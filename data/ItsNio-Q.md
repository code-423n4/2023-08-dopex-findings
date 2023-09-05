# Funding rates can be increased before `genesis` in PerpetualAtlanticVault

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L292

### Impact
`fundingRate` can be updated before `block.timestamp` = `genesis`.

### POC
According to the comment on line [112](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L112), `genesis` is the starting time for fund rate calculation. However, `genesis` can be set to any value, even values greater than `block.timestamp`.

Hence, if `genesis` is set to any value between `block.timestamp + 1` and `block.timestamp + fundingDuration` and the function `purchase()` is called at a time before `genesis`, the funding rate will be updated via the `_updateFundingRate()` function call on line [292](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L292) without reverting. This violates the definition of `genesis`.