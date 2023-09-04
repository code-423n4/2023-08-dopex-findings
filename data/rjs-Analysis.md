# 1. Summary

## 1.1. Approach Taken

**Preparation**
- Reviewed the Dopex documentation with a particular focus on understanding `$rDPX` tokenomics and Atlantic Options
- Reviewed the Dopex [Solidified Audit (2021)](https://github.com/solidified-platform/audits/blob/master/Audit%20Report%20-%20Dopex%20[21.06.2021].pdf)
- Reviewed the Dopex [Solidity Finance Audit (2022)](https://solidity.finance/audits/Dopex/)
- Reviewed the Dopex [Solidity Finance Audit (2022; BNB SSOV)](https://solidity.finance/audits/DopexSSOV/)
- Reviewed the [rDPX V2 technical specification](https://dopex.notion.site/rDPX-V2-RI-b45b5b402af54bcab758d62fb7c69cb4)
- Documented diagram of in-scope system architecture and drafted the mechanism review

**Execution**
- Reviewed the core contracts in principal, and identified Areas of Concern
- Initial code review
- Drafted reports
- Deep code review
- Wrote & submitted reports

**Effort**
21 hours total

## 1.2. Mechanism Review

rDPX V2 is a complex new system involving changes to the existing `rDPX` token which forms part of the Dopex ecosystem. It introduces `dpxETH`, which is pegged to `ETH` in a 1:1 ratio. `dpxETH` will be designated as a staple collateral token for future Dopex options products. The `dpxETH`:`ETH` peg is maintained via an interplay between bonding mechanisms, backing reserves controlled via AMOs, and a peg stability module. There are dependencies on Curve Finance, Uniswap V2 and Uniswap V3 for performing some of these functions.

| Component                  | Notes                                                                                           |
| -------------------------- | ----------------------------------------------------------------------------------------------- |
| Peg Stability Module (PSM) | Ensures `dpxETH` to `ETH` peg on the Curve pool                                                     |
| Regular Bonding Mechanism | Deposit `rDPX` + `ETH` to mint a bond -> send `rDPX`/`ETH` to backing reserves/Curve -> burn/send `rDPX` as rewards -> mint `ERC721` bond token |
| Delegate Bonding | Users with `rDPX`-only or `ETH`-only deposit into pool and then can use pool to perform regular bonding |
| Bonding via `rDPX` Decaying Bond | Similar to regular bonding mechanism, but there is a maturity + redemption process involved |
| Atlantic Perpetual PUTS Options | A perpetual options pool for `rDPX` options which the Core Contract uses for the bonding process with 1 week epoch |
| Receipt Token | Represents `dpxETH`/`ETH` LP tokens on the curve, and are staked in Curve gauges for rewards then distributed to receipt token stakers and the Core Contract |
| Receipt Redemption | Redeemed at current ratio of the `rDPX`/`ETH` backing reserve, less redemption fee |
| AMO | Inspired by Frax's AMO but tailored for rDPX V2; modifications should be carefully reviewed |

## 1.3. Areas of Concern

| Component                 | Concern                             |
| -------------------- | ----------------------------------- |
| Peg Stability Module | Is it possible to depeg `dpxETH`:`ETH`, e.g. under extreme market conditions or aggressive market manipulation? |
| Regular Bonding Mechanism | Is the bonding mechanism susceptible to economic attacks due to its reliance on market prices e.g. flash loan attacks or oracle manipulations |
| Regular Bonding Mechanism | Is liquidity a concern any where in this process, or is it fully controlled by the Dopex contracts? |
| Delegate Bonding | Are there any sequences of interactions in the delegated bonding process that result in unexpected outcomes for `rDPX` and `ETH` holders participating in the pool?  |
| Bonding via `rDPX` Decaying Bond | Do the calculations here always hold through all market and reserve conditions? Can the redemption process be brought forward or exploited? |
| Atlantic Perpetual PUTS Options | Are there surprises on the epoch boundaries? Is it possible to deny service to users by blocking collateral availability? | 
| Receipt Token | Can the current ratio of the Backing Reserve in `rDPX`/`ETH` provide an exploit path to unexpected behavior elsewhere? |
| AMO | Does separating out the Decollateralize and Recollateralize into the Treasury single PSM violate any assumptions in the Frax AMO ecosystem? |
| Dependencies | Can unlikely failure in two components produce an unexpected interaction? |

# 2. Codebase Quality Analysis

## 2.1. Codebase Review

Each source unit was assessed with respect to modularity, access control mechanism, centralization risks, validation completeness and documentation quality:

| Source Unit                                                                                                                                        | Modularity  | Access Control | Centralization         | Validation  | Documentation |
| -------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- | -------------- | ---------------------- | ----------- | ------------- |
| [UniV2LiquidityAmo.sol](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol)                             | 🟢 Good      | 🟢 Role-based   | 🟡 Admin-only functions | 🟢 Excellent     | 🟢 Good        |
| [UniV3LiquidityAmo.sol](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol)                             | 🟢 Good      | 🟢 Role-based   | 🟡 Admin-only functions | 🟢 Excellent     | 🟢 Good        |
| [RdpxV2Core.sol](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol)                                         | 🟢 Excellent | 🟢 Role-based   | 🟡 Admin-only functions | 🟢 Excellent | 🟢 Good        |
| [RdpxV2Bond.sol](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol)                                         | 🟢 Excellent | 🟢 Role-based   | 🟢 Standard             | 🟢 Standard  | 🟢 Standard    |
| [RdpxDecayingBonds.sol](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol)       | 🟢 Excellent | 🟢 Role-based   | 🟢 Standard             | 🟢 Standard  | 🟢 Standard    |
| [DpxEthToken.sol](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol)                                   | 🟢 Excellent | 🟢 Role-based   | 🟢 Standard             | 🟢 Standard  | ⚪ N/A         |
| [PerpetualAtlanticVault.sol](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol)     | 🟢 Excellent | 🟢 Role-based   | 🟡 Admin-only functions | 🟢 Excellent | 🟡 Basic       |
| [PerpetualAtlanticVaultLP.sol](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol) | 🟢 Excellent | 🟢 Creator-only | 🟢 None                 | 🟡 Basic     | 🟡 Basic       |
| [ReLPContract.sol](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol)                                     | 🟢 Excellent | 🟢 Role-based   | 🟡 Admin-only functions | 🟢 Excellent | 🟡 Basic       |

## 2.2. Maturity Level

- "Admin-only functions" represents a centralization risk; ideally, a more granular set of roles would be implemented
- "Basic" validation refers to code where not all parameters or scenarios were validated; details in the QA report
- "Basic" documentation refers to code lacking a clear description of code purpose, assumptions, etc. 
- Overall, the codebase demonstrated high maturity.

## 2.3. Recommendations

* Validation and documentation should be expanded in the areas highlighted yellow above
* Where possible, centralization risks should be reduced by splitting dedicated roles out of the single "admin" role


# 3. Architecture Recommendations & Risks

## 3.1. Consider reducing Peg Stability Module (PSM) dependencies

The PSM is entirely controlled by the Core Contract managers; if these are compromised or act maliciously, they could disrupt the peg.  There could also be scenarios where the Curve pool may not have enough liquidity to correct the peg effectively.

## 3.2. Consider improvements to reduces parameters in the bonding mechanism

Multiple parameters (e.g. discount factor) are set by governance. If governance is compromised or acts inefficiently, it can affect the stability of the bonding process. It is worth considering if improvements are possible to allow parameters to dynamically reflect economic conditions rather than being configured through governance.

## 3.3. Consider making it possible for funding to be calculated without the APP contract admin

"The APP contract admin needs to calculate the funding needs to be called for every strike before the epoch ends".

Ideally any critical processes ought to be executable directly or indirectly by the impacted users in the ordinary course of operations, to reduce centralization risks.

## 3.4. Consider more granular roles for System Parameters

System parameters are set by governance and therefore pose a centralization risk. Consider splitting system parameters by role if possible.

## 3.5. Document system intention through more worked examples

It is possible to create a `forge script` that runs multiple scenarios and assumptions through the protocol and outputs graph points, when can then be rendered graphically. This would greatly improve the documentation and communicate system intention more clearly.

### Time spent:
21 hours