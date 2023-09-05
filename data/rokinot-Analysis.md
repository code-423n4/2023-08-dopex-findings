#### Analysis Introduction

Dopex is a decentralized options protocol, as the initials suggest. The protocol offers classic european options as well as a novel "atlantic" model, a mix between american and european where the collateral can be used for other purposes, leading to a more capital efficient model. 

The protocol centers around two main tokens: DPX, the governance token, and rDPX, the rebate token, with an elastic supply and multitude of uses such as collateral on synthetics, lockup for boosted fees and so on. The new rDPX V2 model aims to introduce a new usage of the later, and was the object of this code review.

#### Approach taken in evaluating the codebase

- Arbitrum 

This is the first and foremost aspect that was taken into consideration. This is important to consider throughout the whole auditing process, as certain patterns can be found in the network that won't be found on Ethereum mainnet, and vice versa. One example of such, the gas limit on Arbitrum is of 1.125 quadrillion, this means any form of DoS by reaching the block gas limit is left off the table, as this would be equivalent. Another example is any vulnerability regarding front running being invalid, as the network's centralized sequencer and low latency make it impossible to front run transactions (with certain exceptions, for example, contract calls that are scheduled to happen at a fixed frequency).

- Personal approach

My approach to reviewing the codebase is to choose a smart contract to begin with and branch off to the next ones from there, based on their dependencies and such. For this contest I began reviewing the AMO contracts instead of the core; some time ago I found that beginning an audit through more peripheral contracts can be a more comfortable experience. The documentation provided in order to understand the new mechanism was somewhat dry, as many details had to be often asked to the developers directly through discord, though it is straightforward.

The more relevant findings are more related to internal accounting rather than systemic risks or mishandling of user funds. Final impressions that were left were more that the protocol's own funds are at risk rather than users' funds.

- Dopex

I was a dopex user in the past, so I had a general overview of the protocol before the audit was even announced. rDPX V2 is a feature which is long awaited by the dopex community, having suffered a complete revamp from it's previous dpxUSD design. These new contracts which are being introduced however have a completely different design from what I previously used, so past experience could not be leveraged in this review.

#### Codebase quality analysis

Based on the style of code writing, some contracts were either not written by the same individual or reviewed by a third party before the audit began. The use of custom errors in RdpxV2Core compared to the use of require statements elsewhere is one parameter that leaves this impression, there were also some comments spelling mistakes and inadequate code discrepancies between core and peripheric contracts as well. The different usage of `transferFrom()` and `safeTransferFrom()` is another factor.

The code is quite intertwined, the core contract requires a lot of vault functions. For a few examples of this:

1. Both vault and vaultLP can update the funding when necessary. An updated funding will lead the core contract to change it's bond behavior.

2. Funds can be transferred from the users to core, from backing to core, from reLP to core and from AMO to core. From there on, they can be further moved to the vault LP when settling options, or the perpetual vault when purchasing options. 

3. Pausing the vault contract means the core contract cannot create bonds or settle options anymore (the latter if purchasing APP are required). The core contract is also unable to update the funding.

Overall, a proficient smart contract developer was required in order to write an extensive smart contract as rDPX V2 core, as well as deploying other contracts which are deeply dependent on one another. 

#### Centralization risks

As it is becoming a more common design pattern in the crypto ecosystem, the administrator has the power to compromise the entire protocol by withdrawing all funds. For this project, the core contract, the AMOs and the perpetual vault all have functions that enable the administrator to withdraw any token whatsoever, these functions being `emergencyWithdraw()` and `recoverERC20()`.

Beyond funds being at risk, there is also the risk of the administrator mishandling atlantic put options. Those are bought when a user is bonding, and sold at the admin's whims. According to a comment from the developer, these options will be settled when the backing reserve is high in rDPX and low in WETH backing. As to how much should be considered high or low, it's up for them to decide.

#### Systemic risks

[Arbitrum's sequencer has gone offline in the past](https://twitter.com/shotaronowhere/status/1666720771421671426), which essentially shuts down the network. This was something that was considered in the auditing process but even after taking this into account I was unable to find any vulnerability pertaining this matter. So assuming no bug was left behind, the protocol should be able to handle these down times.

Matters pertaining oracles are outside the scope of this contest, so nothing can be said about it.

#### Note for judges

I sent one of the vulnerabilities found to be low risk, more specifically the one pertaining to the recovery of ERC721s tokens. I did it so because, based on past contest experiences, judges seem to have a tendency to reduce the gravity of vulnerabilities that pertain to user error (in this case, accidentally sending ERC721 tokens to the core contract). There may be a case to be made about changing this to a medium risk, as it is a function not working as intended even though I sent this as a low risk finding. It's ultimately left for the judge to decide what severity it should be given.



### Time spent:
25 hours