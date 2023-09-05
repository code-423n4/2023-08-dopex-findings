## Gas Optimizations
| Number |Issue|Instances|
|-|:-|:-:|
| [[G-01](#g-01-structs-can-be-packed-into-fewer-storage-slots)] | `Structs` can be packed into fewer storage slots. | 1 | 
| [[G-02](#g-02-cache-state-variables-outside-of-loop-to-avoid-readingwriting-storage-on-every-iteration)] | Cache `state variables` outside of loop to avoid reading/writing storage on every iteration. | 1 |
| [[G-03](#g-03-using-storage-instead-of-memory-for-structsarrays-saves-gas)] | Using `storage` instead of memory for structs/arrays saves gas. | 3 |
| [[G-04](#g-04-use-calldata-instead-of-memory-for-function-arguments-that-do-not-get-mutated)] | Use `calldata` instead of memory for function arguments that do not get mutated. | 3 |
| [[G-05](#g-05-state-variables-can-be-cached-instead-of-re-reading-them-from-storage)] | State variables can be cached instead of re-reading them from storage. | 17 |
| [[G-06](#g-06-multiple-accesses-of-a-mappingarray-should-use-a-local-variable-cache)] | Multiple accesses of a mapping/array should use a local variable cache. | 5 |
| [[G-07](#g-07-state-variables-only-set-during-construction-should-be-declared-constant)]| State variables only set during construction should be declared `constant`. | 2 |
| [[G-08](#g-08-state-variables-can-be-packed-into-fewer-storage-slots)] | State variables can be `packed` into fewer storage slots. | 3 |
| [[G-09](#g-09-use-constants-instead-of-typeuintxmax)] | Use `constants` instead of type(uintx).max. | 17 |
| [[G-10](#g-10-use-hardcode-address-instead-addressthis)] | Use `hardcode address` instead address(this). | 40 |
| [[G-11](#g-11-do-not-calculate-constants-to-save-gas)] | Do not calculate `constants` to save gas. | 2 |
| [[G-12](#g-12-amounts-should-be-checked-for-0-before-calling-a-transfer)] | Amounts should be checked for 0 before calling a transfer. | 3 |
| [[G-13](#g-13-do-not-initialize-state-variables-with-their-default-value)] | Do not initialize state variables with their default value. | 1 |
| [[G-14](#g-14-use--0-instead-of--0-for-unsigned-integer-comparison-to-save-gas)] | Use `!= 0` instead of > 0 for unsigned integer comparison to save gas. | 19 |
| [[G-15](#g-15-use-delete-for-state-variables)] | Use `delete` for state variables. | 2 |
| [[G-16](#g-16-using-ternary-operator-instead-of-if-else-saves-gas)] | Using `ternary` operator instead of if else saves gas. | 1 |
| [[G-17](#g-17-no-need-to-explicitly-initialize-variables-with-default-values)] | No need to explicitly initialize variables with default values. | 8 |
| [[G-18](#g-18-state-variables-only-set-in-the-constructor-should-be-declared-immutable)] | State variables only set in the constructor should be declared immutable. | 2 |
| [[G-19](#g-19-unnecessary-require-check-while-it-already-checked-using-modifier)] | Unnecessary require check while it already checked using modifier. | 1 |
| [[G-20](#g-20-avoid-emitting-storage-values)] | Avoid emitting storage values. | 2 |


## [G-01] Structs can be packed into fewer storage slots.

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to each other in storage and this will pack the values together into a single 32 byte storage slot (if values combined are <= 32 bytes). If the variables packed together are retrieved together in functions (more likely with structs), we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

**_1 Instance - 1 File_**

### Reduce uint type for `expiry` and pack `owner` and `expiry` into a single storage slot to save 1 SLOT (~2000 gas)

```soldity
File : contracts/decaying-bonds/RdpxDecayingBonds.sol

40: struct Bond {
41:    address owner;
42:    uint256 expiry;
43:    uint256 rdpxAmount;
44:  }

```
[40-44](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L40C2-L44C4)

```diff
File : contracts/decaying-bonds/RdpxDecayingBonds.sol

 // Structure to store the bond information
  struct Bond {
-   address owner;
-   uint256 expiry;
+   address owner;
+   uint96 expiry;///@audit change file code according to it's new size 
    uint256 rdpxAmount;
  }

```


## [G-02] Cache state variables outside of loop to avoid reading/writing storage on every iteration.

Reading from storage should always try to be avoided within loops. In the following instances, we are able to cache state variables outside of the loop to save a Gwarmaccess (100 gas) per loop iteration. In addition, update the storage variable outside the loop to save 1 SSTORE per loop iteration.

**Note: These are instances missed by the Bot Race.**

**_1 Instances - 1 File_**

### Cache `totalActiveOptions` outside of the loop to save 1 SSTORE and 1 SLOAD per iteration, update it's state variable after the loop ended.

```solidity
File : contracts/perp-vault/PerpetualAtlanticVault.sol

328: for (uint256 i = 0; i < optionIds.length; i++) {
329:      uint256 strike = optionPositions[optionIds[i]].strike;
330:      uint256 amount = optionPositions[optionIds[i]].amount;

      // check if strike is ITM
333:      _validate(strike >= getUnderlyingPrice(), 7);

335:      ethAmount += (amount * strike) / 1e8;
336:      rdpxAmount += amount;
337:      optionsPerStrike[strike] -= amount;
338:      totalActiveOptions -= amount;//@audit read and write in stack and update state with final stack value 

      // Burn option tokens from user
341:      _burn(optionIds[i]);

343:      optionPositions[optionIds[i]].strike = 0;
    }
```
[328-343](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L328C3-L343C48)

```diff
File : contracts/perp-vault/PerpetualAtlanticVault.sol

contract PerpetualAtlanticVault is
  ...
{
+ uint256 _totalActiveOptions = totalActiveOptions
 for (uint256 i = 0; i < optionIds.length; i++) {
      uint256 strike = optionPositions[optionIds[i]].strike;
      uint256 amount = optionPositions[optionIds[i]].amount;

      // check if strike is ITM
      _validate(strike >= getUnderlyingPrice(), 7);

      ethAmount += (amount * strike) / 1e8;
      rdpxAmount += amount;
      optionsPerStrike[strike] -= amount;
-     totalActiveOptions -= amount;
+     _totalActiveOptions -= amount;

      // Burn option tokens from user
      _burn(optionIds[i]);

      optionPositions[optionIds[i]].strike = 0;
    }
+  totalActiveOptions = _totalActiveOptions;    
 ...}   
```


## [G-03] Using storage instead of memory for structs/arrays saves gas.

Using a memory pointer for a storage struct/array will effectively load all the fields of that data type from storage (SLOAD) into memory (MSTORE). Using a storage pointer will allow you to read specific fields from storage as you need them. If you are not going to use all of the fields of your data type then you should use a storage pointer so that you don't incur extra **Gcoldsload (2100 gas)** for fields that you will never use.

**Note: These are instances missed by the Bot Race.**

**_3 Instances - 2 Files_**

### Using storage variables can potentially save gas by eliminating the cost of copying the struct/array from storage to memory
```diff
File : contracts/amo/UniV3LiquidityAmo.sol

- 218: Position memory pos = positions_array[positionIndex];
+ 218: Position storage pos = positions_array[positionIndex];

```
[218](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L121)


```diff
File : contracts/core/RdpxV2Core.sol

- 1138: ReserveAsset memory asset = reserveAsset[reservesIndex[_token]];
+ 1138: ReserveAsset storage asset = reserveAsset[reservesIndex[_token]];

- 1272: Delegate memory delegatePosition = delegates[_delegateId];
+ 1272: Delegate storage delegatePosition = delegates[_delegateId];

```
[1138](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1138),[1272](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1272)


## [G-04] Use calldata instead of memory for function arguments that do not get mutated.

When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

**_3 Instances - 1 File_**

```solidity
File : contracts/core/RdpxV2Core.sol

764:  function settle(
765:    uint256[] memory optionIds
766:  )

819:  function bondWithDelegate(
820:    address _to,
821:    uint256[] memory _amounts,
822:    uint256[] memory _delegateIds,
823:    uint256 rdpxBondId


```
[[764-766]](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L764C1-L766C4),[[819-823]](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L819C1-L823C23)

```diff
File : contracts/core/RdpxV2Core.sol

contract RdpxV2Core is 
 ...
{
 ...
   function settle(
-   uint256[] memory optionIds
+   uint256[] calldata optionIds
  )
...
  function bondWithDelegate(
    address _to,
-   uint256[] memory _amounts,
-   uint256[] memory _delegateIds,
+   uint256[] calldata _amounts,
+   uint256[] calldata _delegateIds,
    uint256 rdpxBondId

```

## [G-05] State variables can be cached instead of re-reading them from storage.

Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read.

**Note: These are instances missed by the Bot Race.**

**_17 Instances - 5 Files_**

Estimated Gas Saved: **47 * 100 = 4700** 

### Cache `addresses.rdpxV2Core`,`addresses.tokenA` and `addresses.tokenB` to save 3 SLOAD
```solidity
File : contracts/amo/UniV2LiquidityAmo.sol

160:  function _sendTokensToRdpxV2Core() internal {
161:    uint256 tokenABalance = IERC20WithBurn(addresses.tokenA).balanceOf(
162:      address(this)
163:    );
164:    uint256 tokenBBalance = IERC20WithBurn(addresses.tokenB).balanceOf(
165:      address(this)
166:    );
167:    // transfer token A and B from this contract to the rdpxV2Core
168:    IERC20WithBurn(addresses.tokenA).safeTransfer(
169:      addresses.rdpxV2Core,
170:      tokenABalance
171:    );
172:    IERC20WithBurn(addresses.tokenB).safeTransfer(
173:      addresses.rdpxV2Core,
174:      tokenBBalance
175:    );
176:    emit LogAssetsTransfered(msg.sender, tokenABalance, tokenBBalance);
177:  }

```
[160-178](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L160C1-L178C4)

```diff
File : contracts/amo/UniV2LiquidityAmo.sol

contract UniV2LiquidityAMO is AccessControl {
  ...
     function _sendTokensToRdpxV2Core() internal {

     uint256 tokenABalance = IERC20WithBurn(addresses.tokenA).balanceOf(
      address(this)
     );
     uint256 tokenBBalance = IERC20WithBurn(addresses.tokenB).balanceOf(
      address(this)
     );
+   address cached_ad_rdpxV2Core = addresses.rdpxV2Core;
+   address cached_ad_tokenA = addresses.tokenA;
+   address cached_ad_tokenB = addresses.tokenB;

-   IERC20WithBurn(addresses.tokenA).safeTransfer(
+   IERC20WithBurn(cached_ad_tokenA).safeTransfer(
-     addresses.rdpxV2Core,
+     cached_ad_rdpxV2Core,
      tokenABalance
    );
-   IERC20WithBurn(addresses.tokenB).safeTransfer(
+   IERC20WithBurn(cached_ad_tokenB).safeTransfer(
-     addresses.rdpxV2Core,
+     cached_ad_rdpxV2Core,
      tokenBBalance
    );

    emit LogAssetsTransfered(msg.sender, tokenABalance, tokenBBalance);
  }

...}
```

### Cache `addresses.ammRouter`,`addresses.rdpxV2Core`, `addresses.tokenA` and `addresses.tokenB` to save 7 SLOAD
```solidity
File : contracts/amo/UniV2LiquidityAmo.sol

200: IERC20WithBurn(addresses.tokenA).safeApprove(
201:      addresses.ammRouter,
202:      tokenAAmount
203:    );
204:    IERC20WithBurn(addresses.tokenB).safeApprove(
205:      addresses.ammRouter,
206:      tokenBAmount
207:    );

210:  IERC20WithBurn(addresses.tokenA).safeTransferFrom(
211:      addresses.rdpxV2Core,
212:      address(this),
213:      tokenAAmount
214:    );
215:    IERC20WithBurn(addresses.tokenB).safeTransferFrom(
216:      addresses.rdpxV2Core,
217:      address(this),
218:      tokenBAmount
219:    );

222:  (tokenAUsed, tokenBUsed, lpReceived) = IUniswapV2Router(addresses.ammRouter)
223:   .addLiquidity(
224:        addresses.tokenA,
225:        addresses.tokenB,
226:        tokenAAmount,
227:        tokenBAmount,
228:        tokenAAmountMin,
229:        tokenBAmountMin,
230:        address(this),
231:        block.timestamp + 1
232:      );
```
[200-232](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L200C1-L232C9), 

```diff
File : contracts/amo/UniV2LiquidityAmo.sol

contract UniV2LiquidityAMO is AccessControl {
  ...
   function addLiquidity(...
  )
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 tokenAUsed, uint256 tokenBUsed, uint256 lpReceived)
  {
+   address cached_adammRouter = addresses.ammRouter;
+   address cached_adrdpxV2Core = addresses.rdpxV2Core;

+   address cached_ad_tokenA = addresses.tokenA;
+   address cached_ad_tokenB = addresses.tokenB;

-   IERC20WithBurn(addresses.tokenA).safeApprove(
+   IERC20WithBurn(cached_ad_tokenA).safeApprove(
-     addresses.ammRouter,
+     cached_adammRouter,
      tokenAAmount
    );

-   IERC20WithBurn(addresses.tokenB).safeApprove(
+   IERC20WithBurn(cached_ad_tokenB).safeApprove(
-     addresses.ammRouter,
+     cached_adammRouter,
      tokenBAmount
    );

-   IERC20WithBurn(addresses.tokenA).safeTransferFrom(
+   IERC20WithBurn(cached_ad_tokenA).safeTransferFrom(
-     addresses.rdpxV2Core,
+     cached_adrdpxV2Core,
      address(this),
      tokenAAmount
    );
-   IERC20WithBurn(addresses.tokenB).safeTransferFrom(
+   IERC20WithBurn(cached_ad_tokenB).safeTransferFrom(
-     addresses.rdpxV2Core,
+     cached_adrdpxV2Core,
      address(this),
      tokenBAmount
    );
    ...
-   (tokenAUsed, tokenBUsed, lpReceived) = IUniswapV2Router(addresses.ammRouter)
+   (tokenAUsed, tokenBUsed, lpReceived) = IUniswapV2Router(cached_adammRouter)
      .addLiquidity(
-       addresses.tokenA,
-       addresses.tokenB,
+       cached_ad_tokenA,
+       cached_ad_tokenB,
        tokenAAmount,
        tokenBAmount,
        tokenAAmountMin,
        tokenBAmountMin,
        address(this),
        block.timestamp + 1
      );
    ...}
```

### Cache `addresses.ammRouter` to save 1 SLOAD
```solidity
File : contracts/amo/UniV2LiquidityAmo.sol

268:  IERC20WithBurn(addresses.pair).safeApprove(addresses.ammRouter, lpAmount);
270:    // remove liquidity
271:    (tokenAReceived, tokenBReceived) = IUniswapV2Router(addresses.ammRouter)
```
[268-271](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L268C4-L271C77)

```diff
File : contracts/amo/UniV2LiquidityAmo.sol

contract UniV2LiquidityAMO is AccessControl {
  ...
    function removeLiquidity(...
    {...
+   address cached_adammRouter = addresses.ammRouter;  

-   IERC20WithBurn(addresses.pair).safeApprove(addresses.ammRouter, lpAmount);
+   IERC20WithBurn(addresses.pair).safeApprove(cached_adammRouter, lpAmount);

    // remove liquidity
-   (tokenAReceived, tokenBReceived) = IUniswapV2Router(addresses.ammRouter)
+   (tokenAReceived, tokenBReceived) = IUniswapV2Router(cached_adammRouter)
      .removeLiquidity(
        addresses.tokenA,
        addresses.tokenB,
        lpAmount,
        tokenAAmountMin,
        tokenBAmountMin,
        address(this),
        block.timestamp + 1
      );
     ...} 
...}
```

### Cache `addresses.ammRouter` to save 1 SLOAD
```solidity
File : contracts/amo/UniV2LiquidityAmo.sol

328: IERC20WithBurn(token1).safeApprove(addresses.ammRouter, token1Amount);

336:  token2Amount = IUniswapV2Router(addresses.ammRouter)

```
[328](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L328), [336](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L336)

```diff
contract UniV2LiquidityAMO is AccessControl {
  ...
    function swap(...
    {...
+   address cached_adammRouter = addresses.ammRouter;
   ...
-   IERC20WithBurn(token1).safeApprove(addresses.ammRouter, token1Amount);
+   IERC20WithBurn(token1).safeApprove(cached_adammRouter, token1Amount);
   ...
-   token2Amount = IUniswapV2Router(addresses.ammRouter)
+   token2Amount = IUniswapV2Router(cached_adammRouter)
  ...}
```

### Cache `amoAddresses.length` to save 1 SLOAD
```solidity
File : contracts/core/RdpxV2Core.sol

388:  function removeAMOAddress(
389:    uint256 _index
390:  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
391:    _validate(_index < amoAddresses.length, 18);
392:    amoAddresses[_index] = amoAddresses[amoAddresses.length - 1];
393:    amoAddresses.pop();
394:  }
```
[388-394](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L388C2-L394C4)

```diff
File : contracts/core/RdpxV2Core.sol

contract RdpxV2Core is
  ...
{
  ...
     function removeAMOAddress(
+   address cached_amoAddresses_length = amoAddresses.length      
    uint256 _index
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
-   _validate(_index < amoAddresses.length, 18);
-   amoAddresses[_index] = amoAddresses[amoAddresses.length - 1];
+   _validate(_index < cached_amoAddresses_length, 18);
+   amoAddresses[_index] = amoAddresses[cached_amoAddresses_length - 1];
    amoAddresses.pop();
  }
...}  
```

### Cache `<array>.length` to save 1 SLOAD

```solidity
File : contracts/core/RdpxV2Core.sol

966: emit LogAddToDelegate(_amount, _fee, delegates.length - 1);
967: return (delegates.length - 1);
```
[966-967](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L966C5-L967C35)

### Cache `delegate.activeCollateral` to save 1 SLOAD
```solidity
File : contracts/core/RdpxV2Core.sol

975:   function withdraw(
976:    uint256 delegateId
977:  ) external returns (uint256 amountWithdrawn) {
978:    _whenNotPaused();
979:    _validate(delegateId < delegates.length, 14);
980:    Delegate storage delegate = delegates[delegateId];
981:    _validate(delegate.owner == msg.sender, 9);

983:    amountWithdrawn = delegate.amount - delegate.activeCollateral;
984:    _validate(amountWithdrawn > 0, 15);
985:    delegate.amount = delegate.activeCollateral;

987:    IERC20WithBurn(weth).safeTransfer(msg.sender, amountWithdrawn);

989:    emit LogDelegateWithdraw(delegateId, amountWithdrawn);
990:  }
```
[975-990](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L975C1-L990C4)

```diff
File : contracts/core/RdpxV2Core.sol

contract RdpxV2Core is
  ...
{
  ...
      function withdraw(
    uint256 delegateId
  ) external returns (uint256 amountWithdrawn) {
    ...
+   uint256 cached_delegate_activeCollateral = delegate.activeCollateral;

-   amountWithdrawn = delegate.amount - delegate.activeCollateral;
+   amountWithdrawn = delegate.amount - cached_delegate_activeCollateral;
    _validate(amountWithdrawn > 0, 15);
-   delegate.amount = delegate.activeCollateral;
+   delegate.amount = cached_delegate_activeCollateral;

    IERC20WithBurn(weth).safeTransfer(msg.sender, amountWithdrawn);

    emit LogDelegateWithdraw(delegateId, amountWithdrawn);
     }
  ...}   
```

### Cache `addresses.receiptTokenBonds` to save 1 SLOAD
```solidity
File : contracts/core/RdpxV2Core.sol

1016:  function redeem(
1017:    uint256 id,
1018:    address to
1019:  ) external returns (uint256 receiptTokenAmount) {
        
1025:    _validate(
1026:      msg.sender == RdpxV2Bond(addresses.receiptTokenBonds).ownerOf(id),
1027:      9
1028:    );

1032:    RdpxV2Bond(addresses.receiptTokenBonds).burn(id);
```
[1016-1032](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1016C2-L1032C54)

```diff
File : contracts/core/RdpxV2Core.sol

contract RdpxV2Core is
  ...
{
  ...
     function redeem(
    uint256 id,
    address to
  ) external returns (uint256 receiptTokenAmount) {
     ...
+    address cached_adreceiptTokenBonds = addresses.receiptTokenBonds;

    _validate(
-     msg.sender == RdpxV2Bond(addresses.receiptTokenBonds).ownerOf(id),
+     msg.sender == RdpxV2Bond(cached_adreceiptTokenBonds).ownerOf(id),
      9
    );
   ...
-    RdpxV2Bond(addresses.receiptTokenBonds).burn(id);
+    RdpxV2Bond(cached_adreceiptTokenBonds).burn(id);
    ...}
...}
```

### Cache `addresses.perpetualAtlanticVault` to save 2 SLOAD
```solidity
File : contracts/core/RdpxV2Core.sol

1189:     uint256 strike = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
1190:      .roundUp(rdpxPrice - (rdpxPrice / 4)); // 25% below the current price

1192:    uint256 timeToExpiry = IPerpetualAtlanticVault(
1193:      addresses.perpetualAtlanticVault
1194:    ).nextFundingPaymentTimestamp() - block.timestamp;
1195:    if (putOptionsRequired) {
1196:      wethRequired += IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
1197:        .calculatePremium(strike, rdpxRequired, timeToExpiry, 0);
1198:    }
```
[1189-1198](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1189C1-L1198C6)

```diff
File : contracts/core/RdpxV2Core.sol

contract RdpxV2Core is
  ...
{
  ...
    function calculateBondCost(...
     {...
+      address cached_ad_perpetualAtlanticVault = addresses.perpetualAtlanticVault;

-        uint256 strike = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
+        uint256 strike = IPerpetualAtlanticVault(cached_ad_perpetualAtlanticVault)
         .roundUp(rdpxPrice - (rdpxPrice / 4)); // 25% below the current price

       uint256 timeToExpiry = IPerpetualAtlanticVault(
-         addresses.perpetualAtlanticVault
+         cached_ad_perpetualAtlanticVault
       ).nextFundingPaymentTimestamp() - block.timestamp;
       if (putOptionsRequired) {
-         wethRequired += IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
+         wethRequired += IPerpetualAtlanticVault(cached_ad_perpetualAtlanticVault)
         .calculatePremium(strike, rdpxRequired, timeToExpiry, 0);
    }
  }
...}  
```

### Read this `_perpetualAtlanticVaultLP` from params instead of storage because same value assign in to storage saves 1 SLOAD
```diff
File : contracts/perp-vault/PerpetualAtlanticVaultLP.sol

function setAddresses(
    address _optionPricing,
    address _assetPriceOracle,
    address _volatilityOracle,
    address _feeDistributor,
    address _rdpx,
    address _perpetualAtlanticVaultLP,
    address _rdpxV2Core
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    ...

    addresses = Addresses({
      optionPricing: _optionPricing,
      assetPriceOracle: _assetPriceOracle,
      volatilityOracle: _volatilityOracle,
      feeDistributor: _feeDistributor,
      rdpx: _rdpx,
      perpetualAtlanticVaultLP: _perpetualAtlanticVaultLP,
      rdpxV2Core: _rdpxV2Core
    });
    collateralToken.safeApprove(
-     addresses.perpetualAtlanticVaultLP,
+     _perpetualAtlanticVaultLP,
      type(uint256).max
    );
    emit AddressesSet(addresses);
  }

```
[181-211](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L181C1-L212C4)

### Cache `_totalCollateral + proceeds` to save 1 SLOAD and 1 ADD opcodes
```solidity
File : contracts/perp-vault/PerpetualAtlanticVaultLP.sol

191: require(
192:   collateral.balanceOf(address(this)) >= _totalCollateral + proceeds,//@audit 1st sload
193:     "Not enough collateral token was sent"
194:   );
195:   _totalCollateral += proceeds;//@audit 2nd sload

```
[[191-195]](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L191C4-L195C34)

```diff
File : contracts/perp-vault/PerpetualAtlanticVaultLP.sol

contract PerpetualAtlanticVaultLP is ERC20, IPerpetualAtlanticVaultLP {
  ...
  function addProceeds(uint256 proceeds) public onlyPerpVault {
+   uint256 cached_totalCollateralPlusProceeds = _totalCollateral + proceeds;  
      require(
-       collateral.balanceOf(address(this)) >= _totalCollateral + proceeds,
+       collateral.balanceOf(address(this)) >= cached_totalCollateralPlusProceeds,
        "Not enough collateral token was sent"
     );
-          _totalCollateral += proceeds; 
+          _totalCollateral = cached_totalCollateralPlusProceeds;
    }
 ...}   
```

### Cache `_totalCollateral - loss` to save 1 SLOAD and 1 SUB opcode
```solidity
File : contracts/perp-vault/PerpetualAtlanticVaultLP.sol

200:      require(
201:       collateral.balanceOf(address(this)) == _totalCollateral - loss,
202:       "Not enough collateral was sent out"
203:     );
204:     _totalCollateral -= loss;

```
[[200-204]](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L200C4-L204C30)

```diff
File : contracts/perp-vault/PerpetualAtlanticVaultLP.sol

contract PerpetualAtlanticVaultLP is ERC20, IPerpetualAtlanticVaultLP {
  ...
  function subtractLoss(uint256 loss) public onlyPerpVault {
+    uint256 cached_totalCollateralSubProceeds = _totalCollateral - loss;
     require(
-      collateral.balanceOf(address(this)) == _totalCollateral - loss,
+      collateral.balanceOf(address(this)) == cached_totalCollateralSubProceeds,
       "Not enough collateral was sent out"
     );
-    _totalCollateral -= loss;
+    _totalCollateral  = cached_totalCollateralSubProceeds;
     }
 ...}    
```

### Cache `_rdpxCollateral + amount` to save 1 SLOAD and 1 ADD opcode
```solidity
File : contracts/perp-vault/PerpetualAtlanticVaultLP.sol

209: require(
210:     IERC20WithBurn(rdpx).balanceOf(address(this)) >= _rdpxCollateral + amount,
211:    "Not enough rdpx token was sent"
212:    );
213:   _rdpxCollateral += amount;

```
[[209-213]](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L209C5-L213C31)

```diff
File : contracts/perp-vault/PerpetualAtlanticVaultLP.sol

contract PerpetualAtlanticVaultLP is ERC20, IPerpetualAtlanticVaultLP {
  ...
  function addRdpx(uint256 amount) public onlyPerpVault {
+    uint256 cached__rdpxCollateralPlusAmount =  _rdpxCollateral + amount;
     require(
-      IERC20WithBurn(rdpx).balanceOf(address(this)) >= _rdpxCollateral + amount,
+      IERC20WithBurn(rdpx).balanceOf(address(this)) >= cached__rdpxCollateralPlusAmount,
       "Not enough rdpx token was sent"
    );
-    _rdpxCollateral += amount;
+   _rdpxCollateral = cached__rdpxCollateralPlusAmount;
    }
 ...}   
```

### Cache `latestFundingPaymentPointer` to save 2 SLOAD
```solidity
File : contracts/perp-vault/PerpetualAtlanticVault.sol

302: totalActiveOptions += amount;
303:    fundingPaymentsAccountedFor[latestFundingPaymentPointer] += amount;
304:    optionsPerStrike[strike] += amount;
305:
306:    // record the number of options funding has been accounted for the epoch and strike
307:    fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
308:      strike
309:    ] += amount;
```
[302-309](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L302C1-L309C17)

```diff
File : contracts/perp-vault/PerpetualAtlanticVault.sol

function purchase(
    uint256 amount,
    address to
  )
    external
    nonReentrant
    onlyRole(RDPXV2CORE_ROLE)
    returns (uint256 premium, uint256 tokenId)
  {
    ...
+   uint256 cached_latestFundingPaymentPointer = latestFundingPaymentPointer;

    totalActiveOptions += amount;
-   fundingPaymentsAccountedFor[latestFundingPaymentPointer] += amount;
+   fundingPaymentsAccountedFor[cached_latestFundingPaymentPointer] += amount;
    optionsPerStrike[strike] += amount;

    // record the number of options funding has been accounted for the epoch and strike
-   fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
+   fundingPaymentsAccountedForPerStrike[cached_latestFundingPaymentPointer][
      strike
    ] += amount;
  ...}  

```

### Cache `addresses.rdpxV2Core` and `addresses.perpetualAtlanticVaultLP` to save 5 SLOAD
```solidity
File : contracts/perp-vault/PerpetualAtlanticVault.sol

347:  collateralToken.safeTransferFrom(
348:  addresses.perpetualAtlanticVaultLP,
349:  addresses.rdpxV2Core,
350:   ethAmount
351:  );
352:  // Transfer rdpx from rdpx rdpxV2Core to perpetual vault
353:  IERC20WithBurn(addresses.rdpx).safeTransferFrom(
354:  addresses.rdpxV2Core,
355:  addresses.perpetualAtlanticVaultLP,
356:  rdpxAmount
357:  );
358:
359:  IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).subtractLoss(
360:  ethAmount
361:  );
362:  IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP)
363:  .unlockLiquidity(ethAmount);
364:  IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addRdpx(
365:  rdpxAmount
366:  );
```
[347-366](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L347C2-L366C7)

```diff
File : contracts/perp-vault/PerpetualAtlanticVault.sol

function settle(
    uint256[] memory optionIds
  )
    external
    nonReentrant
    onlyRole(RDPXV2CORE_ROLE)
    returns (uint256 ethAmount, uint256 rdpxAmount)
  {
    ...
+   address cached_ad_perpetualAtlanticVaultLP = addresses.perpetualAtlanticVaultLP;
+   address cached_ad_rdpxV2Core = addresses.rdpxV2Core;

    // Transfer collateral token from perpetual vault to rdpx rdpxV2Core
    collateralToken.safeTransferFrom(
-     addresses.perpetualAtlanticVaultLP,
-     addresses.rdpxV2Core,
+     cached_ad_perpetualAtlanticVaultLP,
+     cached_ad_rdpxV2Core,
      ethAmount
    );
    // Transfer rdpx from rdpx rdpxV2Core to perpetual vault
    IERC20WithBurn(addresses.rdpx).safeTransferFrom(
-     addresses.rdpxV2Core,
-     addresses.perpetualAtlanticVaultLP,
+     cached_ad_rdpxV2Core,
+     cached_ad_perpetualAtlanticVaultLP,
      rdpxAmount
    );

-   IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).subtractLoss(
+   IPerpetualAtlanticVaultLP(cached_ad_perpetualAtlanticVaultLP).subtractLoss(
      ethAmount
    );
-   IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP)
+   IPerpetualAtlanticVaultLP(cached_ad_perpetualAtlanticVaultLP)
      .unlockLiquidity(ethAmount);
-   IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addRdpx(
+   IPerpetualAtlanticVaultLP(cached_ad_perpetualAtlanticVaultLP).addRdpx(
      rdpxAmount
    );

    emit Settle(ethAmount, rdpxAmount, optionIds);
  }

```

### Cache `lastUpdateTime` to save 1 SLOAD
```solidity
File : contracts/perp-vault/PerpetualAtlanticVault.sol

502:  function updateFunding() public {
503:    updateFundingPaymentPointer();
504:    uint256 currentFundingRate = fundingRates[latestFundingPaymentPointer];
505:    uint256 startTime = lastUpdateTime == 0
506:      ? (nextFundingPaymentTimestamp() - fundingDuration)
507:      : lastUpdateTime;

```
[502-507](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L502C1-L507C24)

```diff
File : contracts/perp-vault/PerpetualAtlanticVault.sol

contract PerpetualAtlanticVault is
  ...
{
  function updateFunding() public {
+   uint256 _lastUpdateTime = lastUpdateTime  ;  
     updateFundingPaymentPointer();
     uint256 currentFundingRate = fundingRates[latestFundingPaymentPointer];
-    uint256 startTime = lastUpdateTime == 0
+    uint256 startTime = _lastUpdateTime == 0
       ? (nextFundingPaymentTimestamp() - fundingDuration)
-      : lastUpdateTime;
+      : _lastUpdateTime;
     ...}
  ...}   
```

### Cache `addresses.tokenA` , `addresses.tokenB` , `address.pair` , `addresses.amo` , `liquiditySlippageTolerance` , `address.ammRouter` and `addresses.rdpxV2Core` to save 17 SLOAD
```solidity
File : contracts/reLP/ReLPContract.sol

202: function reLP(uint256 _amount) external onlyRole(RDPXV2CORE_ROLE) {
    // get the pool reserves
    (address tokenASorted, address tokenBSorted) = UniswapV2Library.sortTokens(
      addresses.tokenA,
      addresses.tokenB
    );
    (uint256 reserveA, uint256 reserveB) = UniswapV2Library.getReserves(
      addresses.ammFactory,
      tokenASorted,
      tokenBSorted
    );

    TokenAInfo memory tokenAInfo = TokenAInfo(0, 0, 0);

    // get tokenA reserves
    tokenAInfo.tokenAReserve = IRdpxReserve(addresses.tokenAReserve)
      .rdpxReserve(); // rdpx reserves

    // get rdpx price
    tokenAInfo.tokenAPrice = IRdpxEthOracle(addresses.rdpxOracle)
      .getRdpxPriceInEth();

    tokenAInfo.tokenALpReserve = addresses.tokenA == tokenASorted
      ? reserveA
      : reserveB;

    uint256 baseReLpRatio = (reLPFactor *
      Math.sqrt(tokenAInfo.tokenAReserve) *
      1e2) / (Math.sqrt(1e18)); // 1e6 precision

    uint256 tokenAToRemove = ((((_amount * 4) * 1e18) /
      tokenAInfo.tokenAReserve) *
      tokenAInfo.tokenALpReserve *
      baseReLpRatio) / (1e18 * DEFAULT_PRECISION * 1e2);

    uint256 totalLpSupply = IUniswapV2Pair(addresses.pair).totalSupply();

    uint256 lpToRemove = (tokenAToRemove * totalLpSupply) /
      tokenAInfo.tokenALpReserve;

    // transfer LP tokens from the AMO
    IERC20WithBurn(addresses.pair).transferFrom(
      addresses.amo,
      address(this),
      lpToRemove
    );

    // calculate min amounts to remove
    uint256 mintokenAAmount = tokenAToRemove -
      ((tokenAToRemove * liquiditySlippageTolerance) / 1e8);
    uint256 mintokenBAmount = ((tokenAToRemove * tokenAInfo.tokenAPrice) /
      1e8) -
      ((tokenAToRemove * tokenAInfo.tokenAPrice) * liquiditySlippageTolerance) /
      1e16;

    (, uint256 amountB) = IUniswapV2Router(addresses.ammRouter).removeLiquidity(
      addresses.tokenA,
      addresses.tokenB,
      lpToRemove,
      mintokenAAmount,
      mintokenBAmount,
      address(this),
      block.timestamp + 10
    );

    address[] memory path;
    path = new address[](2);
    path[0] = addresses.tokenB;
    path[1] = addresses.tokenA;

    // calculate min amount of tokenA to be received
    mintokenAAmount =
      (((amountB / 2) * tokenAInfo.tokenAPrice) / 1e8) -
      (((amountB / 2) * tokenAInfo.tokenAPrice * slippageTolerance) / 1e16);

    uint256 tokenAAmountOut = IUniswapV2Router(addresses.ammRouter)
      .swapExactTokensForTokens(
        amountB / 2,
        mintokenAAmount,
        path,
        address(this),
        block.timestamp + 10
      )[path.length - 1];

    (, , uint256 lp) = IUniswapV2Router(addresses.ammRouter).addLiquidity(
      addresses.tokenA,
      addresses.tokenB,
      tokenAAmountOut,
      amountB / 2,
      0,
      0,
      address(this),
      block.timestamp + 10
    );

    // transfer the lp to the amo
    IERC20WithBurn(addresses.pair).safeTransfer(addresses.amo, lp);
    IUniV2LiquidityAmo(addresses.amo).sync();

    // transfer rdpx to rdpxV2Core
    IERC20WithBurn(addresses.tokenA).safeTransfer(
      addresses.rdpxV2Core,
      IERC20WithBurn(addresses.tokenA).balanceOf(address(this))
    );
    IRdpxV2Core(addresses.rdpxV2Core).sync();
307:  }

```
[202-307](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L202C3-L308C1)

```diff
File : contracts/reLP/ReLPContract.sol

function reLP(uint256 _amount) external onlyRole(RDPXV2CORE_ROLE) {
    // get the pool reserves

+   address cached_ad_tokenA = addresses.tokenA ;
+   address cached_ad_tokenB = addresses.tokenB ;   
+   address cached_ad_pair = addresses.pair ;
+   address cached_ad_amo = addresses.amo ;
+   uint256 cached_liquiditySlippageTolerance = liquiditySlippageTolerance ;
+   address cached_ad_ammRouter = addresses.ammRouter ;
+   address cached_ad_rdpxV2Core = addresses.rdpxV2Core ;

    (address tokenASorted, address tokenBSorted) = UniswapV2Library.sortTokens(
-     addresses.tokenA,
+     cached_ad_tokenA,
-     addresses.tokenB
+     cached_ad_tokenB
    );
    (uint256 reserveA, uint256 reserveB) = UniswapV2Library.getReserves(
      addresses.ammFactory,
      tokenASorted,
      tokenBSorted
    );

    TokenAInfo memory tokenAInfo = TokenAInfo(0, 0, 0);

    // get tokenA reserves
    tokenAInfo.tokenAReserve = IRdpxReserve(addresses.tokenAReserve)
      .rdpxReserve(); // rdpx reserves

    // get rdpx price
    tokenAInfo.tokenAPrice = IRdpxEthOracle(addresses.rdpxOracle)
      .getRdpxPriceInEth();

-   tokenAInfo.tokenALpReserve = addresses.tokenA == tokenASorted
+   tokenAInfo.tokenALpReserve = cached_ad_tokenA == tokenASorted
      ? reserveA
      : reserveB;

    uint256 baseReLpRatio = (reLPFactor *
      Math.sqrt(tokenAInfo.tokenAReserve) *
      1e2) / (Math.sqrt(1e18)); // 1e6 precision

    uint256 tokenAToRemove = ((((_amount * 4) * 1e18) /
      tokenAInfo.tokenAReserve) *
      tokenAInfo.tokenALpReserve *
      baseReLpRatio) / (1e18 * DEFAULT_PRECISION * 1e2);

-   uint256 totalLpSupply = IUniswapV2Pair(addresses.pair).totalSupply();
+   uint256 totalLpSupply = IUniswapV2Pair(cached_ad_pair).totalSupply();

    uint256 lpToRemove = (tokenAToRemove * totalLpSupply) /
      tokenAInfo.tokenALpReserve;

    // transfer LP tokens from the AMO
-   IERC20WithBurn(addresses.pair).transferFrom(
+   IERC20WithBurn(cached_ad_pair).transferFrom(
-     addresses.amo,
+     cached_ad_amo,
      address(this),
      lpToRemove
    );

    // calculate min amounts to remove
    uint256 mintokenAAmount = tokenAToRemove -
-     ((tokenAToRemove * liquiditySlippageTolerance) / 1e8);
+     ((tokenAToRemove * cached_liquiditySlippageTolerance) / 1e8);
    uint256 mintokenBAmount = ((tokenAToRemove * tokenAInfo.tokenAPrice) /
      1e8) -
-     ((tokenAToRemove * tokenAInfo.tokenAPrice) * liquiditySlippageTolerance) /
+     ((tokenAToRemove * tokenAInfo.tokenAPrice) * cached_liquiditySlippageTolerance) /
      1e16;

-    (, uint256 amountB) = IUniswapV2Router(addresses.ammRouter).removeLiquidity(
+    (, uint256 amountB) = IUniswapV2Router(cached_ad_ammRouter).removeLiquidity(
-     addresses.tokenA,
+     cached_ad_tokenA,
-     addresses.tokenB,
+     cached_ad_tokenB,
      lpToRemove,
      mintokenAAmount,
      mintokenBAmount,
      address(this),
      block.timestamp + 10
    );

    address[] memory path;
    path = new address[](2);
-   path[0] = addresses.tokenB;
+   path[0] = cached_ad_tokenB;
-   path[1] = addresses.tokenA;
+   path[1] = cached_ad_tokenA;

    // calculate min amount of tokenA to be received
    mintokenAAmount =
      (((amountB / 2) * tokenAInfo.tokenAPrice) / 1e8) -
      (((amountB / 2) * tokenAInfo.tokenAPrice * slippageTolerance) / 1e16);

-   uint256 tokenAAmountOut = IUniswapV2Router(addresses.ammRouter)
+   uint256 tokenAAmountOut = IUniswapV2Router(cached_ad_ammRouter)
      .swapExactTokensForTokens(
        amountB / 2,
        mintokenAAmount,
        path,
        address(this),
        block.timestamp + 10
      )[path.length - 1];

-   (, , uint256 lp) = IUniswapV2Router(addresses.ammRouter).addLiquidity(
+   (, , uint256 lp) = IUniswapV2Router(cached_ad_ammRouter).addLiquidity(
-     addresses.tokenA,
+     cached_ad_tokenA,
-     addresses.tokenB,
+     cached_ad_tokenB,
      tokenAAmountOut,
      amountB / 2,
      0,
      0,
      address(this),
      block.timestamp + 10
    );

    // transfer the lp to the amo
-   IERC20WithBurn(addresses.pair).safeTransfer(addresses.amo, lp);
+   IERC20WithBurn(cached_ad_pair).safeTransfer(cached_ad_amo, lp);

-   IUniV2LiquidityAmo(addresses.amo).sync();
+   IUniV2LiquidityAmo(cached_ad_amo).sync();

    // transfer rdpx to rdpxV2Core
-   IERC20WithBurn(addresses.tokenA).safeTransfer(
+   IERC20WithBurn(cached_ad_tokenA).safeTransfer(
-      addresses.rdpxV2Core,
+      cached_ad_rdpxV2Core,
-     IERC20WithBurn(addresses.tokenA).balanceOf(address(this))
+     IERC20WithBurn(cached_ad_tokenA).balanceOf(address(this))
    );
-   IRdpxV2Core(addresses.rdpxV2Core).sync();
+   IRdpxV2Core(cached_ad_rdpxV2Core).sync();
  }

```

## [G-06] Multiple accesses of a mapping/array should use a local variable cache.

Caching a mapping's value in a storage pointer when the value is accessed multiple times saves ~40 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it.
To achieve this, declare a storage pointer for the variable and use it instead of repeatedly fetching the reference in a map or an array. As an example, instead of repeatedly calling stakes[tokenId_], save its reference via a storage pointer: StakeInfo storage stakeInfo = stakes[tokenId_] and use the pointer instead.

**Note: These are instances missed by the Bot Race.**

**_4 instances - 2 Files_**

### Use already cached `IStableSwap(addresses.dpxEthCurvePool)` 
```solidity
File : contracts/core/RdpxV2Core.sol

529: uint256 ethBalance = IStableSwap(addresses.dpxEthCurvePool).balances(a);
530:   uint256 dpxEthBalance = IStableSwap(addresses.dpxEthCurvePool).balances(
531:       b
532:      );
```
[529-532](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L529C6-L532C9)

### Cache `reserveAsset[reservesIndex["RDPX"]]` 
```solidity
File : contracts/core/RdpxV2Core.sol

653: IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress)
654:        .safeTransferFrom(msg.sender, address(this), _rdpxAmount);
655:
656:      // burn the rdpx
657:      IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress).burn(
658:        (_rdpxAmount * rdpxBurnPercentage) / 1e10
659:      );
660:
661:      // transfer the rdpx to the fee distributor
662:      IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress)
663:        .safeTransfer(
664:          addresses.feeDistributor,
665:          (_rdpxAmount * rdpxFeePercentage) / 1e10
666:        );

```
[653-666](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L653C6-L666C11)


### Cache `totalFundingForEpoch[latestFundingPaymentPointer]` and `latestFundingPaymentPointer` to save 8 SLOAD
```solidity
File : contracts/perp-vault/PerpetualAtlanticVault.sol

376: _validate(
377:    totalActiveOptions ==
378:    fundingPaymentsAccountedFor[latestFundingPaymentPointer],
379:    6
380:   );
381:
382:  collateralToken.safeTransferFrom(
383:    addresses.rdpxV2Core,
384:    address(this),
385:  totalFundingForEpoch[latestFundingPaymentPointer] 
386:   );
387:  _updateFundingRate(totalFundingForEpoch[latestFundingPaymentPointer]);
388:
389:   emit PayFunding(
390:     msg.sender,
391:     totalFundingForEpoch[latestFundingPaymentPointer],
392:     latestFundingPaymentPointer
393:    );
395:   return (totalFundingForEpoch[latestFundingPaymentPointer]);

```
[376-395](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L376C4-L395C64)

```diff
File : contracts/perp-vault/PerpetualAtlanticVault.sol

function payFunding() external onlyRole(RDPXV2CORE_ROLE) returns (uint256) {
    _whenNotPaused();
    _isEligibleSender();

+   uint256 cached_latestFundingPaymentPointer = latestFundingPaymentPointer;
+   uint256 cached_totalFundingForEpoch = totalFundingForEpoch[cached_latestFundingPaymentPointer];

    _validate(
      totalActiveOptions ==
-       fundingPaymentsAccountedFor[latestFundingPaymentPointer],
+       fundingPaymentsAccountedFor[cached_latestFundingPaymentPointer],
      6
    );

    collateralToken.safeTransferFrom(
      addresses.rdpxV2Core,
      address(this),
-   totalFundingForEpoch[latestFundingPaymentPointer]
+   cached_totalFundingForEpoch
    );
-   _updateFundingRate(totalFundingForEpoch[latestFundingPaymentPointer]);
+   _updateFundingRate(cached_totalFundingForEpoch);

    emit PayFunding(
      msg.sender,
-     totalFundingForEpoch[latestFundingPaymentPointer],
+     cached_totalFundingForEpoch,
-     latestFundingPaymentPointer
+     cached_latestFundingPaymentPointer
    );

-   return (totalFundingForEpoch[latestFundingPaymentPointer]);
+   return (cached_totalFundingForEpoch);
  }

```


### Cache `fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][strike]` , `optionsPerStrike[strikes[i]]` and `latestFundingPaymentPointer` can also be cached to save 6 SLOAD

**Note: Here we only include those instances which are missed by Bot race . In bot report `latestFundingPaymentPointer` only catch at 3 Places in this function.**

```solidity
File : contracts/perp-vault/PerpetualAtlanticVault.sol

413: for (uint256 i = 0; i < strikes.length; i++) {
414:      _validate(optionsPerStrike[strikes[i]] > 0, 4);
415:      _validate(
416:        latestFundingPerStrike[strikes[i]] != latestFundingPaymentPointer,
417:        5
418:      );
419:      uint256 strike = strikes[i];
420:
421:     uint256 amount = optionsPerStrike[strike] -
422:    fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][strike];
423:
424:      uint256 timeToExpiry = nextFundingPaymentTimestamp() -
425:        (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration));
426:
427:      uint256 premium = calculatePremium(
428:        strike,
429:        amount,
430:        timeToExpiry,
431:        getUnderlyingPrice()
      );

      latestFundingPerStrike[strike] = latestFundingPaymentPointer;
      fundingAmount += premium;
      fundingPaymentsAccountedFor[latestFundingPaymentPointer] += amount;
443:      fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
444:        strike 
445:      ] += amount;
449:       totalFundingForEpoch[latestFundingPaymentPointer][strike]
```
[413-449](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L413C1-L450C1)

```diff
File : contracts/perp-vault/PerpetualAtlanticVault.sol

 function calculateFunding(
    uint256[] memory strikes
  ) external nonReentrant returns (uint256 fundingAmount) {
    _whenNotPaused();
    _isEligibleSender();

    updateFundingPaymentPointer();

    for (uint256 i = 0; i < strikes.length; i++) {

+  uint256 cached_latestFundingPaymentPointer = latestFundingPaymentPointer;      
+  uint256 cached_optionsPerStrike = optionsPerStrike[strikes[i]];
+  uint256 cached_fundingPaymentsAccountedForPerStrike = fundingPaymentsAccountedForPerStrike[cached_latestFundingPaymentPointer][strike]

-     _validate(optionsPerStrike[strikes[i]] > 0, 4);
+     _validate(cached_optionsPerStrike > 0, 4);
      _validate(
        latestFundingPerStrike[strikes[i]] != latestFundingPaymentPointer,///@audit Included in bot report
        5
      );
      uint256 strike = strikes[i];

-     uint256 amount = optionsPerStrike[strike] -
+     uint256 amount = cached_optionsPerStrike -
-        fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][strike];
+        cached_fundingPaymentsAccountedForPerStrike;

      uint256 timeToExpiry = nextFundingPaymentTimestamp() -
        (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration));///@audit Included in bot report

      uint256 premium = calculatePremium(
        strike,
        amount,
        timeToExpiry,
        getUnderlyingPrice()
      );

-     latestFundingPerStrike[strike] = latestFundingPaymentPointer;
+     latestFundingPerStrike[strike] = cached_latestFundingPaymentPointer;
      fundingAmount += premium;

      // Record number of options that funding payments were accounted for, for this epoch
-     fundingPaymentsAccountedFor[latestFundingPaymentPointer] += amount;
+     fundingPaymentsAccountedFor[cached_latestFundingPaymentPointer] += amount;

      // record the number of options funding has been accounted for the epoch and strike
-      fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][strike] += amount;
+      fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][strike] = cached_fundingPaymentsAccountedForPerStrike + amount;

      // Record total funding for this epoch
      // This does not need to be done in purchase() since it's already accounted for using `addProceeds()`
      totalFundingForEpoch[latestFundingPaymentPointer] += premium;

      emit CalculateFunding(
        msg.sender,
        amount,
        strike,
        premium,
        latestFundingPaymentPointer///@audit Included in bot report
      );
    }
  }

```


## [G-07] State variables only set during construction should be declared constant.

The solidity compiler will directly embed the values of constant variables into your contract bytecode, and therefore, will save you from incurring a `Gsset (20000 gas)` when you set storage variables during construction; a `Gcoldsload (2100 gas)` when you access storage variables for the first time in a transaction, and a `Gwarmaccess (100 gas)` for each subsequent access to that storage slot.

**Note: These are instances missed by the Bot Race.**

**_1 Instance - 1 File_**

### Make `roundingPrecision` constant
```solidity
File : contracts/perp-vault/PerpetualAtlanticVault.sol

104 :  uint256 public roundingPrecision = 1e6;
```
[104](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L104C1-L104C42)

```diff
File : contracts/perp-vault/PerpetualAtlanticVault.sol

-104 :  uint256 public roundingPrecision = 1e6;
+104 :  uint256 public constant roundingPrecision = 1e6;
```


## [G-08] State variables can be packed into fewer storage slots.

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions, we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper

**Note: These are instances missed by the Bot Race.**

**_3 Instances - 3 Files_**

Estimated Gas Saved: **8 * 2000 = 16000**

### Pack `rdpxBurnPercentage`,`rdpxFeePercentage`,`slippageTolerance`, `liquiditySlippageTolerance` , `bondMaturity` and `bondDiscountFactor` to save 5 SLOT (~10000 Gas)

```solidity
File : contracts/core/RdpxV2Core.sol

 94:  uint256 public rdpxBurnPercentage = 50 * DEFAULT_PRECISION;

 96:  /// @notice The % of rdpx sent to fee distributor while bonding
 97:  uint256 public rdpxFeePercentage = 50 * DEFAULT_PRECISION;

 99:  /// @notice The slippage tolernce in swaps in 1e8 precision
100:  uint256 public slippageTolerance = 5e5; // 0.5%

102:  /// @notice Liquidity slippage tolerance
103:  uint256 public liquiditySlippageTolerance = 5e5; // 0.5%

106:  uint256 public bondMaturity;

108:  /// @notice rDPX LP bond discount factor
109:  uint256 public bondDiscountFactor;

```
[94-109](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L94C3-L109C37)

```diff
File : contracts/core/RdpxV2Core.sol

contract RdpxV2Core is
    IRdpxV2Core,
    AccessControl,
    ContractWhitelist,
    ERC721Holder,
    Pausable
{...

uint256 public rdpxBurnPercentage = 50 * DEFAULT_PRECISION;

  /// @notice The % of rdpx sent to fee distributor while bonding
- uint256 public rdpxFeePercentage = 50 * DEFAULT_PRECISION;

  /// @notice The slippage tolernce in swaps in 1e8 precision
- uint256 public slippageTolerance = 5e5; // 0.5%

  /// @notice Liquidity slippage tolerance
- uint256 public liquiditySlippageTolerance = 5e5; // 0.5%

  /// @notice Bond maturity
- uint256 public bondMaturity;

  /// @notice rDPX LP bond discount factor
- uint256 public bondDiscountFactor;

+ uint64 public rdpxFeePercentage = 50 * DEFAULT_PRECISION;
+ uint64 public slippageTolerance = 5e5; // 0.5%
+ uint32 public liquiditySlippageTolerance = 5e5; // 0.5%
+ uint32 public bondMaturity;
+ uint32 public bondDiscountFactor;

 ...}

```

### Pack the `uint256 public latestFundingPaymentPointer`,` uint256 public lastUpdateTime` and `uint256 public fundingDuration `into one storage slot to save 2 SLOT (~4000 Gas)

Note:  `lastUpdateTime` hold timestamp, and therefore, a unit type of `uint96` should be big enough to hold those values. Use a smaller data type like `uint64` to store the number of seconds in 7 days (604,800 seconds) and reduce storage costs. `latestFundingPaymentPointer` is holding number of durations passed after genesis timestamp so `uint96` will be much more sufficient to hold this pointer.

```solidity
File : contracts/perp-vault/PerpetualAtlanticVault.sol

89:  uint256 public latestFundingPaymentPointer = 0;
90:
91:  /// @dev the total number of active options
92:  uint256 public totalActiveOptions;
93:
94:  /// @dev genesis timestamp
95:  uint256 public genesis;
96:
97:  /// @dev the timestamp of the last update where funding was paid for
98:  uint256 public lastUpdateTime;
99:
100:  /// @dev the duration between funding payments
101:  uint256 public fundingDuration = 7 days;
102:
103: /// @dev the precision to round up to
104:  uint256 public roundingPrecision = 1e6;

```
[89-104](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L89C1-L104C42)

```diff
File : contracts/perp-vault/PerpetualAtlanticVault.sol

contract PerpetualAtlanticVault is
 ...
{

- uint256 public latestFundingPaymentPointer = 0;

  /// @dev the total number of active options
  uint256 public totalActiveOptions;

  /// @dev genesis timestamp
  uint256 public genesis;

  /// @dev the timestamp of the last update where funding was paid for
- uint256 public lastUpdateTime;

  /// @dev the duration between funding payments
- uint256 public fundingDuration = 7 days;

  /// @dev the precision to round up to
  uint256 public roundingPrecision = 1e6;

+ uint96 public latestFundingPaymentPointer = 0;
+ uint96 public lastUpdateTime;
+ uint64 public fundingDuration = 7 days

  ...
      function updateFundingDuration(
    uint256 _fundingDuration
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
-    fundingDuration = _fundingDuration;
+    fundingDuration =uint64(_fundingDuration);
  }

...
   function updateFundingPaymentPointer() public {
    while (block.timestamp >= nextFundingPaymentTimestamp()) {
      if (lastUpdateTime < nextFundingPaymentTimestamp()) {
        uint256 currentFundingRate = fundingRates[latestFundingPaymentPointer];

        uint256 startTime = lastUpdateTime == 0
          ? (nextFundingPaymentTimestamp() - fundingDuration)
          : lastUpdateTime;

-        lastUpdateTime = nextFundingPaymentTimestamp();
+        lastUpdateTime = uint96(nextFundingPaymentTimestamp());

        collateralToken.safeTransfer(
          addresses.perpetualAtlanticVaultLP,
          (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
            1e18
        );
    ...
        }

      latestFundingPaymentPointer += 1;
      emit FundingPaymentPointerUpdated(latestFundingPaymentPointer);
    }
  }

  ...
   function updateFunding() public {
    updateFundingPaymentPointer();
    uint256 currentFundingRate = fundingRates[latestFundingPaymentPointer];
    uint256 startTime = lastUpdateTime == 0
      ? (nextFundingPaymentTimestamp() - fundingDuration)
      : lastUpdateTime;
-    lastUpdateTime = block.timestamp;
+    lastUpdateTime = uint96(block.timestamp);

    ...}



...}    

```

### Pack the `uint256 public liquiditySlippageTolerance` and `uint256 public slippageTolerance` into one storage slot to save 1 SLOT (~2000 Gas)

These can be packed because for make them 100 percent 1e8 Max can be there so they can easily come into uint128 easily. More then 100 slippageTolerance is in practical. 

```solidity
File : contracts/reLP/ReLPContract.sol 

73:  uint256 public liquiditySlippageTolerance = 5e5;
76:  uint256 public slippageTolerance = 5e5;

```
[73-76](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L73C2-L76C50)

```diff
File : contracts/reLP/ReLPContract.sol 

contract ReLPContract is AccessControl {

  using SafeERC20 for IERC20WithBurn;
  using SafeMath for uint256;

- 73:  uint256 public liquiditySlippageTolerance = 5e5; // 0.5%
- 76:  uint256 public slippageTolerance = 5e5; // 0.5%
+ 73:  uint128 public liquiditySlippageTolerance = 5e5; // 0.5%
+ 76:  uint128 public slippageTolerance = 5e5; // 0.5%

...

 function setLiquiditySlippageTolerance(
    uint256 _liquiditySlippageTolerance
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(
      _liquiditySlippageTolerance > 0,
      "reLPContract: liquidity slippage tolerance must be greater than 0"
    );
-    liquiditySlippageTolerance = _liquiditySlippageTolerance;
+    liquiditySlippageTolerance = uint128(_liquiditySlippageTolerance);
  }

   function setSlippageTolerance(
    uint256 _slippageTolerance
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(
      _slippageTolerance > 0,
      "reLPContract: slippage tolerance must be greater than 0"
    );
-    slippageTolerance = _slippageTolerance;
+    slippageTolerance = uint128(_slippageTolerance);
  }

```



## [G-09] Use constants instead of type(uintx).max.

In Solidity, type(uintX).max is used to represent the maximum value for an unsigned integer of X bits. This value is often used in contracts to represent an "infinity" value or indicate that a certain condition has not been met. However, using type(uintX).max can be expensive in terms of gas cost since it requires the compiler to perform a complex computation to determine the maximum value for an unsigned integer of X bits. 

**_17 Instances - 5 Files_**

### To save gas , it's recommended to use `constants` instead of `type(uint128).max` to represent infinity or other large values.

```solidity
File : contracts/amo/UniV3LiquidityAmo.sol

126: type(uint128).max,
127: type(uint128).max

223: type(uint128).max,
224: type(uint128).max

```
[126-127](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L126C11-L127C28),[223-224](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L223C9-L224C26)

```diff
File : contracts/amo/UniV3LiquidityAmo.sol

contract UniV3LiquidityAMO is AccessControl, ERC721Holder {
 ...
+   uint128 constant MAX_UINT128 = 2**128 - 1;
 ...
    function collectFees() external onlyRole(DEFAULT_ADMIN_ROLE) {...
      INonfungiblePositionManager.CollectParams
        memory collect_params = INonfungiblePositionManager.CollectParams(...
-         type(uint128).max,
-         type(uint128).max
+         MAX_UINT128,
+         MAX_UINT128
        );
     ...}

 ...  
    function removeLiquidity(...
    {...
    INonfungiblePositionManager.CollectParams
      memory collect_params = INonfungiblePositionManager.CollectParams(...
-       type(uint128).max,
-       type(uint128).max
+       MAX_UINT128,
+       MAX_UINT128
      );
    ...}   
    

```

### To save gas , it's recommended to use `constants` instead of `type(uint256).max` to represent infinity or other large values.

```solidity
File : contracts/amo/UniV3LiquidityAmo.sol

190: type(uint256).max

```
[190](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L190)

```diff
File : contracts/amo/UniV3LiquidityAmo.sol

contract UniV3LiquidityAMO is AccessControl, ERC721Holder {
 ...
+   uint256 constant MAX_UINT256 = 2**256 - 1;
 ...
   function addLiquidity(...
    {...
      INonfungiblePositionManager.MintParams
        memory mintParams = INonfungiblePositionManager.MintParams(...
-         type(uint256).max
+         MAX_UINT256
        );
     ...}   

```

```solidity
File : contracts/core/RdpxV2Core.sol

339:  IERC20WithBurn(weth).approve(
340:    addresses.perpetualAtlanticVault,
341:    type(uint256).max
342:  );

343:  IERC20WithBurn(weth).approve(addresses.dopexAMMRouter, type(uint256).max);

344:  IERC20WithBurn(weth).approve(addresses.dpxEthCurvePool, type(uint256).max);

345:  IERC20WithBurn(weth).approve(
346:    addresses.rdpxV2ReceiptToken,
347:    type(uint256).max
348:  );
```
[339-348](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L339C1-L348C7)

```diff
File : contracts/core/RdpxV2Core.sol

contract RdpxV2Core is 
 ...
{
+   uint256 constant MAX_UINT256 = 2**256 - 1;
  ...
   function setAddresses(
    ...
     {...
    IERC20WithBurn(weth).approve(
      addresses.perpetualAtlanticVault,
-     type(uint256).max
+     MAX_UINT256
    );

-   IERC20WithBurn(weth).approve(addresses.dopexAMMRouter, type(uint256).max);
+   IERC20WithBurn(weth).approve(addresses.dopexAMMRouter, MAX_UINT256);

-   IERC20WithBurn(weth).approve(addresses.dpxEthCurvePool, type(uint256).max);
+   IERC20WithBurn(weth).approve(addresses.dpxEthCurvePool, MAX_UINT256);

    IERC20WithBurn(weth).approve(
      addresses.rdpxV2ReceiptToken,
-     type(uint256).max
+     MAX_UINT256
    );
    ...}
  ...  
}
```

```solidity
File : contracts/perp-vault/PerpetualAtlanticVaultLP.sol

106: collateral.approve(_perpetualAtlanticVault, type(uint256).max);
107: ERC20(rdpx).approve(_perpetualAtlanticVault, type(uint256).max);

155: if (allowed != type(uint256).max) {

```
[[106-107]](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L106C1-L107C69),[155](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L155)

```diff
File : contracts/perp-vault/PerpetualAtlanticVaultLP.sol

contract PerpetualAtlanticVaultLP is ERC20, IPerpetualAtlanticVaultLP {
  ...
+   uint256 constant MAX_UINT256 = 2**256 - 1;
  ...
    constructor(...
    ){ ...

-     collateral.approve(_perpetualAtlanticVault, type(uint256).max);
+     collateral.approve(_perpetualAtlanticVault, MAX_UINT256);
-     ERC20(rdpx).approve(_perpetualAtlanticVault, type(uint256).max);
+     ERC20(rdpx).approve(_perpetualAtlanticVault, MAX_UINT256);

   ...

    function redeem(
    ...
  ) public returns (uint256 assets, uint256 rdpxAmount) {
    perpetualAtlanticVault.updateFunding();

    if (msg.sender != owner) {
      uint256 allowed = allowance[owner][msg.sender]; // Saves gas for limited approvals.

-      if (allowed != type(uint256).max) 
+      if (allowed != MAX_UINT256) 
       } 
     }
    }   
}
```

```solidity
File : contracts/perp-vault/PerpetualAtlanticVault.sol

209:   type(uint256).max

247:   type(uint256).max
```
[209](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L209C5-L209C24),[247](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L247)


```diff
File : contracts/perp-vault/PerpetualAtlanticVault.sol

contract PerpetualAtlanticVault is ...
{...
+      uint256 constant MAX_UINT256 = 2**256 - 1;
   ...
      function setAddresses(...) external onlyRole(DEFAULT_ADMIN_ROLE) {
  
    collateralToken.safeApprove(
      addresses.perpetualAtlanticVaultLP,
-      type(uint256).max
+      MAX_UINT256
    );
    emit AddressesSet(addresses);
  }
  ...
     function setLpAllowance(bool increase) external onlyRole(DEFAULT_ADMIN_ROLE) {
    increase
      ? collateralToken.approve(
        addresses.perpetualAtlanticVaultLP,
-        type(uint256).max
+        MAX_UINT256
      )
      : collateralToken.approve(addresses.perpetualAtlanticVaultLP, 0);
  }
}

```
 
```solidity
File : contracts/reLP/ReLPContract.sol

152: type(uint256).max

157: type(uint256).max

162: type(uint256).max

```
[152](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L152C6-L152C24),[157](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L157C7-L157C24),[162](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L162C4-L162C24)

`For example, we can define a constant MAX_UINT256 that represents the maximum value for an unsigned 256-bit integer:`

```diff
File : contracts/reLP/ReLPContract.sol 


contract ReLPContract is AccessControl 
{...
+    uint256 constant MAX_UINT256 = 2**256 - 1;
 ...

  function setAddresses(...)
  { ...

    IERC20WithBurn(addresses.pair).safeApprove(
      addresses.ammRouter,
-      type(uint256).max
+       MAX_UINT256
    );

    IERC20WithBurn(addresses.tokenA).safeApprove(
      addresses.ammRouter,
-      type(uint256).max
+       MAX_UINT256
    );

    IERC20WithBurn(addresses.tokenB).safeApprove(
      addresses.ammRouter,
-      type(uint256).max
+       MAX_UINT256
    );
  }
  ...}

```


## [G-10] Use hardcode address instead address(this).

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this. In Solidity, the address(this) expression returns the address of the current contract instance. This expression is commonly used to send or receive Ether to or from the contract.
Using a hardcoded address instead of the address(this) expression can be more gas-efficient when sending or receiving Ether to or from the contract. This is because using the address(this) expression requires additional gas to be consumed to retrieve the address of the current contract, while using a hardcoded address does not.

**_40 Instances - 6 Files_**

### Use `hardcode` address
```solidity
File : contracts/amo/UniV2LiquidityAmo.sol

149: token.safeTransfer(msg.sender, token.balanceOf(address(this)));

162: address(this)

165: address(this)

212: address(this),

217: address(this),

230: address(this),

278: address(this),

323: address(this),

341: address(this),

363: lpTokenBalance = IERC20WithBurn(addresses.pair).balanceOf(address(this));
```
[149](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L149), [162](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L162), [165](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L165), [212](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L212), [217](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L217), [230](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L230), [278](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L278), [323](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L323), [341](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L341), [363](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L363)

```diff
File : contracts/amo/UniV2LiquidityAmo.sol

contract UniV2LiquidityAMO is AccessControl {...
 ...
+  address public myAddress = 0x1936464994964687447870; 
   function emergencyWithdraw(...
   {...
    for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
-     token.safeTransfer(msg.sender, token.balanceOf(address(this)));
+     token.safeTransfer(msg.sender, token.balanceOf(myAddress));
    }
    ...}

 ...
   function _sendTokensToRdpxV2Core() internal {
    uint256 tokenABalance = IERC20WithBurn(addresses.tokenA).balanceOf(
-     address(this)
+     myAddress
    );
    uint256 tokenBBalance = IERC20WithBurn(addresses.tokenB).balanceOf(
-     address(this)
+     myAddress
    );
    ...}

 ...
   function addLiquidity(...
   {...
    IERC20WithBurn(addresses.tokenA).safeTransferFrom(
      addresses.rdpxV2Core,
-     address(this),
+     myAddress,
      tokenAAmount
    );
    IERC20WithBurn(addresses.tokenB).safeTransferFrom(
      addresses.rdpxV2Core,
-     address(this),
+     myAddress,
      tokenBAmount
    );

    (tokenAUsed, tokenBUsed, lpReceived) = IUniswapV2Router(addresses.ammRouter)
      .addLiquidity(...
-       address(this),
+       myAddress,
        block.timestamp + 1
      );
      ...} 

 ...
   function removeLiquidity(...
     {...
    (tokenAReceived, tokenBReceived) = IUniswapV2Router(addresses.ammRouter)
      .removeLiquidity(...
-       address(this),
+       myAddress,
        block.timestamp + 1
      );  
    ...}

 ...
   function swap(...
   {...
    IERC20WithBurn(token1).safeTransferFrom(
      addresses.rdpxV2Core,
-     address(this),
+     myAddress,
      token1Amount
    );
     ...
    token2Amount = IUniswapV2Router(addresses.ammRouter)
      .swapExactTokensForTokens(...
-       address(this),
+       myAddress,
        block.timestamp + 1
      )[path.length - 1];
    ...}

 ...
     function sync() external {
-   lpTokenBalance = IERC20WithBurn(addresses.pair).balanceOf(address(this));
+   lpTokenBalance = IERC20WithBurn(addresses.pair).balanceOf(myAddress);
     }   
             
```


### Use `hardcode` address
```solidity
File : contracts/amo/UniV3LiquidityAmo.sol

106: keccak256(abi.encodePacked(address(this), _tickLower, _tickUpper))

160: address(this),

165: address(this),

189: address(this),

285: address(this),

294: address(this),

331: address(this),

354:  uint256 tokenABalance = IERC20WithBurn(tokenA).balanceOf(address(this));
355:  uint256 tokenBBalance = IERC20WithBurn(tokenB).balanceOf(address(this));

```
[106](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L106),[160](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L160),[165](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L165),[189](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L189),[285](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L285),[294](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L294),[331](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L331),[354-355](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L354C4-L355C77)

```diff
File : contracts/amo/UniV3LiquidityAmo.sol

contract UniV3LiquidityAMO is AccessControl, ERC721Holder {
  ...
+  address public myAddress = 0x1938523456789012345678901234447870;
 ...
     function liquidityInPool(...
     {...
    (uint128 liquidity, , , , ) = get_pool.positions(
-     keccak256(abi.encodePacked(address(this), _tickLower, _tickUpper))
+     keccak256(abi.encodePacked(myAddress, _tickLower, _tickUpper))
    );
    ...}
...

    function addLiquidity(...
     {...
    IERC20WithBurn(params._tokenA).transferFrom(
      rdpxV2Core,
-     address(this),
+     myAddress,
      params._amount0Desired
    );
    IERC20WithBurn(params._tokenB).transferFrom(
      rdpxV2Core,
-     address(this),
+     myAddress,
      params._amount1Desired
    );
    INonfungiblePositionManager.MintParams
      memory mintParams = INonfungiblePositionManager.MintParams(
        ...
-       address(this),
+       myAddress,
        type(uint256).max
      );
     ...}
...

    function swap(...
    {...
    IERC20WithBurn(_tokenA).transferFrom(
      rdpxV2Core,
-     address(this),
+     myAddress,
      _amountAtoB
    );
    ISwapRouter.ExactInputSingleParams memory swap_params = ISwapRouter
      .ExactInputSingleParams(...
-       address(this),
+       myAddress,
      ...
      );    
    ...}
...  
 
     function recoverERC721(...
     {...
    INonfungiblePositionManager(tokenAddress).safeTransferFrom(
-     address(this),
+     myAddress,
      rdpxV2Core,
      token_id
    );
     ...}
...
  function _sendTokensToRdpxV2Core(address tokenA, address tokenB) internal {
-    uint256 tokenABalance = IERC20WithBurn(tokenA).balanceOf(address(this));
-    uint256 tokenBBalance = IERC20WithBurn(tokenB).balanceOf(address(this));     
+    uint256 tokenABalance = IERC20WithBurn(tokenA).balanceOf(myAddress);
+    uint256 tokenBBalance = IERC20WithBurn(tokenB).balanceOf(myAddress);     
    ...}
```

### Use `hardcode` address
```solidity
File : contracts/core/RdpxV2Core.sol

169: token.safeTransfer(msg.sender, token.balanceOf(address(this)));

483: ).purchase(_amount, address(this));

573: address(this),

654: .safeTransferFrom(msg.sender, address(this), _rdpxAmount);

911: address(this),

953: IERC20WithBurn(weth).safeTransferFrom(msg.sender, address(this), _amount);

998: address(this)

1060: address(this),

1102: address(this),
```
[169](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L169),[483](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L483),[573](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L573),[654](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L654),[911](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L911),[953](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L953),[998](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L998),[1060](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1060),[1102](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1102)

```diff
File : contracts/core/RdpxV2Core.sol

contract RdpxV2Core is 
 ...
{
+  address public myAddress = 0x1938567890123456789012345678901234447870;
  ...
   function emergencyWithdraw(...
    ...{
    for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
-     token.safeTransfer(msg.sender, token.balanceOf(address(this)));
+     token.safeTransfer(msg.sender, token.balanceOf(myAddress));
    }
   ...} 

  ...
   function _purchaseOptions(...
   ...{
    (premium, optionId) = IPerpetualAtlanticVault(
      addresses.perpetualAtlanticVault
-   ).purchase(_amount, address(this)); 
+   ).purchase(_amount, myAddress); 
   ...} 

  ...
     function _stake(...
    ...{
    IDpxEthToken(reserveAsset[reservesIndex["DPXETH"]].tokenAddress).mint(
-     address(this),
+     myAddress,
      _amount / 2
    ); 
    ...}

  ...
    function _transfer(...
    ...{
-       .safeTransferFrom(msg.sender, address(this), _rdpxAmount);
+       .safeTransferFrom(msg.sender, myAddress, _rdpxAmount);
    ...}    

  ...
   function bond(...
   ...{
    IERC20WithBurn(weth).safeTransferFrom(
      msg.sender,
-     address(this),
+     myAddress,
      wethRequired
    ); 
    ...}   

  ...
   function addToDelegate(...
    ...{
-    IERC20WithBurn(weth).safeTransferFrom(msg.sender, address(this), _amount);
+    IERC20WithBurn(weth).safeTransferFrom(msg.sender, myAddress, _amount);
   ...}

  ...
   function sync() external {
    for (uint256 i = 1; i < reserveAsset.length; i++) {
      uint256 balance = IERC20WithBurn(reserveAsset[i].tokenAddress).balanceOf(
-        address(this)
+        myAddress
      );
     ...} 
     ...}

  ...
   function upperDepeg(...
   ...{
    IDpxEthToken(reserveAsset[reservesIndex["DPXETH"]].tokenAddress).mint(
-     address(this),
+     myAddress,
      _amount
    );   
   ...} 

  ...
   function lowerDepeg(...
    ...{
-        address(this),
+        myAddress,
    ...}      
...
} 
```

### Use `hardcode` address
```solidity
File : contracts/perp-vault/PerpetualAtlanticVaultLP.sol

128: collateral.transferFrom(msg.sender, address(this), assets);

192: collateral.balanceOf(address(this)) >= _totalCollateral + proceeds,

201: collateral.balanceOf(address(this)) == _totalCollateral - loss,

210: IERC20WithBurn(rdpx).balanceOf(address(this)) >= _rdpxCollateral + amount,

```
[128](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L128),[192](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L192),[201](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L201),[210](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L210)

```diff
File : contracts/perp-vault/PerpetualAtlanticVaultLP.sol

contract PerpetualAtlanticVaultLP is ERC20, IPerpetualAtlanticVaultLP {
  ...
+   address public myAddress = 0x1934567890123456789012345678901234567870;
    function deposit(
    uint256 assets,
    address receiver
  ) public virtual returns (uint256 shares) {
    ...
-      collateral.transferFrom(msg.sender, address(this), assets);
+      collateral.transferFrom(msg.sender, myAddress, assets);
    ... }
  ...


     function addProceeds(uint256 proceeds) public onlyPerpVault {
      require(
-     collateral.balanceOf(address(this)) >= _totalCollateral + proceeds,
+     collateral.balanceOf(myAddress) >= _totalCollateral + proceeds,
      "Not enough collateral token was sent"
    );
    _totalCollateral += proceeds;
  }


     function subtractLoss(uint256 loss) public onlyPerpVault {
     require(
-      collateral.balanceOf(address(this)) == _totalCollateral - loss,
+      collateral.balanceOf(myAddress) == _totalCollateral - loss,
      "Not enough collateral was sent out"
    );
    _totalCollateral -= loss;
  }


    function addRdpx(uint256 amount) public onlyPerpVault {
     require(
-      IERC20WithBurn(rdpx).balanceOf(address(this)) >= _rdpxCollateral + amount,
+      IERC20WithBurn(rdpx).balanceOf(myAddress) >= _rdpxCollateral + amount,
      "Not enough rdpx token was sent"
    );
    _rdpxCollateral += amount;
  }
}

```

### Use `hardcode` address
```solidity
File : contracts/perp-vault/PerpetualAtlanticVault.sol

227: token.safeTransfer(msg.sender, token.balanceOf(address(this)));

289: collateralToken.safeTransferFrom(msg.sender, address(this), premium);

384:  address(this),
```
[227](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L227C6-L227C70),[289](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L289C5-L289C74),[384](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L384C6-L384C21)

```diff
File : contracts/perp-vault/PerpetualAtlanticVault.sol

contract PerpetualAtlanticVault is ...
{ ...
+   address public myAddress = 0x1934567890123456789012345678901234567870;
 ...
      function emergencyWithdraw(
    address[] calldata tokens
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    ...
-      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
+      token.safeTransfer(msg.sender, token.balanceOf(myAddress));
    }
   ...
     function purchase(
    uint256 amount,
    address to
  )
    external
    nonReentrant
    onlyRole(RDPXV2CORE_ROLE)
    returns (uint256 premium, uint256 tokenId)
  {
    ...
    // Transfer premium from msg.sender to PerpetualAtlantics vault
-    collateralToken.safeTransferFrom(msg.sender, address(this), premium);
+    collateralToken.safeTransferFrom(msg.sender, myAddress, premium);
    
 ... }
  ...
       function payFunding() external onlyRole(RDPXV2CORE_ROLE) returns (uint256) {
  ...
    collateralToken.safeTransferFrom(
      addresses.rdpxV2Core,
-      address(this),
+      myAddress,
      totalFundingForEpoch[latestFundingPaymentPointer]
    );
    ...  }

 ... }


```

### Use `hardcode` address
```solidity
File : contracts/reLP/ReLPContract.sol 

245: address(this),

263: address(this),

282: address(this),

293: address(this),

304: IERC20WithBurn(addresses.tokenA).balanceOf(address(this))

```
[202-307](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L202C3-L307C4)

```diff
File : contracts/reLP/ReLPContract.sol 

contract ReLPContract is AccessControl 
{ ...
+   address public myAddress = 0x1934567890123456789012345678901234567870;
 ...

  function reLP(...)
  { ...
 IERC20WithBurn(addresses.pair).transferFrom(
      addresses.amo,
-      address(this),
+      myAddress,
      lpToRemove
    );

   ...

    (, uint256 amountB) = IUniswapV2Router(addresses.ammRouter).removeLiquidity(
      addresses.tokenA,
      addresses.tokenB,
      lpToRemove,
      mintokenAAmount,
      mintokenBAmount,
-      address(this),
+      myAddress,
      block.timestamp + 10
    );

   ...

    uint256 tokenAAmountOut = IUniswapV2Router(addresses.ammRouter)
      .swapExactTokensForTokens(
        amountB / 2,
        mintokenAAmount,
        path,
-      address(this),
+      myAddress,
        block.timestamp + 10
      )[path.length - 1];

    (, , uint256 lp) = IUniswapV2Router(addresses.ammRouter).addLiquidity(
      addresses.tokenA,
      addresses.tokenB,
      tokenAAmountOut,
      amountB / 2,
      0,
      0,
-      address(this),
+      myAddress,
      block.timestamp + 10
    );

  ...

    IERC20WithBurn(addresses.tokenA).safeTransfer(
      addresses.rdpxV2Core,
-      IERC20WithBurn(addresses.tokenA).balanceOf(address(this))
+      IERC20WithBurn(addresses.tokenA).balanceOf(myAddress)
    );
    IRdpxV2Core(addresses.rdpxV2Core).sync();
  }

  ... }

  ```

## [G-11] Do not calculate constants to save gas.

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.
In Solidity, a constant is a value that is known at compile-time and cannot be changed during runtime. Constants can be defined using the constant keyword or the immutable keyword. if a constant value can be calculated at compile-time, it is more gas-efficient to define the constant directly with the calculated value rather than calculating it at runtime. This is because calculating a constant at runtime requires additional gas to be consumed, whereas defining it directly with the calculated value does not.

**_2 Instance - 1 File_**

```solidity
File : contracts/core/RdpxV2Core.sol

88: uint256 public constant RDPX_RATIO_PERCENTAGE = 25 * DEFAULT_PRECISION;

91: uint256 public constant ETH_RATIO_PERCENTAGE = 75 * DEFAULT_PRECISION;
```
[88](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L88),[91](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L91)

```diff
File : contracts/core/RdpxV2Core.sol

-88: uint256 public constant RDPX_RATIO_PERCENTAGE = 25 * DEFAULT_PRECISION;
+88: uint256 public constant RDPX_RATIO_PERCENTAGE = 2500000000;

-91: uint256 public constant ETH_RATIO_PERCENTAGE = 75 * DEFAULT_PRECISION;
+91: uint256 public constant ETH_RATIO_PERCENTAGE = 7500000000;
```


## [G-12] Amounts should be checked for 0 before calling a transfer.

In Solidity, the transfer() function is commonly used to transfer ether or tokens from one address to another. When calling the transfer() function, it is important to check the amount being transferred to ensure that it is greater than 0, as transferring 0 amounts is not necessary and can consume unnecessary gas.

**_3 Instances - 1 File_**

```solidity
File : contracts/reLP/ReLPContract.sol 

243: IERC20WithBurn(addresses.pair).transferFrom(addresses.amo, address(this),lpToRemove );
298: IERC20WithBurn(addresses.pair).safeTransfer(addresses.amo, lp);
302: IERC20WithBurn(addresses.tokenA).safeTransfer(addresses.rdpxV2Core,IERC20WithBurn(addresses.tokenA).balanceOf(address(this)) );

```
[243](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L243C5-L247C7),[298](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L298C3-L298C68),[302](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L302C3-L305C7)

## [G-13] Do not initialize state variables with their default value. 

Every variable assignment in Solidity costs gas. When initializing variables, we often waste gas by assigning default values that will never be used.

**_1 Instance - 1 File_** 

```diff
File : contracts/perp-vault/PerpetualAtlanticVault.sol

-89: uint256 public latestFundingPaymentPointer = 0;
+89: uint256 public latestFundingPaymentPointer;

```
[89](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L89C2-L89C50)

## [G-14] Use != 0 instead of > 0 for unsigned integer comparison to save gas.

The statement "Use != 0 instead of > 0 for unsigned integer comparison to save gas cost" means that using the != 0 comparison operator is more gas-efficient than using the > 0 comparison operator when comparing unsigned integers. This is because the Solidity compiler can optimize the != 0 comparison to a single operation, while the > 0 comparison requires two operations.

**_19 Instances - 4 Files_**


```diff
File : contracts/amo/UniV2LiquidityAmo.sol

-113: _slippageTolerance > 0,
+113: _slippageTolerance != 0,

-133: require(_amount > 0, "reLPContract: amount must be greater than 0");
+133: require(_amount != 0, "reLPContract: amount must be greater than 0");

```
[113](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L113),[133](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L133)


```diff
File : contracts/core/RdpxV2Core.sol

-183: _validate(_rdpxBurnPercentage > 0, 3);
+183: _validate(_rdpxBurnPercentage != 0, 3);

-196: _validate(_rdpxFeePercentage > 0, 3);
+196: _validate(_rdpxFeePercentage != 0, 3);

-231: _validate(_bondMaturity > 0, 3);
+231: _validate(_bondMaturity != 0, 3);

-410: _validate(_amount > 0, 17);
+410: _validate(_amount != 0, 17);

-444: _validate(_bondDiscountFactor > 0, 3);
+444: _validate(_bondDiscountFactor != 0, 3);

-458: _validate(_slippageTolerance > 0, 3);
+458: _validate(_slippageTolerance != 0, 3);

-556: minAmount > 0 ? minAmount : minOut
+556: minAmount != 0 ? minAmount : minOut

-838: _validate(_amounts[i] > 0, 4);
+838: _validate(_amounts[i] != 0, 4);

-901: _validate(_amount > 0, 4);
+901: _validate(_amount != 0, 4);

-984: _validate(amountWithdrawn > 0, 15);
+984: _validate(amountWithdrawn != 0, 15);

-1021: _validate(bonds[id].timestamp > 0, 6);
+1021: _validate(bonds[id].timestamp != 0, 6);

```
[183](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L183),[196](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L196),[231](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L231),[410](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L410),[444](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L444),[458](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L458),[556](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L556),[838](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L838),[901](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L901),[984](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L984),[1021](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1021)

 

```diff
File : contracts/reLP/ReLPContract.sol 

- 94:  _reLPFactor > 0,
+ 94:  _reLPFactor !=0,

-175:  _liquiditySlippageTolerance > 0,
+175:  _liquiditySlippageTolerance !=0,

-190:  _slippageTolerance > 0,
+190:  _slippageTolerance !=0,

```
[94](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L94C5-L94C23),[175](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L175C3-L175C39),[190](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L190C7-L190C30)



```diff
File : contracts/perp-vault/PerpetualAtlanticVault.sol

-265 :   _validate(amount > 0, 2);
+265 :   _validate(amount !=0, 2);

-414 :   _validate(optionsPerStrike[strikes[i]] > 0, 4);
+414 :   _validate(optionsPerStrike[strikes[i]] !=0, 4);

-547 :    _price > 0 ? _price : getUnderlyingPrice(),
+547 :    _price !=0 ? _price : getUnderlyingPrice(),
```
[265](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L265),[414](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L414C6-L414C54),[547](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L547C4-L547C50)

## [G-15] Use `delete` for state variables.

 Use `delete` to reinitialize with their default value rather than assigning default value saves gas.

**_2 Instance - 2 File_**

```diff
File : contracts/core/RdpxV2Core.sol

- 277: reservesIndex[_assetSymbol] = 0;
+ 277: delete reservesIndex[_assetSymbol];
```
[277](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L277)


 ```diff
 File : contracts/perp-vault/PerpetualAtlanticVault.sol

- 343: optionPositions[optionIds[i]].strike = 0;
+ 343: delete optionPositions[optionIds[i]].strike;

 ```
 [343](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L343C6-L343C48)


## [G-16] Using ternary operator instead of if else saves gas.

When using the if-else construct, Solidity generates bytecode that includes a JUMP operation. The JUMP operation requires the EVM to change the program counter to a different location in the bytecode, resulting in additional gas costs. On the other hand, the ternary operator doesn't involve a JUMP operation, as it generates bytecode that utilizes conditional stack manipulation.
The exact amount of gas saved by using the ternary operator instead of the if-else construct in Solidity can vary depending on the specific scenario, the complexity of the surrounding code, and the conditions involved

**_1 Instance - 1 File_**

```solidity
File : contracts/perp-vault/PerpetualAtlanticVault.sol

578:    if (remainder == 0) {
579:      return _strike;
580:    } else {
581:      return _strike - remainder + roundingPrecision;
582:    }

```
[578-582](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L578C3-L582C6)

```diff
File : contracts/perp-vault/PerpetualAtlanticVault.sol

- if (remainder == 0) {
-      return _strike;
-    } else {
-      return _strike - remainder + roundingPrecision;
-    }

+  return remainder == 0 ? _strike : _strike - remainder + roundingPrecision;
```

## [G-17] No need to explicitly initialize variables with default values.

If a variable is not set/initialized, it is assumed to have the default value (0 for uint, false for bool, address(0) for address, etc.). Explicitly initializing it with its default value is an anti-pattern and wastes gas. 

**_8 Instances - 4 Files_**

### *`for (uint256 i = 0; i < numIterations; ++i)`*  should be replaced with: *`for (uint256 i; i < numIterations; ++i)`* 

```solidity
File : contracts/amo/UniV2LiquidityAmo.sol

147: for (uint256 i = 0; i < tokens.length; i++) {
```
[147](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L147)


```solidity
File : contracts/amo/UniV3LiquidityAmo.sol

120: for (uint i = 0; i < positions_array.length; i++) {
```
[120](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L120)


```solidity
File : contracts/core/RdpxV2Core.sol

167: for (uint256 i = 0; i < tokens.length; i++) {

775: for (uint256 i = 0; i < optionIds.length; i++) {

836: for (uint256 i = 0; i < _amounts.length; i++) {
```
[167](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L167),[775](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L775),[836](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L836)


```solidity
File : contracts/perp-vault/PerpetualAtlanticVault.sol

225: for (uint256 i = 0; i < tokens.length; i++) { 

328: for (uint256 i = 0; i < optionIds.length; i++) { 

413: for (uint256 i = 0; i < strikes.length; i++) { 
```
[225](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L225),[338](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L328),[413](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L413)

## [G‑18] State variables only set in the constructor should be declared immutable.

Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

**Note: These are instances missed by the Bot Race.**

**_2 Instances -2 Files_**

### Make `rdpx` and `rdpxV2Core` immutable 
```solidity
File : contracts/amo/UniV3LiquidityAmo.sol

69: address public rdpx;
  
72: address public rdpxV2Core;
```
[69-72](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L69C1-L72C29)

```diff
File : contracts/amo/UniV3LiquidityAmo.sol

-  address public rdpx;
+  address public immutable rdpx;

-  address public rdpxV2Core;
+  address public immutable rdpxV2Core;

```

### Make `perpetualAtlanticVault` and `rdpx` immutable
```solidity
File : contracts/perp-vault/PerpetualAtlanticVaultLP.sol

46: IPerpetualAtlanticVault public perpetualAtlanticVault;

67: address public rdpx;
```
[46](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L46), [67](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L67)

```diff
File : contracts/perp-vault/PerpetualAtlanticVaultLP.sol

-46: IPerpetualAtlanticVault public perpetualAtlanticVault;
+46: IPerpetualAtlanticVault public immutable perpetualAtlanticVault;

-67: address public rdpx;
+67: address public immutable rdpx;
```

## [G-19] Unnecessary require check while it already checked using modifier. 

**_1 Instance -1 File_**

```solidity
File : 

114:   function mint(
    ) {...
119:    _whenNotPaused();
120:    require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
   ...}
```
[114-120](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L114C1-L120C73)


## [G-20] Avoid emitting storage values.

Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. We can avoid unnecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.

**_2 Instances -2 Files_**

```solidity
File : contracts/core/RdpxV2Core.sol

358:  function setPricingOracleAddresses(
    address _rdpxPriceOracle,
    address _dpxEthPriceOracle
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_rdpxPriceOracle != address(0), 17);
    _validate(_dpxEthPriceOracle != address(0), 17);

    pricingOracleAddresses = PricingOracleAddresses({
      rdpxPriceOracle: _rdpxPriceOracle,
      dpxEthPriceOracle: _dpxEthPriceOracle
    });

    emit LogSetPricingOracleAddresses(pricingOracleAddresses);
371:  }
```
[358-371](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L358C1-L372C1)


```solidity
File : contracts/perp-vault/PerpetualAtlanticVault.sol

181:  function setAddresses(
    address _optionPricing,
    address _assetPriceOracle,
    address _volatilityOracle,
    address _feeDistributor,
    address _rdpx,
    address _perpetualAtlanticVaultLP,
    address _rdpxV2Core
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    ...
    addresses = Addresses({
      optionPricing: _optionPricing,
      assetPriceOracle: _assetPriceOracle,
      volatilityOracle: _volatilityOracle,
      feeDistributor: _feeDistributor,
      rdpx: _rdpx,
      perpetualAtlanticVaultLP: _perpetualAtlanticVaultLP,
      rdpxV2Core: _rdpxV2Core
    });
    collateralToken.safeApprove(
      addresses.perpetualAtlanticVaultLP,
      type(uint256).max
    );
211: emit AddressesSet(addresses);
  }
```
[181-211](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L181C1-L212C4)


