## Low 1: Owner can steal assets

The contract/protocol owner can steal assets, i.e. reserves / user deposits at the following instances:  
[UniV2LiquidityAmo.emergencyWithdraw(...)](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L137-L153)  
[RdpxV2Core.emergencyWithdraw(...)](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L156-L173)  
[RdpxDecayingBonds.emergencyWithdraw(...)](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L80-L107), including native token  
[PerpetualAtlanticVault.emergencyWithdraw(...)](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L214-L228)  
[UniV3LiquidityAmo.approveTarget(...)](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L139-L152)  
The above emergency methods work for all tokens and are **not** restricted to non-protocol relevant "stranded" tokens.  

**Severity:**  
In the real world this is critical since this stablecoin is dependent on incentivizing users to interact with the bonding mechanism in order to have enough reserves and hold the peg.  
However, such emergency withdrawal / rug-pull methods are trust eroding and disincentivize anyone from interacting with the protocol which in turn endangers the peg.  
Unfortunately such issues are typically not taken seriously by protocol owners, thus it is listed as *Low* in this report.


## Low 2: Custodian vs. `DEFAULT_ADMIN_ROLE`
The [UniV3LiquidityAmo.collectFees()](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L116-L133) method is meant to be called by the custodian (core contract) according to the comment above the method. However, the methods is restricted to default admin access which is the deployer of the `UniV3LiquidityAmo` contract and not the custodian.   

Due to the fact that that the core contract does not and cannot call `collectFees()`, the comment seems to be false/misleading and the access modifier correct.

## Low 3: Protocol is susceptible to oracle price manipulation
The protocol is heavily dependent on the rDPX/ETH LP, dpxETH, ETH and rDPX prices which are given by two protocol owned oracles, see also [RdpxV2Core L1201-L1242](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1201-L1242).  
Since those oracles are out of scope, we cannot know how reliable the price outputs are, e.g. how stable they are in case of a flash-loan attack.  
However, the bonding price, the underlying option premium and the LP share price directly related to these oracle price outputs and therefore vulnerable to oracle price manipulation.  
This can be abused to generate an advantage on bonding/withdrawal for the attacker and a disadvantage for the protocol which in turn endangers the peg.