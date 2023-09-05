# Overview

Dopex is a decentralized options protocol with the primary goals of increasing liquidity, minimizing potential losses for option writers, and maximizing profits for option buyers. The audit contest on Code4rena is specifically focused on evaluating the newly introduced contract for the synthetic coin called dpxETH, which maintains a peg to the value of ETH. This is achieved through a bonding mechanism that generates new dpxETH tokens and ensures their value is secured through a combination of rDPX tokens and an ETH reserve.

In rDPX v2, the bonding mechanism is similar to OlympusDAO's approach. This mechanism allows users to lock their assets within the protocol in return for a different asset at a reduced rate, with a predetermined vesting period. These locked assets transform into what's known as Protocol-Owned-Liquidity (POL). This POL can serve various purposes, including providing backing for $dpxETH, supporting permissioned Automated Market Makers (AMMs), contributing to peg stability, or even participating in yield farming across other DeFi protocols.

The based on README of the contest, scope of audit is:

| Contract                                                                                                                                                     | Purpose                                                        |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------- |
| [contracts/amo/UniV2LiquidityAmo.sol](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol)                             | This contract encompasses all functions for the Uniswap V2 AMO |
| [contracts/amo/UniV3LiquidityAmo.sol](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol)                             | This contract encompasses all functions for the Uniswap V3 AMO |
| [contracts/core/RdpxV2Core.sol](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol)                                         | This is the core contract of rDPX V2                           |
| [contracts/core/RdpxV2Bond.sol](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol)                                         | ERC721 contract for minting NFT bonds via the core contract    |
| [contracts/decaying-bonds/RdpxDecayingBonds.sol](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol)       | Contract responsible to mint rDPX decaying bonds               |
| [contracts/dpxETH/DpxEthToken.sol](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol)                                   | ERC20 dpxETH token contract                                    |
| [contracts/perp-vault/PerpetualAtlanticVault.sol](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol)     | Contract for the Perpetual Atlantic Vault (ERC721)             |
| [contracts/perp-vault/PerpetualAtlanticVaultLP.sol](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol) | Contract for the Perpetual Atlantic Vault LP (ERC4626)         |
| [contracts/reLP/ReLPContract.sol](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol)                                     | Contract to perform the reLP process on the Uniswap V2 AMO     |

# Audit Methodology

Steps

1. **Read audit README to understand scope and what's expected from audit:** Begin by reviewing the audit's README document for Dopex. This step helps me grasp the scope of the audit and what's expected and objectives of the protocol

2. **Read Official Dopex docs:** Read and understand any relevant official project documentation. Understanding the project's specifications and design can provide valuable context for the audit.

3. **Read any existing audit or similar project audit reports:** If there are previous audit reports for Dopex or similar projects, I review it. This can uncover issues that were found and resolved in the past and give insights into potential vulnerabilities within Dopex.

4. **Screening through Dopex codebase and analyze entry points:** Start by skimming through the codebase of Dopex to identify entry points and key components. This initial review helps me get a high-level understanding of Dopex's code structure.

5. **Run test cases:** Execute predefined test cases (inside tests folder) to check if the code in Dopex behaves as expected. This step ensures basic functionality is intact before diving deeper into the code.

6. **Analyze logic of Dopex code base one by one thoroughly:** Go through the code of Dopex line by line, method by method, and thoroughly analyze the logic. Look for vulnerabilities, edge cases, and potential security issues specific to Dopex.

7. **Create different edge case scenarios:** Generate various edge case scenarios that the code in Dopex might encounter. This step helps test Dopex's resilience and its ability to handle unusual or unexpected inputs or conditions, which is crucial for security.

These steps collectively form a comprehensive audit methodology, allowing me to systematically assess the Dopex codebase for vulnerabilities and ensure the security and reliability of the Dopex project.

I seldom rely on audit tools since I presume that most projects have already undergone such analysis, and most auditors typically employ these tools. Therefore, my focus isn't on utilizing well-known automatic analysis tools.

# Codebase Quality Analysis

The audit scope encompasses a specific subset of Dopex's codebase. Notably, the staking contract, reserve contract, and oracles fall outside the audit's scope. However, it's great that the code within the audit scope is well crafted. It includes natspec annotations and well-documented comments for major functions and contracts.

The modular organization of the code, with distinct folders for core, amo, faucet, helper, interface, and libraries, greatly facilitates codebase navigation and comprehension.

It's also worth acknowledging the developer's meticulous preparation of contract interfaces. These interfaces feature clear and concise descriptions of functions and variables, enhancing code readability and usability.

In summary, the audited code exhibits a high standard of quality, characterized by well-documented annotations, modular organization, and thoughtful interface design.

# Architecture Design Analysis & Recommendation

the rDPX v2 consists of multiple components and based on the benefit of each, we can summarize as:

