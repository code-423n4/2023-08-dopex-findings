## Vulnerability Detail
```solidity
require(
    _perpetualAtlanticVault != address(0) || _rdpx != address(0),
    "ZERO_ADDRESS"
);

```
- this check ensures that _perpetualAtlanticVault and _rdpx are not the zero address, it does not prevent both of them from being the zero address simultaneously.
- If both _perpetualAtlanticVault and _rdpx are set to the zero address, the condition _perpetualAtlanticVault != address(0) || _rdpx != address(0)) would still evaluate to true, allowing the contract to be deployed with invalid addresses.
so both addresses from being the zero address, it better to use the logical AND operator (&&) instead of OR (||) to ensure that both conditions are met simultaneously.
## Impact
lead contract to be deployed with invalid addresses.
## Code Snippet
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L94C5-L97C7
## Tool used
Manual Review
## Recommendation
both _perpetualAtlanticVault and _rdpx must be valid addresses for the contract to be deployed, Here's the updated code
```solidity
require(
    _perpetualAtlanticVault != address(0) && _rdpx != address(0),
    "ZERO_ADDRESS"
);
```