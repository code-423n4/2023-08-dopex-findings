# Dopex analysis report

Aug/Sept 2023

*Dr. cmark0v*


## Methods and materials

Clone repo; Get errors; fix typos in filenames and scripts; furnish docker containers with build stack and security tools. Run all existing tests, flatten all in-scope code, Read code starting with ``DpxEthToken.sol``.  Read docs a little. Focus on ``RdpxV2Bond.sol`` and ``RdpxV2Core.sol`` and anywhere I see math, authentication, questionable standards or ambiguous quality I give extra effort. Scan with `mythril`. Work on PoCs using the existing test codebase. Try to pay special attention to the interactions between the contracts and foreign ones, as well as potential misuse of libraries. 



## Overview

### Impression

First impressions of the code

**overall**
- Anemic set of tests. Emaciated even. 
- signs of a struggle(with math)


**design**
- unused code features from libraries, meaning increased attack surface without feature gains
- absence of implemented/codified governance functionality 


**practices**
- absence of consistent commenting standards
- inaccurate comments
- lack of cohesive standards, code doesn't all look like it was written by the same person
- various linear combinations of incoherent , inconsistent, and absent natspec

**docs**
- 'we have docs at home' -mom 


### Auth


1. **The role based authentication is overkill if you don't need multiple reconfigurable role holders then do not have the features for them.** It is just another place for a potential disaster. It is also a waste of gas. It looks suspicious to people reading the code to see who can burn their tokens and things. namely when you need to backtrack through a chain of functions and over rides and modifiers to even know who actually has these roles. 

2. **Unilateral unchecked power**  - There is an excess of unrequired power indefinitely available to a single administrative wallet. This is very opaque to users, many wallets can be added and remain somewhat opaque by blockchain standards.

3. **Emergency withdrawl function** - Allows arbitrary and inconsequential withdrawl of all funds by admin at discretion. Does nothing but issue an event. This is the worst

4. **Pausability** - same as aforementioned emergency administrative withdrawl, eveything can be paused and unpaused at total dsicretion of a singular wallet. Blocking withdrawls for non-admins  . second worse


4. **lack of procedures defined**

5. **excessive manual procedures** - more things should be done in constructors or with launch scripts that are dept as part of the codeibase. 

6. **role based auth is overkill and leaves too many features for potential abuse** - it uses mappings so its not trivial to enumerate all roles and admins. It is way more than what is needed




### Math


1. **Using single point precision** - There is no advantage to this and it is teetering on the edge of manifesting as critical problems see

2. **redundant algebraic forms**  - not properly formulated for the task. sloppy, unconsolidated algrabraic forms. 

3. **inconsistent use of  parameters** - digits of accuracy defined in a parameter that is neglected to be used in favor of using hard coded values. it should be used everywhere 1e8 denominator or other appearance deterministically dependent on that choice of parameter, so that if it is changed, it wont break the whole system.  

4. **excessive use of hard coded constants with no comments** - when you hard code a constant in it has no variable name so there is not even a hint as to where it could have come from or why it is there so it is good to put a comment. its also memory efficiency problem, AKA a gas problem. 

5. **lack of comments on key equations** - computationally efficient(cheap) and viable(accurate) form of an expression is generally not the nicest on the human eyes, so the form it is derived from should be present in the natspec or other comments. Which should be traceable back to whatever the spec speed on the tech is. 

6. **odd use of percentages in computation** - use of percents converted to a synthetic float fraction over 100 in computation has no computational benefit, no basis on any convention. calculation does not use percents. It uses decimal or fraction representations of floating point numbers. multiplying the synthetic decimal form of a parameter by 100 and storing it like that at runtime is odd and bound to result in operator error or confusion. Percents are a display form of a scaling factor. 




### Tests


1. **Emaciated set of tests** - maybe it somehow has high code line coverage but it does not have execution state space covered

2. **non parameterized tests** comparing things to known outcomes, no parametric tests. Math is being 'tested' against a few trivial values against a few mysterious hard coded outcomes with no visible connection to anything that shows us

3. **no fuzztests** - can cover a lot more ground


## Details, mitigation




### Mathematics

Most of the themes discussed for the math above are well addressed by some guidelines for approximating rational numbers on these sets of integers we work with in solidity:


