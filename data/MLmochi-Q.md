By MLmochi
email wooddustsniffer@gmail.com
discord MLmochi#0400
code4rena contest: https://github.com/code-423n4/2023-08-dopex/tree/main

Overview
- LOW 01- Conflicting reservesIndex mappings
- LOW 02- Incorrect reserveTokens element popped

--- 
# LOW 01- Conflicting reservesIndex mappings
## Description


In the Core-contract: `contract/core/RdpxV2Core.sol` the function:`addAssetTotokenReserves(address _asset, string memory _assetSymbol)`  
allows the **Admin** to add new Assets to the reserve. It checks that the reserveAssets's **address** is not already present, **however** it does not check if the input `string memory _assetSymbol` is already present. 

Repo Link: https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L240C36-L240C36

```
  function addAssetTotokenReserves(
    address _asset,
    string memory _assetSymbol
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");

    for (uint256 i = 1; i < reserveAsset.length; i++) {
      require(
        reserveAsset[i].tokenAddress != _asset,
        "RdpxV2Core: asset already exists"
      );
    }

    ReserveAsset memory asset = ReserveAsset({
      tokenAddress: _asset,
      tokenBalance: 0,
      tokenSymbol: _assetSymbol
    });
    reserveAsset.push(asset);
    reserveTokens.push(_assetSymbol);

    reservesIndex[_assetSymbol] = reserveAsset.length - 1;

    emit LogAssetAddedTotokenReserves(_asset, _assetSymbol);
  }
```

Adding a new reserveAsset with an existing `_assetSymbol` will render the entire previous existing `reseveAsset` inaccessible, which can result a loss of user funds. Due to the dependency on `reservesIndex` to extract a `reserveAsset` in this contract essential functions such as : 

- `_purchaseOptions(...)`
- `_stake(...)`
- `_transfer(...)`
- `_bondWithDelegate(...)` 
- `settle(...)
- etc .. 
### Assessed type
**admin privilege**
## Burden of proof
Adding a new asset to the reserve with the same `string memory _assetSymbol` of already an existing mapping inside the `reservesIndex` will make any existing `reserveAsset` useless.  This can cause further downstream issues when  calling `reserveAsset` and potentially breakage of the contract functionality.

Consider the following scenario:
**State (0) before calling `addAssetTotokenReserves(<0xNewcontractAddress>, "RDPX")`**:

	reserveTokens[0]=: RDPX
	reserveTokens[1]=: WETH
	reserveTokens[2]=: DPXETH
	reservesIndex['RDPX']=: 1
	reservesIndex['WETH']=: 2
	reservesIndex['DPXETH']=: 3
	reservesAsset[0]=: ZERO
	reservesAsset[1]=: RDPX
	reservesAsset[2]=: WETH
	reservesAsset[3]=: DPXETH

**Admin calls`addAssetTotokenReserves(<0xNewcontractAddress>, "RDPX") `**

**State (1) after call:**

	reserveTokens[0]=: RDPX
	reserveTokens[1]=: WETH
	reserveTokens[2]=: DPXETH
	reserveTokens[3]=: RDPX
	reservesIndex['RDPX']=: 4
	reservesIndex['WETH']=: 2
	reservesIndex['DPXETH']=: 3
	reservesAsset[0]=: ZERO
	reservesAsset[1]=: RDPX
	reservesAsset[2]=: WETH
	reservesAsset[3]=: DPXETH
	reservesAsset[4]=: RDPX

Which now shows that `"RDPX" -> 4` hence getting  `reserveAsset` element that corresponds to `RDPX`, we get a new and empty `ReserveAsset` structure. Any balance on the previous `ReserveAsset` is still present in the storage but inaccessible. 

For example in the function `settle`  inside `contract/core/RdpxV2Core.sol` will find that the `reserveAsset[reservesIndex["RDPX"]].tokenBalance` is 0 and any  `rdpxAmount>0` will make the settle function revert. 

```
  function settle(
    uint256[] memory optionIds
  )
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 amountOfWeth, uint256 rdpxAmount)
  {
    _whenNotPaused();
    (amountOfWeth, rdpxAmount) = IPerpetualAtlanticVault(
      addresses.perpetualAtlanticVault
    ).settle(optionIds);
    for (uint256 i = 0; i < optionIds.length; i++) {
      optionsOwned[optionIds[i]] = false;
    }

    reserveAsset[reservesIndex["WETH"]].tokenBalance += amountOfWeth;
    reserveAsset[reservesIndex["RDPX"]].tokenBalance -= rdpxAmount;

    emit LogSettle(optionIds);
  }
