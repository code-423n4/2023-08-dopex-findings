**contracts/decaying-bonds/RdpxDecayingBonds.sol**
- L53 - The EmergencyWithdraw event is created, but it is never used, therefore it should be eliminated so that it does not take up unnecessary space in storage.

- L118/12 - The same thing is validated twice, from the modifier with onlyRole(MINTER_ROLE) and in the require, hasRole(MINTER_ROLE, msg.sender), this generates unnecessary extra gas spending.


**contracts/perp-vault/PerpetualAtlanticVaultLP.sol**
- L52 - The variable underlyingSymbol is created in storage, but it was never used, so it should be removed.

- L192/195/201/204/210/213 - If a validation is performed and then the same operation is performed, it could return unchecked the second time.


**contracts/perp-vault/PerpetualAtlanticVault.sol**
- L89/225/328/413 - A variable is generated and then it is set with the default value, this is an unnecessary waste of gas, since that value is already defined.
