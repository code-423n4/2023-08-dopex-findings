# 1) Owner will not be able to receive recovery withdrawal, because of the modifier `onlyRole(DEFAULT_ADMIN_ROLE)`.
Only Admin is able to get recovery withdrawal, because of the modifier `onlyRole(DEFAULT_ADMIN_ROLE)`. Consider allowing owners obtaining recovery too, if it's intended.
According to the function's comments - owner should have access to it.

Instance:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L327

###### Recommended mitigation steps:
Consider changing the modifier of the function `recoverERC721` to `onlyRole(OWNER)` or add `onlyOwner` modifier.
