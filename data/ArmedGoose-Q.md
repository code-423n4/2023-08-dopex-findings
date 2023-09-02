[Low-01] Redundant code
Some of the code is not used throughout the project. Examples are:
in `UniV2Liquidity.sol`:
 - [Unused variable slippageTolerance](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L51)
 - since `slippageTolerance` is not user, therefore the function setting it is also not needed: [setSlippageTolerance()](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L109-L117)

in `UniV3LiquidityAmo.sol`:
 - Abstract contract [OracleLike](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L22-L26) which is never used

[Low-02] Key and predictable state variables are not initialized
There are some key state variables that in order for the protocol to work properly, have to be initialized. For example, the roles that are critical for the protocol or key tokens that anyway will be used. 
They still can be updated later, but initializing it as the values are predictable (e.g. we already know what tokens will be used and what roles will be used) makes the protocol more error-prone, as during manual setup some steps might be accidentally omitted, leaving the protocol in incorrect state. 

The occurences are:
- [RdpxV2Core.sol](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L52-L58) only the first asset is initialized in constructor
- Key roles are not set upfront, e.g. RDPXV2CORE_ROLE in RdpxDecayingBonds.sol is used in `decreaseAmount` and  but never initialized, the same role is int PerpetualAtlanticVault.sol used in `purchase`, `settle` and `payFunding` but never initialized

Recommendation: Pre-initialize the known variables in constructor or in an initialization-like function.

[Low-03] RPC disclosure
The unit tests contain a valid RPC URL which presumably may have some limitation and be related to a project maintained by the team. Once disclosed, it can be freely used by users throughout the internet or scrapped from github, and the team's limit for this RPC might be abused.
Recommendation: After the contest, remove the URL and regenerate new one.

[Info-01] Bond mapping is not cleared after burn
When a bond is burnt, in 
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1032
later its associated mapping is used once few lines later, in
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1035
however the bond mapping is not cleared at this point regardless its already burnt. There is no reuse of the mapping in code, and it was not proven to have any significant impact, however even for sake of little gas refund for releasing the mapping from storage its worth to clear it after burn.
