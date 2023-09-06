## Any comments for the judge to contextualize your findings

My findings were based on several key areas of the protocol as mentioned below:

1. Unexpected state changes during execution of key functionalities of the protocol.
2. Errorneous logic implementation in slippage calculation of the `_curveSwap` function.
3. Wrong decimal precision used in the price feeds used for calculations.
4. Lack of input and return validation in important functions.
5. Possiblity of using `stale price data` returned from UniswapV2 TWAP oracles. 
6. Vulnerabilities arising from single logic failure in a `for loop` execution which could revert the entire transaction.
7. Errorneous logic implementation of arithmetic operations in key functions of the protocol.
8. Lack of deadline check in the `swap` transaction of the `_curveSwap` function.

## Approach taken in evaluating the codebase

I initially started analysing the documentation provided for the audit. The provided `dopex notion site` was exetremely valuable in understanding the core workings of the `dopex` protcol. After understandings the core concepts of the protocol such as `Regular bonding`, `Delegate bonding`, `Bonding via rDPX Decaying Bond`, `Atlantic Perpetual PUTS Options`, `rDPX V2 Bond Receipt Token` and `AMO`, I started learning about how they interact with each other. Once I obtained a sufficient enough understanding of the protocol I started auditing the codebase.

I initially started on the `RdpxV2Core.sol` contract. It provided me the initial structure of the protocol to audit on. `RdpxV2Core.sol` contract mainly used the utility functions of the `PerpetualAtlanticVault.sol` and `PerpetualAtlanticVaultLP.sol` contracts. Hence I was able to parallely audit `RdpxV2Core.sol`, `PerpetualAtlanticVault.sol`, `PerpetualAtlanticVaultLP.sol` contracts following thier execution flow.  

Then I started auditing the other utility contracts of the core `RdpxV2Core.sol` contract. I started auditing `RdpxDecayingBonds.sol` contract because it was directly related to minting `rDPX decaying bonds` and was used in the `RdpxV2Core.bondWithDelegate` function execution. After that the `DpxEthToken.sol` and `ReLPContract.sol` were audited.

Finally I moved on to auditing the AMO contracts. I audited the `UniV2LiquidityAmo.sol` and `UniV3LiquidityAmo.sol` contracts. These contracts were responsible for managing the addition and removal of liquidity on `rDPX/ETH pool` of `UniswapV2` and `UniswapV3`.

## Architecture recommendations

rDPX V2 is a comprehensive and forward-looking solution to increase the utility of the rDPX token. It brings in a combination of `bonding mechanisms`, `algorithmic market operations`, `perpetual options pools`, and `stability modules` to maintain the peg of the newly introduced `synthetic coin`, dpxETH, to ETH. The system is integrated with DeFi protocols such as Uniswap and Curve to maximize benefits and yields for its users.

Currently the protocol is controlled by the `DEFUALT_ADMIN_ROLE` since main functions of the protocol are only accessible by that role. But it is recommended to introduce a `DAO` for the governance of this protocol to make the solution more decentralized. If `DAO` is not introduced a `multisig` should be used as the `DEFAULT_ADMIN_ROLE` user.

The protocol uses `UnsiwapV2 TWAP Oracles` for real-time price feed for ETH and rDPX to adjust bonding mechanisms, options pricing, and peg adjustments. If the `UniswapV2` gets compramised in the future due to an upgrade or any other reason this could impact the operations of the `rDPXV2 protocol`. Hence it is recommended to analyse other options for oracles such as Chainlink, MakerDAO's oracles, or other reputable price feeds.

## Codebase quality analysis

The code was proprely structured. The core contract of the codebase was the `RdpxV2Core.sol` contract. Other contracts mainly were utility contracts which provided the required functionality for the execution of the core contract logics such as bonding, put option purchase and AMOs.

