## 1. Audit Approach

|     |     |     |
| --- | --- | --- |
| Step | Task | Details |
| 1   | Run Tests | Foundry tests run successfully |
| 2   | Coverage | Some minor code modification is required as explained in the README |
| 3   | Slither | Reviewed Slither results, no vulnerabilities discovered |
| 4   | Surya | Generate graphs to understand the overall project structure. Provided an initial insight to the contract inheritance and function call flow |
| 5   | Solidity Metrics | Generate metrics reports to obtain initial insight on the codebase, noting areas of potential concern |
| 6   | Code Review | Line by line code review |
| 7   | Test Review | Review of each test and it's purpose |

## 2. Mechanism Summary

Dopex offers decentralised options, and their latest update, rDPX v2, adds a synthetic asset pegged to ETH, `DpxEthToken`. The main contract intended to be used by the user is `RdpxV2Core`, which manages bonds represented by an ERC-721 token in `RdpxV2Bond`. `RdpxDecayingBonds` are used to represent bonds which expire after a period of time and are minted to users who incur losses on Dopex Options. The system uses Frax bank's [AMO](https://docs.frax.finance/amo/overview) model to control the liquidity on Uniswap using the `UniV2LiquidityAmo` and `UniV3LiquidityAmo` contracts. The ReLP process is controlled by the `ReLPContract`. An options pool used by the system is the `PerpetualAtlanticVault`, an ERC-721 compliant pool controlled by the core contract. The `PerpetualAtlanticVaultLP` is used to add liquidity to this pool and is an ERC-4626 compliant tokenized vault.

## 3. Centralisation Risks

The contracts use a role based access control system, with privileged functionality being separated into the following roles, which are initially assigned to the contract deployer:

- UniV2LiquidityAmo
    - DEFAULT_ADMIN_ROLE
- UniV3LiquidityAmo
    - DEFAULT_ADMIN_ROLE
- RdpxV2Core
    - DEFAULT_ADMIN_ROLE
- RdpxV2Bond
    - DEFAULT_ADMIN_ROLE
    - MINTER_ROLE
- RdpxDecayingBonds
    - DEFAULT_ADMIN_ROLE
    - MINTER_ROLE
- DpxEthToken
    - DEFAULT_ADMIN_ROLE
    - MINTER_ROLE
    - PAUSER_ROLE
- PerpetualAtlanticVault
    - DEFAULT_ADMIN_ROLE
    - MINTER_ROLE
- ReLPContract
    - DEFAULT_ADMIN_ROLE
    - RDPXV2CORE_ROLE

The `PerpetualAtlanticVaultLP` contract does not use roles, access control for the `PerpetualAtlanticVault` is implemented directly on the address with a modifier. Although roles are a good way to separate account privilege levels, there is a centralisation risk with them all initially being assigned to the deployer. No further context, such as deployment scripts or documentation is provided to show which accounts will have these roles post deployment.

## 4. Quality Analysis

### 4.1 Codebase

The code is well structured and organised into separate contracts, each with a clear purpose. The [automated findings](https://github.com/code-423n4/2023-08-dopex/blob/main/bot-report.md) show that there are issues needing to be addressed such as unchecked return values and missing zero address checks. There are also many small changes identified which would result in saving gas. Events could be better used, there are four parameters in `ReLPContract` which the admin can change but only one emits an event.

### 4.2 Documentation

[Basic documentation](https://dopex.notion.site/rDPX-V2-RI-b45b5b402af54bcab758d62fb7c69cb4) is provided, which clearly describes the models used in the options contracts. The mathematical side of things are explained clearly, but is lacking in relating this to the code itself. There is a basic definition provided for each contract, but no further explanations of the functions and their arguments for a user to interact with the contract directly.

### 4.3 Tests

Good tests are written which test the functions used in the core flow of the contracts, coverage is sufficient at 95%. Testing could be further improved with the use of fuzz testing, or formal verification to give users the extra assurance that their funds are safe.

## 5. Systemic Risks

No deployment code is included in the repository. As permissioned roles are initially granted to the deployer it would be useful to include scripts which transfer the roles as desired. This would reduce the risk of mistakes being made when initialising the system as any mistakes made are more likely to be discovered before deployment. Mistakes here could have unintended consequences, in the worst cases funds could be lost.

## 6. Architecture Improvements

### 6.1 Consider using OpenZeppelin contracts

A custom version of `Pausable` is implemented as it is "lighter". Consider using the original contract which is already trusted by the community.

### Time spent:
30 hours