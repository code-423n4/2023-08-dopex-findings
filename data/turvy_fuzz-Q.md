## Using || here is not correct
Description:
the require check here is to ensure no zero address is mistakenly passed in the specified values however this won't be the case any one of them can still be mistakenly set to address(0) due to the || operator in the require.

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L95

Recommendation:
Instead of ||, use && here to ensure the intended check