There were few `Natspec` comments errors prevalent in the codebase. Due to the discrepency between the Natspec comments and logic implementation it was difficult at times to understand the key workings of the main functions of the contracts during the audit. Hence it is recommended to put more attention on the correctness of the `Natspec` comments which would make it easier for the developers as well as the auditors to understand the codebase.

It is recommended to put more attention on the decimal precision of the price feeds used in the `RdpxV2Core.sol` contract and `UniV2LiquidityAmo.sol` contract. This is because the contracts expect the decimal precision of the price feeds to be in `1e8` precision where as the price feed precision returned from the library functions are in `1e18`. This breaks the core accounting of the protocol. This issue has been reported under H/M vulnerabilites of the audit.

Futhermore it is recommended to put more attention on the `gas optimization` of the contracts. Because there were multiple cases where the state variables were used more than once within the function exectuion scope. This was extremely gas consuming and could have easily being avoided by caching the state variable into a memory variable and using the memory variable for respective operations. 

## Centralization risks

There is a high level of centralization risk in this protocol. Since almost all the critical functions of the contracts were controlled by the `DEFAULT_ADMIN_ROLE`. Hence the address with this role has the complete authority over the protocol. The `DEFAULT_ADMIN_ROLE` can decide to grant or revoke other roles such as `MINTER_ROLE` from other users.

Since most of the functions of the protocol were access controlled by the `openzeppeling` `AccessControl.sol` contract the `DEFAULT_ADMIN_ROLE` becomes a critical factor of the protocol. 

Hence it is recommended to use a `multisig wallet` as the `DEFAULT_ADMIN_ROLE` or later move on to a `DAO` based governance system to mitigate the centralization risks present in this protocol. 

## Mechanism review

rDPX V2 introduces dpxETH, a `synthetic coin` pegged to ETH, enabling `boosted yields on ETH` and serving as `key collateral` for upcoming Dopex Options Products. The rDPX bonding process allows for `minting of dpxETH` tokens. Upon bonding with the rDPX V2 contract, users are given a receipt token, signifying ETH and dpxETH LP on curve. 

New dpxETH minting through bonding is supported by rDPX and ETH reserves, managed by AMOs. To safely and effectively scale rDPX V2 and dpxETH, the `AMO concept from Frax Finance` has been adopted.

The bonding process also has variations such as `delegate bonding` and `rDPX decaying bonds`. The `delegate bonding` enables users with only `weth` to delegate thier `weth` so that the users with `rDPX` can use them to bond and recieve the respective share of the receipt tokens. The `rDPX decaying bonds` are minted as `rebates for losses incurred` on the Dopex Options Protocol or for `incentives`. These `decaying bonds` can be used in the `regular bonding` or `delegate bonding` process.

Furthermore `AMOs` a concept used in the `Frax ecosystem` is used in the protocol in the form of `Uniswap V2 AMO` and `Uniswap V3 AMO`. These `AMOs` are used in the `UniswapV2` and `UniswapV3` respectively to `add and remove liquidity` on `rDPX/ETH pools`.

## Systemic risks

The protocol uses the `UniswapV2` and `UniswapV3` pools for `rDPX/ETH` pair in the AMO for liqudity addition and removal. Hence this creates a risk if the `Uniswap` protocol is compramised in the future due to an upgrade or attack this could direclty impact the `dopex` protocol as well.

Futher `dopex` protocol uses the `Uniswap TWAP oracles` for price discovery with in the protocol. Since the `TWAP` is used as the price feeds the manipulation risks are less but still exist. If a high networth individual can manipulate the respective liquidity pool long enough (especially when the liquidity is less in the pool), he may be able to manipulate the price thus resulting in errorneous accounting which could be disadvantageous to the users as well as to the protocol.

The `curve` protocol was used for `swap` operations between dpxEth and ETH tokens. The `curve` swap does not include `deadline` checks for the transaction execution. This could be problematic if a malicious validator decides to delay the swap transaction and executes the same at a differnt price point which is disadvantageous to the protocol.

### Time spent:
40 hours