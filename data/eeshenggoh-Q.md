## Impact
The variable 2105300114 in the function swap is used to set the expiration time of the swap. It appears to be a hardcoded Unix timestamp.

When creating a variable for a timestamp, it's best to follow a few best practices:

Use a descriptive name: The variable name should clearly indicate what it represents. In this case, a name like expirationTimestamp would be more descriptive and easy to understand.

Use the appropriate data type: In Solidity, timestamps are represented as uint256. This is because the EVM deals with time in terms of seconds since the Unix epoch.

Avoid hardcoding values: Hardcoding a specific value, like 2105300114, can lead to problems if the timestamp is not updated. It's better to calculate the timestamp dynamically based on the current block time (block.timestamp)

SLOC
`https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L295`

## Mitigation
Since 2105300114 seconds / 86400 seconds/day = 24375 days

`uint256 expirationTimestamp = 24375 days; // Expires in 1 day`

In this example, expirationTimestamp is a uint256 variable that is set to 24375 days. This makes it clear that the variable represents a timestamp and when it will expire.

This timestamp is used in the ISwapRouter.ExactInputSingleParams struct which is part of the Uniswap V3 Periphery library. It sets a deadline for when the swap must be included in a block. If the transaction isn't included in a block before this timestamp, it will fail. This is a common pattern in DeFi to prevent transactions from being manipulated by miners

=======================
## Impact
Wrong Error Codes may confuse Users

## PoC
```
_validate(bondDiscount < 100e8, 14);

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1167
```
```
// ERROR CODES
// E1: "Insufficient bond amount",
// E2: "Bond has expired",
// E3: "Invalid parameters",
// E4: "Invalid amount",
// E5: "Not enough ETH available from the delegate",
// E6: "Invalid bond ID",
// E7: "Bond has not matured",
// E8: "Fee cannot be more than 100",
// E9: "msg.sender is not the owner",
// E10: "Price is not above the upper peg",
// E11: "Amount greater that max permissible amount",
// E12: "Price is not lower than first lower peg",
// E13: "Amount greater than max permissible amount"
// E14: "Invalid delegate Id"
// E15: "Invalid amount"
// E16: "Funding already paid for the epoch"
// E17: "Zero address"
// E18: "Asset not found"
// E19: "Token not found"
```
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1289C1-L1308C26
```

## Mitigation
Either change the error code to a newly created one or to E15 as it makes more sense

=======================
## Impact
Wrong initialization in Setup.t.sol test. Developers may think that the test case is valid an initialize in wrong other against documentation. 

## PoC
```
  /* Inital tokens in the reserve
     index0: ZERO address
     index1: weth
     index2: rdpx
     index3: dpxEth
     index4: crv
  */
    rdpxV2Core.addAssetTotokenReserves(address(rdpx), "RDPX");
    rdpxV2Core.addAssetTotokenReserves(address(weth), "WETH");
    rdpxV2Core.addAssetTotokenReserves(address(dpxETH), "DPXETH");
```

```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/tests/rdpxV2-core/Setup.t.sol#L260
```

## Mitigation
Change to WETH first as stated in the RdpxV2 Contract:
```
    rdpxV2Core.addAssetTotokenReserves(address(weth), "WETH");
    rdpxV2Core.addAssetTotokenReserves(address(rdpx), "RDPX");
    rdpxV2Core.addAssetTotokenReserves(address(dpxETH), "DPXETH");
```


=======================

## Summary
Just to prove the removeAssetFromtokenReserves working. It's working, I just want to input some of my hardwork test cases here for devs to test out.
Test Codes PoC:

Add into RdpxV2Core.sol:
```
  function getCount() public view returns (uint){
    return reserveAsset.length;
  }

    function getIndex(string memory _assetSymbol) public view returns (uint){
    return reservesIndex[_assetSymbol];
  }
  function getAsset(uint index) public view returns (address,uint,string memory) {
      return (reserveAsset[index].tokenAddress,reserveAsset[index].tokenBalance,reserveAsset[index].tokenSymbol);
  }
```

Add into Unit.t.sol it's used to prove the function removeAssetFromtokenReserves() in RdpxV2Core.sol for intentions.
```
  function test_EqReserveIndex() public {
    
    assertEq(rdpxV2Core.getCount(),3,'test_eq');
  }


  function test_EqIndex() public {
      string memory assetSymbol = "WETH"; 
      uint expectedValue = 1;
      uint actualValue = rdpxV2Core.getIndex(assetSymbol);

      assertEq(actualValue, expectedValue, "The actual value does not match the expected value");
  }
  function test_getAsset(uint index) public {
    (address tokenAddress,,string memory tokenSymbol) = rdpxV2Core.getAsset(index);
    assertEq(tokenAddress,address(weth));
    assertEq(tokenSymbol,"WETH");

  }
  // /rdpx address=0x2e234dae75c793f67a35089c9d99245e1c58470b
  // /weth address0x5615deb798bb3e4dfa0139dfa1b3d433cc23b72f

  function test_Add() public {
    rdpxV2Core.addAssetTotokenReserves(address(weth), "WETH");
    rdpxV2Core.addAssetTotokenReserves(address(rdpx), "RDPX");
    test_EqReserveIndex();
  }
  function test_Remove() public {
    test_Add();
    rdpxV2Core.removeAssetFromtokenReserves("WETH");
    test_getAsset(1);
  }


=======================

## Impact
Potential reentrancy: Code does not follow the best practice of check-effects-interaction

Code should follow the best practice of check-effects-interaction, where state variables are updated before any external calls are made. Doing so prevents a large class of reentrancy bugs.

```
    IERC20WithBurn(weth).safeTransferFrom(
      msg.sender,
      address(this),
      wethRequired
    );

    // update weth reserve
    reserveAsset[reservesIndex["WETH"]].tokenBalance += wethRequired;
```

```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L909C1-L917C1
```
## Mitigation 
Follow the check-effect-interact pattern