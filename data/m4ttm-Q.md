### Non-Critical Issues List
|Number|Issues Details|Occurrences|
|:--:|:-------|:--:|
| [NC-01] |Missing Event for critical parameters change | 3 |

Total 1 issues

### [NC-01] Missing Event for critical parameters change

```solidity
contracts/reLP/ReLPContract.sol:
  115:  function setAddresses(

contracts/reLP/ReLPContract.sol:
  171:  function setLiquiditySlippageTolerance(

contracts/reLP/ReLPContract.sol:
  186:  function setSlippageTolerance(
```

**Description:**
Events help non-contract tools to track changes, and events prevent users from being surprised by changes

**Recommendation:**
Add Event and Emit