##  <img src="https://user-images.githubusercontent.com/68193826/265804925-d7e1d5bb-e425-4d75-a573-c155c81a402a.svg" loading="lazy" width="30" alt="" class="image-9"> Description
Dopex is an advanced decentralised option trading solution. with this audit dopex is going to introduce a new coin that is pegged to the value of eth backed by the reserves assets of Ethereum and rdpx. Bonding will happen via options and will also lead to minting out of the stable coin.


To maintain the peg, dopex have meny noval mechanims, such as amo, Relp and over collateralizing and de collateralizing techniques.

Along with these there are vaults that will be managing the Lp and their funding via the premiums paid. For the first epoch option buyer will himself pay the premium and later on all the premium will be paid by protocol in start of each epoch.

**The key contracts of the protocol for this Audit are**:

- **RdpxV2Core**: 
    This function is responsible for performing all the functionalities by using other contracts. It is the entry point for most of things.

- **Rdpx Decaying Bond Contract**: 
     Only admin can mint these bonds and are source of compensation for the users for loss accured on the protocol.

- **PerpetualAtlanticVault Contract**: 
     Contract which is used to pay all the premiums and fundings to the lp's.

- **PerpetualAtlanticVaultlp Contract**: 
    Main entry point for the liquidity provider and is also used by the vault to keep record of funds and assets availible to be used.

- **AMOContract**: 
     Two type of AMO are being used, this concept is inspired from the FRAX. Use to add and remove liquidity to the pools

- **ReLP Contract**: 
     This contract simply removes the eth liquidity and swap back it into dpx to increase the its value and value of backing reserves.

**Existing Patterns**: The protocol uses standard Solidity and Ethereum patterns. It uses the `ERC721` standards for most part and `Accesibility` pattern to grant different roles and also there is `whitelisting` and `pausing` mechanism too.
Protocol is also partially based on ERC4626 standard too but does not follow it completely and it is intentional.

##  <img src="https://user-images.githubusercontent.com/68193826/265804925-d7e1d5bb-e425-4d75-a573-c155c81a402a.svg" loading="lazy" width="30" alt="" class="image-9"> Approach
I took the top to down approach for this audit, so general flow is as follow

```
1. rdpxV2Core.sol
2. perpetualAtlanticVault.sol
3. perpetualAtlanticVaultLP.sol
3. Amo and Relp
4. RdpxDecayingBonds.sol
```
Vaults, Amo and ReLp were reviewed side by side as they have close relationship with each other and balance out each other functionalities or build on top of each other and core interact with each of them. But decaying bonds contract can be reviewed separately.

##  <img src="https://user-images.githubusercontent.com/68193826/265804925-d7e1d5bb-e425-4d75-a573-c155c81a402a.svg" loading="lazy" width="30" alt="" class="image-9"> Codebase Quality
I would say the codebase quality is good but can be improved, there are checks in place to handle different roles, each standard is ensured to be followed. And if something is not fully being followed that have been informed. But still it can be improved in following areas


| Codebase Quality Categories  | Comments |
| --- | --- |
| **Unit Testing**  | Code base is well-tested but there is still place to add fuzz testing and invariant testing and foundry have now solid support for those, I would say best in class.|
| **Code Comments**  | Overall comments in code base were not enough, more commenting can be done to more specifically describe specifically mathematical calculation. At many point if there were more detail commenting and natspec it would have been bit easy for the auditor and less question would have been asked from sponser, saving time for both. Specifically perpetual vault needs more comments as its flow is hard to understand |
| **Documentation** | Documentation was to the point but not detailed enough, such novelty requires more documentation than one notion page, that can be improved and I am sure will be done as previous products have full proof documentation on dopex website.|
| **Organization** | Code orgranization was great for core, decaying bonds, amo's and Relp. A bit of improvement is needed in vault and lp vault, specifically in vault every thing was all over the place. If some one is not familiar with the flow of function he will remain lost. |

##  <img src="https://user-images.githubusercontent.com/68193826/265804925-d7e1d5bb-e425-4d75-a573-c155c81a402a.svg" loading="lazy" width="30" alt="" class="image-9"> Systematic and Centralisation Risks

### 1. Centralization Risk: 

It is important to note that there are many previliged function in the system that if went rogue can cause significant problems for the protocol, for example roles for minter, fee setters, deployer and other roles granted using access control. But the bigger threat is that in every contract except for ReLP contract there is an emergency withdraw function that can be easily misused at some point and is a very big threat to the protocol. 
                                        
Emergency withdraw function for either ERC20 or ERC721 or for both is present in following contract:

1. core have the Emergency withdraw function
2. V3 amo have recoverERC20 and recoverERC721 functions.
3. Vault have emergency withdraw function
4. Lp vault also have the emergency withdraw function.

Theses function may seem necessary in some cases but are constant risk to the protocol. Even for multisig wallets keys keep getting compromised.

### 2. Pools liquidity and manipulation risk

The whole system is overall very dependent upon the rdpx, dpx, and weth pools on uniswap but while anaysing these on a tool developed by  chaos lab here:

https://community.chaoslabs.xyz/uniswap/twap

It can be seen that rdpx/weth pool on both uniswap v2 and v3 can be manipulated for a very less cost, and the highly dependence of dopex on these pool is an innate risk for the protocol.

Lets have a look here on both uniswap v2 and v3

[Uniswap v2 analysis](https://community.chaoslabs.xyz/uniswap/twap/pools/arbitrum/0x310a09b51501d893988401fba75c47dc59f1b42a)
[Uniswap V3 analysis](https://community.chaoslabs.xyz/uniswap/twap/pools/arbitrum/0xb52781c275431bd48d290a4318e338fe0df89eb9)

![image](https://user-images.githubusercontent.com/68193826/265802514-573dd11b-d995-4c6e-95ad-9403cb65056b.png)
As we can see that manipulating the price of rdpx/weth on uniswap v3 just cost around 7 grands and also the exhaustion cost is not that high.

And even worse for dpx/weth pool as the manipulating the price by 10% only cost around $2000. And exhaustion cost is also way lower than v3 pool.

And dopex is highly dependence on these. so my recommendation is first focus on increasing the liquidity of these pools and making than more manipulation proof will be a good move.

##  <img src="https://user-images.githubusercontent.com/68193826/265804925-d7e1d5bb-e425-4d75-a573-c155c81a402a.svg" loading="lazy" width="30" alt="" class="image-9"> Conclusion
Overall dopex came up with a very strong solution with some weak points in commenting, docs and some centralisation risk. But the approach used for the core working of protocol is really solid and up to the industry standard and fixing above recommendation will make it even more robust.


### Time spent:
20 hours