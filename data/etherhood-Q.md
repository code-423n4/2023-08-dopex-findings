- Use of safeApprove of SafeERC20
A lot of places, approve calls are being used, which can fail for non standard tokens
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L411
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L243
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L173

- Use of safeTransfer/safeTransferFrom of SafeERC20
There are places where transferFrom/transfer is being used, which can fail for non standard token which don't return any bool value
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L158
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L128
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L170

- Use of FullMath.mulDiv of OZ FullMath library to avoid overflow error while multiplying
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L232
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L250
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L273
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L335

- Incorrect validation
  - `slippageTolerance` in ReLPContract can be limited to 5-10% max value
  - `fundingDuration` in PerpetualAtlanticVault can be limited to 30 days as max value
  - `bondDiscountFactor` in RdpxV2Core can be limited to 1e6/1e8 as max value
Values greater than a certain number can lead all kind of mis calculations.

- No zero address check
  - constructor of RdpxV2Core

- Multiple redundant checks at `settle` https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L315, of _isEligibleSender(), leading to need to set RdpxV2Core contract as whitelist as well, where as there is no need for it


