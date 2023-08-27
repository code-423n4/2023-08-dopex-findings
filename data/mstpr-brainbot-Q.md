1- RdpxReserve contracts "emergencyWithdraw" function can transfer ETH balance from the contract address to an address on call. However, this contract can't receive plain ether by receive/fallback or via payable functions. 
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reserve/RdpxReserve.sol#L60-L78


2- UniswapV3 AMO can have USDT as a liquidity asset. It also commented above in here that USDT can be an asset that the AMO handle. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L146
However, main functions are still using transferFrom and approve which is not adaptable with USDT alike tokens. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L158-L176

