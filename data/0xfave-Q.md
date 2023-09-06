# L-01 Missing zero-address checks
## Impact
Checking addresses against zero-address during initialization or during setting is a security best-practice. However, such checks are missing in all address variable initializations.

Impact: Allowing zero-addresses will lead to contract reverts and force redeployments if there are no setters for such address variables.
## Proof of concept
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L124-L126
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L76-L78
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L274-L287
## Tools Used
manual review

## Recommended Mitigation Steps
add zero address validation
like
```
require(_weth != address(0), "RdpxV2Core: _weth cannot be 0 address");
```