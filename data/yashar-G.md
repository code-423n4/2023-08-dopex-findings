In the RdpxDecayingBonds.sol contract, the mint function double checks the caller to ensure that he has the MINTER_ROLE or not, which is neither gas-efficient nor necessary while there is a function modifier that checks if the caller has the MINTER_ROLE.

First Check (Modifier):
https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/decaying-bonds/RdpxDecayingBonds.sol#L118

Double Check (Not-necessary):
https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/decaying-bonds/RdpxDecayingBonds.sol#L120

## Recommendation:
Remove the second check (require)
```
require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
```