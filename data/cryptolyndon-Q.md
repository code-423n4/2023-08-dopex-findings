OK a last minute one..

In `removeAssetFromtokenReserves()`, ([RdpxV2Core.sol, line 270](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L270)), the array of token symbols (`reserveTokens`) has its last element popped, but not moved into the vacated slot as is done for `reserveAssets` ([line 283](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#283))


