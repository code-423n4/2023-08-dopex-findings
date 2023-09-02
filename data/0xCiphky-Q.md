## Restriction for zero approvals can lead to a loss of funds if contract gets compromised

### **Relevant GitHub Links:**

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L133

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L410

### **Summary:**

The protocol ensures a significant degree of independence between its contracts, further strengthened by the provision of methods to pause a contract and execute emergency withdrawals. Nonetheless, there remains a latent vulnerability. Contracts like **`UniV2LiquidityAMO`** and **`RdpxV2Core`** have the function **`approveContractToSpend`** which, under administrative privileges, can set an approval to any non-zero value. In the unfortunate event of a compromised contract, the majority of the funds can be shielded by resetting the approval but not all as we can’t set approvals to zero.

### **Impact:**

Severity: Low.  The majority (about 99%) of the funds can be shielded by resetting the approval to the minimal value.

Likelihood: Low. This vulnerability becomes relevant only if a contract is compromised.

### **Tools Used:**

Manual analysis

### **Recommendation:**

Allow the following contracts to reset approvals to zero.

## Event Emits Incorrect tokenId in UniV3LiquidityAMO's removeLiquidity Function

### **Severity:**

- Low Risk

### **Relevant GitHub Links:**

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L269

### **Summary:**

The **`removeLiquidity`** function within the **`UniV3LiquidityAMO`** contract is designed to emit an event that logs the **`tokenId`** for the liquidity position being removed. However, due to the sequence of operations, the emitted **`tokenId`** is always zero. This is because the function emits the **`tokenId`** after it's deleted from the **`positions_mapping`** structure.

The code is as follows:

```solidity
    function removeLiquidity(uint256 positionIndex, uint256 minAmount0, uint256 minAmount1)
        public
        onlyRole(DEFAULT_ADMIN_ROLE)
    {
        // rest of the code

        positions_array[positionIndex] = positions_array[positions_array.length - 1];
        positions_array.pop();
        delete positions_mapping[pos.token_id];

        // send tokens to rdpxV2Core
        _sendTokensToRdpxV2Core(tokenA, tokenB);

        emit log(positions_array.length);
        emit log(positions_mapping[pos.token_id].token_id);
    }
```

### **Impact:**

The inaccurate logging of the **`tokenId`** can lead to confusion and misinterpretation when analyzing contract events.

### **Tools Used:**

- Manual analysis

### **Recommendation:**

To address this issue, the **`tokenId`** should be cached before its deletion from the **`positions_mapping`** structure. Then, the cached **`tokenId`** should be used in the event emission. Here's a suggested code modification:

```solidity
    function removeLiquidity(uint256 positionIndex, uint256 minAmount0, uint256 minAmount1)
        public
        onlyRole(DEFAULT_ADMIN_ROLE)
    {
        // rest of the code

        positions_array[positionIndex] = positions_array[positions_array.length - 1];
        positions_array.pop();
	uint256 tokenId = positions_mapping[pos.token_id]
        delete positions_mapping[pos.token_id];

        // send tokens to rdpxV2Core
        _sendTokensToRdpxV2Core(tokenA, tokenB);

        emit log(positions_array.length);
        emit log(tokenId);
    }
```