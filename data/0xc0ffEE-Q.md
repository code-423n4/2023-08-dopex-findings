1. View function `UniV3LiquidityAMO#liquidityInPool` does not work properly as expected : https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L94-L109. The specification for this function is `Returns this contract's liquidity in a specific [Rdpx]-[collateral] uni v3 pool`. This function calls to `UniswapV3 Pool` contract to get position liquidity, but meanwhile liquidity is added through `UniswapV3 NonfungiblePositionManager` contract (which will manage and track added liquidity through NFT positions). In short, the function get pool position liquidity instead of NFT position liquidity, which does not match with specification. 
This issue is non-critical because it does not affect contracts in scope, but maybe it could affect contracts that integrate with it in the future
```diff
diff --git a/tests/rdpxV2-core/Periphery.t.sol b/tests/rdpxV2-core/Periphery.t.sol
index aed7de4..590c8bb 100644
--- a/tests/rdpxV2-core/Periphery.t.sol
+++ b/tests/rdpxV2-core/Periphery.t.sol
@@ -177,6 +177,8 @@ contract Periphery is ERC721Holder, Setup {
 
     uniV3LiquidityAMO.addLiquidity(params);
 
+    assertEq(uniV3LiquidityAMO.liquidityInPool(address(weth), minTick, maxTick, fee), 0);
+
     assertEq(rdpx.balanceOf(address(uniV3LiquidityAMO)), 0);
     assertEq(weth.balanceOf(address(uniV3LiquidityAMO)), 0);
```