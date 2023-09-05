## [LOW] Emited event is always address(0)
In `UniV3LiquidityAmo::removeLiquidity()` the event `log()` is emited with a mapping element that has just been deleted as parameter. This will always log `address(0)`.
```solidity
File UniV3LiquidityAmo.sol

L263: delete positions_mapping[pos.token_id]; //@audit delete mapping element

    // send tokens to rdpxV2Core
    _sendTokensToRdpxV2Core(tokenA, tokenB);

    emit log(positions_array.length);
    emit log(positions_mapping[pos.token_id].token_id); //@audit has just been deleted
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L269

## [LOW] `UniV3LiquidityAmo` will stop working in 2036
In `UniV3LiquidityAmo::swap()` the deadline parameter of the swap configuration is hardcoded to `2105300114` which is the unix timestamp of `17 September 2036`. That means that all swaps will revert after this date and the function will become unusable.
```solidity
File UniV3LiquidityAmo.sol

L289: ISwapRouter.ExactInputSingleParams memory swap_params = ISwapRouter
      .ExactInputSingleParams(
        _tokenA,
        _tokenB,
        _fee_tier,
        address(this),
        2105300114, // Expiration: a long time from now //@audit Deadline is in 13 years (2036)
        _amountAtoB,
        _amountOutMinimum,
        _sqrtPriceLimitX96
      );

    // Approval
    TransferHelper.safeApprove(_tokenA, address(univ3_router), _amountAtoB);

    uint256 amountOut = univ3_router.exactInputSingle(swap_params);
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L295

## [QA] Wrong revert messages
*Instances(6)*
```solidity
File UniV2LiquidityAmo.sol

L91: "reLPContract: address cannot be 0" //@audit should be "UniV2LiquidityAmo: address cannot be 0"

L114: "reLPContract: slippage tolerance must be greater than 0" //@audit same typo

L131: require(_token != address(0), "reLPContract: token cannot be 0"); //@audit same typo
L132: require(_spender != address(0), "reLPContract: spender cannot be 0"); //@audit same typo
L133: require(_amount > 0, "reLPContract: amount must be greater than 0"); //@audit same typo
```
```solidity
File: RdpxDecayingBonds.sol

L99: require(success, "RdpxReserve: transfer failed"); // @audit should not be `RdpxReserve` --> `RdpxDecayingBonds.sol`
```
