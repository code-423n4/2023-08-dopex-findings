# Dopex: Analysis
Audit contest: `2023-08-dopex`

## Table of Contents

- [1. Executive Summary](#1-executive-summary)
- [2. Code Audit Approach](#2-code-audit-approach)
  - [2.1 Scope](#21-scope)
  - [2.2 Setup](#22-setup)
  - [2.3 Tests](#23-tests)
  - [2.4 Code Review](#24-code-review)
  - [2.5 Documentation](#25-documentation)
  - [2.6 Threat Modelling](#26-threat-modelling)
  - [2.7 Exploitation and Proofs of Concept](#27-exploitation-and-proofs-of-concept)
  - [2.8 Report Issues](#28-report-issues)
- [3. Architecture Overview](#3-architecture-overview)
  - [3.1 Centralization Risks](#31-centralization-risks)
  - [3.2 `contracts/core/RdpxV2Core.sol`](#32-contracts-corerdpxv2coresol)
  - [3.3 OpenZeppelin's AccessControl and RDPXV2CORE_ROLE](#33-openzeppelins-accesscontrol-and-rdpxv2core_role)
  - [3.4 Slither's Printer](#34-slithers-printer)
- [4. Implementation Notes](#4-implementation-notes)
  - [4.1 `setAddresses` vs `constructor` in AMO Contracts](#41-setaddresses-vs-constructor-in-amo-contracts)
  - [4.2 `contracts/decaying-bonds/RdpxDecayingBonds.sol`'s `decreaseAmount`](#42-contractsdecaying-bondrdpxdecayingbondssol-decreaseamount)
  - [4.3 Comments](#43-comments)
  - [4.4 Delegates Array](#44-delegates-array)
  - [4.5 Inconsistent Deadlines](#45-inconsistent-deadlines)
- [5. Conclusion](#5-conclusion)

## 1. Executive Summary

For this analysis, I will center my attention on the scope of the ongoing audit contest `2023-08-dopex`,
although there are certain insights that can be extended to the entire codebase. I begin by outlining the code audit approach employed for
the in-scope contracts, followed by sharing my perspective on the architecture, and concluding with observations about the implementation's code.

**Please note** that unless explicitly stated otherwise, any architectural risks mentioned in this document should not be considered vulnerabilities
or recommendations for altering the architecture solely based on this analysis. As an auditor, I acknowledge the importance of thorough evaluation
for design decisions in a complex project, taking associated risks into consideration as one single part of an overarching process. It is also important
to recognize that the project team may have already assessed these risks and determined the most suitable approach to address or coexist with them.

## 2. Code Audit Approach

Time spent: 24 hours

### 2.1 Scope
The initial step involved examining [the scope](https://github.com/code-423n4/2023-08-dopex/tree/main#scope)
to grasp the audit's boundaries and effectively prioritize my efforts within the 30 hours available.

### 2.2 Setup
I meticulously reviewed the [setup](https://github.com/code-423n4/2023-08-dopex/tree/main#setup) to ensure the
in-scope code was locally avaiable, and the provided test harness executed flawlessly. This enabled me to seamlessly
integrate my own tests as part of the audit process. It is worth emphasizing that the provision of well-structured automated
tests to auditors yields significant benefits. They enhance our comprehension of the most intricate expectations that developers
may hold regarding their software, and we can allocate less time to testing intricate use cases and potential exploits.

I encountered some challenges while setting up the test harness. While it is possible that idiosyncrasies within my own setup played
a role, I found it necessary to make the following import modifications in these test files:

`/tests/rdpxV2-core/Periphery.t.sol`
```diff
// Contracts
-import { UniV2LiquidityAMO } from "contracts/amo/UniV2LiquidityAMO.sol";
-import { UniV3LiquidityAMO } from "contracts/amo/UniV3LiquidityAMO.sol";
+import { UniV2LiquidityAMO } from "contracts/amo/UniV2LiquidityAmo.sol";
+import { UniV3LiquidityAMO } from "contracts/amo/UniV3LiquidityAmo.sol";
```

`/tests/rdpxV2-core/Setup.t.sol`
```diff
 // Peripheral contracts
 import { RdpxDecayingBonds } from "contracts/decaying-bonds/RdpxDecayingBonds.sol";
-import { ReLPContract } from "contracts/relp/ReLPContract.sol";
+import { ReLPContract } from "../../contracts/reLP/ReLPContract.sol";
```

Following these minor adjustments, the command `forge test` executed without any issues.

### 2.3 Tests

Some time was spent analysing the scope and purpose of the tests.

### 2.4 Code review

The code review commenced with an examination of `contracts/core/RdpxV2Core.sol`, chosen for its significance and intricate
role within the system's architecture. Subsequently, our focus in the code review centered on comprehending
`contracts/core/RdpxV2Core.sol` and its interrelation with other project components. Throughout this process,
I documented observations and raised questions concerning potential exploits.

### 2.5 Documentation

I favor comprehending the majority of a system through its code, allowing me to grasp the implementation more
intimately before delving into the documentation. After some hours of code review, I began reading the [full product spec](https://dopex.notion.site/rDPX-V2-RI-b45b5b402af54bcab758d62fb7c69cb4) thoughtfully furnished by the Dopex team. I employed this resource to enhance my comprehension of the code,
ensuring a closer alignment between my conceptual abstractions and expectations and those held by the development team. My primary objective was to identify
any critical disparities that might potentially lead to security vulnerabilities. Code is the ultimate authority and, as such, I prioritize it over documentation.
I did not uncover any notable distinctions between the code and documentation that could raise concerns about potential security vulnerabilities or relevant
deviations from critical assumptions.

### 2.6 Threat Modelling

I began formulating specific assumptions that, if compromised, could pose security risks to the system. This process aids me in determining the most effective exploitation strategies. While not a comprehensive threat modeling exercise, it shares numerous similarities with one.

### 2.7 Exploitation and Proofs of Concept

From now on, the majority of our time will be dedicated to a repetitive cycle conditionally encompassing each of the steps 4, 5, 6, and 7, involving attempts at exploitation and
the creation of proofs of concept. Our primary objective is to challenge critical assumptions, generate new ones in the process, and enhance this process by leveraging coded
proofs of concept to expedite the development of successful exploits.

### 2.8 Report Issues

Alghough this step may seem self-explanatory, it comes with certain nuances. Rushing to report vulnerabilities immediately and then neglecting them is unwise. The most effective
approach to bring more value to sponsors (and, hopefully, to auditors) is to document what can be gained by exploiting each vulnerability. This assessment helps in evaluating whether
these exploits can be strategically combined to create a more significant impact on the system's security. In some cases, seemingly minor and moderate issues can compound to form a
critical vulnerability when leveraged wisely. This has to be balanced with any risks that users may face. In the context of Code4rena audit contests, zero-days or highly sensitive bugs
affecting deployed contracts are given a more cautious and immediate reporting channel.

## 3. Architecture overview

### 3.1 Centralization Risks

In the context of this analysis, it is imperative to underscore that our usage of "centralization" specifically pertains to the potential single point of failure within the
project team's assets, such as the scenario where only one key needs to be compromised. We are not delving into concerns associated with the likelihood of collusion
amongst key personnel.

With exclusive control over the codebase, the potential for centralization is significant. In the implementation, there are critically trusted roles, and it's unclear how their keys
will be managed. The code cannot disallow them to be controlled by a single private key. However, in the audit contest's Discord channel, the sponsor has confirmed that:

>"the contracts that get deployed will be by the admin and all the roles and permissions will be granted to a mutisig and the admin will revoke all the unnecessary role."

Since analyzing the architecture to achieve the sponsor's stated goals is beyond the scope of this audit, I believe it wouldn't add much value to dwell extensively
on this topic. Nevertheless, the sponsor's statement underscores that the team is well-informed and ready to address this issue.

### 3.2 `contracts/core/RdpxV2Core.sol`

The architecture of the in-scope contracts has `contracts/core/RdpxV2Core.sol` as its central point. It imports interfaces to use features in most of the other in-scope contracts.
Notably, two exceptions exist: `contracts/amo/UniV2LiquidityAmo.sol` and `contracts/amo/UniV3LiquidityAmo.sol`, which import an interface and maintain an address in storage to deal with
`contracts/core/RdpxV2Core.sol` in their role as AMOs. This implies that every in-scope contract inevitably engages with `contracts/core/RdpxV2Core.sol` in some capacity,
underscoring its pivotal role not only within the design and security of the in-scope contracts but also in our comprehension of the overall architecture.

### 3.3 OpenZeppelin's AccessControl` and `RDPXV2CORE_ROLE`

The `RDPXV2CORE_ROLE` used by `contracts/decaying-bonds/RdpxDecayingBonds.sol`, `contracts/perp-vault/PerpetualAtlanticVault.sol`, `contracts/reLP/ReLPContract.sol`, and
`contracts/reserve/RdpxReserve.sol` (**out of scope**) relies on OpenZeppelin's `AccessControl.sol`. a "contract module that allows children to implement role-based access
control mechanisms". The way it is used raises a significant architectural concern: **who has the role `RDPXV2CORE_ROLE`?**

Assuming that there will be only one `RdpxV2Core` contract to interact with at any given moment, with its address expected to be static most of the time, the `AccessControl`
module does nothing to prevent that more than one address holds the role:

```c++
function _grantRole(bytes32 role, address account) internal virtual {
    if (!hasRole(role, account)) {
        _roles[role].members[account] = true;
        emit RoleGranted(role, account, _msgSender());
    }
}
```

It does add a non-negligible risk of having more than one address with the role `RDPXV2CORE_ROLE` when there may be no use case for this. Another aspect that
contributes a bit to this risk is that `AccessControl`:

```
[...] is a lightweight version that doesn't allow enumerating role
members except through off-chain means by accessing the contract event logs. Some
applications may benefit from on-chain enumerability, for those cases see
{AccessControlEnumerable}.
```

Additional, stricter yet equally lightweight approaches to address this requirement could also be considered.

### 3.4 Slither's Printer

[Slither's Printer](https://github.com/crytic/slither/wiki/Printer-documentation) serves as a valuable tool for gaining a deeper insight into the project's architecture and its overall
interactions. While it cannot replace a careful codebase analysis, it can provide a swift overview of the entire project. There is no value added by flooding this analysis with 
Slither-generated graphs, but I would like to highlight two useful commands that aided me in analysing the project's architecture. 

The first command is:

`$ slither --solc-remaps @openzeppelin=./lib/openzeppelin-contracts/contracts contracts/core/RdpxV2Core.sol --print call-graph`

Recognizing that `RdpxV2Core` holds a pivotal role within the architecture, this command generates a set of call-graphs to provide a comprehensive visualization of the interactions and potential data flows within the project. The same command can also be applied to contracts not depicted in the generated graphs; however, due to the specific structure of contract
interactions, this 'call-graph' command proved to be the most valuable for auditing the in-scope contracts.

Another useful command to use in this codebase is:

`$ slither --solc-remaps @openzeppelin=./lib/openzeppelin-contracts contracts/core/RdpxV2Core.sol --print human-summary`

Here is part of the output this command gave me for the project:

```
[... snip ...]
Compiled with solc      
Number of lines: 4926 (+ 0 in dependencies, + 0 in tests)                                                
Number of assembly lines: 0
Number of contracts: 39 (+ 0 in dependencies, + 0 tests)                                                 

Number of optimization issues: 19                                                                                                                                           
Number of informational issues: 83                                                                                                                                          
Number of low issues: 40                                                                                                                                                    
Number of medium issues: 15                                                                                                                                                 
Number of high issues: 3                                                                                                                                                    
ERCs: ERC20, ERC721, ERC165                                                                                                                                                 
                                                                                                                                                                            
+-------------------------+-------------+---------------+--------------------+--------------+--------------------+
|           Name          | # functions |      ERCS     |     ERC20 info     | Complex code |      Features      |
+-------------------------+-------------+---------------+--------------------+--------------+--------------------+
|       IERC20Permit      |      3      |               |                    |      No      |                    |
|        SafeERC20        |      9      |               |                    |      No      |      Send ETH      |
|                         |             |               |                    |              | Tokens interaction |
|         Address         |      13     |               |                    |      No      |      Send ETH      |
|                         |             |               |                    |              |    Delegatecall    |
|                         |             |               |                    |              |      Assembly      |
|         Counters        |      4      |               |                    |      No      |                    |
|         Strings         |      7      |               |                    |      No      |      Assembly      |
|           Math          |      14     |               |                    |     Yes      |      Assembly      |
|        SignedMath       |      4      |               |                    |      No      |                    |
|      EnumerableSet      |      24     |               |                    |      No      |      Assembly      |
|        RdpxV2Bond       |      90     | ERC165,ERC721 |                    |      No      |      Assembly      |
|        RdpxV2Core       |      82     |     ERC165    |                    |      No      |      Send ETH      |
|                         |             |               |                    |              | Tokens interaction |
|                         |             |               |                    |              |      Assembly      |
|    IRdpxDecayingBonds   |      4      |               |                    |      No      |                    |
|       IDpxEthToken      |      11     |     ERC20     |     ∞ Minting      |      No      |                    |
|                         |             |               | Approve Race Cond. |              |                    |
|                         |             |               |                    |              |                    |
|      IERC20WithBurn     |      10     |     ERC20     |     No Minting     |      No      |                    |
|                         |             |               | Approve Race Cond. |              |                    |
|                         |             |               |                    |              |                    |
|      IRdpxEthOracle     |      3      |               |                    |      No      |                    |
|   IRdpxV2ReceiptToken   |      1      |               |                    |      No      |                    |
|          IReLP          |      1      |               |                    |      No      |                    |                                                                                                
|       IStableSwap       |      8      |               |                    |      No      |                    |                                                                                                
|      IDpxEthOracle      |      2      |               |                    |      No      |                    |                                                                                                
| IPerpetualAtlanticVault |      13     |               |                    |      No      |                    |                                                                                                
|       IRdpxReserve      |      2      |               |                    |      No      |                    |                                                                                                
|     IUniswapV2Router    |      17     |               |                    |      No      |    Receive ETH     |                                                                                                
+-------------------------+-------------+---------------+--------------------+--------------+--------------------+ 
INFO:Slither:contracts/core/RdpxV2Core.sol analysed (39 contracts)
```

There are numerous other Slither Printer commands and outputs available, but it's easy to lose focus on what truly matters in the audited architecture by
generating various less essential outputs. There are the ones that I extracted some value for the audit.

## 4. Implementation Notes

Throughout the audit, certain implementation details stood out as noteworthy, and a portion of these could prove valuable for this analysis.

In general, I suggest placing more emphasis on Quality Assurance (QA), code quality, and standardizing the in-scope codebase. This will not only enhance
the experience for both developers and auditors but also help prevent potential future bugs.

More specifically, I will show some of the more notable perceptions without being unnecessarily exhaustive with examples that would be of little value to the project team.

As a general comment, I think a bit more attention may be given to some details on QA, code quality, and standardization of the in-scope codebase. These may be helpful for
developer's and auditor's experience alike, and avoid some future bugs.

### 4.1 `setAddresses` vs `constructor` in AMO Contracts

Unlike [the `RDPXV2CORE_ROLE` case](#openzeppelins-accesscontrol-and-rdpxv2core_role), I believe this is better suited as an implementation note.

Both `contracts/amo/UniV2LiquidityAmo.sol` and `contracts/amo/UniV3LiquidityAmo.sol` (hereafter, called "AMO contracts") store `RdpxV2Core`'s address for interactions. Nevertheless,
they do it in inherently different ways and elicit some questions about the `setAddresses` function.

Let us analyse how the address `rdpxV2Core` is stored in each of the AMO contracts. `UniV3LiquidityAmo` declares a public variable (`address public rdpxV2Core;`)
and attributes an address value to it in the `constructor`, indicating that this address should remain unchanged as there are no other functions to change it. In contrast, `UniV2LiquidityAmo` employs a `setAddresses` function to modify key addresses in the `addresses` struct, including `rdpxV2Core`. Unlike the `constructor`, this function can be
called multiple times by any address with the `DEFAULT_ADMIN_ROLE` (using OpenZeppelin's `AccessControl` module). With this context, I have a few questions to consider:

Why do AMO contracts have divergent approaches? If `rdpxV2Core` is meant to remain static, why increase risk by permitting alterations in one contract? Conversely,
if changes are expected, why restrict them in the other contract?

If other addresses in the `addresses` struct are required to change over time while `rdpxV2Core` is not, their attribution should be handled separately. Otherwise, consider
consolidating everything into the constructor function or equally restricting the `setAddress` function if setting after deployment is a requirement.

Regarding the use of the `AccessControl` module, the same limitations and questions previously discussed apply to the role `DEFAULT_ADMIN_ROLE`, so I will not delve further into this
topic again.

### 4.2 `contracts/decaying-bonds/RdpxDecayingBonds.sol`'s `decreaseAmount`

Here is an interesting case in the codebase that may serve as a good example on QA and code quality issues. This is a simple and short function commented and declared as:

```c++
  /// @notice Decreases the bond amount
  /// @dev Can only be called by the rdpxV2Core
  /// @param bondId id of the bond to decrease
  /// @param amount amount to decrease
  function decreaseAmount(
    uint256 bondId,
    uint256 amount
  ) public onlyRole(RDPXV2CORE_ROLE) {
    _whenNotPaused();
    bonds[bondId].rdpxAmount = amount;
  }
```

Though, does it truly achieve its intended purpose? Nothing is decreased, only set.
Did a `-` (minus) mistakenly disappear before the `=` (equal)? Or did requirements suddenly shift, leaving the function in this state?

According to a call to this function on `RdpxV2Core`, the latter seems more likely:

```c++
IRdpxDecayingBonds(addresses.rdpxDecayingBonds).decreaseAmount(
  _bondId,
  amount - _rdpxAmount
);
```

The code seems to correctly use `decreaseAmount` decrease `_rdpxAmount` from the bond's `amount`.
I didn't report this bug as a Medium issue because it is limited to calls from `RdpxV2Core`, and the only call from `RdpxV2Core` to this function seems to use it correctly, despite
the unusual behaviour.

### 4.3 Comments

Many parts of the code have commendable documentation via comments, which I appreciate. However, some critical sections require more commentary. For instance, in the 
`contracts/reLP/ReLPContract.sol` contract's `reLP` function, there are calculations that, while somewhat easy to follow, would benefit from additional comments to clarify
their purpose and avoid critical bugs.

```c++
uint256 baseReLpRatio = (reLPFactor *
  Math.sqrt(tokenAInfo.tokenAReserve) *
  1e2) / (Math.sqrt(1e18));

uint256 tokenAToRemove = ((((_amount * 4) * 1e18) /
  tokenAInfo.tokenAReserve) *
  tokenAInfo.tokenALpReserve *
  baseReLpRatio) / (1e18 * DEFAULT_PRECISION * 1e2);
```

These comments are crucial important because there are two phases of finding bugs in mathematical operations in a code. Usually, the easier one for most auditors is to spot a
discrepancy between the intention the comments documenting the code transmit and the code itself. The other one is questioning the mathematical foundations and correctness
of the formulas used. Commenting on the expected outcomes of calculations within the code greatly aids to address the former.

### 4.4 Delegates Array

I've observed that in the `RdpxV2Core` contract, the `delegates` array (`Delegate[] public delegates;`) is pushed but never popped. While I didn't find any
loops utilizing this array or any other more problematic operations, it's worth considering the use of a `mapping` to assess potential cost and safety benefits of
handling delegates.

### 4.5 Inconsistent Deadlines

There are some interactions that ask for a deadline to be set by the caller. Here are some examples in the in-scope contracts:

[`RdpxV2Core.sol:L1097-L1104`](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1097-L1104):
```c++
      amountOfWethOut = IUniswapV2Router(addresses.dopexAMMRouter)
        .swapExactTokensForTokens(
          _rdpxAmount,
          minamountOfWeth,
          path,
          address(this),
          block.timestamp + 10
        )[path.length - 1];
```


[`UniV3LiquidityAmo.sol:L243-L251`](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L243-L251):
```c++
    INonfungiblePositionManager.DecreaseLiquidityParams
      memory decreaseLiquidityParams = INonfungiblePositionManager
        .DecreaseLiquidityParams(
          pos.token_id,
          liquidity,
          minAmount0,
          minAmount1,
          block.timestamp
        );
```

[`UniV3LiquidityAmo.sol:L289-L299`](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L289-L299):
```c++
    ISwapRouter.ExactInputSingleParams memory swap_params = ISwapRouter
      .ExactInputSingleParams(
        _tokenA,
        _tokenB,
        _fee_tier,
        address(this),
        2105300114, // Expiration: a long time from now
        _amountAtoB,
        _amountOutMinimum,
        _sqrtPriceLimitX96
      );
```

[`UniV2LiquidityAmo.sol:L222-L232`](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L222-L232):
```c++
    (tokenAUsed, tokenBUsed, lpReceived) = IUniswapV2Router(addresses.ammRouter)
      .addLiquidity(
        addresses.tokenA,
        addresses.tokenB,
        tokenAAmount,
        tokenBAmount,
        tokenAAmountMin,
        tokenBAmountMin,
        address(this),
        block.timestamp + 1
      );
```

Within `UniV2LiquidityAmo.sol` there are other instances where `block.timestamp + 1` is used.

Depending on the caller's requirements, utilizing `block.timestamp` as a deadline is a valid approach. However, it's crucial to acknowledge that employing
`block.timestamp + {some_number}` yields an equivalent outcome in this context. The use of `block.timestamp + {some_number}` might suggest an unintended utilization of this feature.
Nevertheless, it should be noted that some developers opt for `block.timestamp + {some_number}` for minor stylistic or readability reasons and possess the necessary understanding of
the feature. Naturally, this might be the case in the audited code. However, the instance mentioned, where `2105300114, // Expiration: a long time from now` is used as a deadline,
along with the provided comment, suggests that the feature may not be fully comprehended by at least one of the developers involved.

Continuing our examination, here is a more dangerous construct discovered on
[ReLPContract.sol:L286-L295](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L286-L295):

```c++
    (, , uint256 lp) = IUniswapV2Router(addresses.ammRouter).addLiquidity(
      addresses.tokenA,
      addresses.tokenB,
      tokenAAmountOut,
      amountB / 2,
      0, // @audit amountAMin
      0, // @audit amountBMin
      address(this),
      block.timestamp + 10
    );
```

Using `block.timestamp` with `amountXMin` offers absolutely no slippage protection. An analysis of this, along with its potential consequences, is warranted.

In conclusion, the most fitting recommendation for the purpose of this analysis report is to ensure developers' comprehension of these features and, ideally,
establish a standardized pattern across the codebase. Alongside other secure development practices, standardization will prevent errors and facilitate
the identification and auditing of exceptions, including those that may be necessary.


## 5. Conclusion

I hope that I have been able to offer a valuable overview of the methodology utilized during the audit of the contracts within scope,
along with pertinent insights for the project team.


### Time spent:
24 hours