```

### Foundry test demonstration
For *visual* reproduction of bug used the Foundry test setup and created a the following contract inside the  `2023-08-dopex/test/` : 

```
// SPDX-License-Identifier: UNLICENSED
/**
Code4Rena Contest August
MLmochi tests
*/
pragma solidity 0.8.19;
import { Test } from "forge-std/Test.sol";
import { IRdpxV2Core } from "contracts/core/IRdpxV2Core.sol";
import { Setup } from "../rdpxV2-core/Setup.t.sol";
import { ERC721Holder } from "@openzeppelin/contracts/token/ERC721/utils/ERC721Holder.sol";

contract conflictingAddAssetTotokenReserves is ERC721Holder, Setup{

    function testConflictingReservesIndex() external {
        // Get the TokenSymbol pre
        ( ,  , string memory N0 )  = rdpxV2Core.reserveAsset(0);
        ( , ,  string memory N1)   = rdpxV2Core.reserveAsset(1);
        ( ,  , string memory N2 )  = rdpxV2Core.reserveAsset(2); 
        ( ,  , string memory N3)  = rdpxV2Core.reserveAsset(3);


        // Log States from contract Before Calling  removeAssetFromtokenReserves
        emit log("Before Function");
        emit log_named_string("reserveTokens[0]=",rdpxV2Core.reserveTokens(0) );
        emit log_named_string("reserveTokens[1]=",rdpxV2Core.reserveTokens(1) );
        emit log_named_string("reserveTokens[2]=",rdpxV2Core.reserveTokens(2) );


        emit log_named_uint("reservesIndex['RDPX']=",rdpxV2Core.reservesIndex("RDPX") );
        emit log_named_uint("reservesIndex['WETH']=",rdpxV2Core.reservesIndex("WETH") );
        emit log_named_uint("reservesIndex['DPXETH']=",rdpxV2Core.reservesIndex("DPXETH") );

        emit log_named_string("reservesAsset[0]=", N0 );
        emit log_named_string("reservesAsset[1]=", N1 );
        emit log_named_string("reservesAsset[2]=", N2 );
        emit log_named_string("reservesAsset[3]=", N3);

        //add New rDPX token (update)
        MockToken rdpxToken = new MockToken("RDPX TOKEN", "RDPX");
        rdpxV2Core.addAssetTotokenReserves(address(rdpxToken), "RDPX");

        // Log States After calling removeAssetFromtokenReserves
        emit log("After Function");
        // Here on reservTokens we will see an wrong element deletion
        emit log_named_string("reserveTokens[0]=",rdpxV2Core.reserveTokens(0) );
        emit log_named_string("reserveTokens[1]=",rdpxV2Core.reserveTokens(1) );
        emit log_named_string("reserveTokens[2]=",rdpxV2Core.reserveTokens(2) );
        emit log_named_string("reserveTokens[3]=",rdpxV2Core.reserveTokens(3) );
        // emit log_named_string("reserveTokens[4]=",rdpxV2Core.reserveTokens(4) );

        emit log_named_uint("reservesIndex['RDPX']=",rdpxV2Core.reservesIndex("RDPX") );
        emit log_named_uint("reservesIndex['WETH']=",rdpxV2Core.reservesIndex("WETH") );
        emit log_named_uint("reservesIndex['DPXETH']=",rdpxV2Core.reservesIndex("DPXETH") );

        ( ,, string memory N0_a )  = rdpxV2Core.reserveAsset(0);
        ( ,, string memory N1_a)   = rdpxV2Core.reserveAsset(1);
        ( ,, string memory N2_a )  = rdpxV2Core.reserveAsset(2); 
        ( ,, string memory N3_a )  = rdpxV2Core.reserveAsset(3); 
        ( ,, string memory N4_a )  = rdpxV2Core.reserveAsset(4); 

        emit log_named_string("reservesAsset[0]=", N0_a );
        emit log_named_string("reservesAsset[1]=", N1_a );
        emit log_named_string("reservesAsset[2]=", N2_a );
        emit log_named_string("reservesAsset[3]=", N3_a );
        emit log_named_string("reservesAsset[4]=", N4_a );
      
    }

```

which can be tested with bash:
`forge test --match-contract conflictingAddAssetTotokenReserves --match-test testConflictingReservesIndex -vv`

## Tools used 
Visual Studio Code 
## Impact 
Impact of this finding can be considered Low because only an address with **Admin rights** can call this function. However, accidents can happen, with this double reserveAssset addition, many essential functionalities can break. 

## Recommended Mitigation Steps
This potential security issue can be easily resolved by also adding an simple require on the function input `string memory _assetSymbol`  between the *for loop*  and the creation of the a new `ReserveAsset`

```
require(reservesIndex[_assetSymbol] == 0, "RdpxV2Core: _assetSymbol already exists" );

```

---
# LOW 02- Incorrect reserveTokens element popped
## Description

In the Core-contract: `contract/core/RdpxV2Core.sol` the function:`removeAssetFromtokenReserves(string memory _assetSymbol)`  incorrectly pops the elements of `reserveTokens` without rearranging the elements. The function: 

(repo link: https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L270)

```
  function removeAssetFromtokenReserves(
    string memory _assetSymbol
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    uint256 index = reservesIndex[_assetSymbol];
    _validate(index != 0, 18);

    // remove the asset from the mapping
    reservesIndex[_assetSymbol] = 0;

    // add new index for the last element
    reservesIndex[reserveTokens[reserveTokens.length - 1]] = index;

    // update the index of reserveAsset with the last element
    reserveAsset[index] = reserveAsset[reserveAsset.length - 1];

    // remove the last element
    reserveAsset.pop();
    reserveTokens.pop();

    emit LogAssetRemovedFromtokenReserves(_assetSymbol, index);
  }
```

### Assessed type
**admin privilege**
## Burden of proof
Removing an Reserve asset from the contract that is not the last element of the `reserveTokens` array results in an inconsistent contract state with respect to `reserveAsset` and `reserveIndex`

Consider the following scenario:
**State (0) before calling `removeAssetFromtokenReserves`**:
`reserveTokens[0] = "RDPX"` 
`reserveTokens[1] = "WETH"` 
`reserveTokens[2] = "DPXETH"` 
`reservesIndex['RDPX'] =  1`
`reservesIndex['WETH'] = 2`
`reservesIndex['DPXETH'] = 3`
`reservesAsset[0].tokenSymbol = "ZERO"`
`reservesAsset[1].tokenSymbol = "RDPX"`
`reservesAsset[2].tokenSymbol = "WETH"`
`reservesAsset[3].tokenSymbol = "DPXETH"`

**Admin calls `removeAssetFromtokenReserves("RDPX")`** 

**State (1) after call:**
`reserveTokens[0] = "RDPX"` 
`reserveTokens[1] = "WETH"` 
`reservesIndex['RDPX'] =  0`
`reservesIndex['WETH'] = 2`
`reservesIndex['DPXETH'] = 1`
`reservesAsset[0].tokenSymbol = "ZERO"`
`reservesAsset[1].tokenSymbol = "DPXETH"`
`reservesAsset[2].tokenSymbol = "WETH"`

Which clearly shows that  `"DPXETH"` was popped from `reserveTokens` which **should have been** `RDPX`

### Foundry test demonstration
For reproduction of bug used the Foundry test setup and created a the following contract inside the  `2023-08-dopex/test/` : 
```
// SPDX-License-Identifier: UNLICENSED
/**
Code4Rena Contest August
MLmochi tests
*/
pragma solidity 0.8.19;
import { Test } from "forge-std/Test.sol";
import { IRdpxV2Core } from "contracts/core/IRdpxV2Core.sol";
import { Setup } from "../rdpxV2-core/Setup.t.sol";
import { ERC721Holder } from "@openzeppelin/contracts/token/ERC721/utils/ERC721Holder.sol";

contract incorrectRemoveAssetFromtokenReserves  is ERC721Holder, Setup{
    function testRemoveAssetFromtokenReserves_logs() external {
        // Get the TokenSymbol pre
        ( ,  , string memory N0 )  = rdpxV2Core.reserveAsset(0);
        ( , ,  string memory N1)   = rdpxV2Core.reserveAsset(1);
        ( ,  , string memory N2 )  = rdpxV2Core.reserveAsset(2); 
        ( ,  , string memory N3)  = rdpxV2Core.reserveAsset(3);

        // Log States from contract Before Calling  removeAssetFromtokenReserves
        emit log("Before Function");
        emit log_named_string("reserveTokens[0]=",rdpxV2Core.reserveTokens(0) );
        emit log_named_string("reserveTokens[1]=",rdpxV2Core.reserveTokens(1) );
        emit log_named_string("reserveTokens[2]=",rdpxV2Core.reserveTokens(2) );


        emit log_named_uint("reservesIndex['RDPX']=",rdpxV2Core.reservesIndex("RDPX") );
        emit log_named_uint("reservesIndex['WETH']=",rdpxV2Core.reservesIndex("WETH") );
        emit log_named_uint("reservesIndex['DPXETH']=",rdpxV2Core.reservesIndex("DPXETH") );
        emit log_named_string("reservesAsset[0]=", N0 );
        emit log_named_string("reservesAsset[1]=", N1 );
        emit log_named_string("reservesAsset[2]=", N2 );
        emit log_named_string("reservesAsset[3]=", N3);

        // Delete the 0 element of reserveTokens-> which is RDPX
        rdpxV2Core.removeAssetFromtokenReserves("RDPX");
        
        // Log States After calling removeAssetFromtokenReserves
        emit log("After Function");
        // Here on reservTokens we will see an wrong element deletion
        emit log_named_string("reserveTokens[0]=",rdpxV2Core.reserveTokens(0) );
        emit log_named_string("reserveTokens[1]=",rdpxV2Core.reserveTokens(1) );

        emit log_named_uint("reservesIndex['RDPX']=",rdpxV2Core.reservesIndex("RDPX") );
        emit log_named_uint("reservesIndex['WETH']=",rdpxV2Core.reservesIndex("WETH") );
        emit log_named_uint("reservesIndex['DPXETH']=",rdpxV2Core.reservesIndex("DPXETH") );

        ( ,, string memory N0_a )  = rdpxV2Core.reserveAsset(0);
        ( ,, string memory N1_a)   = rdpxV2Core.reserveAsset(1);
        ( ,, string memory N2_a )  = rdpxV2Core.reserveAsset(2); 

        emit log_named_string("reservesAsset[0]=", N0_a );
        emit log_named_string("reservesAsset[1]=", N1_a );
        emit log_named_string("reservesAsset[2]=", N2_a );
    }

    function testRemoveAssetFromtokenReserves() external {
        // State Before Calling removeAssetFromtokenReserves
        // Contains the assets: rDPX, weth, dpxEth
        // Check state Before
        assertEq(keccak256(abi.encodePacked((rdpxV2Core.reserveTokens(0)))) == keccak256(abi.encodePacked(("RDPX")))   , true     );
        assertEq(keccak256(abi.encodePacked((rdpxV2Core.reserveTokens(1)))) == keccak256(abi.encodePacked(("WETH")))   , true     );
        assertEq(keccak256(abi.encodePacked((rdpxV2Core.reserveTokens(2)))) == keccak256(abi.encodePacked(("DPXETH"))) , true     );
        // remove is DPXETH
        rdpxV2Core.removeAssetFromtokenReserves("RDPX");
        // Check state After: Should NOT revert, but will due to bug!
        assertEq(keccak256(abi.encodePacked((rdpxV2Core.reserveTokens(rdpxV2Core.reservesIndex("DPXETH")-1)))) == keccak256(abi.encodePacked(("DPXETH")))  , true ); 
        assertEq(keccak256(abi.encodePacked((rdpxV2Core.reserveTokens(rdpxV2Core.reservesIndex("WETH")-1)))) == keccak256(abi.encodePacked(("WETH")))   , true     );
    }
}

```

which can be tested with bash:
`forge test --match-contract incorrectRemoveAssetFromtokenReserves --match-test testRemoveAssetFromtokenReserves -vv`

## Tools used 
Visual Studio Code 
## Impact 
Despite  the `reserveIndex` inside the same function using the `reserveToken`, the impact is considered Low. There appears no further downstream security issues as the `reserveIndex` will always properly refer to the right `reserveAsset` element, which operates independent to `reserverToken`.  Additionally this **special admin privilege** function, which makes it even lower risk.

## Recommended Mitigation Steps
This potential security issue can be easily resolved by rearranging `reservesTokens` just like is done in `reserveAssets` :

```
  function removeAssetFromtokenReserves(
    string memory _assetSymbol
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    uint256 index = reservesIndex[_assetSymbol];
    _validate(index != 0, 18);

    // remove the asset from the mapping
    reservesIndex[_assetSymbol] = 0;

    // add new index for the last element
    reservesIndex[reserveTokens[reserveTokens.length - 1]] = index;

    // update the index of reserveAsset with the last element
    reserveAsset[index] = reserveAsset[reserveAsset.length - 1];

    // update the reserveToken Array
    ++ reserveTokens[index-1] = reserveTokens[reserveTokens.length - 1];
    
    // remove the last element
    reserveAsset.pop();
    reserveTokens.pop();

    emit LogAssetRemovedFromtokenReserves(_assetSymbol, index);
  }
```

