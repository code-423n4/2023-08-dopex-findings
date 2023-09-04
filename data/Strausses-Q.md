###Create an error when the first value is bigger than the second one.

Add this piece of code.
```
_validate(delegate.amount - delegate.activeCollateral, 15);
```
Otherwise tx could just revert without any error message
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L983


###Missing check for 0 address - missed spots by Bot race

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L120C13-L120C21
Missing check that ```reciever``` is not 0 address

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1018
Missing check that ```to``` is not 0 address


###Private func, should be named starting from "_" 
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L286


###Add custom error message when allowance is already 0/no allowance
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L156


###It would be better to add Sanity Checks

Sanity Check is a call to a contract to make sure that this is the expected/correct input

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L74-L102
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L305
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L115

