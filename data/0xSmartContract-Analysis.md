# 🛠️ Analysis - Dopex Protocol  ![image](https://github.com/code-423n4/2023-08-dopex/assets/104318932/6c6fb6aa-df12-4c09-aa49-fc7b6709a628)


### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Centralization risks | How was the risk of centralization handled in the project, what could be alternatives? |
|e) |Systemic risks | Potential systemic risks in the project |
|f) |Competition analysis| What are similar projects? |
|g) |Security Approach of the Project | Audit approach of the Project |
|h) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|i) |Gas Optimization | Gas usage approach of the project and alternative solutions to it |
|j) |Other recommendations | What is unique? How are the existing patterns used? |
|k) |New insights and learning from this audit | Things learned from the project |

## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2023-08-dopex

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-08-dopex#running-tests)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review|The [Dopex](https://dopex.notion.site/rDPX-V2-RI-b45b5b402af54bcab758d62fb7c69cb4) explainer provides a high-level overview of the Project system and the describe the core components.|Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/code-423n4/2023-08-dopex/blob/main/slither.txt)| The project already has a slither output, this output has been reviewed |
|5|Test Suits|[Tests](https://github.com/code-423n4/2023-08-dopex/tree/main/tests)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2023-08-dopex#scope)|Top-down analysis of codes according to architectural design, IDE used: VsCode|
|7|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|
|8|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol)|Evolving Proteus's algorithm has six parameters that determine liquidity concentration|

## b) Analysis of the code base

