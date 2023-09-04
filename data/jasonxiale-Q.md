# missing `IRdpxV2Core(rdpxV2Core).sync();` in UniV2LiquidityAmo._sendTokensToRdpxV2Core
In [UniV3LiquidityAmo._sendTokensToRdpxV2Core](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L353-L364), while `_sendTokensToRdpxV2Core` is called, the function will call `IRdpxV2Core(rdpxV2Core).sync();` to sync the balance  of tokens, so I think it's better to call the same function in [UniV2LiquidityAmo._sendTokensToRdpxV2Core](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L160C12-L178)

# Incorrect event messages in UniV2LiquidityAmo.sol which should using `"UniV2LiquidityAmo"` instead of `"reLPContract"`
In UniV2LiquidityAmo.sol, there are some incorrect event messages with string as __"reLPContract"__, which I think should be changed to __"UniV2LiquidityAmo"__
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L91
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L114C8-L114C20
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L131-L133

# 
