# G-01 Cache Function return value once and use it instead of making extra function calls to get the same value

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L594-L614

In `PerpetualAtlanticVault._updateFundingRate()` function, function calls are made to `nextFundingPaymentTimestamp()` in these lines [597](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L597), [600](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L600), [602](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L602) and [608](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L608) to get the `nextFundingPaymentTimestamp`. 

Instead of this, the return value of `nextFundingPaymentTimestamp()` can be cached at the start of the function and used when needed. This saves gas by avoiding an extra function call.

# G-02 Redundant check for role can be removed

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L114-L125
