# Analysis -Dopex  
# Summary

| List |Head |Details|
|:--|:----------------|:------|
|1 | Overview of Dopex  | overview of the key components and features of Dopex |
|2 |Audit approach | Process and steps i followed  |
|3 |Learnings | Learnings from this protocol|
|4 |Possible Systemic Risks | The possible systemic risks based on my analysis |
|5 |Code Commentary | Suggestions for existing code base |
|6 | Centralization risks | Concerns associated with centralized systems |
|7 |Gas Optimizations | Details about my gas optimizations findings and gas savings  |
|8 |Risks as per Analysis | Possible risks |
|9 |Non-functional aspects | General suggestions |
|10 |Time spent on analysis  | The Over all time spend for this reports |

## Overview
> A rebate system for option writers in the Dopex Protocol

``rDPX V2`` introduces a new ``synthetic coin`` ``dpxETH`` which is pegged to ``ETH``.`` dpxETH`` will be used to earn boosted yields on ``ETH`` and will be a staple collateral token for future ``Dopex Options Products``

The ``rDPX`` bonding process represents the method in which new ``dpxETH`` tokens can be minted. When a user bonds with the ``rDPX V2`` contract they receive a receipt token. A receipt token represents ``ETH`` and ``dpxETH LP`` on curve

### Bonding Mechanism

#### Types

 - ``Regular bonding``
    To mint a ``bond carrying`` approx. 1 Receipt Token you will deposit ``25 rDPX (25%) ``& ``0.75 ETH (75%)`` + the premium for the size of the ``rDPX`` provided for an ``rDPX Atlantic Perpetual`` PUT option which is ``25%`` out of the money. You receive a discount on the ``rDPX & ETH`` provided
 - ``Delegate bonding``
    Delegate bonding is a system wherein users come in with ``ETH`` deposit it in a pool (via the Core Contract) and set a fee for the usage of their ``ETH``. Now users holding ``rDPX`` can use this pool of ``ETH`` to bond (using the regular bonding mechanism). This way users only with ``rDPX`` and users only with ETH both can come together and bond to receive their share of receipt tokens
 - ``Bonding via rDPX Decaying Bond``
    Each decaying bond carries a particular amount of ``rDPX`` in it. This amount is decided when these decaying bonds are minted to users. ``rDPX`` Decaying bonds can be minted to users as rebates for losses incurred on the ``Dopex Options Protocol`` or for incentives
    
### Atlantic Perpetual PUTS Options
A perpetual options pools for ``rDPX`` options the Core Contract uses for the bonding process. The ``liquidity`` for this will be provided via the open market and will be incentivized via ``DPX``. The Core Contract pays this pool LPs funding every week

### Receipt Token

The ``rDPX V2`` bonds hold receipt tokens in them, a receipt token represent ``dpxETH/ETH`` LP tokens on curve. These LP tokens are staked in curve gauges with boosted rewards. Boosted rewards come from pointing the ``CVX`` held by the ``Dopex`` Team and Partners to the curve gauge

### AMO

Diving deeper into the ``Frax ecosystem``, All Frax AMOs hold 4 key properties: ``Recollateralize``, ``Decollateralize``, ``Market Operations`` and ``FXS1559``

 - Decollateralize
 - Market operations
 - Recollateralize
 - FXS1559

#### AMO Versions

  - Uniswap V2 AMO
  - Uniswap V3 AMO

### Peg Stability Module (PSM)

The ``peg stability`` module is part of the Core Contract that ensures that the peg of ``dpxETH`` to ``ETH`` on the ``curve pool`` is maintained

Core Contract managers and It encompasses 3 functions:

 - upperDepeg (if dpxETH > 1 ETH)
 - lowerDepeg (if dpxETH < 1 ETH)
 - settle

 
## Audit approach

I followed below steps while analyzing and auditing the code base.

1. Read the contest ``Readme.md`` and took the required notes.

  - Dopex Protocol
    - Inheritance
    - Test Coverage - 95%
    - Timelock function, NFT, AMM, ERC-20 Token
    - Uses the ``custom oracle``
    - Alternate implementation of ``Uniswap``
    - Area need to addressed in ``DpxEth-Eth peg``
      
2. Analyzed the over all codebase one iterations very fast

3. Study of documentation to understand each contract purposes, its functionality, how it is connected with other contracts, etc.

4. Then i read old audits and already known findings. Then go through the bot races findings 

5. Then setup my testing environment things. Run the tests to checks all test passed. I used ``foundryup`` to test Dopex Protocol . 

#### Commands Used for testing: 

  - forge build
  - forge test

