## [G-01] EnumerableSet is not used in RdpxV2Core.sol and should be removed
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L26
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L40

The EnumerableSet is never used in RdpxV2Core.sol, so no need to import them.

## [G-02] feeDistributor in addresses variable in PerpetualAtlanticVault.sol is never used, should be removed
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L181-L212

Since this contract never use the feeDistributor, this can be removed from the addresses variable.