This rule is mentioned in https://docs.openzeppelin.com/contracts/3.x/extending-contracts#using-hooks that **1. Whenever you override a parent’s hook, re-apply the virtual attribute to the hook. That will allow child contracts to add more functionality to the hook.**

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L50
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L167
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L59
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L640