6. Finally, I started with the auditing the code base in ``depth`` way I started understanding line by line code and took the ``necessary notes`` to ask some questions to ``sponsors``.

## Stages of audit

- ``The first stage of the audit``

During the ``initial stage`` of the audit for ``Dopex Protocol``, the primary focus was on analyzing ``GAS`` usage and ``Quality assurance (QA)`` aspects. This phase of the audit aimed to ensure the efficiency of gas consumption and verify the robustness of the platform.

#### Gas Optimizations

Found ``12 impactful`` gas optimizations and saved around ``50000 GAS``

#### QA Analysis

Also Found ``16 LOW RISK FINDINGS``
 

- ``The second stage of the audit``

In the ``second stage`` of the audit for ``Dopex Protocol``, the focus shifted towards understanding the ``protocol`` usage in more detail. This involved identifying and analyzing the ``important contracts`` and functions within the system. By examining these key components, the audit aimed to gain a comprehensive understanding of the protocol's ``functionality`` and ``potential risks``. 

- ``The third stage of the audit``

During the ``third stage`` of the audit for ``Dopex Protocol``, the focus was on thoroughly examining and marking any doubtful or vulnerable areas within the protocol. This stage involved conducting comprehensive ``vulnerability assessments ``and identifying ``potential weaknesses`` in the system. Found ``60-70`` ``vulnerable`` and ``weakness`` code parts all marked with ``@audit tags``.

- ``The fourth stage of the audit``

During the ``fourth stage`` of the audit for ``Dopex Protocol``, a ``comprehensive analysis`` and ``testing`` of the previously identified ``doubtful`` and ``vulnerable`` areas were conducted. This stage involved diving deeper into these areas, performing ``in-depth examinations``, and subjecting them to ``rigorous testing``, including ``fuzzing`` with various inputs. Finally concluded findings after all ``research's`` and ``tests``. Then i reported ``C4`` with ``proper formats`` 


## Learnings

The ``rDPX V2`` system introduces a range of technical enhancements to the existing framework. It revolves around the ``creation`` and ``management`` of the ``synthetic coin dpxETH``, ``pegged to ETH``. The core infrastructure includes the ``rDPX Backing Reserve``, ``ETH Backing Reserve``, and ``Treasury Reserve``, each serving ``distinct collateral`` purposes. The Core Contract ``orchestrates`` the ``Peg Stability Module (PSM)``, ``bonding mechanisms``, and ``Algorithmic Market Operations Controllers`` (AMOs).

The bonding process is ``multifaceted``, encompassing ``regular bonding``, ``delegate bonding``, and ``decaying bonds``, each involving ``rDPX`` and ``ETH deposits``. ``Atlantic Perpetual`` PUT Options ``drive bonding``, ``incentivizing liquidity`` supply and rewarding ``option holders``. Receipt tokens represent ``dpxETH/ETH`` LP tokens on Curve, staked for boosted yields and ``DPX rewards``. These tokens can be redeemed based on the backing ratio.

``Integral`` to stability are system parameters ``governed`` by the ``community``, setting ``veDPX`` ``Bonding Fee``, ``Reward Distribution Percentages``, ``Discount Factor``, ``Bond Maturity``, and ``Redemption Fee``. A meticulous launch strategy involves ``Treasury`` and ``Partner participation`` before the ``public announcement``.

In a ``nutshell``, ``rDPX V2`` leverages these intricate elements to establish a ``refined ecosystem``, ``generating dpxETH``, supporting Dopex Options Products, and optimizing overall ``system stability`` and ``efficiency``


## Possible Systemic Risks

### Imbalanced Reserves :
  The ``dual`` token reserves (rDPX Backing Reserve and ETH Backing Reserve) are critical to supporting ``dpxETH``. If the reserves become imbalanced or ``inadequately managed``, it could ``threaten`` the ``stability`` of the ``peg`` and ``overall system integrity``.

### Overreliance on AMOs :
The ``Algorithmic Market Operations Controllers`` (AMOs) play a central ``role`` in managing the ``reserves`` and ``market operations``. Depending too heavily on ``AMOs``, without proper ``monitoring`` and ``adjustments``, could ``expose`` the system to ``vulnerabilities`` in ``volatile market conditions``.

### Bonding Mechanism Complexity :
The ``bonding process``, with its ``regular bonding``, ``delegate bonding``, and ``decaying bonds``, ``introduces complexity``. If not well-understood by users or not functioning as intended, it could lead to ``errors``, ``unexpected consequences``, and a ``loss of user trust``

