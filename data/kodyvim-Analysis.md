# Dopex Analysis Report
## Mechanism Review
**Bonding Mechanism:**
The bonding mechanism is a crucial part of the rDPX V2 system, as it allows users to mint new dpxETH tokens. This mechanism can be divided into three main methods: regular bonding, delegate bonding, and bonding via rDPX decaying bonds.

**Regular Bonding:**
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L894
Users deposit rDPX and ETH to mint dpxETH tokens.
A discount is applied to the rDPX and ETH provided based on a discount factor set by governance.
rDPX and a portion of ETH are sent to the Backing Reserve to mint dpxETH.
Some rDPX is burnt from the Treasury Reserve, and the rest is sent as emissions to veDPX holders.
Discounts provided are covered by the Treasury Reserve.
A reLP process is applied to Uniswap V2 liquidity to increase the price of rDPX.

**Delegate Bonding:**
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L699
Users with ETH deposit it in a pool and set a fee for usage.
rDPX holders can use this pool of ETH to bond using the regular bonding mechanism.
This allows both rDPX and ETH holders to participate in bonding.
Bonding via rDPX Decaying Bond:

Decaying bonds are minted to users as rebates for losses or incentives.
Users deposit ETH to mint dpxETH tokens.
A portion of the ETH is sent to the Backing Reserve to mint dpxETH.
The full amount of rDPX is provided by the Treasury Reserve.

**Atlantic Perpetual PUTS Options:**
The Atlantic Perpetual PUTS Options system provides a pool for rDPX options. It operates on an epoch basis, and funding is calculated based on options' activity. Funding comes from a portion of CRV and is used to pay option LPs.

**Receipt Token:**
Receipt tokens represent dpxETH/ETH LP tokens on the curve. They are staked in curve gauges with boosted rewards. Users can redeem receipt tokens, and a redemption fee is charged.
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L495

**AMO (Algorithmic Market Operations):**
AMOs in rDPX V2 are responsible for market operations, including adding/removing liquidity on Uniswap V2 and Uniswap V3 pools. These AMOs help maintain liquidity and market stability.
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L22
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L28

**Peg Stability Module (PSM):**
The PSM is responsible for maintaining the peg of dpxETH to ETH. It includes functions for upperDepeg, lowerDepeg, and settling to ensure the peg remains close to 1 ETH.
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1051
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1080

**System Parameters:**
System parameters, set by governance, include veDPX Bonding Fee, reward distribution percentages, discount factor, bond maturity, and redemption fee. These parameters are crucial for system stability and incentives.

## Codebase quality analysis
| Category | Description |
|--- | --- |
|Access Controls | **Satisfactory**. Appropriate access controls were in place for performing privileged operations.|
|Arithmetic | **Moderate**. The contracts uses solidity version ^0.8.0 potentially safe from overflow/underflow while some decimal precision are not clearly stated |
| Centralization | **Moderate**. critical functionalities could be changed by an admin.|
| Code Complexity | **Satisfactory**. Most part of the protocol were easy to understand |
|Contract Upgradeability | **Satisfactory**. Contracts not upgradable.|
| Documentation | **Satisfactory**. High-level documentation and some in-line comments describing the functionality of the protocol were available.|
| Monitoring | **Satisfactory**. Events are emitted on performing critical actions.|

The provided test files were very easy to POC and very organised.

## Centralization risks
**Smart Contract Ownership:** Ownership and control of critical smart contracts can be a centralization risk. If the private keys of privilege individuals within the system are compromise the whole system Would be destablized.
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L82
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L126
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L144
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L407

**Oracles:** Relying on a single centralized oracle or a limited number of oracles also introduces centralization risk.
it's quite unknown yet given the scope if rdpxV2 would use a centralized or decentralized oracle for price determination.
Manipulation or failure of the oracle can affect the accuracy of data used in the system.

**Treasury Control:** If a single entity or a small group has control over the project's treasury, they may have significant influence over funding allocation and decision-making.

**Collateral Concentration:** If a significant portion of the system's collateral is held by a few large players, it can centralize risk. Large collateral holders may have disproportionate control over the system's stability which can have direct impact on the peg to WETH.

**AMO Control:** The AMOs (Algorithmic Market Operations Controllers) responsible for market operations can introduce centralization risk if they are controlled by a single entity or a small group.


## Systemic Risks
**Oracle Failures:** Any Reliance on centralized oracles or single data sources can pose systemic risks. If oracles provide inaccurate or manipulated data, it can lead to incorrect pricing, liquidations, or other financial disruptions.

**Liquidity Shortages:** peg on DpxETH/ETH relies on liquidity providers on curve pool. If there's a sudden withdrawal of liquidity from key pools or platforms, it can trigger cascading effects, causing slippage, instability, and potential insolvency.

**Flash Loan Attacks:** peg could be susceptible to flash loans attacks that exploit arbitrage opportunities and manipulate prices, leading to systemic instability if not well-mitigated.

**Regulatory Risks:** Regulatory actions or changes in regulations could impact the legality and operation of DeFi projects, potentially leading to disruptions.

## Architecture Recommendations.
* roles assignment should be more decentralized, admin possess all roles
  Pauser role could be a different entity as well as Minter role
* Additional checks are not required when using safe transfer and should be removed.

## Approach taken
Read through the provided Notion notes.
Skim through the code provided and take note of relevant points.
Checking assumptions against the provided test files.
Proceed with quality assurance (QA) and report writing.

### Time spent:
94 hours