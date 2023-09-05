## 1. Use of deprecated _setupRole function
Multiple contracts make use of the deprecated function _setupRole from the AccessControl contract. 
https://github.com/OpenZeppelin/openzeppelin-contracts/issues/3488

Total Instances: 8

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L125

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L25

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L26

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L80

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L58

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L80

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L126

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L127

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L61

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L62

Change _setupRole to _grantRole

## 2. Remove SafeMath (Unnecessary After Solidity 0.8.0)

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L29

Consider removing the SafeMath import and its usage in the contract to simplify the code. Solidity 0.8.0 and later versions have built-in overflow and underflow checks, making SafeMath redundant.

By removing SafeMath, you will streamline the contract code and potentially reduce gas costs for contract deployment and function execution.

## 3. Describe events

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L29

```    
emit log(positions_array.length); 
emit log(positions_mapping[pos.token_id].token_id);
```
The contract uses generic log events without any descriptive labels or context. This makes it difficult to understand what the emitted events signify, especially for debugging or monitoring contract activities.

## 4. Check _fundingDuration value

The function updateFundingDuration does not check if the passed-in _fundingDuration value is zero before setting it. Failing to validate this could lead to unintended behavior or potential vulnerabilities in the contract.