Unchecked return value of transferFrom can allow a user to mint some shares for free

URL : https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L128

The PerpetualAtlanticVaultLP contract has a function deposit() which calls an unsafe transferFrom() to transfer assets from msg.sender to address(this) on the collateral contract, A call to transferFrom is frequently done without checking the results. For certain ERC20 tokens, if insufficient tokens are present, no revert occurs but a result of “false” is returned. check : https://consensys.net/diligence/audits/2021/01/fei-protocol/#unchecked-return-value-for-iweth-transfer-call

And in this function case, if the collateral.transferFrom() returns false, it would continue the call to mint shares on the PerpetualAtlanticVaultLP contract. 