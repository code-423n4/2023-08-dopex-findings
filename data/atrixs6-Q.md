[RdpxDecayingBonds.sol#L144](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L144)

```solidity
  bonds[bondId].rdpxAmount = amount;
```

Should be

```solidity
  bonds[bondId].rdpxAmount -= amount;
```