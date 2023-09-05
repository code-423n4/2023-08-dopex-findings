# ![Dopex](https://dopex.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F39e1706c-1d8d-4771-9c02-e8affcfed0ed%2Flogo.png?table=block&id=b45b5b40-2af5-4bca-b758-d62fb7c69cb4&spaceId=7ec03ae2-8e00-4620-b583-264f9ddf1ae0&width=40&userId=&cache=v2) Dopex Protocol Smart Contracts Analysis

## Description overview of Dopex Protocol

The `Dopex protocol` has introduced a new version called `rDPX V2`, which brings an interesting way to reward those who `write options` on their platform using a rebate system. Additionally, they've created a new synthetic coin called `dpxETH`, which is tied to the price of `ETH`. The idea behind this coin is to use it for earning higher yields with `ETH` and as a collateral asset for future options products on `Dopex`.

The bonding process in `rDPX V2` is how new `dpxETH` coins can be created. When someone bonds, they receive a `"receipt"` that represents a combination of `ETH` and `dpxETH` in a system called `Curve`. Through this bonding process, new `dpxETH` coins are minted, and to ensure they always have backing, there are `reserves` of `rDPX` and `ETH` called `"Backing Reserves."` These reserves are controlled by entities called `"AMOs."`

To achieve a safe and controlled scaling of `rDPX V2` and `dpxETH`, the protocol has adopted an idea called `"AMO ideology"` from `Frax Finance`, which is another similar project. Essentially, they are drawing inspiration from how `Frax Finance` manages its system to ensure that the new version of `Dopex` can grow and operate in a stable and sustainable manner.

