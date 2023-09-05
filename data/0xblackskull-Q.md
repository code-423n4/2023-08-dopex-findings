In contract PerpetualAtlanticVaultLP constructor use `||` instead of `&&` so deployer could set zero  value between _perpetualAtlanticVault or _rdpx
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L95
``` use &&
require(_perpetualAtlanticVault != address(0) && _rdpx != address(0), "ZERO_ADDRESS");
```