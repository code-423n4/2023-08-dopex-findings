## `removeAssetFromtokenReserves` dosnt check if the token still has funds in the contract 
If the owner Removes Weth it will be reset to `tokenBalance=0`.
If we where to add it again it will become [`0`](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L255)