![Dopex1](https://github.com/catellaTech/dopex/blob/main/Dopex1.drawio.png?raw=true)

## Summary

![Dopex2](https://github.com/catellaTech/dopex/blob/main/Dopex2.drawio.png?raw=true)

## 1- Approach we followed when reviewing the code

To begin, we assessed the code's scope, which guided our approach to reviewing and analyzing the project.

### **Scope**

- 1. **UniV2LiquidityAMO.sol**:

  - This contract is utilized to manage liquidity in the `Uniswap V2` token pair, as well as to execute automated exchange operations. The contract encompasses administrative functions, AMO (Automated Market Maker Operator) functions, and view functions to oversee a variety of liquidity-related operations and token exchanges.

- 2. **UniV3LiquidityAmo.sol**:

  - This contract is designed to interact with the `Uniswap V3` protocol and perform liquidity management operations.

- 3. **RdpxV2Core.sol**:

  - This contract is the core of the protocol and is responsible for managing crucial operations such as interacting with `delegates`, `asset management`, `option settlement`, `balance synchronization`, and `repeg handling`. Internal `_validate` functions are used to check conditions throughout the contract and generate exceptions if a condition is not met. This contract ensures that operations are carried out in an authorized manner and in accordance with predefined protocol rules.

- 4. **RdpxV2Bond.sol**:

  - this contract defines an `ERC721-based` NFT token with the ability to `mint` new tokens, `pause/unpause` token transfers, and enforce checks during token transfers. It leverages functionality from various `OpenZeppelin` contracts to implement these features.

- 5. **RdpxDecayingBonds.sol**:
  - `RdpxDecayingBonds` contract provides a way to `mint` and manage decaying `rDPX bonds` as non-fungible tokens. Users with specific roles can create bonds, perform emergency withdrawals, and query the bonds they own. The contract is designed to be paused in critical situations and features a structure that ensures security and control over operations at all times.
- 6. **DpxEthToken.sol**:

  - This contract defines a synthetic token with `ERC-20` functionalities and adds features for `pausing`, `minting`, and `burning` tokens. The contract leverages `role-based` access control to manage the interactions and operations on the token.

- 7. **PerpetualAtlanticVault.sol**:

  - This contract represent a vault for perpetual atlantic `rDPX` PUT options. It is designed to handle option issuance, settlement, and funding calculation. The contract utilizes various roles and `OpenZeppelin` libraries to ensure its functionality and security.

- 8. **PerpetualAtlanticVaultLP.sol**:

  - this contract handles liquidity provision for the `PerpetualAtlanticVault` contract, allowing users to deposit and withdraw assets, while also managing aspects related to the `PerpetualAtlanticVault` contract and calculating conversions and previews for operations. And use the standard `ERC4626`.

- 9. **ReLPContract.sol**:
  - this contract manages the process of `re-liquidity` provisioning for a DEX pair using the `Uniswap V2` protocol. It calculates the optimal amount of Token A `reLP` to remove from the pool, performs swaps, and adds the new liquidity back to the pool. The contract is designed to ensure efficient and accurate `re-liquidity` provision while syncing balances with other relevant contracts.

### Privileged Roles 
There are several important roles in the system.

- **DEFAULT_ADMIN_ROLE**: This role is essential in every contract. It grants administrative permissions and control over critical functions. The holder of this role can manage other roles, configure contract parameters, and perform various administrative tasks.
```Solidity
 constructor() {
    _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
  }

```
- **MANAGER_ROLE**: This role is present in the [PerpetualAtlanticVault](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol) contract. It is used for general contract management, including setting addresses, adjusting parameters, and controlling administrative functions. The manager role helps ensure proper operation and configuration.
```Solidity
 bytes32 public constant MANAGER_ROLE = keccak256("MANAGER_ROLE");
```
- **MINTER_ROLE** - **PAUSER_ROLE** - **BURNER_ROLE** :Found in contracts like [DpxEthToken](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol), [RdpxDecayingBonds](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol) and [RdpxV2Bond](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol) this role permits the creation or minting of new tokens or bonds. It's important for controlling the issuance of new tokens, maintaining the token supply, and potentially providing incentives.
```Solidity
  bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
  bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
```

Initially, the `accounts` responsible for fully controlling these `roles` will be managed by the `Dopex` team as confirmed by the sponsor 
```shell
Psytama — 31/08/2023 at 7:24. 
Yes, they will be controlled by the protocol through a multisig. 
```
Given the level of control that these `roles` possess within the system, users must place full trust in the fact that the entities overseeing these `roles` will always act correctly and in the best interest of the `system` and its `users`.

After grasping the code's functionality, we carefully analyzed and audited it step by step. Our goal was to ensure its security and proper operation.

## 2- Analysis of the code base

The main most important contracts of `rDPX V2`.

- Here's a detailed diagrams of the essential components of these contracts:

### 1. RdpxV2Core:

![Dopex3Core](https://github.com/catellaTech/dopex/blob/main/Dopex3Core.drawio.png?raw=true)

### 2. PerpetualAtlanticVault:

![Dopex4](https://github.com/catellaTech/dopex/blob/main/Dopex4.drawio.png?raw=true)

### 3. PerpetualAtlanticVaultLP:

![Dopex5LP](https://github.com/catellaTech/dopex/blob/main/Dopex5LP.drawio.png?raw=true)

### 4. UniV3LiquidityAmo:

![Dopex7UniV3](https://github.com/catellaTech/dopex/blob/main/Dopex7UniV3.drawio.png?raw=true)

### 5. RdpxV2Bond:

![Dopex8Bond](https://github.com/catellaTech/dopex/blob/main/Dopex8Bond.drawio.png?raw=true)

### 6. RdpxDecayingBonds:

![Dopex9Decay](https://github.com/catellaTech/dopex/blob/main/Dopex9Decay.drawio.png?raw=true)

### 7. ReLPContract:

![Dopex10reLP](https://github.com/catellaTech/dopex/blob/main/Dopex10reLP.drawio.png?raw=true)

### What's unique? What's using existing patterns?

- **Unique Features**:
  - *Decaying Bonds*: Issuance of `ERC721` decaying bonds offering rebates and indirect `rDPX` emission.

- **Utilization of Existing Patterns**:
  - *Token Standards*: `ERC721` and `ERC20` token standards for bonds, `LP tokens`, and assets.
  - *Liquidity Pools and AMOs*: Utilizing `Uniswap V2` and `V3 functions` for liquidity and `AMO` operations.
  - *Staking and Rewards*: Implementing staking and proportional rewards mechanisms.
  - *Oracle Integration*: Using oracles for accurate price information.
  - *Access Control*: `Role-based` access control for secure interactions.
  - *Strategy Contracts*: Automating liquidity provision with strategy contracts.

## 3- Architecture of the rDPX V2

This diagram illustrates how the different contracts and modules interact within the `rDPX V2` ecosystem.

![Dopex6Arqui](https://github.com/catellaTech/dopex/blob/main/Dopex6Arqui.drawio.png?raw=true)

- **How a user interacts** 🧑‍💻:
  Users interact with the `rDPX V2` through their actions, such as:

  - **`Bonding and Interaction with Core Modules`**:

    - A user can initiate bonding processes through the Bonding Module. They can acquire `rDPX` Bond tokens, which represent their bond balance.
    - The user can interact with the Peg Stability module indirectly by participating in bonding processes. The module aims to maintain the peg of `dpxETH`.
    - Users can engage with the Backing Reserves module by providing reserves (such as `rDPX` or `ETH`) to be used by Automated Market Operators (AMOs)..

  - **`Liquidity Provision and Yield Generation`**:

    - Users can provide liquidity to the `AtlanticPerpetualPoolLP`, contributing to the liquidity of the `AtlanticPerpetualPool`.
    - By adding liquidity, users earn rewards which they can stake in the `AtlanticPerpetualPoolStaking` contract for further yield.

  - **`Option Trading`**:

    - Users can mint option tokens (NFTs) by interacting with the `AtlanticPerpetualPool` contract.
    - These option tokens can be traded or used for various purposes within the ecosystem.

  - **`Decaying Bonds and Loss Mitigation`**:

    - If a user experiences losses from Dopex Option Products, they might receive decaying bonds from the rDPX Decaying Bonds contract. These bonds act as a rebate for their losses.

  - **`Staking and Yield Farming`**:

    - Users can stake `RdpxV2ReceiptToken` in the `RdpxV2ReceiptTokenStaking` contract to earn boosted yields from the `Curve LP` and `DPX incentives`.

  - **`Utilizing Oracles`**:

    - Users can access real-time price information from the `dpxETH/ETH` Oracle and `rDPX/ETH` Oracle to make informed decisions.

  - **`AMO Interaction`**:

    - If users wish to use `Uniswap V2` or `V3 functions`, they can do so through the respective `AMO` contracts. This involves actions like `adding` or `removing` liquidity and `making swaps`.

  - **`Redeeming Rewards and Tokens`**:
    - Users can redeem rewards earned from staking, providing liquidity, or participating in other activities within the ecosystem.

## 4- Documents

- The documentation of the `Dopex rDPX V2` project is quite comprehensive and detailed, providing a solid overview of how `rDPX V2` is structured and how its various aspects function. However, we have noticed that there is room for additional details, such as `diagrams`, to gain a deeper understanding of how different contracts interact and the functions they implement. With considerable enthusiasm, we have dedicated several days to creating diagrams for each of the contracts. 
We are confident that these diagrams will bring significant value to the protocol as they can be seamlessly integrated into the existing `documentation`, enriching it and providing a more comprehensive and detailed understanding for `users`, `developers` and `auditors`.

- We suggest including high-quality articles on `Medium`, as it's an excellent way to provide an in-depth understanding of many project topics, and it's a common practice in many blockchain projects.

## 5- Systemic & Centralization Risks

Here's an analysis of potential systemic and centralization risks in the provided contracts:

### UniV2LiquidityAmo:

- **Systemic Risks**:

  - Calculation errors in exchange and liquidity operations.
  - Authorization mistakes and manipulation vulnerabilities.
  - Errors in swaps and reliance on price oracles.

- **Centralization Risks**:
  - Dependency on external contracts and administrator control.

### UniV3LiquidityAmo:

- **Systemic Risks**:

  - Calculation errors and oracle dependency.
  - Unintended position changes due to mistakes.

- **Centralization Risks**:
  - Administrator control and reliance on external contracts.

### RdpxV2Core:

- **Systemic Risks**:

  - Centralized control and oracle dependence (The oracle is out the scope in this contests, but we recommend to consider auditing and make code reviews in those contracts).
  - Delegation mechanism and `AMO` dependency.

- **Centralization Risks**:
  - Administrator control over `pause` and `withdrawal`.
  - Minting control and reliance on external contracts.

### RdpxV2Bond:

**Centralization Risks**:

- Minting control and `bond` amount.
- `Ownership` and contract governance.

### RdpxDecayingBonds:

- **Systemic Risks**:
  - Vulnerability in `emergencyWithdraw()` function could lead to fund loss during emergencies.
  - Improper implementation or malicious exploitation of `pausing` could disrupt contract functionality.
  - Interaction with external contracts introduces security and behavior risks.
  - Lack of mechanisms to limit bond supply may cause unintended inflation.

**Centralization Risks**:

- Admin's control over pausing and withdrawal might centralize power.
- Concentrated `MINTER_ROLE` control could lead to centralized `bond` issuance.
- Centralized `RDPXV2CORE_ROLE` control might impact `bond` adjustments.
- Absence of governance could centralize `upgrade` decisions.
- Initial `deployer's role ownership` could centralize contract control.

### DpxEthToken:

- **Systemic Risks**:

  - Lack of burn limits and reliance on external contracts.

- **Centralization Risks**:
  - Administrator control and centralized role assignment.

### PerpetualAtlanticVault:

- **Systemic Risks**:

  - Funding calculation errors and stale data.
  - External contract dependencies and reentrancy.

- **Centralization Risks**:
  - `Manager role` control and `administrator` authority.
  - `Whitelist` control and contract upgrades.

### PerpetualAtlanticVaultLP:

- **Systemic and Centralization Risks**:
  - Centralized control due to dependency on `perpetualAtlanticVault` contract.
  - Vulnerabilities in external contracts and `liquidity` risk.

### ReLPContract:

- **Systemic Risks**:

  - `Slippage` tolerance risk and `role` centralization.

- **Centralization Risks**:
  - Administrator control and dependence on external contracts.

These summaries outline the major risks that each contract is exposed to in relation to potential `system-wide` problems and `concentration of control`.

### **Dependency risk**:
   - We observed that old and unsafe versions of Openzeppelin are used in the project, this should be updated to the latest one:

  ```solidity
      package.json#L4
      4: "version": "4.9.0",
  ```
  - Latest is `4.9.3` (as of July 28, 2023), version used in project is `4.9.0`
  - You can read more of this issue in our `QA`.

## 6- New insights and learning from this audit

Our new insights from this `Dopex rDPX V2` protocol audit have been exponential. We have learned about the intricate `bonding mechanisms`, including `regular`, `delegate`, and `decaying bonds`, which cater to various scenarios but require user understanding. Inspired by `Frax Finance`, Algorithmic Market Operations `AMOs` are integrated to mitigate risks. The addition of `perpetual options` and increased `yield` issuance for `receipt token` holders aims to incentivize participation. Governance-controlled parameters ensure adaptability, but effective governance is essential. The `fee` structure of the `redemption mechanism` impacts liquidity. The implementation strategy emphasizes initial liquidity and participation.

## 7- Test analysis

The audit scope of the contracts to be reviewed is 95%, with the aim of reaching 100%. However, the tests provided by the protocol are at a high level of comprehension, giving us the opportunity to easily conduct Proof of Concepts `(PoCs)` to test certain bugs present in the project.

### How Could They Have Improved?

While the provided codes seems to be functional, there are always areas for improvement to ensure better security, efficiency, and readability in both the contracts and the protocol.

- Here are some potential areas for improvement:

  1. **Modularization and Separation of Concerns**:

     - The contract `RdpxV2Core` code is lengthy and contains many functions in a single unit. Consider separating different responsibilities into smaller, specific modules.
       For example, you can create modules for:
       - Bond handling
       - Delegate interaction
       - Price calculations etc.

     This will improve code readability and maintainability.

  2. **Comments and Detailed Documentation**:

     - While the code has comments, certain sections may necessitate more comprehensive explanations, such as the `Execute` function in the `UniV3LiquidityAmo.sol` contract.

     It is recommended to document it as per the instructions provided by the sponsor in the Discord DM:

     ```Shell
     psytama — 22/08/2023 21:18
         - "We just added a generic proxy function in case an unforeseeable situation arises; we would utilize it"
     ```

     Enhance the clarity and comprehensibility of the code by incorporating explicit and elucidative comments for every function and segment. This practice will not only enhance understanding but also facilitate seamless maintenance.

  3. **Structured Comments**:
     - Use structured comments, such as standardized function headers (NATS), to describe the function, its parameters, returns, and behavior. This will make the code more understandable for other developers.

## Conclusion

In general, the `Dopex rDPX V2 project` exhibits an interesting and well-developed architecture we believe the team has done a good job regarding the code, but the identified risks need to be addressed, and measures should be implemented to protect the protocol from potential malicious use cases. It is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.

## Time Spent ⏱

A total of `8 days` were dedicated to completing this audit, distributed as follows:

1. 1st Day: Devoted to comprehending the protocol's functionalities and implementation.
2. 2nd Day: Initiated the analysis process, leveraging the documentation offered by the `Dopex Protocol`.
3. 3nd Day & 4rd Day: We dedicated ourselves to taking notes while reading through the contracts line by line.
4. 5th Day: From the notes taken, we decided what goes into the QA report and started it.
5. 6th Day: After finishing the QA, we started with some PoCs that we wanted to do to confirm our doubts about certain functions.
6. 7th Day: We focused on writing the reports for the medium findings we discovered.
7. 8rd Day: Focused on conducting a thorough analysis, incorporating meticulously crafted diagrams derived from the contracts and the note taken by us.




### Time spent:
72 hours