### Market Price Fluctuations :
Since the system pegs ``dpxETH`` to ``ETH``, significant fluctuations in ETH price could potentially ``disrupt`` the peg and create challenges for ``maintaining`` system ``stability``.

### Unpredictable Option Pool Dynamics :
The ``perpetual options pool's`` rewards and dynamics depend on market conditions and user behavior. If these dynamics become ``unpredictable`` or ``unmanageable``, it could impact the effectiveness of the ``system's incentivization mechanism``.

### Oracle Inaccuracies :
The reliance on ``oracles`` to determine the value of assets ``(e.g., ETH, rDPX)`` could expose the system to risks if these oracles provide ``inaccurate`` or ``manipulated data``.



## Centralization risks

A single point of failure is not acceptable for this project Centrality risk is high in the project, the role of ``DEFAULT_ADMIN_ROLE``detailed below has very critical and important powers.

Project and funds may be compromised by a malicious or stolen private key ``DEFAULT_ADMIN_ROLE`` ``msg.sender``

By designating a single ``admin role`` for all functions, you create a single point of control in your smart contract. This can be a security risk because if the ``admin's private key`` is ``compromised`` or if the admin acts ``maliciously``, they could effectively take over the entire contract.

``Overusing`` the ``DEFAULT_ADMIN_ROLE`` can undermine the ``security model`` of your smart contract. If all functions rely on the ``same role``, it becomes harder to limit access to specific functions or data, increasing the risk of ``unauthorized actions``.

In a decentralized application (dApp) or protocol, ``the goal`` is often to minimize ``centralization`` and control. Relying too heavily on a ``single admin`` role goes against this principle and could lead to accusations of ``abuse of power`` or ``centralization concerns``.

#### Mitigation:
Implementing a ``role-based access control`` system that defines different roles with specific permissions can provide a more flexible and secure approach to managing access control in your contract. This allows you to grant the necessary privileges to different entities or users ``without relying`` solely on the ``DEFAULT_ADMIN_ROLE``.


## Gas Optimizations

### Report Based on ``GAS OPTIMIZATION``

Several gas optimization opportunities have been identified. First, by packing the ``owner`` and ``expiry fields`` within the same ``storage slot`` in the ``Bond`` struct. Additionally, ``caching`` the results of expensive ``external`` function calls, such as ``getEthPrice()``, ``getDpxEthPrice()``, and ``getRdpxPrice()``, can save some high value . To optimize gas costs for ``read-only external`` function arguments, using ``calldata`` instead of ``memory`` is recommended, resulting in ``substantial gas`` savings in ``multiple instances``. Moreover, replacing ``memory`` with ``storage`` for structs and arrays can reduce gas costs by minimizing ``expensive storage reads``. Utilizing smaller data types for state variables, such as ``uint96`` and ``uint128`` instead of ``uint256``, can save gas and storage slots. Lastly, avoiding unnecessary ``initialization`` of state variables with default values can lead to gas savings.

Overall, these ``optimizations`` enhance ``gas efficiency`` and reduce costs in ``smart contract execution``.

#### General Gas Suggestions

- When calling ``safeApprove()`` and ``safeTransfer()`` functions, consider checking the current allowance or balance before making the approval or transfer to minimize unnecessary operations.
- ``_sendTokensToRdpxV2Core()`` optimize this function by consolidating multiple token transfers into a single transaction, reducing the number of external calls and saving gas
- When ``adding`` or ``removing`` assets from the ``reserveAsset`` array, consider the gas cost implications. These operations involve ``updating arrays`` and ``mappings``. If the array is expected to be very long, these ``operations`` could become ``costly``. You might want to evaluate ``alternative approaches`` for ``managing assets``, such as using a different ``data structure`` or ``limiting the maximum number`` of assets.
- The ``delegates`` array is used to store delegates. If this array is expected to grow significantly, consider implementing ``gas-efficient`` ways to manage and access this data. For example, you could use a ``mapping`` to associate addresses with ``delegate information``, reducing the need to ``iterate`` through the array.
- Array operations, such as ``pop``. When ``removing elements`` from arrays, especially if they could become large, consider the gas cost implications.
- Consider using ``gas-efficient libraries`` or ``built-in`` Solidity functions for common operations like ``mathematical calculations``. For example, Solidity's`` built-in`` ``add`` and ``sub`` functions can be more ``gas-efficient`` than custom ``SafeMath`` libraries in some cases.
- Ensure that functions are correctly marked as ``view`` or ``pure`` when appropriate. Functions that do not modify state should be marked as ``view`` to indicate this to the ``compiler`` and ``reduce gas costs``
- Avoid ``unnecessary`` state changes. Gas is consumed when you write to storage. Ensure that state changes are essential and can't be optimized away. For instance, consider if you can reduce the number of times ``tokenIdCounter`` is updated 
- Review your code for any ``redundant`` or ``unused ``functions``, ``variables``, or ``conditionals``. Removing ``dead code`` can reduce the ``contract size`` and ``gas costs``
-  Emitting ``events`` can be costly in ``terms of gas``. Emit ``events`` only when necessary, and consider emitting fewer details if possible.
-  External contract calls can be expensive. ``Minimize calls`` to external contracts, especially those with ``complex logic`` or ``unknown behavior``



