## Audit Details

Project Name: Dupex

Client Name: Dupex

Auditor: Franklin

Languages: Solidity

Date: August 30th 2023

**Audit Goals** : The focus of the audit was to verify that the smart contract system is secure, resilient and working according to its specifications. The audit activities can be grouped in the following

**Security**: Identifying security related issues within each contract and within the system of the contracts. 
Sound Architecture:  Evaluation of the architecture of this system through the lens of established smart contract best practices and general software best practices
Code correctness and Quality:  A full review of the contract source code. The primary area of focus includes:

**Correctness,
Readability,
Sections of code with high complexity,
Quantity and quality of test coverage,**

**Documentation**
The following documentation(s) was available to wardens and provides the necessary context for this report

https://dopex.notion.site/rDPX-V2-RI-b45b5b402af54bcab758d62fb7c69cb4


**Key Oberservations**
On the quality and preparedness of Dupex

Dupex documentation/Notion white paper is thorough and well written. The codebase contains natspec comments in most of its functions. The code is written defensively with frequent assert statements to revert calls with Invalid input. 

**Test Suite and Code Coverage**

The analysis of code coverage for the protocol revealed that only 39% of functions were accounted for in the test suites. Out of total of 279 functions, only 111 functions were exercised by the test. This indicates that there is a substantial portion of the codebase that remains untested, potentially leaving critical parts of the protocol without validation through automated testing. 

**Recommendations**

Addressing this lower coverage percentage would be beneficial to ensure comprehensive testing and improved reliability of the protocol implementation

**Overview**
Dopex is a Decentralized Options Exchange that provides permissionless and non-custodial access to options trading.
rDPX V2 introduces a new synthetic coin dpxETH which is pegged to ETH. dpxETH will be used to earn boosted yields on ETH and will be a staple collateral token for future Dopex Options Products.
Issue Overview

 The following are some of the issues discovered during the audit. 

1) `The Zero address check in the constructor of PerpetualAtlanticVaultLP.sol can be by-passed.` 

The require statement validating for a zero address check in PerpetualAtlanticVaultLP.sol uses the || (OR) operator instead of && (And) operator, which means it only requires one of the check to be true for it to be passed. 

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L95

**Recommended Mitigation Steps**
Use the && operator instead. 

2) `Remove unused variable` 

In the process of code review and analysis, i identified `liquiditySlippageTolerance` in RdpxV2Core.sol was declared but never used. This can potentially lead to confusion and increase code complexity.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L103

**Recommendation**
Remove `liquiditySlippageTolerance` in RdpxV2Core if not being used. 

3) `Inconsistent use of solidity pragma version`
IStableSwap.sol uses a different pragma version 0.8.9 which was imported in RdpxV2Core.sol that used pragma 0.8.19.

**Recommended Mitigation Steps**
Consider changing this to match with the rest of the contracts pragma versions (0.8.19). Preferably lock pragma version to a particular version, in this case 0.8.19,  to ensure contracts don’t compile to a separate version and behaves consistently over time. 

 
4) `If there is an emergencyWithdraw function in UniV2liquidity, then this function should only be called when contract is paused.`

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L142

**Recommendation**
Consider making this contract pauseable and enable the emergency withdraw function only when contract is paused. 

**Conculsion** 

In conclusion, it's advisable to address the recommendations outlined above to enhance the overall quality and performance of the project. Further collaboration and testing will undoubtedly contribute to achieving the project's objectives effectively.

**Tools used**
All the issues stated above were found during Manual Review

**Time spent on analysis**
16hrs


### Time spent:
16 hours