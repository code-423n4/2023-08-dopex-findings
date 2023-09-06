## Summary

### Gas Optimization

| no | Issue | Instances |
| --- | --- | --- |
| [G-01] |  Before transfer of some functions, we should check some variables for possible gas save | 2 |
| [G-02] | Can Make The Variable Outside The Loop To Save Gas | 3 |
| [G-03] | Use calldata instead of memory for function parameters | 5 |
| [G-04] | Using storage instead of memory for structs/arrays saves gas | 1 |
| [G-05] | Assigning state variables to default value can be omitted to save gas | 1 |
| [G-06] | Compute hashing off-chain to save gas at deployment | 9 |
| [G-07] | State variables should be cached outside the loop to save gas | 1 |

### [G-01] **Before transfer of some functions, we should check some variables for possible gas save**

Before transfer, we should check for amount being 0 so the function doesn't run when its not gonna do anything:

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1036

```solidity
File: RdpxV2Core.sol
1036: IERC20WithBurn(addresses.rdpxV2ReceiptToken).safeTransfer(
1037:      to,
1038:      receiptTokenAmount
1039:    );
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L172

```solidity
File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
172: IERC20WithBurn(rdpx).safeTransfer(receiver, rdpxAmount);
```

### [G-02] Can Make The Variable Outside The Loop To Save Gas

In Solidity, declaring variables inside loops can lead to unnecessary gas costs due to the creation of new instances of the variable for each iteration. To enhance the efficiency of your contract and reduce gas consumption, it is recommended to declare such variables outside the loop.

When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

```
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;

        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }

        return total;
    }
}

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L329C5-L331C1

```solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol

329: uint256 strike = optionPositions[optionIds[i]].strike; 
330: uint256 amount = optionPositions[optionIds[i]].amount;
...
419: uint256 strike = strikes[i];
```

## [G-03] Use `calldata` instead of memory for function parameters

When you specify a data location as `memory`, that value will be copied into memory. When you specify the location as `calldata`, the value will stay static within `calldata`. If the value is a large, complex type, using memory may result in extra memory expansion costs.

*Note: This is a commonly known high impact finding; however, due to lack of available tests the exact gas costs can not be known. We will therefore be conservative in our projected gas savings and give each instance a projected savings of 100 gas.*

**`strikes` array is passed as a memory array. Consider using `calldata` instead of `memory` to save gas costs for large input arrays. 312 per interation**

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L405C3-L407C60

```solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol
405: function calculateFunding(
406:    uint256[] memory strikes //@audit use calldata
407:  ) external nonReentrant returns (uint256 fundingAmount) {
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L315C2-L317C4

```solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol

315: function settle(
316:    uint256[] memory optionIds   
317:  )
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L240C1-L242C31

```solidity
File: contracts/core/RdpxV2Core.sol

240: function addAssetTotokenReserves(
241:    address _asset,
242:    string memory _assetSymbol      //@audit Use calldata
243:  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L270C2-L272C44

```solidity
File: contracts/core/RdpxV2Core.sol

270: function removeAssetFromtokenReserves(
271:    string memory _assetSymbol
272:  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L764C3-L766C4

```solidity
File: contracts/core/RdpxV2Core.sol

764: function settle(
765:    uint256[] memory optionIds //@audit use calldata
766:  )
```

### [G-04] Using `storage` instead of `memory` for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a `memory` variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (**2100 gas**) for *each* field of the struct/array. If the fields are read from the new memory variable, they incur an additional `MLOAD` rather than a cheap stack read. Instead of declearing the variable with the `memory` keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires `memory`, or if the array/struct is being read from another `memory` array/struct

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1092C6-L1092C29

```solidity
File: contracts/core/RdpxV2Core.sol

1092: address[] memory path; //@audit Use storage
```

### [G-05] Assigning state variables to default value can be omitted to save gas

In the `PerpetualAtlanticVault.sol` contract the `latestFundingPaymentPointer` state variable is assigned to the default value of `0` as shown below:

- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L89

```solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol
89: uint256 public latestFundingPaymentPointer = 0; //@audit dont initialize state variable
```

This can be omitted to save gas, since by default  variable will have the value `0`. This will save an extra `SSTORE` operation.

### [G-06] Compute hashing off-chain to save gas at deployment

 [Here](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L44C1-L46C83) we can compute these values off-chain to save gas at deployment as hashing is expensive for example:

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L19C1-L22C1

```solidity
File: contracts/dpxETH/DpxEthToken.sol
19:  bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
20:  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
21:  bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L70

```solidity
File: contracts/reLP/ReLPContract.sol
70: bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L31C1-L34C74

```solidity
File: contracts/decaying-bonds/RdpxDecayingBonds.sol
31: bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE"); 
32:
33:  // Create a new role identifier for the Rdpx v2 core role
34:  bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L22

```solidity
File: contracts/core/RdpxV2Bond.sol
22: bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L45C1-L49C1

```solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol
45: bytes32 public constant MANAGER_ROLE = keccak256("MANAGER_ROLE"); //@audit
46:
47:  /// @dev Rdpx v2 core role which can purchase and settle options
48:  bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
```

### [G-07] State variables should be cached outside the loop to save gas

State variables should be cached outside the loop to save gas. This is because the gas cost of accessing a state variable inside a loop is higher than the gas cost of accessing a state variable outside a loop

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L121

```solidity
File: contracts/amo/UniV3LiquidityAMO.sol
121: Position memory current_position = positions_array[i]; //@audit cache variable outside loop
```