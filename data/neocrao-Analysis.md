# Analysis Report

## Auditor
NeoCrao

## Auditing Approach
Below are the steps that was followed to go through the project and audit it completely:

* Setup the project: Start by setting up the project, and run the tests.

* Start by reading the documentation: The documentation https://dopex.notion.site/rDPX-V2-RI-b45b5b402af54bcab758d62fb7c69cb4 provided a good overview of what the V2 version is introducing. The documentation was pretty thorough, and helped in grasping a fair idea of what the project was bringing.

* Round 1 - Skimming through the code: Started by going through each of the contract one by one. The primary focus was to understand how the code is organized, and the hot spots, including the public entry points, admin functions, and similar observations. If somethign caught my eye, then they were tagged with @audit.

* Discord channel (Ongoing) - I started this contest a couple of days late, so there was a lot of catch up on Discord. Went through all the messages in Discord to improve my understanding, and get answers to some questions that I might have which people would have already posted.

* Round 2 - Picked the contracts from smallest to the largest sLOCs, and went line by line. I am not too good with DeFi math. So I tried not waste too much time on it. Kept marking @audit to whatever seemed suspicious. At this step, I had to go back to Discord and the documentations multiple times to get a better understanding of what I was reading.

* Review all the @audit tag items: Now I started by going through all the items that were tagged with @audit one by one. I start with tackling the lows/QAs first as a warm up, and then start reviewing the potential high/meds one by one. For highs/meds I started with the simpler ones, and created tests as needed. And then I went after the harder ones.

## Total Time Spent

I spend around 30 hours on this project.

## Codebase Quality Summary
* The codebase was organized pretty well. It made use of OpenZepplin for implementing standard features, and ensured that none of the inherited specs were missed. It also ensured to inherit the correct contracts like BurnableERC20, and others that allowed pausing and unpausing of the contract.
* The documentation throughout the code was very well, and it really helped in understanding the code.
* Most of the methods were very short, which really made it convenient when reviewing the code as individual units.
* Some of the conditions seemed overly complex, and I have made suggestions in my report on how to simplify them.

## Architecture Feedback
The architecture of the code seems pretty simple. Most of the entry points were in the Core contract, which really made it easy to follow the sequence of function calls. However, there are opportunities to make these contracts upgradeable, timelocks and add governance to it, as there are too many parameters across the contracts that have setter methods, and its always better to handle these either through upgradeability, governance, and timelocks.

More specifically, the bond maturity dates can be updates, several slippage rates can be adjusted, and contract addresses can be updated directly, which is very prone to errors.

## Centralization Risk
Most of the critical functions require admin role, and hence the protocol is heavily prone to centralization risks. 

The most dangerous feature was the emergency withdrawal functions in the contracts which can drain all the tokens from the contract. These operations should require voting, multisigs by multiple admins, or atleast some form of DAO control.

## Recommendations
* There was a lot of math in the codebase which was not too clear. Either it requires more inline documentation, or the actual notion documentation needs to talk about it.

### Time spent:
30 hours