## Code Commentary

1. Ensure that your ``variable`` and ``function names`` follow a consistent ``naming convention``. Use ``camelCase``as per solidity naming standards 
2. Lack of documentations for many critical functions 
3. There no clear information about when the ``emergencyWithdraw()`` called
4. Missing ``same value`` check for ``bool`` to avoid unnecessary write operations 
5. The ``SafeMath`` library is not necessary after Solidity version ``0.8.10``. Solidity 0.8.10 includes ``built-in overflow checks``, so there is no need to use the SafeMath library
6. Implement thorough parameter ``validation`` at the beginning of functions that ``interact`` with ``external`` contracts or involve critical operations. Check if addresses are ``not zero addresses`` and if ``amounts are greater`` than zero
7. Replace some of the ``if`` statements for ``parameter validation`` with require statements. This ensures that functions ``revert`` immediately if conditions are not met and returns gas to the caller.
8. Consider implementing ``access control`` checks for ``critical functions``. You already have an ``admin role``, but ensure that ``only authorized`` users can execute specific functions.
9. Consider marking ``addresses`` that won't change as ``immutable`` to make the contract more efficient.
10.  Use ``descriptive variable`` and ``function names`` that clearly ``convey`` their purpose and usage. Avoid ``single-letter`` variable names or ``cryptic abbreviations``.
11. Optimize for gas efficiency where possible. For example, in ``removeLiquidity``, you could check if ``lpAmount`` is zero before executing the ``remove liquidity`` operation.
12. The code could ``benefit`` from the use of ``function modifiers`` to reduce ``code duplication`` and improve ``readability``. For instance, a ``modifier`` for ``admin-only`` functions could be used. 
13. Consider breaking down ``complex functions`` into ``smaller``, modular functions with single ``responsibilities``. This improves code readability and makes it easier to test and maintain.
14. The contract includes ``error handling`` through the use of ``require`` statements. However, consider providing more ``informative error messages`` to enhance ``user experience`` and ``debugging``.
15.  Constants like ``DEFAULT_PRECISION``, ``RDPX_RATIO_PERCENTAGE``, and ``ETH_RATIO_PERCENTAGE`` are defined to improve code readability. This is a ``good practice`` for avoiding ``magic numbers`` in the code.
16.  Maintain consistency when ``using mappings``. For example, consider using a ``mapping`` for ``optionsOwned`` instead of an ``array`` if it offers better ``gas efficiency`` for ``lookups``.
17. Ensure that ``external contracts`` and ``addresses`` used by the contract are ``well-tested`` and ``secure``. Conduct external audits if necessary.
18. Be cautious about ``uninitialized variables``, as they can lead to ``unexpected behavior`` or ``vulnerabilities``. Ensure that all variables are initialized correctly.
19. Use access control modifiers such as onlyRole effectively to restrict the execution of certain functions to authorized addresses. Clearly define and document the ``roles`` and their ``responsibilities`` within the contract.
20. Remove ``redundant`` or ``unused code`` to keep the contract ``concise`` and ``maintainable``
21. Consider adding a ``version identifier`` or ``contract name`` to your contract to distinguish between ``different versions`` or ``instances``

## Risks as per Analysis

