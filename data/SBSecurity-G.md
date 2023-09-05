## Gas Optimizations
| Number |Issue|Instances| Estimated Gas Saved |
|-|:-|:-:|:-:| 
| [G-01](#g-01-use-calldata-instead-of-memory-for-function-arguments-that-do-not-get-mutated) | Use `calldata` instead of `memory` for function arguments that do not get mutated | 1 | 580 |

*Total Estimated Gas Saved: 580*

# [G-01] Use `calldata` instead of `memory` for function arguments that do not get mutated

When you specify a data location as `memory`, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

Total instances: `2`

Gas Savings for `RdpxV2Core.bondWithDelegate`, obtained via protocol’s tests: Avg 580 gas

|||
|-|-|
| Memory | 788179 |
| Calldata | 787599 |

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L819-L824

```solidity
File: contracts/core/RdpxV2Core.sol.sol
819:      function bondWithDelegate(
820:          address _to,
821:          uint256[] memory _amounts,
822:          uint256[] memory _delegateIds,
823:          uint256 rdpxBondId
824:        ) public returns (uint256 receiptTokenAmount, uint256[] memory) { {
```

```diff
function bondWithDelegate(
    address _to,
-   uint256[] memory _amounts,
-   uint256[] memory _delegateIds,
+   uint256[] calldata _amounts,
+   uint256[] calldata _delegateIds,
    uint256 rdpxBondId
) public returns (uint256 receiptTokenAmount, uint256[] memory) {

```