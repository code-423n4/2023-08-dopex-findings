# QA Report

## Low Severity Findings
- ### Unused function parameter 
The `redeem` function in `PerpetualAtlanticVaultLP` calls `beforeWithdraw` and passes in two arguments (assets, shares), however only one of these values (assets) is used in the `beforeWithdraw` function. [Redeem](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L166) and [beforeWithdraw](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L286) functions. Consider removing shares argument if it is unneeded to prevent ambiguity and unexpected behaviours.