1. Use ``WAD = '1 ether' = 1e18`` as the  default choice

2. deviate from ``WAD`` to ``RAY=1e27`` or from ``WAD`` to ``1e8`` if it solves a problem

3. multiply and add then subtract and divide. you want to finish with a division by the base or some positive function of it

4. analyze your schemes and verify the bounds and order of ops

5. spec out the human readable(physically relevant) form of the equation in docs and in comment

6. after rearranging according to 3 and 4 to keep every operation in bounds, combine all like terms and reduce the number of operations if possible

7. store parameters in the same precision as the calculations unless otherwise labeled in setters and variable names. 

Put the work in your docs preferably so it can be checked efficiently. The more detail in docs the less bandwidth it takes to check this stuff. The probability of errors goes down asymptotically as a function of the number of people who checked it in full honest detail. 


Division and multiplication are the same operation(with different elements) on the real numbers but division and multiplication are not the same operation in these sets of integers. Neither is subtraction the same operation as addition. If you are thinking in terms of algebraic structures formed by closed operations on a set. 



### Auth/Gov




1. Role based auth system The wallet launching the contracts ``RdpxDecayingBonds.sol``, ``RdpxV2Bond.sol``, ``RdpxV2Core.sol``, ``RdpxV2Core.sol``, ``ReLPContract.sol``, ``UniV2LiquidityAMO.sol``, ``UniV3LiquidityAMO.sol`` is grated ``DEFAULT_ADMIN_ROLE``, which can be used to unilaterally drain all funds, pause contracts, grant other permissions to mint and burn, etc. This is by no means necessary from a technical perspective and exposes the team to unnecessary risk and potential liabilities. If it is a multisig that is still an opaque mechanism that involves trusting a group of people who know eachother and have like interests. 


These things should be controlled through a governance contract with transparent, well documented, verified procedure for any serious administrative events. Multiple parties with transparent incentives to operate with the protocol's best interest in mind. When it is launched, the governance contract address is supplied in the transaction and never has to be done through separate manual processes. 


```solidity
// File: contracts/reLP/ReLPContract.sol
// Lines: 70-80

70   bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
71 
72   /// @notice liquidity slippage tolerance
73   uint256 public liquiditySlippageTolerance = 5e5; // 0.5%
74 
75   /// @notice The slippage tolernce in swaps in 1e8 precision
76   uint256 public slippageTolerance = 5e5; // 0.5%
77 
78   // ================================ CONSTRUCTOR ================================ //
79   constructor() {
80     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
```


2. ``MINTER_ROLE`` is granted to ``msg.sender`` on launch of these token contracts. This is even less necessary than having the admin role defined in this manner. The contract that controls minting should be supplied at launch as a argument to constructor, and reconfigurable through a transparent governance mechanism controlled by a contract. It should reject configuration of more than one minter if the protocol doesn't require it. It should verify that the minter is not a wallet and is known to the governance contract. 


3. **emergency withdrawal function lacks safeguards and formality** The emergency withdrawal should be only callable if the contract is permanently locked and can not be re-activated. Or alternatively, involve some other irreversible action. These kinds of functions need to be controlled by governance **OR** they must NEVER act on the assets that are in play within the given contract during nominal operations. IE it is only for shuffling misplaced blind token sends by inexperienced users,


```solidity
// File: contracts/core/RdpxV2Core.sol
// Lines: 161-170

161   function emergencyWithdraw(
162     address[] calldata tokens
163   ) external onlyRole(DEFAULT_ADMIN_ROLE) {
164     _whenPaused();
165     IERC20WithBurn token;
166 
167     for (uint256 i = 0; i < tokens.length; i++) {
168       token = IERC20WithBurn(tokens[i]);
169       token.safeTransfer(msg.sender, token.balanceOf(address(this)));
170     }
```


At the very least, it should not be possible to call this for the tokens the contract is holding for its functions



4. **Unilateral governance** - the governance functions, all very powerful and capable of serious harm to users and collapse of the protocol, are formally controlled by a single wallet that launches them. This is not acceptable. No one cares if it is a multi-sig wallet. This is supposed to be trust-agnostic, transparent  platform as much as possible. Dumping out funds should authorize depositors to withdraw or require their approval in a vote








### Time spent:
35 hours