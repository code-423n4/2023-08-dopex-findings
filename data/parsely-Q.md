# [L-01] Missing checks for the contract being paused.
## Context
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1016-L1042
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1051-L1070
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1080-L1124


## Description
The `redeem`, `upperDepeg` and `lowerDepeg` functions are missing the check as to if the contract is paused. These functions should check if the contract is paused as confirmed with the sponsor.

## Resolution
Add the check `_whenNotPaused()` to the functions.

# [L-02] RdpxV2Core `redeem` function runs successfully if the user sends in address(0) as the `to` parameter.
## Context
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1016-L1042

## Impact
User funds would be lost, as the token and transfer function do not seem to catch it be default.

## Description
Although it would be a user mistake, it would still lead to loss of funds.
The call in a PoC using parameter address(0) runs successfully.
```
    uint256 redeemRet = rdpxV2Core.redeem(2, address(0));
    console.log("Amount redeemed : " , redeemRet);
```

Gives output as below indicating it was successfully actioned:
```
  Contract Balance of TOKEN before call :  5000000000000000001
  Test user Balance of TOKEN before call :  0
  Amount redeemed :  1
  Contract Balance of TOKEN after call :  5000000000000000000
  Test user Balance of TOKEN after call :  0
```
and emits to event as below:
```
emit LogRedeem(to: 0x0000000000000000000000000000000000000000, amount: 1)
```

## Resolution
Add the check for address(0). 

# [L-03] No way for user to swap receiptToken back to dpxETH and WETH currently.
## Context
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1016-L1042

## Description
The `redeem` function only returns receiptToken to the user after the bond has matured. I confirmed with the sponsor and currently the receiptToken swap back to WETH and/or dpxETH is not implemented yet. 

## Impact
The user currently can only get receiptToken but cannot get WETH or dpxETH back out.

## Resolution
Implement the functionality in the receiptToken before launching.

# [NC-01] Unused local variable : `amoAddresses`.
## Context
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L63-L64
```
address[] public amoAddresses;
```
## Description
The local variable `amoAddresses` is controlled by the functions `addAMOAddress(address _addr)` and `removeAMOAddress(uint256 _index)` but the actual addresses saved to the array are never used in anyway throughout all the code.

## Resolution
Consider removing the array and functions, if it will not be used in future.

# [NC-02] Unused function parameter : `shares` in PerpetualAtlanticVaultLP.sol function `beforeWithdraw`.
## Context
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L63-L64
```
function beforeWithdraw(uint256 assets, uint256 /*shares*/) internal {
    require(
      assets <= totalAvailableCollateral(),
      "Not enough available assets to satisfy withdrawal"
    );
    _totalCollateral -= assets;
  }
```
## Description
The function parameter `shares` is not used by the function `beforeWithdraw(uint256 assets, uint256 /*shares*/)` but is not used within the function in any way.

## Resolution
Consider removing the function parameter, if it will not be used in future.

# [NC-03] Comment about functionality is incorrect in the `_transfer` function in the RdpxV2Core contract.
## Context
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L652
```
// Transfer rDPX and ETH token from user
```
## Description
TThe comment mentions that it transfers ETH from the user, but as confirmed with the sponsor this is an old comment and the function does not transfer any ETH from the user.

## Resolution
Correct comment to be
```
// Transfer rDPX
```