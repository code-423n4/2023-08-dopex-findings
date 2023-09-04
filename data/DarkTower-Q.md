## Unchecked zero address in the tokens array of `emergencyWithdraw` will fail to complete withdrawal/transfers
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L147-L149

The erc20 tokens to withdraw will be attempted to be transferred to the caller of this `emergencyWithdraw` function in `UniV2LiquidityAmo.sol`.

Presumably, this will fail in cases where there are tokens with zero addresses in the array that neither have an address nor a controlling contract address that's valid.

The recommendation is to exclude instances of a zero address occurrence in the token array and proceed with the withdrawal.


## The function `emergencyWithdraw` in contract `RpdxDecayingBonds` should emit a proper event

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L89

Similar to the instance of the function in `UniV2LiquidityAmo.sol`, this function in `RpdxDecayingBonds` should emit an event when the function call finishes exec. There's even a defined event for this `event EmergencyWithdraw(address sender)` which is unused in the contract.