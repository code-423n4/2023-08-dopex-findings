# Findings

| ID     |  Title                                                                                        | Severity      |
| :------| :------------------------------------------------------------------------------------------   | :------------ |
| [L-01] | Lack of zero address check in deposit function | Low |
| [L-02] | approveContractToSpend will revert for USDT token| Low |
| [L-03] | Wrong error code | Low |
| [L-04] | bondWithDelegate does not revert with empty arrays as input | Low |
| [L-05] | Add check for slippage tolerance to be less than 100% | Low |
| [L-06] | Do not use hard-coded addresses | Low |
| [L-07] | The event emits zero every time. | Low |
| [L-08] |Wrong recipient address during the swap | Low |
| [L-09] | rdpxFeePercentage and rdpxBurnPercentage should always be equal to 100%| Low |
| [L-10] | The _issueBond function returns a bondId but is never handled | Low |


## [L-01] Lack of zero address check in deposit function
### Impact 
Lack of zero address check in `deposit` function for `receiver` address. It is possible to min shares to address(0)

### Recommended Mitigation Steps
Add additional check for preventing minting to address(0).

## [L-02] approveContractToSpend will revert for USDT token
### Impact 
The `approveContractToSpend` function faces a zero-approval race condition for the USDT token. It is not possible to perform a zero approval due to the above validation `_validate(_amount > 0, 17);`.

### Recommended Mitigation Steps
Remove this validation` _validate(_amount > 0, 17);` because only the admin can call `approveContractToSpend`

## [L-03] Wrong errorCode
In some places in the codebase the wrong error code is used:

[1](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L410). It should be 4

[2](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L951). Here should be other error code because nmber 8 is for: `"Fee cannot be more than 100",` as here is cheked for fee less than 1.

## [L-04] bondWithDelegate does not revert with empty arrays as input
The `bondWithDelegate` function will execute successfully if both `_amounts` and `_delegateIds` are empty, [Instance](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L821-L822).. However, this is not the expected behavior and the stake and reLP functions will be executed with zero amount as input.

### Recommended Mitigation Steps
Add additional check for that.
`_validate(_amounts.length != 0, )`

## [L-05] Add check for slippage tolerance to be less than 100%
In the `setSlippageTolerance` function, it is currently checked to be greater than 0 but not checked to be less than 100%.

` _validate(_slippageTolerance > 0, 3);`
## [L-06] Do not use hard-coded addresses
In the constructor of `UniV3LiquidityAMO`, hard-coded addresses are used for `UniswapV3Factory`, `NonfungiblePositionManager` and `SwapRouter`. This is considered bad practice because if the contract is deployed to another EVM-compatible blockchain, these addresses will be set incorrectly.

## [L-07] The event emits zero every time.
This event:
`emit log(positions_mapping[pos.token_id].token_id);` will emit zero every time because the `token_id` of` positions_mapping[pos.token_id]` is deleted before emitting of the event.

Instance of deleting:
[delete positions_mapping[pos.token_id]](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L263)

## [L-08] Wrong recipient address during the swap
During the swapping of tokens, the recipient address is set to `address(this)`. However, it would be more appropriate for it to be the address of the `rdpxV2Core` contract, as the tokens will be transferred to the core contract after swapping. 

Swapping should be handled similarly to the `removeLiquidity` and `collectFees` functions.

## [L-09] rdpxFeePercentage and rdpxBurnPercentage should always be equal to 100%
The setting of variables `rdpxBurnPercentage` and `rdpxFeePercentage` is a critical aspect of the core protocol. The sum of these percentages should always equal 100%. Therefore, implementing an additional check to ensure their combined sum is 100% will prevent any potential loss of funds from the protocol.

## [L-10] The _issueBond function returns a bondId but is never handled
The `bondId` function returns a `bondId` value but is never handled by the `_stake` function. An additional check can be added to verify that the `bondId` is not equal to zero.

```solidity
// mint receipt token bonds
    _issueBond(_to, receiptTokenAmount); //@audit not handled bondid
```