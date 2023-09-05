# ![dopex img](https://www.gitbook.com/cdn-cgi/image/width=256,dpr=2,height=40,fit=contain,format=auto/https%3A%2F%2F3368698341-files.gitbook.io%2F~%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FAPIycFsj5hwiV3mOSG75%252Flogo%252FwYtDXbJoJxaakMTkPACy%252Flogo-gradient%25403x.png%3Falt%3Dmedia%26token%3D70db5826-d00a-44e6-81fc-46ac71181d79) Dopex Analysis Report  

## Description  

The contest focus was on the newly developed `DpxEthToken` and it's supporting contracts. This is V2 for the Dopex team, who have built a thriving options ecosystem.  

The key contracts in this contest are:  

**RdpxV2Core**: This is the main entry oint for users into the system. A user can bond their `rdpx` and `WETH` through this contract. The discount factor in this contract determines the discount users receive for bonding. The bonding mechanism supports the backing reserves of `rdpx` and `WETH`. This contract also contains Curve style Peg Stability Module functionality.   

**RdpxV2Bonds**: The bonds are minted as ERC721 tokens and are held by the aforementioned `RdpxV2Core` contract, which is the only contract that can mint bonds.  

**PerpetualAtlanticVault**: This contract is responsible for the perpetual options business logic. The contract allows the `RdpxV2Core` contract to purchase perpetual put options with a strike price that is 25% below the current price. The price is denominated in `rdpxPriceInEth`, and so these puts are in the money if either `ETH` increases by more than 25% compared to `rdpx` or if `rdpx` weakens by more than 25% compared to `ETH`.  

**PerpetualAtlanticVaultLP**: This contract holds the collateral for put options and allows users to supply liquidity in the form of `WETH` which is used as collateral by the `PerpetualAtlanticVault`. In addition, it allows liquidity providers to earn yield on their deposited `WETH`, as there is an epoch-based funding mechanism for holding the put options. 

**DpxEthToken**: The `DpxEthToken` is the star of the show. The Dopex team intends to make this a core asset used in their products. This token is intended to be pegged to `ETH` and the token economic mechanisms of bonding and put options are crafted to support this peg.  

## Approach  
The focus of the review was the implementation of the perpetual put options, the bonding mechanism and the business logic for the `DpxEthToken`. 

Manual review of the code and the provided protocol documents was done. Approximately 4 hours per day over the course of 10 days were spent reviewing the code.  

## Architectural recommendations  

The protocol consists of permissioned and non-permissioned components. The Algorithmic Market Operations module and other token management functionality cannot be made permissionless (at least at inception). The current architecture is suitable for a new product being deployed, which may require manual intervention in response to market actions. For example, when to exercise in the money put options requires team analysis and decision making which should not be automated at the start of the project. 

Once the deployment has stabilized and the project token economics are up and running, moving to a less permissioned architecture should be considered.

## Qualitative Analysis  

Overall the code reviewed was of a great quality with good test coverage.

| Category | Comments |
| ---- | ---- |
| Test suite | Both unit and integration tests in foundry were provided with good coverage | 
| Natspec | There were occasional, small natspec errors and instances where the comments were not clear |

It is not standard, but it is advisable to stress test the token economic assumptions in the protocol across market movements. The contracts are very exposed to `ETH` price movements (see Token Economic Risks below).

## Centralization Risks  

The codebase is heavily permissioned and depends on the Dopex Team to manage treasury functions. Multiple protocol functions require role-based access. For example, the settlement of put options when the puts are in the money is triggered manually through an EOA or multisig, which in itself may introduce risk; it must be noted, however, that this functionality cannot be safely automated away at the moment. 

In addition, there are multiple instances of deployer accounts being granted access to sensitive functions. This increases attack surface and should be avoided.

## Systemic Risks  

The `DpxEthToken` and the supporting mechanisms are exposed to smart contract risk from the chosen `WETH` token. A failure of trust in the aforementioned tokens contract will contaminate the `DpxEthToken`. A possible consideration would be to use various `WETH` tokens from multiple established projects.

## Token Economic Risks  

There is significant concentrated asset risk in the audited contracts. The `DpxEthToken` is backed by 75% `ether` (used interchangeably here with `ETH` and `WETH`) and 25% `rdpx` up front; in addition it is supported by a put option which backstops the collateral to be around 93% `WETH` when the `rdpx` price falls (75% straight `WETH`, with an additional 25% `rdpx` of which 75% is effectively `WETH` through the put option). 

The upside of this mechanic is that it becomes easier for `DpxEth` to maintain it's peg to `ETH`. This is also the biggest downside: `WETH` price movements will greatly impact the TVL. 

The edge case is also worth mentioning: in a market scenario where the `WETH` price is falling, the `rdpx` price is likely to come under pressure as well. If the prices of both of these assets fall simultaneously, and to similar degrees, the `Core` contracts will be unable to exercise their put options (because the strike price is in `rdpxInEth`).

It can be argued that this is by design:`DpxEthToken` is, after all, intended to be pegged to `ETH`. But it must be conceded that this leads to a scenario where the TVL can reduce drastically in USD terms in a relatively short time frame and the protocol would be unable to reduce it's exposure to these assets.  

Protocol survival in times of market distress is crucial, and negative price movement in `ETH` will trigger flight to stablecoins; away from both `ETH`,`rdpx` and `DpxEth`, further putting pressure on *all* the assets. The slump in `rdpx` in such a scenario is unavoidable (as evidenced by most tokens' historical price movements in downturns).

It is recommended that the team prepares and implements various hedges to account for such market scenarios. It is likely that the team already does this, but it is important to note nonetheless.

## Time Spent  

40 hours.  

### Time spent:
40 hours