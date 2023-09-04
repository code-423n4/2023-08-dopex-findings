# L-01: UniV2LiquidityAmo._sendTokensToRdpxV2Core does not call IRdpxV2Core(rdpxV2Core).sync()

`UniV2LiquidityAmo.addLiquidity`/`removeLiquidity`/`swap` will call [_sendTokensToRdpxV2Core()](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L160) at the end to send all tokenA and tokenB of this contract to the rdpxV2Core contract. Since rdpxV2Core uses the [reserveAsset](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L73) array to record all token balances, this is a caching mechanism. The [sync](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L995-L1008) function needs to be called in time to update these caches. However, `_sendTokensToRdpxV2Core()` sends tokens to rdpxV2Core without calling `sync` to update the `reserveAsset` array. In this case, using the old value may lead to unexpected behavior.

For example, `swap` is called to swap rdpx for weth, this function will transfer the rdpx of rdpxV2Core to this contract, and then call `IUniswapV2Router(addresses.ammRouter).swapExactTokensForTokens` for weth, and finally call `_sendTokensToRdpxV2Core` to transfer all rdpx and weth in this contract to rdpxV2Core. **In this case, the rdpx decreases and weth increases in rdpxV2Core**. But `rdpxV2Core.sync()` is not called, so `reserveAsset` still holds the stale value. If at this time, [RdpxV2Core.lowerDepeg](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1080-L1085) is called, this function uses the value of the `reserveAsset` array to buy back and burn DpxEth. Due to this problem, `reservesAsset[reservesIndex["RDPX"]].tokenBalance` is larger than `rdpx.balanceOf(address(RdpxV2Core))`, which is an illusion. Because the previous `swap` has transferred part of rdpx from RdpxV2Core. Then, it may cause the `IUniswapV2Router(addresses.dopexAMMRouter).swapExactTokensForTokens` revert [here](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1097-L1104) due to not enough rdpx. Similarly, `reserveAsset[reservesIndex["WETH"]].tokenBalance` is smaller than `weth.balanceOf(address(RdpxV2Core))`, which is also an illusion. Because the previous `swap` has transferred weth to RdpxV2Core. `lowerDepeg` [could have used more _wethAmount](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1110) to buy back and burn DpxEth.

## Recommended Mitigation Steps

```fix
File: contracts\amo\UniV2LiquidityAmo.sol
160:   function _sendTokensToRdpxV2Core() internal {
......
172:     IERC20WithBurn(addresses.tokenB).safeTransfer(
173:       addresses.rdpxV2Core,
174:       tokenBBalance
175:     );
176:+    IRdpxV2Core(rdpxV2Core).sync();
177:     emit LogAssetsTransfered(msg.sender, tokenABalance, tokenBBalance);
178:   }
```

# L-02: RdpxV2Core.removeAssetFromtokenReserves lacks updating the index of reserveTokens with the last element

[removeAssetFromtokenReserves](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L270-L272) is used to remove a asset from the reserves tokens. To delete an element from an array, a common method is to assign the last element of the array to the element to be removed, and then call `array.pop()` to delete the last element. `RemoveAssetFromtokenReserves` lacks updating the index of `reserveTokens` with the last element, and directly calls `reserveTokens.pop()` to delete the last element. This causes the elements to be deleted to still exist in the array, while the elements that do not need to be deleted are deleted.

## Recommended Mitigation Steps

```fix
File: contracts\core\RdpxV2Core.sol
279:     // add new index for the last element
280:     reservesIndex[reserveTokens[reserveTokens.length - 1]] = index;
281: 
282:     // update the index of reserveAsset with the last element
283:     reserveAsset[index] = reserveAsset[reserveAsset.length - 1];
284:+    reserveTokens[index -1] = reserveTokens[reserveTokens.length - 1];
285:     // remove the last element
286:     reserveAsset.pop();
287:     reserveTokens.pop();
```