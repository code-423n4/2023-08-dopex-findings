[Low-01] Redundant code
Some of the code is not used throughout the project. Examples are:
in `UniV2Liquidity.sol`:
 - [Unused variable slippageTolerance](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L51)
 - since `slippageTolerance` is not user, therefore the function setting it is also not needed: [setSlippageTolerance()](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L109-L117)

in `UniV3LiquidityAmo.sol`:
 - Abstract contract [OracleLike](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L22-L26) which is never used