- **$rDPX bonding**

  - For Dopex: This component allows Dopex users to utilize POL to farm yield, which in turn generates more revenue for the protocol.

  - For rDPX holders: rDPX holders benefit by gaining profit through the discount offered when minting ETH.

- **v2 AMM and liquidity management**

  - For rDPX holders: This aspect benefits rDPX holders by contributing to a stronger rDPX price, potentially increasing the value of their holdings.

  - For rDPX option writers: Option writers experience reduced risk of loss due to the strengthened rDPX price.

  - For Dopex: A higher collateral value in the v2 Treasury results in a more robust dpxETH peg, enhancing stability within Dopex.

- **Peg Stability Module**

  - For DPX holders: The Peg Stability Module creates higher demand for veDPX, leading to improved price performance for DPX.

- **CRV emissions**

  - For dpxETH holders: This feature offers additional yield through farming and stimulates higher demand for dpxETH. Increased liquidity, making it less likely for dpxETH to de-peg.

In summary, each component of rDPX v2 offers specific benefits to different stakeholders within the Dopex ecosystem, ranging from protocol revenue generation and risk mitigation to price stability and yield enhancement. These components collectively contribute to the overall strength and sustainability of the Dopex platform.

## Design Analysis

Initially, I must admit that I am struggling to fully comprehend and grasp the dopex protocol's rDPX v2 upgrade. This is due to rDPX v2 architecture appears quite complicated, with many components serving different purposes. When a complicated components works together we need to make sure each component doesn't have any failure, because it will bring down the whole system because changes or problems in one can affect others.

## Recommendations

Consider to make a clearer documentation and perhaps create more youtube video content explaining how each of components in Dopex works together. While I grapple with understanding these components in my earnest effort to do so for my professional role, I can't help but wonder how other users might be experiencing the same challenges. Therefore, having a user-friendly manual and accessible documentation would greatly enhance the overall understanding of Dopex for a wider audience.

# Centralization & Systemic Risks

## Centralization Risk

Upon examination of the core contract (RdpxV2Core) and other crucial contracts (Vault, Reserve, Decaying Bonds contracts), it's clear that the current architecture of Dopex relies heavily on a single role known as DEFAULT_ADMIN_ROLE. While this admin role can potentially be a multisign or subject to a timelock with DAO governance proposals for execution, depending on a single role for such extensive capabilities is not an ideal practice.

Furthermore, in some contracts, although additional roles are defined alongside the default admin role, these roles are not assigned to any specific functions. For instance, consider the MANAGER_ROLE in the vault or the PAUSER_ROLE in RdpxReserve.

For instance, the PAUSER_ROLE in RdpxReserve, as the name implies, should have the ability to pause (and possibly unpause) RdpxReserve operations. However, it turns out that the pause() and unpause() functions are exclusively accessible to the admin role.

This situation effectively grants the admin an extraordinary level of authority. In the unfortunate event of a compromise or breach of this admin role, there may be limited recourse available to address the situation.

## Systemic Risk

Based on documentation, the bonding with 25% rDPX, 75% ETH, and A premium for a 25% out-of-the-money (OTM) rDPX/ETH perpetual put, resulting there will be a theoretical backing of 93.75% of ETH for dpxETH, but still, it's crucial to understand that absolute security at 100% cannot be guaranteed. The probability of de-pegging with less than 93.75% of ETH is exceptionally low due to the tangible assets supporting it.

However, it's important to consider extreme case scenarios, just like what happen with LUNA-TERRA. For instance, if the price of rDPX experiences an unexpected and substantial surge, questions may arise regarding the system's overall stability. This could potentially impact the ability of the system to maintain its desired peg to ETH.

if $rDPX were to plummet to zero in value, this could potentially disrupt the mechanisms designed to uphold the peg. Therefore, it's imperative to conduct a thorough evaluation and develop backup plans for such extreme scenarios to ensure the system's resilience and reliability under all possible circumstances.

# Conclusion, Notes, Additional Information and Suggestions

1. **Enhanced Option Writer Attraction:** In the past, option writers faced higher exposure to risk and found their positions less appealing, which resulted in reduced liquidity. However, rDPX v2 serves as a compensatory solution, introducing a rebate token designed to attract option writers. This is a great move by Dopex.

2. **Dynamic Bond Ratio Consideration:** Instead of adhering to a fixed 1:3 (25:75) bond ratio between rDPX and ETH, there's potential to implement a dynamic value range. This range could automatically adjust based on current market conditions and asset reserves, providing more flexibility and adaptability.

3. **Positive Feedback on RDPX V2 for the Mentally Challenged Post:** I found the [RDPX V2 FOR THE MENTALLY CHALLENGED](https://blog.dopex.io/articles/dopex-papers/rdpx-v2-for-the-mentally-challenged) post highly informative and insightful. Creating a dedicated video to explain rDPX v2 using the same concept as presented in the post would be a valuable addition.


### Time spent:
20 hours