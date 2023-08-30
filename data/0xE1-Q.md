Admin can mistakenly add address(0) to addToContractWhitelist because there is no check:
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L419

Recommended is to avoid this by adding:
require(_addr != address(0));