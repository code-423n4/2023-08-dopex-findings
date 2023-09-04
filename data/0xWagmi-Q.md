### Q/A Report 

| Letter | Name | Description |
|:--:|:-------:|:-------:|
| L  | Low risk | Potential risk |
| NC |  Non-critical | Non risky findings |
| R  | Refactor | Changing the code |
| O | Ordinary | Often found issues |

| Total Found Issues | 3 |
|:--:|:--:|


---

 
## Low[0]  `calculateFunding()` function  could be bypassed  by passing a array with a length 0 ;

CODE : https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L405


### Vulnerability details 

In `PerpetualAtlanticVault::calculateFunding()` it takes an array of `strikes` then it checks `_isEligibleSender()`;
```js
 function _isEligibleSender() internal view {
        // the below condition checks whether the caller is a contract or not
        if (msg.sender != tx.origin) {
            require(whitelistedContracts[msg.sender], "Contract must be whitelisted");
        }
    }
```
Consider this also another low severity vulnerability . we should avoid using  `tx.origin` much as possible, https://twitter.com/pashovkrum/status/1692151232503144578 

Ok , so  we have ` for (uint256 i = 0; i < strikes.length; i++) {` which takes a strikes array and calculate  the `fundingAccrued`  
based on that.But it never checks that `array.length > 0` so the whole logic inside the for loop could be bypassed  ,


### POC 
`copy this test into  /tests/perp-vault/Integration.sol`
`forge test --match-path  ./tests/perp-vault/Integration.t.sol   -vvvv`

```js
function test_strike_bypass_checks() public {
    uint256[] memory strikes = new uint256[](0);
    uint256 fundingAccrued = vault.calculateFunding(strikes);
}

```

### Tools Used
Manual Review / foundry 

### Mitigation steps 
Consider checking the length of the array is greater than 0 else revert the function   

---


## Low[1] Re-assignable  `assetSymbol` result in pointing to a invalid reserveAsset 

### Vulnerability details 
In `rdpxV2Core::addAssetTotokenReserves` it isn't checked that the token symbol is used or not ,In a rare case  if owner reuse the same `symbol` with a different address  and call the function `addAssetTotokenReserves(...)` it will result in changing the index in `reservesIndex["symbol"]` so it will result in breaking  core functionalities in the protocol

```js
// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.19;

import { Test } from "forge-std/Test.sol";

import { ERC721Holder } from "@openzeppelin/contracts/token/ERC721/utils/ERC721Holder.sol";
import { Setup,MockToken } from "./Setup.t.sol";

contract MyTest is ERC721Holder, Setup {
    
    MockToken test_token ;

function test_recallWith_same_symbol() public {
       test_token = new MockToken("Test", "Tst");      
      rdpxV2Core.addAssetTotokenReserves(address(test_token), "WETH");
  }
}
```
### Tools Used
Manual Review /Foundry  

### Impact
Breaking of core functionalities of the protocol ,As this is a rare case and because only the owner is allowed  low severity

### Recommendation 
Consider checking  the token  `symbol` is used or not , `if (keccak256(reserveAsset[i].tokenSymbol) == keccak256(assetSymbol){ revert SymbolExist__() } `

---


 
## Low[2]  An attacker can frontrun by creating a malicious pool if the token Pair doesn't exists

### Vulnerability details 
In `UniV3LiquidityAmo::swap` it uses `exactInputSingle` to swap tokens in uniswap3 pools if the tokens doesn't exists the call should revert 
but a bad actor can take advantage of this by creating a malicious pool by frontrunning to steal tokens    
### Impact
The tokens will be  lost due to the bad exchange rates in the malicious pool.This is very unlikely to happen because owner may use tokens that 
has already pools created with sufficient liquidity.

### Tools Used
Manual Review 
