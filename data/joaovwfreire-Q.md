# File: RdpxV2Core.sol

### DDoS Risk with RDpx Burn Percentage

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L180-L186

The contract does not apply any sanity checks on the maximum RDpx burn percentage. This lack of constraints can potentially expose the system to DDoS or other related attacks where malicious actors manipulate burn percentages.
Recommendation: Introduce constraints on permissible values for RDpx burn percentages to ensure controlled behavior and prevent potential misuse.

### DDoS Risk with RDpx Fee Percentage

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L193-L199

The contract does not apply any sanity checks on the maximum RDpx fee percentage. This lack of constraints can potentially expose the system to DDoS or other related attacks where malicious actors manipulate burn percentages.
Recommendation: Introduce constraints on permissible values for RDpx fee percentages to ensure controlled behavior and prevent potential misuse.

### Unbounded Bond Maturity

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L228-L234
The contract does not set an upper limit for bond maturity. Without constraints, users can set very long bond maturities, potentially causing liquidity issues.
Recommendation: Introduce an upper limit for bond maturity to ensure controlled behavior.

### Repeated AMO addresses

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L378-L381

The contract allows for repeated AMO addresses, which can lead to unnecessary redundancy and confusion.
Recommendation: Introduce a mechanism to prevent the addition of duplicate AMO addresses.

## Absence of Upper Limits for slippage tolerance and bond discount

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L437-L462

Those variables contract lack defined upper bounds. Without constraints, these parameters can be set to extreme values, potentially causing free bonds and 100% slippage on swaps.
Recommendation: Clearly define and implement limit bounds at the functions.


# File: UniV2LiquidityAmo.sol

### No Upper Limit for Slippage Tolerance

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L109-L117

The contract does not define an upper limit for slippage tolerance. This oversight can expose users to high slippages during token swaps, leading to potential significant financial losses.
Recommendation: Implement an upper limit for slippage tolerance, ensuring that users are protected from extreme price deviations during swaps.

### Unlinked Token Approval

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L126-L135

The token approval mechanism in the contract is not linked to any specific function or operation. This broad approval can potentially expose the system to malicious actors who can misuse the approvals.
Recommendation: Restrict token approval to specific functions or operational scenarios, ensuring a more secure contract environment.

# File: UniV3LiquidityAmo.sol

### Hardcoded Addresses
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L88

The contract contains hardcoded addresses. Using fixed values can be inflexible and might cause operational issues if these addresses change or become obsolete in the future.
Recommendation: Implement a more dynamic approach to manage these addresses, possibly through administrative functions that allow updates when necessary, ensuring contract adaptability.

# File: RdpxDecayingBonds.sol

### Decrease amount function actually sets amount
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L135-L145
The decrease amount fucntion doesn't decrease the amount value, but rather sets it based on the argument passed.
Recommendation: Introduce a check to verify bonds[bondId].rdpxAmount > amount.

### Immediate Bond Expiry

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L114-L125

The bond expiry can be set to a value less than the current timestamp, resulting in immediate bond expiry. This can lead to unintentional behaviors and financial implications for users.
Recommendation: Implement checks or constraints to ensure bonds have a valid and future expiry date.

# File: PerpetualAtlanticVaultLP.sol
QA Observations:
Missing Zero Address Checks
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L94-L97

The contract does not perform checks against the zero address for _rdpxRdpxV2Core and _collateral‎ at the constructor. This suggests it's okay not to have addresses for those arguments. Interactions involving the zero address can lead to lost funds or other unintended behaviors.
Recommendation: Implement checks to prevent any operations involving the Ethereum zero address, enhancing the contract's safety.