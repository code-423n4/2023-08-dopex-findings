# Genesis should be before or equal to current timestamp
In the [constructor of the PerpetualAtlanticVault](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L124) the `_genesis` parameter should be checked to be before or equal to the current timestamp.

# Proposed fix
Add the following line to the constructor:
```sol
    _validate(block.timesamp >= _genesis, 1);
```