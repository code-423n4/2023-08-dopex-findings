## Summary
[G-01] Remove `require` when there is a `modifier` with the same validation.
[G-02] Store calculations in a variable instead of doing the same calculations multiple times.

### [G-01] Remove `require` when there is a modifier with the same validation.
The function `mint` in `RdpxDecayingBonds.sol` has the modifier `onlyRole(MINTER_ROLE)` which checks if the `msg.sender` has the role `MINTER_ROLE`. The function then has a `require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");` which basically does the same as the modifier.

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/decaying-bonds/RdpxDecayingBonds.sol#L114-L125

#### Recommended Mitigation Steps
Remove the `require` statement.

### [G-02] Store calculations in a variable instead of doing the same calculations multiple times.
The function `updateFunding` in `PerpetualAtlanticVault.sol` performs the same calculation 3 times. It is better to save the result in a variable and than use the variable. The calculation is `(currentFundingRate * (block.timestamp - startTime)) / 1e18`.

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/perp-vault/PerpetualAtlanticVault.sol#L502-L524

#### Recommended Mitigation Steps
Store the value in a variable.