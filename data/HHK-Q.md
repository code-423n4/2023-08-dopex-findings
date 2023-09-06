### INFO: Natspec return values are inversed for [`_calculateAmounts()`](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L598).

#### Technical Details

In the function `_calculateAmounts()` the `amount1` returned is the amount received by the delegatee and the `amount2` is the amount received by the delegate but the natspec for this function says the opposite.

#### Impact

Can be missleading for future auditors or integrators.

#### Recommendation

Update the natspec.

```diff
- * @return amount1 The amount received by the delegate
- * @return amount2 The amount received by the delegatee
+ * @return amount1 The amount received by the delegatee
+ * @return amount2 The amount received by the delegate
```

### LOW: No  `_whenNotPaused()` in `redeem()`

#### Technical Details

Almost all state changing functions have `_whenNotPaused()` in the core contract but it is not the case for [`redeem()`](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1016).

The NFT it interact with has a pause/unpause functionnality so the function will revert if the NFT is paused thus the impact seems low.

#### Impact

User could still redeem even tho the core contract is paused.

#### Recommendation

Add `_whenNotPaused()` to the function.

### LOW: Calling `liquidityInPool()` will always return 0 on UniV3 AMO

#### Technical Details

Because uniswap v3 position are minted using the nonFungiblePositionManager, asking the liquidity of the AMO address directly from the pool will return 0 in [`liquidityInPool()`](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L94).

#### Impact

Incorrect liquidity returned.

#### Recommendation

Query the liquidity on the nonFungiblePositionManager instead of the pool.

### LOW: `collectFees()` might become gas intensive over time

#### Technical Details

The function [`collectFees()`](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L119) collect the fees on all positions, but because each time this contract adds liquidity it creates a new position.

Over time a lot of position could be created and might become very gas intensive or not worth collecting fees for some positions.

#### Impact

No ability to select the position to collect.

#### Recommendation

Add a start index and end index to the function params or an index array to decide which positions to collect from.

### LOW: `reLP()` add liquidity is not efficient and result in some dust

#### Technical Details

At the end of the function [`reLP()`](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L202) the contract tries to add liquidity with the weth left. But by swapping half of the ETH for rdpx it changes the reserve of rdpx and so the price on the univ2 pool.

This result in some rdpx dust left after the `addLiquidity()`, the impact is low as all rdpx are sent to the core contract.

#### Impact

The liquidity added will be slightly less than what we wanted.

#### Recommendation

Use the right formula to calculate how much ETH to swap. See: https://blog.alphaventuredao.io/onesideduniswap/