![image](https://github.com/code-423n4/2023-08-dopex/assets/104318932/a0d95374-c8ae-4a88-b2d8-f57635fbaa31)

</br>
----------------------------------------------------------------------------------------------------------------------------------------
</br>


![image](https://github.com/code-423n4/2023-08-dopex/assets/104318932/a2919932-08d7-4196-a32d-f6e82292b144)

</br>
----------------------------------------------------------------------------------------------------------------------------------------
</br>

![image](https://github.com/code-423n4/2023-08-dopex/assets/104318932/45a12226-6f89-4a95-aef7-aac32e6063e0)

</br>
----------------------------------------------------------------------------------------------------------------------------------------
</br>

![image](https://github.com/code-423n4/2023-08-dopex/assets/104318932/3eb3a0f5-b18d-47ce-86eb-b8e99b275a11)

</br>
----------------------------------------------------------------------------------------------------------------------------------------
</br>

![image](https://github.com/code-423n4/2023-08-dopex/assets/104318932/ecdddfd0-1a35-4385-ac45-ced5dbaf01c0)

## c) Test analysis

This is an area where we think the developers have done a nice job. While the test coverage could be higher and involve more complex scenarios, we found the PoCs extremely easy and seamless to implement. The project uses a test suite written in solidity that has been written and tested with the Foundry framework.

#### 1- Test Coverage 
The audit scope of the contracts to be audited is 95% and it should be aimed to be 100%.

```js
Readme.md
- What is the overall line coverage percentage provided by your tests?: 95%
```

#### 2- Invariant Tests
There are many unit tests in the project, inveriant tests in which the interaction of contracts with each other are modeled should be increased.
For example, with the `RdpxV2Core.redeem()` function, we can summarize it as follows;

##### Invariant Test Phases
| Number |Invariant Test Phase |Details|
|:--|:----------------|:------|
|1 | Function | Specifying the function to be tested for Invariant Test |
|2 |Test List |List of attack vectors specific to the function to be Invariant Tested |
|3 |Test Suit | Writing the codes of the tests that will try the attack vectors one by one |

- Function:

```solidity
contracts\core\RdpxV2Core.sol:
  1015     **/
  1016:   function redeem(
  1017:     uint256 id,
  1018:     address to
  1019:   ) external returns (uint256 receiptTokenAmount) {
  1020:     // Validate bond ID
  1021:     _validate(bonds[id].timestamp > 0, 6);
  1022:     // Validate if bond has matured
  1023:     _validate(block.timestamp > bonds[id].maturity, 7);
  1024: 
  1025:     _validate(
  1026:       msg.sender == RdpxV2Bond(addresses.receiptTokenBonds).ownerOf(id),
  1027:       9
  1028:     );
  1029: 
  1030:     // Burn the bond token
  1031:     // Note requires an approval of the bond token to this contract
  1032:     RdpxV2Bond(addresses.receiptTokenBonds).burn(id);
  1033: 
  1034:     // transfer receipt tokens to user
  1035:     receiptTokenAmount = bonds[id].amount;
  1036:     IERC20WithBurn(addresses.rdpxV2ReceiptToken).safeTransfer(
  1037:       to,
  1038:       receiptTokenAmount
  1039:     );
  1040: 
  1041:     emit LogRedeem(to, receiptTokenAmount);
  1042:   }
```

- Test List:

Bond Integrity: Once a bond has been paid, the information (ID) of that bond should no longer be present in the contract.

Token Protection: The total amount of tokens should be checked before and after the transaction. This ensures that there is no token loss or creation in the transaction.

Access Control: Only the owner of the bond should be able to call the redeem function. If any other user tries to call this function, the operation is expected to fail.

Bond Maturity: The ability to call the redeem function should only be possible when the bond is due.

Token Transfer Check: If the token transfer was successful, the balance of the destination address should increase and the balance of the contract should decrease. The amount of token must also be equal to the amount of the bond.

Event Log: After each redeem call, check that the LogRedeem event is triggered with correct information.

Bond Token Confirmation: When the redeem function is called, verify that the bond token is burnt by the contract.

- Test Suit:

```solidity
pragma solidity ^0.8.0;

import { Test } from "forge-std/Test.sol";
import { RdpxV2Core} from "../contracts/core/RdpxV2Core.sol";

contract RedeemTests is Test {
    RdpxV2Core testRdpxV2Core;
    address public user; 
    address public testAddress;
    function beforeEach() public {
        testRdpxV2Core = new RdpxV2Core();
       
    }


    // Bond Integrity Test
    function testBondIntegrity() public {
        testRdpxV2Core.redeem(1, user);
        bool hasBond = // Check bond existence, e.g., `testRdpxV2Core.bonds(1).timestamp > 0`;
        assertFalse(hasBond);
    }

    // Token Protection Test
    function testTokenProtection() public {
        uint256 beforeBalance = IERC20WithBurn(testRdpxV2Core.addresses.rdpxV2ReceiptToken()).balanceOf(address(this));
        testRdpxV2Core.redeem(1, user);
        uint256 afterBalance = IERC20WithBurn(testRdpxV2Core.addresses.rdpxV2ReceiptToken()).balanceOf(address(this));
        assertEq(beforeBalance - afterBalance, testRdpxV2Core.bonds(1).amount);
    }

    // Access Control Test
    function testAccessControl() public {
        // Assuming anotherAddress is not the bond owner
        address anotherAddress = address(testAddress);
        expectRevertWith("NOT_BOND_OWNER", testRdpxV2Core.redeem(1, anotherAddress));
    }

    // Bond Maturity Test
    function testBondMaturity() public {
        // Assuming bond's maturity hasn't reached yet.
        expectRevertWith("BOND_NOT_MATURED", testRdpxV2Core.redeem(1, user));
    }

    // Token Transfer Check Test
    function testTokenTransferCheck() public {
        uint256 userBalanceBefore = IERC20WithBurn(testRdpxV2Core.addresses.rdpxV2ReceiptToken()).balanceOf(user);
        testRdpxV2Core.redeem(1, user);
        uint256 userBalanceAfter = IERC20WithBurn(testRdpxV2Core.addresses.rdpxV2ReceiptToken()).balanceOf(user);
        assertEq(userBalanceAfter - userBalanceBefore, testRdpxV2Core.bonds(1).amount);
    }

    // Event Log Test
    function testEventLog() public logs_gas {
        expectEvent("LogRedeem", abi.encode(user, testRdpxV2Core.bonds(1).amount));
        testRdpxV2Core.redeem(1, user);
    }

    // Bond Token Confirmation Test
    function testBondTokenConfirmation() public {
        testRdpxV2Core.redeem(1, user);
        bool bondTokenExists = RdpxV2Bond(testRdpxV2Core.addresses.receiptTokenBonds()).ownerOf(1) == address(0); 
        assertTrue(bondTokenExists);
    }

}
```

#### 3 -Test suites do not test for attack vectors, especially re-entrancy
Test teams are testing many functions and variables, but recently, due to the vulnerability in the Vyper Compiler, the hacking of the projects using certain Vyper compiler and losing 50 million $ has revealed the security weakness here.
https://cointelegraph.com/news/curve-finance-pools-exploited-over-24-reentrancy-vulnerability

Attack vectors, especially re-entrancy, seem untested and trusted

We recommend adding integration tests that show whether Re-Entrancy modifiers in the project are effective against re-entry.
In addition, removing other attack vectors and adding integration tests according to them will make the project more secure.

```solidity

contracts\perp-vault\PerpetualAtlanticVault.sol:
  255:   function purchase(
  256:     uint256 amount,
  257:     address to
  258:   )
  259:     external
  260:     nonReentrant                              // @audit nonReentrant
  261:     onlyRole(RDPXV2CORE_ROLE)
  262:     returns (uint256 premium, uint256 tokenId)
  263:   {


contracts\perp-vault\PerpetualAtlanticVault.sol:
  315:   function settle(
  316:     uint256[] memory optionIds
  317:   )
  318:     external
  319:     nonReentrant    // @audit nonReentrant
  320:     onlyRole(RDPXV2CORE_ROLE)
  321:     returns (uint256 ethAmount, uint256 rdpxAmount)
  322:   {

contracts\perp-vault\PerpetualAtlanticVault.sol:
  404     **/
  405:   function calculateFunding(
  406:     uint256[] memory strikes
  407:   ) external nonReentrant returns (uint256 fundingAmount) { // @audit nonReentrant


```

#### 4- Other Test Recomendations

- For testing to be more effective, I recommend deploying the contracts to be tested on the testnet and have them tested there, closer to the scope.

- I recommend that the tests of Oracle-related functions be done comprehensively (0 value control, price staleness control, etc.)

```solidity

contracts\amo\UniV2LiquidityAmo.sol:
  380     **/
  381:   function getLpPrice() public view returns (uint256) {
  382:     return IRdpxEthOracle(addresses.rdpxOracle).getLpPriceInEth();
  383:   }

```
- The project, while designing the tests, modeled it with the Threat Model, wrote the tests and documented it and presented it to the auditors, which increases the auditability and gives us the project team's view of the threat models specifically for the project.

- I recommend having complex tests in the test suite that include setup functions and nested operations within the call

##  d) Centralization risks 
The project uses Openzeppelin's implementation of the role-based access control library AccessControl, we will look at the centrality review through this architecture;

##### Role-Based List
| Role | Contract | Function|Who|
|:--|:----------------|:------|:------|
|MINTER_ROLE | RdpxV2Bond | mint() |msg.sender (defined in Constructor)|
|MINTER_ROLE | RdpxDecayingBonds | mint() |msg.sender (defined in Constructor)|
|MINTER_ROLE | DpxEthToken | mint() |msg.sender (defined in Constructor)|
|RDPXV2CORE_ROLE | RdpxDecayingBonds | decreaseAmount() |msg.sender (defined in Constructor)|
|RDPXV2CORE_ROLE | PerpetualAtlanticVault | purchase() |msg.sender (defined in Constructor)|
|RDPXV2CORE_ROLE | PerpetualAtlanticVault | settle() |msg.sender (defined in Constructor)|
|RDPXV2CORE_ROLE | PerpetualAtlanticVault | payFunding() |msg.sender (defined in Constructor)|
|RDPXV2CORE_ROLE | ReLPContract | reLP() |msg.sender (defined in Constructor)|
|RDPXV2CORE_ROLE | RdpxReserve | withdraw() |msg.sender (defined in Constructor)|
|PAUSER_ROLE | DpxEthToken | pause() |msg.sender (defined in Constructor)|
|PAUSER_ROLE | DpxEthToken | unpause() |msg.sender (defined in Constructor)|
|BURNER_ROLE | DpxEthToken | burn() |msg.sender (defined in Constructor)|
|BURNER_ROLE | DpxEthToken | burnFrom() |msg.sender (defined in Constructor)|
|KEEPER_ROLE | DpxEthOracle | updatePrice() |msg.sender (defined in Constructor)|
|MANAGER_ROLE | Faucet | setTimeout() |msg.sender (defined in Constructor)|
|MANAGER_ROLE | Faucet | setToken() |msg.sender (defined in Constructor)|
|MANAGER_ROLE | Faucet | toggle() |msg.sender (defined in Constructor)|
|MANAGER_ROLE | Faucet | managerMint() |msg.sender (defined in Constructor)|

Analysis of Role-Based Access Control Mechanism:
1 - Role Based Access Control Architecture has the risk of centrality, in all roles consctructor, it is defined as the address that deploys the contract

2- There is no information that the addresses of these roles are EOA or Contract addresses, and deploy was not simulated in the tests.

3- There is no information whether these addresses are MultiSign wallets that will reduce the risk of single point.

4-  Role-based role definitions with onlyOwner are handled differently in terms of security and the risk is lower in the Role-Based approach, but in this project, all roles are set as msg.sender in the constructor, that is, the deploying address is all roles, so it is no different from onlyOwner.

Role-Based Access Control Mechanism Suggestions:
1-Multi-Signature (Multi-Sig) Wallets: Instead of having a single owner with full control, you can implement multi-signature wallets. Multi-sig wallets require multiple parties to sign off on transactions or changes to the smart contract, making it more decentralized and secure.

2-Timelocks and Governance Delays: Introduce timelocks or governance delays for critical operations or updates. This means that proposed changes need to wait for a specified period before being executed. During this period, the community can review and potentially veto the changes if they identify any issues.

3-Decentralized Governance Mechanisms: Implement decentralized governance mechanisms, such as DAOs (Decentralized Autonomous Organizations) or community voting systems. This allows token holders or stakeholders to have a say in important decisions, making the project more community-driven.

Great summary of GalloDaSballo with examples of centrality risk and how to reduce it in previous Code4rena audits; https://gist.github.com/GalloDaSballo/881e7a45ac14481519fb88f34fdb8837


  
##  d) Systemic risks 

The most important system risk in the project; Risks arising from the use of Oracle, the project uses  Uniswap V3 and Uniswap V2 oracles, security exploits from oracles have been increasing recently, so using Oracle is a risk in itself and it is important to minimize this risk.

<br />



## f) Competition analysis
Frax Finance's specifically AMO ideology is forked

[Frax Finance](https://docs.frax.finance/amo/overview)
![image](https://github.com/code-423n4/2023-08-dopex/assets/104318932/d80695e4-490c-4d1a-bd0b-13f83fc85154)
Frax v2 expands on the idea of fractional-algorithmic stability by introducing the idea of the “Algorithmic Market Operations Controller” (AMO). An AMO module is an autonomous contract(s) that enacts arbitrary  monetary policy so long as it does not change the FRAX price off its peg. This means that AMO controllers can perform open market operations algorithmically (as in the name), but they cannot arbitrarily mint FRAX out of thin air and break the peg. This keeps FRAX’s base layer stability mechanism pure and untouched, which has been the core of what makes our protocol special and inspired other smaller projects. 

<br />


## g) Security Approach of the Project

#### What the project can add in the understanding of Security;

1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)

2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)

3-  Pause Mechanism
This is a chaotic situation, which can be thought of as a choice between decentralization and security. Having a pause mechanism makes sense in order not to damage user funds in case of a possible problem in the project.

4- Upgradability Mechanism
There are use cases of the Upgradable pattern in defi projects using mathematical models, but it is a design and security option.

5- Add On-Chain Monitoring System; If On-Chain Monitoring systems such as Forta are added to the project, its security will increase.

For example ;
This bot tracks any DEFI transactions in which wrapping, unwrapping, swapping, depositing, or withdrawals occur over a threshold amount. If transactions occur with unusually high token amounts, the bot sends out an alert.
https://app.forta.network/bot/0x7f9afc392329ed5a473bcf304565adf9c2588ba4bc060f7d215519005b8303e3

6- After the Code4rena audit is completed and the project is live, I recommend the audit process to continue, projects like immunefi do this.
https://immunefi.com/


<br />

## h) Other Audit Reports and Automated Findings 

**Automated Findings:**
https://github.com/code-423n4/2023-08-dopex/blob/main/bot-report.md

Especially Medium  detections in the Automated Finding Report should be taken into account;



## i) Gas Optimization
The project is generally efficient in terms of gas optimizations, many generally accepted gas optimizations have been implemented, gas optimizations with minor effects are already mentioned in automatic finding, but gas optimizations will not be a priority considering code readability and code base size


When the project is analyzed in terms of Gas Optimization, there is a very important gas optimization; "Using Mapping instead of Openzeppelin's EnumerableSet library provides high gas optimization"

```solidity
contracts\core\RdpxV2Core.sol:
  26: import { EnumerableSet } from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

  40:   using EnumerableSet for EnumerableSet.AddressSet;
  41:   using SafeERC20 for IERC20WithBurn;
  42:   using SafeERC20 for IDpxEthToken;

```

The use of EnumerableSet takes place in the following steps; 1- Define Openzeppelin library with import 2- With using EnumerableSet for EnumerableSet.AddressSet; it can be used directly without specifying the library name with its definition. 3- In EnumerableSet.AddressSet private s_priceUpdaters; a private s_priceUpdaters type variable is declared using the EnumerableSet library

In the following state variables for Gas Optimization, using mapping instead of EnumerableSet will save gas;

```solidity
contracts\core\RdpxV2Core.sol:
  40:   using EnumerableSet for EnumerableSet.AddressSet;
```

using Mapping in this project seems more efficient

Because while EnumerableSet provides enumeration capabilities, iterating over a large set consumes a significant amount of gas, especially if you need to perform operations on each item, which is exactly what happens in this project. It's more efficient to use a mapping if accessing them by key has a fixed gas cost

Gas optimization could not be measured due to the complete change of the architecture, but it is obvious that it will provide a significant gas gain in the context of use cases.


#### Other Gas Optimizations;
- The architecture pattern `_msg.sender ` is only used if there are meta-processes, but it does not appear in the project, so it consumes extra gas.

```solidity
contracts\dpxETH\DpxEthToken.sol:
  43    ) public override(ERC20Burnable, IDpxEthToken) onlyRole(BURNER_ROLE) {
  44:     _burn(_msgSender(), _amount);
  45    }

  50    ) public override onlyRole(BURNER_ROLE) {
  51:     _spendAllowance(account, _msgSender(), amount);
```





## j) Other recommendations


- The use of assembly in project codes is very low, I especially recommend using such useful and gas-optimized code patterns;
https://github.com/dragonfly-xyz/useful-solidity-patterns/tree/main/patterns/assembly-tricks-1

- It is seen that the latest versions of imported important libraries such as Openzeppelin are not used in the project codes, it should be noted.
https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts/

- A good model can be used to systematically assess the risk of the project, for example this modeling is recommended;
https://www.notion.so/Smart-Contract-Risk-Assessment-3b067bc099ce4c31a35ef28b011b92ff#7770b3b385444779bf11e677f16e101e

- It is stated in the readme document of the project that there is a timelock function, but the timelock function is not found, I suggest that it be fixed

- I recommend that the project follow a typical two-stage process in order to achieve gradual decentralization in the next stage.
Recommended 2-stage transition;
  Initially, teams delegate more privileged roles to multi-signature (at least 2 out of 3) smart contracts. The reasoning behind this is to maintain speed and control in the early stages, but when projects mature and gain the right attention, they eventually pass it on to the DAO, which should be managed by the community involved in the project.

![image](https://github.com/code-423n4/2023-08-dopex/assets/104318932/271aa8d9-2d5f-4d2e-83c8-be77d761fa97)

## k) New insights and learning from this audit 

- I learned deep knowledge about 'DpxEth-Eth' peg mechanism and synthetic Ether mechanism

- I learned about bonding mechanisms (regular bonding, delegate bonding and using decaying bonding).

- I learned in detail the advantages of Algorithmic Market Operations (AMO) mechanisms and their use in smart contracts.

### Time spent:
13 hours