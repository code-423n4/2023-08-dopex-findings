# [G-1] Consider make variable constant since it doesn't change anywhere.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L104

```solidity
--  uint256 public roundingPrecision = 1e6; // @audit make constant and private to save gas
++  uint256 private constant roundingPrecision = 1e6;
```

Deployment Cost `3802550` - Before gas optimization

Deployment Cost `3776436` - After gas optimization.

Difference: `26114`