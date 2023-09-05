## Finding Summary 

| ID | Severity | Description |
| - | - | :-: |
| [L-01](#l-01-Missing-checks-for-zero-amount-transfers-in-univ3liquidityamo.sol) | Low | Missing checks for zero amount transfers in `UniV2LiquidityAmo.sol` and `UniV3LiquidityAmo.sol`
| [L-02](#l-02-Typo-in-the-comments) | Low | Typo in the comments 
| [L-03](#l-03-safeerc20-and-safetransferlib-libraries-can-be-used-interchangeably) | Low | `SafeERC20` and `SafeTransferLib` libraries can be used interchangeably
| [L-04](#l-04-there-is-no-need-for-two-slippage-variables-in-relpcontract.sol) | Low | There is no need for two slippage variables in `ReLPContract.sol`
| [L-05](#l-05-unchecked-return-data-when-making-arbitrary-calls-to-the-to-address) | Low | Unchecked return data when making arbitrary calls to the `to` address
| [L-06](#l-06-erc721burnable-is-initialized-but-its-functionality-is-never-used-in-rpdxdecayingbonds.sol) | Low | ERC721Burnable is initialized but its functionality is never used in RpdxDecayingBonds.sol
| [L-07](#l-07-admin-is-a-single-point-of-failure-when-calling-emergencywithdraw-due-to-unspecified-to-address) | Low | Admin is a single point of failure when calling `emergencyWithdraw()` due to unspecified `to` address


## [L-01] Missing checks for zero amount transfers in `UniV2LiquidityAmo.sol` and `UniV3LiquidityAmo.sol`

In `UniV2LiquidityAmo.sol` and `UniV3LiquiidtyAmo.sol`, there is no check whether `tokenABalance` and `tokenBBalance` being sent to the `RpdxV2Core` via `safeTransfer()` are not zero. Therefore, there can be a situation where there are zero amount transfers:

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L168-175
```
  IERC20WithBurn(addresses.tokenA).safeTransfer(
      addresses.rdpxV2Core,
      tokenABalance
    );
    IERC20WithBurn(addresses.tokenB).safeTransfer(
      addresses.rdpxV2Core,
      tokenBBalance
    );
```

### Recommendation

Consider adding the following checks: 

```
require(tokenABalance > 0)
require(tokenBBalance > 0)
```

## [L-02] Typo in the comments  

There is a typo in the comments before ```lowerDepeg()``` function in ```RpdxV2Core.sol```:

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1073

### Recommendation

Change ```Rdpx V2 Core can redeem Rdpx/ETH lp tokens ot buy back and burn DpxEth``` here on ```or buy back```

## [L-03] `SafeERC20` and `SafeTransferLib` libraries can be used interchangeably

In `PerpetualAtlanticVaultLP.sol`, two different libraries for handling non-standard ERC20 tokens are used. The problem is that they can be used interchangeably as they are used for one purpose:

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L11-12
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L22-23

### Recommendation

Consider using only one library when dealing with `IERC20WithBurn` and `uint256`.

## [L-04] There is no need for two slippage variables in `ReLPContract.sol`

In `ReLPContract.sol` two variables (`liquiditySlippageTolerance` and `slippageTolerance`) are used to manage slippage protection. However, as they are both constant values, only one variable can be created with `5e5` as the value.

### Recommendation

Consider leaving only `slippageTolerance` variable that would represent 0.5%.

## [L-05] Unchecked return data when making arbitrary calls to the `to` address

There are several functions where `call` opcode is used for the admin to perform arbitrary calls for different purposes:

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L98-99
```
(bool success, ) = to.call{ value: amount, gas: gas }("");
      require(success, "RdpxReserve: transfer failed");
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L344-345

```
(bool success, bytes memory result) = _to.call{ value: _value }(_data);
    return (success, result);
```

The problem is that the _to address can return a huge payload that will lead to very costly memory allocation.

### Recommendation

Consider using low-level assembly `call` to avoid copying return data to memory:

```
bool success;
assembly {
    success := call(gas, receiver, amount, 0, 0, 0, 0)
}
```

## [L-06] `ERC721Burnable` is initialized but its functionality is never used in `RpdxDecayingBonds.sol`

Inside of `RpdxDecayingBonds.sol`, there is an import of `ERC721Burnable` OZ library which extends `ERC721` standard by adding `burn()` function. However, there is no any examples of usage this function in the contract
where decaying bonds are minted to the user.

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L11
`import { ERC721Burnable } from "@openzeppelin/contracts/token/ERC721/extensions/ERC721Burnable.sol";`

### Recommendation

Consider removing this library or use the `burn()` function.

## [L-07] Admin is a single point of failure when calling `emergencyWithdraw()` due to unspecified `to` address

Inside of the `RpdxDecayingBonds` contract there is `emergencyWithdraw()` function where `to` address is specified by the admin. If the address is incorrect, all the funds from this contract will be lost:

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L89-107
```
 if (transferNative) {
      (bool success, ) = to.call{ value: amount, gas: gas }("");
      require(success, "RdpxReserve: transfer failed");
    }
```
### Recommendation

Due to importance of this function, consider using `msg.sender` instead of `to` address.
