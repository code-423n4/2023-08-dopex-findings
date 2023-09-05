| | issue |
| ----------- | ----------- |
| 1 | [State variables only set in the constructor should be declared immutable.](#1-state-variables-only-set-in-the-constructor-should-be-declared-immutableones-not-caught-by-bot) |
| 2 | [Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate](#2-multiple-addressid-mappings-can-be-combined-into-a-single-mapping-of-an-addressid-to-a-struct-where-appropriate) |
| 3 | [state variables should be cached in stack variables rather than re-reading them from storage](#3-state-variables-should-be-cached-in-stack-variables-rather-than-re-reading-them-from-storage-ones-not-caught-by-bot) |
| 4 | [should use arguments instead of state variable](#4-should-use-arguments-instead-of-state-variable) |
| 5 | [it costs more gas to initialize non-constant/non-immutable variables to zero than to let the default of zero be applied](#5-it-costs-more-gas-to-initialize-non-constantnon-immutable-variables-to-zero-than-to-let-the-default-of-zero-be-applied) |
| 6 | [Use assembly to check for address(0)](#6-use-assembly-to-check-for-address0) |
| 7 | [state variable should be declared as constant which saves lots of gas](#7-state-variable-should-be-declared-as-constant-which-saves-lots-of-gas) |
| 8 | [Use hardcoded address instead address(this)](#8-use-hardcoded-address-instead-addressthis) |
| 9 | [Unbounded Gas Consumption Risk Due to External Call Recipients](#9-unbounded-gas-consumption-risk-due-to-external-call-recipients) |


## 1. State variables only set in the constructor should be declared immutable.(ones not caught by bot)

Avoids a Gsset (20000 gas) in the constructor, and replaces each Gwarmaccess (100 gas) with a PUSH32 (3 gas). While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

`rdpx` which is a state variable in `UniV3LiquidityAmo` is a address value that is never changed in any place, so it can be made as a immutable because it doesnt change other than in constructor.
- [UniV3LiquidityAmo.sol#L69](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L69)

same thing with `rdpxV2Core` as well
- [UniV3LiquidityAmo.sol#L72](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L72)

`rdpx` also doesnt change value after constructor so it can be immutable to save a 20000 Gsset gas and every call for it will be 97 gas cheaper
- [PerpetualAtlanticVaultLP.sol#L101](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L101)


## 2. Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. 

`optionsOwned` and `fundingPaidFor` can be turned into a struct to avoid a Gsset (20000 gas). this will not save any other gas because there is no use that uses both of them. (although they are different this will not break anything and will save the gas mentioned for a bit of a hit to readability)
- [RdpxV2Core.sol#L79-L82](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L79-L82)


## 3. state variables should be cached in stack variables rather than re-reading them from storage (ones not caught by bot)

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses. 

(NOTE: none of these are in the findings in disputed category and some of them are in the same places but they are valid)

`addresses.tokenA` can be cached because its address type and its value doesnt change and is read from storage twice. caching it will reduce a complex SLOAD to save gas. (although marked as disputed for caching `addresses` alone, caching a entity like `addresses.tokenA` is right and doable to save gas). cache it before #L161
- [UniV2LiquidityAmo.sol#L168](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L168)

same thing as above for `addresses.tokenB`. cache before #L164
- [UniV2LiquidityAmo.sol#L172](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L172)

`addresses.rdpxV2Core` is also being read twice and its a address type and its value doesnt change between reads from storage so it can be cached to save gas. cache before #L168
- [UniV2LiquidityAmo.sol#L173](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L173)

`addresses.tokenA` again this value is a address type and it is not changed inside the `addLiquidity` function, so there is no need to read it 3 times from storage. cache it before #L200 to reduce 2 extra complex SLOAD.
- [UniV2LiquidityAmo.sol#L210](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L210)
- [UniV2LiquidityAmo.sol#L224](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L224)

same as above for `addresses.tokenB` 
- [UniV2LiquidityAmo.sol#L214](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L214)
- [UniV2LiquidityAmo.sol#L225](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L225)

cache `addresses.rdpxV2Core` before #L210 because it is read from storage twice without any change in its value. this will reduce one complex SLOAD
- [UniV2LiquidityAmo.sol#L216](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L216)

same as above for `addresses.ammRouter` but there is 3 reads from storage for this one and there is gonna double gas save for it. cache before #L200
- [UniV2LiquidityAmo.sol#L205](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L205)
- [UniV2LiquidityAmo.sol#L222](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L222)

also in `removeLiquidity` function `addresses.ammRouter` should be cached befoer #L268 to reduce 1 complex SLOAD.
- [UniV2LiquidityAmo.sol#L271](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L271)

same thing in `swap` function for `addresses.ammRouter`. cache it before #L328
- [UniV2LiquidityAmo.sol#L336](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L336)

`positions_array.length` can be cached before #L259 so we can calculate its value in #L268, this is possible because there is no ifs or requires and the changes will happen. reduces one SLOAD
- [UniV3LiquidityAmo.sol#L268](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L268)

we can have a way cheaper version of #L349 if instead of assigning to the `addresses` state variable in #L327 we assign the values to a stack variable and then assign that variable to the `addresses` state varible, with this method we can use the stack variable that we made in line #349 which is gonna be way cheaper
- [RdpxV2Core.sol#L349](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L349)

same thing as above here too
- [PerpetualAtlanticVault.sol#L211](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L211)

same thing here too
- [ReLPContract.sol#L138](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L138)

same as above for `pricingOracleAddresses` in the `setPricingOracleAddresses` function
- [RdpxV2Core.sol#L370](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L370)

`amoAddresses.length` can be cached before #L391 so we can use the cached version in #L392 instead. reduces one SLOAD
- [RdpxV2Core.sol#L392](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L392)

`addresses.dpxEthCurvePool` can be cached to reduce SLOADs by 2 if we cache it before #L521 and use the cached version.
- [RdpxV2Core.sol#L529](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L529)
- [RdpxV2Core.sol#L530](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L530)

`addresses.rdpxDecayingBonds` can be cached because it does not change value and its being read from storage 3 times. cahcing it will reduce 2 complex SLOADs
- [RdpxV2Core.sol#L638](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L638)
- [RdpxV2Core.sol#L643](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L643)

`reserveAsset[reservesIndex["RDPX"]].tokenAddress` can be cached to save lots of gas because its being read from storage 3 times. (also if `reservesIndex["RDPX"]` be cached beforehand this will also save more gas also because that part is being used in #L681 and this will reduce 1 complex SLOAD as well)
- [RdpxV2Core.sol#L657](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L657)
- [RdpxV2Core.sol#L662](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L662)

`addresses.perpetualAtlanticVault` can be cached to reduce one SLOAD. cache it before #L796.
- [RdpxV2Core.sol#L800](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L800)

`delegates.length - 1` can be cached beforehand so we can use the cached version in emit
- [RdpxV2Core.sol#L966](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L966)

`reserveAsset[i].tokenAddress` can be cached at the start of for loop to reduce one complex SLOAD every iteration
- [RdpxV2Core.sol#L1001](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1001)

`addresses.receiptTokenBonds` can be cached to reduce one SLOAD. cache it before #L1025
- [RdpxV2Core.sol#L1032](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1032)

`reservesIndex["RDPX"]` is used both in #L1094 and #L1106 but the value hasnt changed so it can be cached beforehand
- [RdpxV2Core.sol#L1106](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1106)

`addresses.perpetualAtlanticVault` can be cached to reduce 2 SLOADs. cache it before #1189
- [RdpxV2Core.sol#L1193](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1193)
- [RdpxV2Core.sol#L1196](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1196)

`latestFundingPaymentPointer` can be cached before #L303. this will result in one less SLOAD reducing gas cost by 100.
- [PerpetualAtlanticVault.sol#L303](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L303)

`totalActiveOptions` should be cached before the for loop and do the operations we want on that stack variable and after the loop the value be assigned to `totalActiveOptions`. this will result in `totalActiveOptions` not being read every time in loop and also not assigning to it every iteration of the loop
- [PerpetualAtlanticVault.sol#L338](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L338)

`addresses.perpetualAtlanticVaultLP` is being read 5 times from storage without the value changing. so we can cache it before #L347 to reduce 4 SLOADs.
- [PerpetualAtlanticVault.sol#L348](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L348)

`addresses.rdpxV2Core` is being read 5 times from storage without the value changing. so we can cache it before #L347 to reduce 4 SLOADs.
- [PerpetualAtlanticVault.sol#L349](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L349)

`latestFundingPaymentPointer` should be cached because the value is never changed inside the function and its being read from storage 6 times. cache it before #L376 to save 497 gas.
- [PerpetualAtlanticVault.sol#L385-L395](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L385-L395)

`addresses.perpetualAtlanticVaultLP` is read from storage in both #L511 and #L515. cache it before #L510 to reduce one SLOAD
- [PerpetualAtlanticVault.sol#L515](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L515)

`latestFundingPaymentPointer` is read from storage in both #L504 and #L522. cache it before #L504 to reduce one SLOAD
- [PerpetualAtlanticVault.sol#L522](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L522)

caching `latestFundingPaymentPointer` can save up to 297 gas (197 if the if #L595 happens and 297 if the #606 else happens).cache it before the #L595
- [PerpetualAtlanticVault.sol#L603](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L603)

`_totalCollateral` can be cached before the require to most likely save 97 gas. even if it doesnt happen and require reverts we only wasted 3 gas which is completely worth it. for this we should change the += to simple assignment and use cached version on the right side.(changing += to assign saves gas on its part too)
- [PerpetualAtlanticVaultLP.sol#L195](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L195)

same logic as above for these

`_totalCollateral` 
- [PerpetualAtlanticVaultLP.sol#L204](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L204)

`_rdpxCollateral`
- [PerpetualAtlanticVaultLP.sol#L213](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L213)

`addresses.tokenA` is being read from storage 7 times without changing in value , cache it before #L204
- [ReLPContract.sol#L204](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L204)

`addresses.tokenB`is being read from storage 4 times without changing in value , cache it before #L204
- [ReLPContract.sol#L204](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L204)

`addresses.pair` is being read from storage 3 times without changing in value , cache it before #L237
- [ReLPContract.sol#L237](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L237)

`addresses.amo` is being read from storage 3 times without changing in value , cache it before #L243
- [ReLPContract.sol#L244](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L244)

`addresses.ammRouter` is being read from storage 3 times without changing in value , cache it before #L257
- [ReLPContract.sol#L257](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L257)


## 4. should use arguments instead of state variable

This will save 100 gas normally (higher for complex SLOADs) because it gets rid of a storage read and instead uses a argument that we already have which gives the same result

instead of `addresses.perpetualAtlanticVault` used in #L340 we can use the argument we had beforehand `_perpetualAtlanticVault` that we assigned to `addresses.perpetualAtlanticVault` this will stop a SLOAD 
- [RdpxV2Core.sol#L340](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L340)

same thing as above for these 3
instead of `addresses.dopexAMMRouter` use `_dopexAMMRouter`
- [RdpxV2Core.sol#L343](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L343)

instead of `addresses.dpxEthCurvePool` use `_dpxEthCurvePool`
- [RdpxV2Core.sol#L344](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L344)

instead of `addresses.rdpxV2ReceiptToken` use `_rdpxV2ReceiptToken`
- [RdpxV2Core.sol#L346](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L346)

instead of `addresses.perpetualAtlanticVaultLP` use `_perpetualAtlanticVaultLP`
- [PerpetualAtlanticVault.sol#L208](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L208)

use `_rdpx` instead of `rdpx`
- [PerpetualAtlanticVaultLP.sol#L107](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L107)

instead of `addresses.pair` use `_pair`
- [ReLPContract.sol#L150](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L150)

instead of `addresses.tokenA` use `_tokenA`
- [ReLPContract.sol#L155](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L155)

instead of `addresses.tokenB` use `_tokenB`
- [ReLPContract.sol#L160](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L160)

instead of `addresses.ammRouter` use `_ammRouter`
- [ReLPContract.sol#L151-L161](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L151-L161)


## 5. it costs more gas to initialize non-constant/non-immutable variables to zero than to let the default of zero be applied

- [UniV2LiquidityAmo.sol#L147](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L147)

- [UniV3LiquidityAmo.sol#L120](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L120)

- [RdpxV2Core.sol#L167](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L167)
- [RdpxV2Core.sol#L775](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L775)
- [RdpxV2Core.sol#L836](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L836)

- [RdpxDecayingBonds.sol#L103](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L103)

- [PerpetualAtlanticVault.sol#L89](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L89)
- [PerpetualAtlanticVault.sol#L225](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L225)
- [PerpetualAtlanticVault.sol#L328](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L328)
- [PerpetualAtlanticVault.sol#L413](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L413)


## 6. Use assembly to check for address(0)

saves 6 gas per instance

- [UniV2LiquidityAmo.sol#L84](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L84)
- [UniV2LiquidityAmo.sol#L85](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L85)
- [UniV2LiquidityAmo.sol#L86](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L86)
- [UniV2LiquidityAmo.sol#L87](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L87)
- [UniV2LiquidityAmo.sol#L88](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L88)
- [UniV2LiquidityAmo.sol#L89](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L89)
- [UniV2LiquidityAmo.sol#L90](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L90)
- [UniV2LiquidityAmo.sol#L131](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L131)
- [UniV2LiquidityAmo.sol#L132](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L132)

- [RdpxV2Core.sol#L244](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L244)
- [RdpxV2Core.sol#L316](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L316)
- [RdpxV2Core.sol#L317](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L316)
- [RdpxV2Core.sol#L318](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L316)
- [RdpxV2Core.sol#L319](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L316)
- [RdpxV2Core.sol#L320](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L316)
- [RdpxV2Core.sol#L321](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L316)
- [RdpxV2Core.sol#L322](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L316)
- [RdpxV2Core.sol#L323](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L316)
- [RdpxV2Core.sol#L324](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L316)
- [RdpxV2Core.sol#L325](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L316)
- [RdpxV2Core.sol#L362](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L362)
- [RdpxV2Core.sol#L363](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L363)
- [RdpxV2Core.sol#L379](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L379)
- [RdpxV2Core.sol#L408](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L408)
- [RdpxV2Core.sol#L409](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L409)

- [PerpetualAtlanticVault.sol#L119](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L119)
- [PerpetualAtlanticVault.sol#L190](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L190)
- [PerpetualAtlanticVault.sol#L191](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L191)
- [PerpetualAtlanticVault.sol#L192](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L192)
- [PerpetualAtlanticVault.sol#L193](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L193)
- [PerpetualAtlanticVault.sol#L194](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L194)
- [PerpetualAtlanticVault.sol#L195](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L195)
- [PerpetualAtlanticVault.sol#L196](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L196)

- [PerpetualAtlanticVaultLP.sol#L95](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L95)

- [ReLPContract.sol#L127](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L127)
- [ReLPContract.sol#L128](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L128)
- [ReLPContract.sol#L129](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L129)
- [ReLPContract.sol#L130](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L130)
- [ReLPContract.sol#L131](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L131)
- [ReLPContract.sol#L132](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L132)
- [ReLPContract.sol#L133](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L133)
- [ReLPContract.sol#L134](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L134)
- [ReLPContract.sol#L135](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L135)


## 7. state variable should be declared as constant which saves lots of gas

when a state variable doesnt change value from compile time, we can instead create it as a constant to avoid a Gsset 20000 gas and every use of the state variable also will be way cheaper.

`liquiditySlippageTolerance` only saves the 20000 gas for Gsset
- [RdpxV2Core.sol#L103](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L103)

`roundingPrecision` will save the 20000 gas for Gsset and also there is 2 uses so they will be cheaper as well saving 194 gas for that
- [PerpetualAtlanticVault.sol#L104](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L104)


## 8. Use hardcoded address instead address(this)

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry's `script.sol` and solmate's `LibRlp.sol` contracts can help achieve this. - [Reference](https://twitter.com/transmissions11/status/1518507047943245824)

- [UniV2LiquidityAmo.sol#L149](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L149)
- [UniV2LiquidityAmo.sol#L162](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L162)
- [UniV2LiquidityAmo.sol#L165](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L165)
- [UniV2LiquidityAmo.sol#L212](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L212)
- [UniV2LiquidityAmo.sol#L217](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L217)
- [UniV2LiquidityAmo.sol#L230](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L230)
- [UniV2LiquidityAmo.sol#L278](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L278)
- [UniV2LiquidityAmo.sol#L323](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L323)
- [UniV2LiquidityAmo.sol#L341](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L341)
- [UniV2LiquidityAmo.sol#L363](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L363)

- [UniV3LiquidityAmo.sol#L104](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L104)
- [UniV3LiquidityAmo.sol#L160](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L160)
- [UniV3LiquidityAmo.sol#L165](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L165)
- [UniV3LiquidityAmo.sol#L189](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L189)
- [UniV3LiquidityAmo.sol#L285](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L285)
- [UniV3LiquidityAmo.sol#L294](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L294)
- [UniV3LiquidityAmo.sol#L331](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L331)
- [UniV3LiquidityAmo.sol#L354](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L354)
- [UniV3LiquidityAmo.sol#L355](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L355)

- [RdpxV2Core.sol#L169](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L169)
- [RdpxV2Core.sol#L483](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L483)
- [RdpxV2Core.sol#L573](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L573)
- [RdpxV2Core.sol#L654](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L654)
- [RdpxV2Core.sol#L911](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L911)
- [RdpxV2Core.sol#L953](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L953)
- [RdpxV2Core.sol#L998](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L998)
- [RdpxV2Core.sol#L1060](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1060)

- [PerpetualAtlanticVault.sol#L227](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L227)
- [PerpetualAtlanticVault.sol#L289](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L289)
- [PerpetualAtlanticVault.sol#L384](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L384)

- [PerpetualAtlanticVaultLP.sol#L128](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L128)
- [PerpetualAtlanticVaultLP.sol#L192](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L192)
- [PerpetualAtlanticVaultLP.sol#L201](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L201)
- [PerpetualAtlanticVaultLP.sol#L210](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L210)

- [ReLPContract.sol#L245](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L245)
- [ReLPContract.sol#L263](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L263)
- [ReLPContract.sol#L282](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L282)
- [ReLPContract.sol#L293](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L293)
- [ReLPContract.sol#L304](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L304)


## 9. Unbounded Gas Consumption Risk Due to External Call Recipients

In the context of Solidity, external function calls without a specified gas limit present a significant risk. The callee contract has the potential to consume all the gas allocated to the transaction, causing an undesired revert and disrupt the function's execution. To mitigate this, it's recommended to explicitly set a gas limit when performing external calls using `addr.call`{gas: }. This limits the gas forwarded to the callee, preventing potential pitfalls and offering better control over transaction execution.

- [UniV3LiquidityAmo.sol#L344](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L344)

