0. QA/LOW: RdpxV2Core - Likely incorrect/misleading comment regarding precision for state variable `slippageTolerance`, or precision hasn't been added to variable value yet. 

https://github.com/code-423n4/2023-08-dopex/blob/e96aaa5ea21f11b29d828dbe2d0745974cd046ed/contracts/core/RdpxV2Core.sol#L99-L100

L100:
`slippageTolerance = 5e5` but comment mentions 1e8 precision as per below:

```solidity
  /// @notice The slippage tolernce in swaps in 1e8 precision
  uint256 public slippageTolerance = 5e5; // 0.5%
```

Recommendation:

Fix the comment or fix the assignment value, depending on which one is incorrect.

If assignment incorrect, then:

```solidity
uint256 public slippageTolerance = 5e5 * 1e8; // 0.5%
```

1. QA: RdpxV2Core - `liquiditySlippageTolerance` state variable not used in this contract.

https://github.com/code-423n4/2023-08-dopex/blob/e96aaa5ea21f11b29d828dbe2d0745974cd046ed/contracts/core/RdpxV2Core.sol#L102-L103

```solidity
  /// @notice Liquidity slippage tolerance
  uint256 public liquiditySlippageTolerance = 5e5; // 0.5%
```

Recommendation:

Remove the variable to save on gas costs during deployments, or comment it out if planning to use later, also consider its storage slot.


