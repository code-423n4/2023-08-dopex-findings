## G1 
### Double Check
In RdpxDecayingBonds.mint, onlyRole(MINTER_ROLE) modifier has been used to restrict the function, but another check was done for the same using a require statement. 
```require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");```
This check is no longer necessary as the modifier already checked for it.