- ``_slippageTolerance > 0``,``_rdpxBurnPercentage > 0``,``_bondMaturity > 0`` does not yield the expected results. Setting ``_slippageTolerance`` to ``1`` makes it logically true, but this can lead to issues within the ``protocol``.
- ``Timelock`` functions not implemented efficiantly 
- Consider adding a ``fallback`` function with appropriate error handling to prevent ``accidental Ether`` transfers to the contract.
- The code does not include ``any mechanisms`` for ``contract upgradeability``. If this is a requirement, consider implementing an upgradeable contract pattern.
- There is no explicit ``reentrancy protection`` in the contract. Consider adding the ``nonReentrant`` modifier to functions that interact with ``external contracts`` to ``prevent reentrancy attacks``.
-  In the ``_sendTokensToRdpxV2Core`` function, multiple token transfers are performed in a loop. This can be ``gas-inefficient`` if there are many tokens to transfer. Consider ``batching`` or ``optimizing`` these transfers to reduce gas costs
-  The code includes several ``external contract calls`` without checking their ``return values``. Always check the ``return values`` of ``external calls`` and handle failure cases appropriately to ``prevent unexpected behavior``
-  The contract has several ``hardcoded addresses``. Consider using ``constructor parameters`` or ``governance`` mechanisms to set these addresses to improve ``flexibility`` and ``upgradability``
-  The ``approveContractToSpend`` function allows the ``contract owner`` to approve any contract to ``spend tokens`` on its ``behalf``. This could lead to ``unintended approvals``. Implement checks to restrict approvals to trusted contracts.
-  The contract ``RdpxV2Core`` may be susceptible to front-running attacks, where malicious users or miners observe pending transactions and exploit them for their advantage. Ensure that sensitive operations are protected against such attacks, especially when interacting with external contracts.
-  The contract interacts with external contracts, such as ``perpetualAtlanticVault``, in functions like ``_purchaseOptions`` and ``_issueBond``. There is a risk of ``reentrancy attacks`` if these external contracts do not have reentrancy protection.
-  Minting a ``Uniswap V3 position`` can consume a significant ``amount of gas``, especially if the ``liquidity pool`` has ``many tokens``. Ensure that the ``gas limit`` for the transaction is sufficient to cover these costs.
-  The function calls the ``mint`` function on the ``NonfungiblePositionManager`` contract. This function mints a new ``non-fungible`` position token and returns the ``token ID``, the amount of liquidity, and the tick range. This could be a problem if the ``NonfungiblePositionManager`` contract is not secure or if it is not working properly.
-  The function takes an additional parameter, ``positionIndex``, which specifies the index of the position to be removed. This could be a problem if the caller does not specify a valid position index or if the position index is out of bounds. The validity of the ``positionIndex`` not checked.

### Risks Found Based on QA Reports

1. The use of ``DEFAULT_ADMIN_ROLE`` for ``unrestricted token withdrawals`` can create conflicts of interest and ``jeopardize`` the ``protocol's integrity``.
2. Extremely ``short deadlines`` in the ``addLiquidity()`` function may discourage ``liquidity providers`` and hinder transaction processing
3. The absence of ``gas checks`` in ``Uniswap transactions`` may result in transaction failures during times of ``network congestion``
4. Excessive ``reliance`` on ``DEFAULT_ADMIN_ROLE`` centralizes control, presenting ``security risks`` and ``potential conflicts``
5. Certain functions ``lack checks`` to prevent their execution during ``paused states``, potentially leading to ``unintended outcomes``
6. Certain tokens, such as OpenZeppelin, may trigger a revert if an attempt is made to approve the zero address to spend tokens (e.g., approve(address(0), amt))
7. Several ``divisions`` lack checks for ``zero-value inputs``, which could lead to contract ``reverts`` when zero is provided as an input.
8. The codebase relies on an ``older`` version of ``OpenZeppelin-Contracts (v4.9.0)``, which may contain known issues related to ``encoding``, ``input validation``, and ``authorization``
9. The codebase lacks ``comprehensive checks`` for critical parameter assignments, leaving room for ``data corruption`` or ``inconsistencies`` within the protocol
10. Given the presence of ``NFTs`` within the platform, adding a ``blocklist`` function for NFTs is advisable
11. The codebase ``employs`` the deprecated ``Counters.sol`` library, which is ``not recommended`` by ``OpenZeppelin``
12. The ``swapExactTokensForTokens()`` function lacks validation checks to ensure that ``token swap``s occur as expected. This ``oversight`` could result in a ``loss of funds`` if the received amount of ``token ``B falls short of the `` requested amount``
13. The codebase lacks checks to ascertain whether tokens are ``blacklisted`` before executing token transfers
14. The absence of a setter function for the ``rdpxV2Core`` variable poses a potential risk to the overall integrity of the protocol
15. The ``addAssetTotokenReserves`` function does not verify whether an asset ``symbol already exists``, potentially permitting the addition of ``duplicate symbols`` to token reserves.

## Non-functional aspects

- Aim for high test coverage to validate the contract's behavior and catch potential bugs or vulnerabilities
- The protocol execution flow is not explained in efficient way. 
- Its best to explain over all protocol with architecture is easy to understandings
- Consider Using upgradable contracts  

## Time spent on analysis 
``15 Hours``



































































### Time spent:
15 hours