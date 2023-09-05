# Codebase Optimization Report

## Auditor's Disclaimer 

While we try our best to maintain readability in the provided code snippets, some functions have been truncated to highlight the affected portions.

It's important to note that during the implementation of these suggested changes, developers must exercise caution to avoid introducing vulnerabilities. Although the optimizations have been tested prior to these recommendations, it is the responsibility of the developers to conduct thorough testing again.

Code reviews and additional testing are strongly advised to minimize any potential risks associated with the refactoring process.

## Table Of Contents

- [Codebase Optimization Report](#codebase-optimization-report)
  - [Auditor's Disclaimer](#auditors-disclaimer)
  - [Table Of Contents](#table-of-contents)
  - [Note on Gas estimates.](#note-on-gas-estimates)
  - [\[G-01\]Using immutable on variables that are only set in the constructor and never after (Save 8400 Gas)](#g-01using-immutable-on-variables-that-are-only-set-in-the-constructor-and-never-after-save-8400-gas)
  - [\[G-02\]Use a constant variable for `roundingPrecision`(Save 1 SLOT: 2.1k Gas)](#g-02use-a-constant-variable-for-roundingprecisionsave-1-slot-21k-gas)
  - [\[G-03\]Tighly pack storage variables/optimize the order of variable declaration](#g-03tighly-pack-storage-variablesoptimize-the-order-of-variable-declaration)
    - [\[G-03-01\]We can save one more slot compared to the bot(use `uint40` for `genesis` variable and `lastUpdateTime` and pack them with `collateralToken` ) - Save 2k gas](#g-03-01we-can-save-one-more-slot-compared-to-the-botuse-uint40-for-genesis-variable-and-lastupdatetime-and-pack-them-with-collateraltoken----save-2k-gas)
    - [Reordering the order of inheritance can help us pack one more variable](#reordering-the-order-of-inheritance-can-help-us-pack-one-more-variable)
    - [\[G-03-02\]We can save 1 SLOT here by packing `bool _paused` from the contract `pausable` with the variable `collateralToken` in the inheriting contract. (Save 1 SLOT: 2K gas)](#g-03-02we-can-save-1-slot-here-by-packing-bool-_paused-from-the-contract-pausable-with-the-variable-collateraltoken-in-the-inheriting-contract-save-1-slot-2k-gas)
    - [\[G-03-03\]Reorder the order of variable declaration to allow packing variables from child contracts.(Save 1 SLOT: 2K gas)](#g-03-03reorder-the-order-of-variable-declaration-to-allow-packing-variables-from-child-contractssave-1-slot-2k-gas)
  - [\[G-04\]Function calls should be cached instead of calling them more than once in same transaction](#g-04function-calls-should-be-cached-instead-of-calling-them-more-than-once-in-same-transaction)
    - [\[G-04-01\]Result of `nextFundingPaymentTimestamp()` should be cached instead of repeatedly calling it (Save 777 Gas on average)](#g-04-01result-of-nextfundingpaymenttimestamp-should-be-cached-instead-of-repeatedly-calling-it-save-777-gas-on-average)
  - [\[G-05\]Use calldata instead of memory for function parameters](#g-05use-calldata-instead-of-memory-for-function-parameters)
    - [\[G-05-01\]Use calldata for `_assetSymbol`(Save 318 Gas on average)](#g-05-01use-calldata-for-_assetsymbolsave-318-gas-on-average)
    - [\[G-05-02\]Use calldata for `_assetSymbol`](#g-05-02use-calldata-for-_assetsymbol)
    - [\[G-05-03\]Use calldata for `optionIds`(Save 646 Gas on average)](#g-05-03use-calldata-for-optionidssave-646-gas-on-average)
    - [\[G-05-04\]Change to external and use calldata for `_amounts` and `_delegateIds `(Save 768 Gas on average)](#g-05-04change-to-external-and-use-calldata-for-_amounts-and-_delegateids-save-768-gas-on-average)
    - [\[G-05-05\]Change to external and use calldata on `_token`(Save 536 Gas on average)](#g-05-05change-to-external-and-use-calldata-on-_tokensave-536-gas-on-average)
    - [\[G-05-06\]use calldata for `optionIds`(Save 596 Gas on average)](#g-05-06use-calldata-for-optionidssave-596-gas-on-average)
    - [\[G-05-07\]use calldata for `strikes`(Save 233 Gas on average)](#g-05-07use-calldata-for-strikessave-233-gas-on-average)
  - [\[G-06\]Use uint256 instead of the counter library](#g-06use-uint256-instead-of-the-counter-library)
    - [\[G-06-01\]RdpxV2Bond.sol.mint(): get rid of counters(Save 109 Gas)](#g-06-01rdpxv2bondsolmint-get-rid-of-counterssave-109-gas)
    - [\[G-06-02\]RdpxDecayingBonds.sol.mintToken() : save 162 Gas](#g-06-02rdpxdecayingbondssolminttoken--save-162-gas)
    - [\[G-06-03\]PerpetualAtlanticVault.sol.\_mintOptionToken(): (Save ~132 Gas on average)](#g-06-03perpetualatlanticvaultsol_mintoptiontoken-save-132-gas-on-average)
  - [\[G-07\]Since we are assigning local values to the struct, we shouldn't read the struct(state) in the same transactions, simply use the local variables(Save 427 Gas)](#g-07since-we-are-assigning-local-values-to-the-struct-we-shouldnt-read-the-structstate-in-the-same-transactions-simply-use-the-local-variablessave-427-gas)
  - [\[G-08\]when using storage instead of memory, we should cache any fields that need to be re-read in stack variables](#g-08when-using-storage-instead-of-memory-we-should-cache-any-fields-that-need-to-be-re-read-in-stack-variables)
  - [\[G-09\]Leverage  dot notation method for struct assignment](#g-09leverage--dot-notation-method-for-struct-assignment)
    - [\[G-09-01\]Refactor the struct assignment style(Save 159 Gas)](#g-09-01refactor-the-struct-assignment-stylesave-159-gas)
    - [\[G-09-02\]PerpetualAtlanticVault.sol.setAddresses(): (Save 196 Gas on average)](#g-09-02perpetualatlanticvaultsolsetaddresses-save-196-gas-on-average)
    - [\[G-09-03\]PerpetualAtlanticVault.sol.purchase(): Save 90 on Gas on average](#g-09-03perpetualatlanticvaultsolpurchase-save-90-on-gas-on-average)
    - [\[G-09-04\]ReLPContract.sol.setAddresses(Save 238 Gas)](#g-09-04relpcontractsolsetaddressessave-238-gas)
    - [\[G-09-05\]RdpxV2Core.sol:setAddresses(Save 268 Gas)](#g-09-05rdpxv2coresolsetaddressessave-268-gas)
  - [\[G-10\]We can cache some gas intensive operations](#g-10we-can-cache-some-gas-intensive-operations)
    - [\[G-10-01\]PerpetualAtlanticVault.sol.payFunding(): Cache the call `totalFundingForEpoch[latestFundingPaymentPointer]`(Save 669 Gas on average)](#g-10-01perpetualatlanticvaultsolpayfunding-cache-the-call-totalfundingforepochlatestfundingpaymentpointersave-669-gas-on-average)
    - [\[G-10-02\]We can cache some operations instead of repeatedly doing them(Save 490 Gas on average)](#g-10-02we-can-cache-some-operations-instead-of-repeatedly-doing-themsave-490-gas-on-average)
    - [\[G-10-03\]Cache `latestFundingPaymentPointer` in memory (Save  291 Gas)](#g-10-03cache-latestfundingpaymentpointer-in-memory-save--291-gas)
    - [\[G-10-04\]We don't have to do `delegates.length - 1` twice especially because we are reading from storage(Save 136 Gas on average)](#g-10-04we-dont-have-to-do-delegateslength---1-twice-especially-because-we-are-reading-from-storagesave-136-gas-on-average)
    - [\[G-10-05\]PerpetualAtlanticVault.sol.settle(): `addresses.perpetualAtlanticVaultLP` should be cached (Save 139 Gas)](#g-10-05perpetualatlanticvaultsolsettle-addressesperpetualatlanticvaultlp-should-be-cached-save-139-gas)
  - [\[G-11\]We can avoid reading from state here](#g-11we-can-avoid-reading-from-state-here)
  - [\[G-12\]Since we have cached values, we should reference them instead of making a state read](#g-12since-we-have-cached-values-we-should-reference-them-instead-of-making-a-state-read)
  - [\[G-13\]Unused libraries](#g-13unused-libraries)
  - [\[G-14\]No need to initialize `state variables` with their default values](#g-14no-need-to-initialize-state-variables-with-their-default-values)
  - [Conclusion](#conclusion)



## Note on Gas estimates.

we've tried to give the exact amount of gas being saved from running the included tests whenever the function is within the test coverage

Some functions are not covered by the test cases or are internal/private functions. In this case, the gas benchmarks are based on functions that call this internal/private functions.
Some like packing/immutables we can easily tell the amount being saved based on the number of slots being saved.

**In some cases , one issue title will have many instances under it, the main issue will be labeled as `G-mainTitleNumber` while the instances follows the format `G-mainTitleNumber-InstanceNumber**

**The bot did an amazing job on this audit,but still missed some findings**

## [G-01]Using immutable on variables that are only set in the constructor and never after (Save 8400 Gas)

**This instance were missed by the bot**

**Gas per instance: 2.1K**
**Total Instances: 4**
Total Gas Saved: `2.1 * 4 = 8400 Gas`

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L68-L72

```solidity
File: /contracts/amo/UniV3LiquidityAmo.sol
68:  // Rdpx address
69:  address public rdpx;

71:  // RdpxV2Core address
72:  address public rdpxV2Core;
```

```diff
   // Rdpx address
-  address public rdpx;
+  address public immutable rdpx;

   // RdpxV2Core address
-  address public rdpxV2Core;
+  address public immutable rdpxV2Core;

```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L46
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVaultLP.sol
46:  IPerpetualAtlanticVault public perpetualAtlanticVault;
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L66-L70
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVaultLP.sol
66:  /// @dev address of rdpx token
67:  address public rdpx;
```

```diff
   /// @dev address of rdpx token
-  address public rdpx;
+  address public immutable rdpx;
```

## [G-02]Use a constant variable for `roundingPrecision`(Save 1 SLOT: 2.1k Gas)

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L104
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
104:  uint256 public roundingPrecision = 1e6;
```
The variable `roundingPrecision` is being referenced in two places, see below
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L576-L583
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
576:  function roundUp(uint256 _strike) public view returns (uint256 strike) {
577:    uint256 remainder = _strike % roundingPrecision;
578:    if (remainder == 0) {
579:      return _strike;
580:    } else {
581:      return _strike - remainder + roundingPrecision;
582:    }
583:  }
```

Note, at no instance is our variable being modified, we can therefore save the SLOT it's currently occupying by switching to a constant variable

```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
index 88ac2b3..fefe065 100644
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
@@ -101,7 +101,7 @@ contract PerpetualAtlanticVault is
   uint256 public fundingDuration = 7 days;

   /// @dev the precision to round up to
-  uint256 public roundingPrecision = 1e6;
+  uint256 public  constant roundingPrecision = 1e6;
```

## [G-03]Tighly pack storage variables/optimize the order of variable declaration

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a `Gwarmaccess (100 gas)` versus a `Gcoldsload (2100 gas)`.
Here, the storage variables can be tightly packed

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L57-L98

### [G-03-01]We can save one more slot compared to the bot(use `uint40` for `genesis` variable and `lastUpdateTime` and pack them with `collateralToken` ) - Save 2k gas

**Note: The bot didn't pack the genesis variable: we therefore only count the one slot not found by the bot**
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
  IERC20WithBurn public collateralToken;

  /// @dev genesis timestamp
  uint256 public genesis;

  /// @dev the timestamp of the last update where funding was paid for
  uint256 public lastUpdateTime;
```


```diff
   /// @dev Collateral Token
   IERC20WithBurn public collateralToken;
+    /// @dev genesis timestamp
+  uint40 public genesis;
+
+  /// @dev the timestamp of the last update where funding was paid for
+  uint40 public lastUpdateTime;

   /// @dev The precision of the collateral token
   uint256 public collateralPrecision;
@@ -91,12 +96,6 @@ contract PerpetualAtlanticVault is
   /// @dev the total number of active options
   uint256 public totalActiveOptions;

-  /// @dev genesis timestamp
-  uint256 public genesis;
-
-  /// @dev the timestamp of the last update where funding was paid for
-  uint256 public lastUpdateTime;
```

### Reordering the order of inheritance can help us pack one more variable 
[From the docs](https://docs.soliditylang.org/en/v0.8.21/internals/layout_in_storage.html):"For contracts that use inheritance, the ordering of state variables is determined by the C3-linearized order of contracts starting with the most base-ward contract. If allowed by the above rules, state variables from different contracts do share the same storage slot."

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L28-L98

### [G-03-02]We can save 1 SLOT here by packing `bool _paused` from the contract `pausable` with the variable `collateralToken` in the inheriting contract. (Save 1 SLOT: 2K gas)

*Part of this borrows some part from the above finding*
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
contract PerpetualAtlanticVault is
  IPerpetualAtlanticVault,
  ReentrancyGuard,
  Pausable,
  ERC721,
  ERC721Enumerable,
  ERC721Burnable,
  AccessControl,
  ContractWhitelist
{
42:  Counters.Counter private _tokenIdCounter;
    <Truncated>
56:  /// @dev Collateral Token
57:  IERC20WithBurn public collateralToken;
      <Truncated>

 94: /// @dev genesis timestamp
 95: uint256 public genesis;
	<Truncated>

97:  /// @dev the timestamp of the last update where funding was paid for
98:  uint256 public lastUpdateTime;
```

In the contract `Pausable`  we have a state variable of type `boolean`. According to solidity docs, if allowed by the rules of inheritance, variables from different contracts can be packed together.
This would mean, we move the `Pausable` to be the last contract inherited so that the `bool _paused` from that contract can be packed with variables in the inheriting contract. I've moved the `collateralToken, genesis,lastUpdateTime ` to the top of the contract so that the variable `_paused` is packed together with `collateralToken` as they are accessed in the same transaction

```diff
 contract PerpetualAtlanticVault is
   IPerpetualAtlanticVault,
   ReentrancyGuard,
-  Pausable,
   ERC721,
   ERC721Enumerable,
   ERC721Burnable,
   AccessControl,
-  ContractWhitelist
+  ContractWhitelist,
+  Pausable
 {
   using SafeERC20 for IERC20WithBurn;
   using Counters for Counters.Counter;

+  /// @dev Collateral Token
+  IERC20WithBurn public collateralToken;
+    /// @dev genesis timestamp
+  uint40 public genesis;
+
+  /// @dev the timestamp of the last update where funding was paid for
+  uint40 public lastUpdateTime;
   /// @dev Token ID counter for write positions
   Counters.Counter private _tokenIdCounter;

@@ -53,9 +60,6 @@ contract PerpetualAtlanticVault is
   /// @dev Contract addresses
   Addresses public addresses;

-  /// @dev Collateral Token
-  IERC20WithBurn public collateralToken;
-
   /// @dev The precision of the collateral token
   uint256 public collateralPrecision;

@@ -91,12 +95,6 @@ contract PerpetualAtlanticVault is
   /// @dev the total number of active options
   uint256 public totalActiveOptions;

-  /// @dev genesis timestamp
-  uint256 public genesis;
-
-  /// @dev the timestamp of the last update where funding was paid for
-  uint256 public lastUpdateTime;
```



https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L33-L67

### [G-03-03]Reorder the order of variable declaration to allow packing variables from child contracts.(Save 1 SLOT: 2K gas)

**We can pack the variable `_paused` from the contract `Pausable` with the variable `isReLPActive and putOptionsRequired` in the inheriting contract**
```solidity
File: /contracts/core/RdpxV2Core.sol
33:contract RdpxV2Core is
34:  IRdpxV2Core,
35:  AccessControl,
36:  ContractWhitelist,
37:  ERC721Holder,
38:  Pausable
39: {

47:  Addresses public addresses;

50:  PricingOracleAddresses public pricingOracleAddresses;

115:  bool public isReLPActive;

118:  bool public putOptionsRequired;
```
By just moving the bools to the top, we allow the inherited variable `_paused` to be packed with the variables we've just moved to the top
*There was a possibility of packing with address `weth` but this was suggested to be made immutable by the bot*
```diff
+  /// @notice Whether reLP is active or not
+  bool public isReLPActive;

+  /// @notice Whether put options are requred
+  bool public putOptionsRequired;
+
   /// @notice Addresses used by the contract
   Addresses public addresses;

@@ -111,12 +116,6 @@ contract RdpxV2Core is
   /// @notice Total weth delegated
   uint256 public totalWethDelegated;

-  /// @notice Whether reLP is active or not
-  bool public isReLPActive;
-
-  /// @notice Whether put options are requred
-  bool public putOptionsRequired;
```


## [G-04]Function calls should be cached instead of calling them more than once in same transaction

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L594-L614

### [G-04-01]Result of `nextFundingPaymentTimestamp()` should be cached instead of repeatedly calling it (Save 777 Gas on average)

Gas benchmarks based on function `Purchase()` which calls our private function `_updateFundingRate`

*`Test: Purchase(uint256,address)(uint256)`*

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 312368    | 333657   | 333657 | 354946 |
| After  | 311846    | 332880   | 332880 | 353914 |

*`Test: Purchase(uint256,address)(uint256,uint256)`*

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 17877    | 256930   | 221256 | 359646 |
| After | 17877 | 256498 | 221256 | 358614 |

```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
594:  function _updateFundingRate(uint256 amount) private {
595:    if (fundingRates[latestFundingPaymentPointer] == 0) {
596:      uint256 startTime;
597:      if (lastUpdateTime > nextFundingPaymentTimestamp() - fundingDuration) {
598:        startTime = lastUpdateTime;
599:      } else {
600:        startTime = nextFundingPaymentTimestamp() - fundingDuration;
601:      }
602:      uint256 endTime = nextFundingPaymentTimestamp();

606:    } else {
607:      uint256 startTime = lastUpdateTime;
608:      uint256 endTime = nextFundingPaymentTimestamp();
```

If we look into the implementation of the function `nextFundingPaymentTimestamp` which is being called several times, we can see we read three state variables `genesis,latestFundingPaymentPointer,fundingDuration`
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L563-L569
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
563:  function nextFundingPaymentTimestamp()
564:    public
565:    view
566:    returns (uint256 timestamp)
567:  {
568:    return genesis + (latestFundingPaymentPointer * fundingDuration);
569:  }
```
Every time we call this function, we end up making 3 state reads, which can be quite expensive. We should refactor the code as follows to only call this function once, ie reading from state only once and caching the function result.

```diff

   function _updateFundingRate(uint256 amount) private {
+     uint256 _nextFundingPaymentTimestamp = nextFundingPaymentTimestamp();
     if (fundingRates[latestFundingPaymentPointer] == 0) {
       uint256 startTime;
-      if (lastUpdateTime > nextFundingPaymentTimestamp() - fundingDuration) {
+
+      if (lastUpdateTime > _nextFundingPaymentTimestamp - fundingDuration) {
         startTime = lastUpdateTime;
       } else {
-        startTime = nextFundingPaymentTimestamp() - fundingDuration;
+        startTime = _nextFundingPaymentTimestamp - fundingDuration;
       }
-      uint256 endTime = nextFundingPaymentTimestamp();
+
       fundingRates[latestFundingPaymentPointer] =
         (amount * 1e18) /
-        (endTime - startTime);
+        (_nextFundingPaymentTimestamp - startTime);
     } else {
       uint256 startTime = lastUpdateTime;
-      uint256 endTime = nextFundingPaymentTimestamp();
-      if (endTime == startTime) return;
+      if (_nextFundingPaymentTimestamp == startTime) return;
       fundingRates[latestFundingPaymentPointer] =
         fundingRates[latestFundingPaymentPointer] +
-        ((amount * 1e18) / (endTime - startTime));
+        ((amount * 1e18) / (_nextFundingPaymentTimestamp - startTime));
     }
   }

```

## [G-05]Use calldata instead of memory for function parameters

If a reference type function parameter is read-only, it is cheaper in gas to use calldata instead of memory. Calldata is a non-modifiable, non-persistent area where function arguments are stored, and behaves mostly like memory.

Note that I've also flagged instances where the function is public but can be marked as external since it's not called by the contract, and cases where a constructor is involved

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L240-L243

### [G-05-01]Use calldata for `_assetSymbol`(Save 318 Gas on average)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 96795    | 104095   | 97365 | 118125 |
| After | 96477 | 103777 | 97047 | 117807 |
```solidity
File: /contracts/core/RdpxV2Core.sol
240:  function addAssetTotokenReserves(
241:    address _asset,
242:    string memory _assetSymbol
243:  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
```

```diff
   function addAssetTotokenReserves(
     address _asset,
-    string memory _assetSymbol
+    string calldata _assetSymbol
   ) external onlyRole(DEFAULT_ADMIN_ROLE) {
     require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L270-L272

### [G-05-02]Use calldata for `_assetSymbol`

```solidity
File: /contracts/core/RdpxV2Core.sol
270:  function removeAssetFromtokenReserves(
271:    string memory _assetSymbol
272:  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
```

```diff
   function removeAssetFromtokenReserves(
-    string memory _assetSymbol
+    string calldata _assetSymbol
   ) external onlyRole(DEFAULT_ADMIN_ROLE) {
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L764-L766

### [G-05-03]Use calldata for `optionIds`(Save 646 Gas on average)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 19746    | 64123   | 42398 | 136328 |
| After | 19070 | 63477 | 41831 | 135036 |
```solidity
File: /contracts/core/RdpxV2Core.sol
764:  function settle(
765:    uint256[] memory optionIds
766:  )
```

```diff
   function settle(
-    uint256[] memory optionIds
+    uint256[] calldata optionIds
   )
```


https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L819-L824

### [G-05-04]Change to external and use calldata for `_amounts` and `_delegateIds `(Save 768 Gas on average)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 2159    | 847375   | 1092279 | 1705471 |
| After | 1584 | 846607 | 1091594 | 1704182 |
```solidity
File: /contracts/core/RdpxV2Core.sol
819:  function bondWithDelegate(
820:    address _to,
821:    uint256[] memory _amounts,
822:    uint256[] memory _delegateIds,
823:    uint256 rdpxBondId
824:  ) public returns (uint256 receiptTokenAmount, uint256[] memory) {
```

```diff
   function bondWithDelegate(
     address _to,
-    uint256[] memory _amounts,
-    uint256[] memory _delegateIds,
+    uint256[] calldata _amounts,
+    uint256[] calldata _delegateIds,
     uint256 rdpxBondId
   ) public returns (uint256 receiptTokenAmount, uint256[] memory) {
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1135-L1137

### [G-05-05]Change to external and use calldata on `_token`(Save 536 Gas on average)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 3699    | 5099   | 3699 | 13699 |
| After | 3163 | 4563 | 3163 | 13163 |
```solidity
File: /contracts/core/RdpxV2Core.sol
1135:  function getReserveTokenInfo(
1136:    string memory _token
1137:  ) public view returns (address, uint256, string memory) {
```

```diff
   function getReserveTokenInfo(
-    string memory _token
+    string calldata _token
   ) public view returns (address, uint256, string memory) {
```



https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L315-L317

### [G-05-06]use calldata for `optionIds`(Save 596 Gas on average)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 17113    | 60186   | 56770 | 127485 |
| After | 16897 | 59890 | 56420 | 126616 |
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
315:  function settle(
316:    uint256[] memory optionIds
317:  )
```

```diff
   /// @inheritdoc      IPerpetualAtlanticVault
   function settle(
-    uint256[] memory optionIds
+    uint256[] calldata optionIds
   )
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L405-L407

### [G-05-07]use calldata for `strikes`(Save 233 Gas on average)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 64779    | 99013   | 101124 | 154379 |
| After | 64485 | 98780 | 100909 | 154085 |
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
405:  function calculateFunding(
406:    uint256[] memory strikes
407:  ) external nonReentrant returns (uint256 fundingAmount) {
```

```diff
   function calculateFunding(
-    uint256[] memory strikes
+    uint256[] calldata strikes
   ) external nonReentrant returns (uint256 fundingAmount) {
```


## [G-06]Use uint256 instead of the counter library

The counters version runs 6 more instructions than the uint version  due to the counters code needing to jump into the Counters library.
The 6 added instructions are: 2 JUMPs, 2 JUMPDESTs, and 2 PUSH1s. These are used to jump from the current executing code to the library function that actually handles the increment logic. The total gas costs of these operations is 24 units
In addition to using more gas, using the Counters library also results in a larger deployment size
See [More Info](https://twitter.com/0xCygaar/status/1608562486508417025)


**Note:The counters library is also being deprecated**https://github.com/OpenZeppelin/openzeppelin-contracts/pull/4289

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L9

### [G-06-01]RdpxV2Bond.sol.mint(): get rid of counters(Save 109 Gas)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 105847    | 113510   | 115747 | 117747 |
| After  | 105738    | 113401   | 115638 | 117638 |

```solidity
File: /contracts/core/RdpxV2Bond.sol
9:import { Counters } from "@openzeppelin/contracts/utils/Counters.sol";

18:  using Counters for Counters.Counter;


20:  Counters.Counter private _tokenIdCounter;
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L37-L43
```solidity
File: /contracts/core/RdpxV2Bond.sol
37:  function mint(
38:    address to
39:  ) public onlyRole(MINTER_ROLE) returns (uint256 tokenId) {
40:    tokenId = _tokenIdCounter.current();
41:    _tokenIdCounter.increment();
42:    _mint(to, tokenId);
43:  }
```


```diff
-import { Counters } from "@openzeppelin/contracts/utils/Counters.sol";

 contract RdpxV2Bond is
   ERC721,
@@ -15,9 +14,8 @@ contract RdpxV2Bond is
   AccessControl,
   ERC721Burnable
 {
-  using Counters for Counters.Counter;

-  Counters.Counter private _tokenIdCounter;
+  uint256 private _tokenIdCounter;

   bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

@@ -37,8 +35,10 @@ contract RdpxV2Bond is
   function mint(
     address to
   ) public onlyRole(MINTER_ROLE) returns (uint256 tokenId) {
-    tokenId = _tokenIdCounter.current();
-    _tokenIdCounter.increment();
+    tokenId = _tokenIdCounter;
+    unchecked{
+      ++_tokenIdCounter;
+    }
     _mint(to, tokenId);
   }

```


**Similar Instance**
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L13

### [G-06-02]RdpxDecayingBonds.sol.mintToken() : save 162 Gas

**Benchmarks based on function `mint()` which calls the function `mintToken()`

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 184661    | 192194   | 195461 | 197461 |
| After  | 184529    | 192062   | 195329 | 197329 |
```solidity
File: /contracts/decaying-bonds/RdpxDecayingBonds.sol
13:import { Counters } from "@openzeppelin/contracts/utils/Counters.sol";

26:  using Counters for Counters.Counter;

28:  Counters.Counter private _tokenIdCounter;

56:  constructor(

59:  ) ERC721(_name, _symbol) {

63:    _tokenIdCounter.increment();
64:  }
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L129-L133
```solidity
File: /contracts/decaying-bonds/RdpxDecayingBonds.sol
129:  function _mintToken(address to) private returns (uint256 tokenId) {
130:    tokenId = _tokenIdCounter.current();
131:    _tokenIdCounter.increment();
132:    _mint(to, tokenId);
133:  }
```

```diff
-import { Counters } from "@openzeppelin/contracts/utils/Counters.sol";

 // Interfaces
 import { IERC20WithBurn } from "../interfaces/IERC20WithBurn.sol";
@@ -23,9 +22,8 @@ contract RdpxDecayingBonds is
   AccessControl
 {
   using SafeERC20 for IERC20WithBurn;
-  using Counters for Counters.Counter;

-  Counters.Counter private _tokenIdCounter;
+  uint256 private _tokenIdCounter;

   // Create a new role identifier for the minter role
   bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
@@ -60,7 +58,9 @@ contract RdpxDecayingBonds is
     // Grant the minter role and admin role to deployer
     _setupRole(MINTER_ROLE, msg.sender);
     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
-    _tokenIdCounter.increment();
+    unchecked {
+      ++_tokenIdCounter;
+    }
   }

   /*============ ADMIN FUNCTIONS ============*/
@@ -127,8 +127,10 @@ contract RdpxDecayingBonds is
   /// @dev Internal function to mint a bond position token
   /// @param to the address to mint the position to
   function _mintToken(address to) private returns (uint256 tokenId) {
-    tokenId = _tokenIdCounter.current();
-    _tokenIdCounter.increment();
+    tokenId = _tokenIdCounter;
+    unchecked{
+      ++_tokenIdCounter;
+    }
     _mint(to, tokenId);
   }

```


**Another instance**
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L14

### [G-06-03]PerpetualAtlanticVault.sol.\_mintOptionToken(): (Save ~132 Gas on average)

*Benchmarks based on function purchase which calls the private function `_mintOptionToken()`*

*`Test: Purchase(uint256,address)(uint256)`*

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 312368    | 333657   | 333657 | 354946 |
| After  | 312236    | 333525   | 333525 | 354814 |

*`Test: Purchase(uint256,address)(uint256,uint256)`*

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 17877    | 256930   | 221256 | 359646 |
| After  | 17877    | 256809   | 221124 | 359515 |

```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
14:import { Counters } from "@openzeppelin/contracts/utils/Counters.sol";

39:  using Counters for Counters.Counter;

41:  /// @dev Token ID counter for write positions
42:  Counters.Counter private _tokenIdCounter;
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L588-L592
```solidity
588:  function _mintOptionToken() private returns (uint256 tokenId) {
589:    tokenId = _tokenIdCounter.current();
590:    _tokenIdCounter.increment();
591:    _mint(addresses.rdpxV2Core, tokenId);
592:  }
```

```diff
-import { Counters } from "@openzeppelin/contracts/utils/Counters.sol";
 import { IPerpetualAtlanticVaultLP } from "./IPerpetualAtlanticVaultLP.sol";
 import { ContractWhitelist } from "../helper/ContractWhitelist.sol";
 import { Pausable } from "../helper/Pausable.sol";
@@ -36,10 +35,9 @@ contract PerpetualAtlanticVault is
   ContractWhitelist
 {
   using SafeERC20 for IERC20WithBurn;
-  using Counters for Counters.Counter;

   /// @dev Token ID counter for write positions
-  Counters.Counter private _tokenIdCounter;
+  uint256 private _tokenIdCounter;

   /// @dev Manager role which handles bootstrapping
   bytes32 public constant MANAGER_ROLE = keccak256("MANAGER_ROLE");
@@ -352,7 +350,7 @@ contract PerpetualAtlanticVault is
     // Transfer rdpx from rdpx rdpxV2Core to perpetual vault
     IERC20WithBurn(addresses.rdpx).safeTransferFrom(
       addresses.rdpxV2Core,
-      addresses.perpetualAtlanticVaultLP,
+      addresses.perpetualAtlanticVaultLP,//@audit Cache this call?
       rdpxAmount
     );

@@ -586,8 +584,10 @@ contract PerpetualAtlanticVault is

   /// @dev Internal function to mint a option token
   function _mintOptionToken() private returns (uint256 tokenId) {
-    tokenId = _tokenIdCounter.current();
-    _tokenIdCounter.increment();
+    tokenId = _tokenIdCounter;
+    unchecked {
+      ++_tokenIdCounter;
+    }
     _mint(addresses.rdpxV2Core, tokenId);
   }

```



https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L138-L164

## [G-07]Since we are assigning local values to the struct, we shouldn't read the struct(state) in the same transactions, simply use the local variables(Save 427 Gas)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 285281    | 285281   | 285281 | 285281 |
| After | 284854 | 284854 | 284854 | 284854 |
```solidity
File: /contracts/reLP/ReLPContract.sol
138:    addresses = Addresses({
139:      tokenA: _tokenA,
140:      tokenB: _tokenB,
141:      pair: _pair,
        <Truncated>
145:      rdpxOracle: _rdpxOracle,
146:      ammFactory: _ammFactory,
147:      ammRouter: _ammRouter
148:    });


150:    IERC20WithBurn(addresses.pair).safeApprove(
151:      addresses.ammRouter,
152:      type(uint256).max
153:    );


155:    IERC20WithBurn(addresses.tokenA).safeApprove(
156:      addresses.ammRouter,
157:      type(uint256).max
158:    );


160:    IERC20WithBurn(addresses.tokenB).safeApprove(
161:      addresses.ammRouter,
162:      type(uint256).max
163:    );
164:  }
```

```diff

-    IERC20WithBurn(addresses.pair).safeApprove(
-      addresses.ammRouter,
+    IERC20WithBurn(_pair).safeApprove(
+      _ammRouter,
       type(uint256).max
     );

-    IERC20WithBurn(addresses.tokenA).safeApprove(
-      addresses.ammRouter,
+    IERC20WithBurn(_tokenA).safeApprove(
+      _ammRouter,
       type(uint256).max
     );

-    IERC20WithBurn(addresses.tokenB).safeApprove(
-      addresses.ammRouter,
+    IERC20WithBurn(_tokenB).safeApprove(
+      _ammRouter,
       type(uint256).max
     );
   }
```




## [G-08]when using storage instead of memory, we should cache any fields that need to be re-read in stack variables 

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L980-L985
```solidity
File: /contracts/core/RdpxV2Core.sol
980:    Delegate storage delegate = delegates[delegateId];
981:    _validate(delegate.owner == msg.sender, 9);

983:    amountWithdrawn = delegate.amount - delegate.activeCollateral;
984:    _validate(amountWithdrawn > 0, 15);
985:    delegate.amount = delegate.activeCollateral;
```

```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
index e28a65c..951ab24 100644
--- a/contracts/core/RdpxV2Core.sol
+++ b/contracts/core/RdpxV2Core.sol
@@ -979,10 +979,11 @@ contract RdpxV2Core is
     _validate(delegateId < delegates.length, 14);
     Delegate storage delegate = delegates[delegateId];
     _validate(delegate.owner == msg.sender, 9);
+    uint256 _activeCollateral = delegate.activeCollateral;

-    amountWithdrawn = delegate.amount - delegate.activeCollateral;
+    amountWithdrawn = delegate.amount - _activeCollateral;
     _validate(amountWithdrawn > 0, 15);
-    delegate.amount = delegate.activeCollateral;
+    delegate.amount = _activeCollateral;
```

## [G-09]Leverage  dot notation method for struct assignment

We have a few methods we can use when assigning values to struct

- Type({field1: value1, field2: value2});
- Type(value1,value2);
- dot notation(structObject.field = value1);

```solidity
struct Example{
    uint a;
    uint b;
  }
  Example public example;

  function sample1(uint value1, uint value2) external {
    example.a = value1;
    example.b = value2;
  }
  function sample2(uint value1,uint value2) external{
      example = Example(value1, value2);
    }

  function sample3(uint value1, uint value2) external{
    example = Example({
      a : value1,
      b : value2
    });
  }

```

When we use the first two examples(sample1,sample2), the values are directly written to storage variable(example), however using the last method(sample3)
the compiler will allocate some memory to store the struct instance first before writing it to storage
Example({a:value1,b:value2}) will create new Example in memory to store the values a and b. Then once we have this in memory we need to write it to the storage var example.
That said, we can see where the gas difference would come from, the first two writes the values directly to storage while the last one needs to copy to memory first.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L93-L101

### [G-09-01]Refactor the struct assignment style(Save 159 Gas)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 156876    | 156876   | 156876 | 156876 |
| After  | 156717    | 156717   | 156717 | 156717 |
```solidity
File: /contracts/amo/UniV2LiquidityAmo.sol
93:    addresses = Addresses({
94:      tokenA: _tokenA,
95:      tokenB: _tokenB,
96:      pair: _pair,
97:      rdpxV2Core: _rdpxV2Core,
98:      rdpxOracle: _rdpxOracle,
99:      ammFactory: _ammFactory,
100:      ammRouter: _ammRouter
101:    });
```


```diff
diff --git a/contracts/amo/UniV2LiquidityAmo.sol b/contracts/amo/UniV2LiquidityAmo.sol
index ce92d43..2381841 100644
--- a/contracts/amo/UniV2LiquidityAmo.sol
+++ b/contracts/amo/UniV2LiquidityAmo.sol
@@ -90,15 +90,13 @@ contract UniV2LiquidityAMO is AccessControl {
         _ammRouter != address(0),
       "reLPContract: address cannot be 0"
     );
-    addresses = Addresses({
-      tokenA: _tokenA,
-      tokenB: _tokenB,
-      pair: _pair,
-      rdpxV2Core: _rdpxV2Core,
-      rdpxOracle: _rdpxOracle,
-      ammFactory: _ammFactory,
-      ammRouter: _ammRouter
-    });
+    addresses.tokenA = _tokenA;
+    addresses.tokenB = _tokenB;
+    addresses.pair = _pair;
+    addresses.rdpxV2Core = _rdpxV2Core;
+    addresses.rdpxOracle = _rdpxOracle;
+    addresses.ammFactory = _ammFactory;
+    addresses.ammRouter = _ammRouter;
   }
```


https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L198-L206

### [G-09-02]PerpetualAtlanticVault.sol.setAddresses(): (Save 196 Gas on average)

|        | Min    | Average | Median | Max    |
| ------ | ------ | ------- | ------ | ------ |
| Before | 71992 | 185092  | 187992 | 187992 |
| After  | 71796 | 184896  | 187796 | 187796 |
|        |        |         |        |        |
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
198:    addresses = Addresses({
199:      optionPricing: _optionPricing,
200:      assetPriceOracle: _assetPriceOracle,
201:      volatilityOracle: _volatilityOracle,
202:      feeDistributor: _feeDistributor,
203:      rdpx: _rdpx,
204:      perpetualAtlanticVaultLP: _perpetualAtlanticVaultLP,
205:      rdpxV2Core: _rdpxV2Core
206:    });
```

```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
index 88ac2b3..9aabe3d 100644
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
@@ -195,15 +195,15 @@ contract PerpetualAtlanticVault is
     _validate(_perpetualAtlanticVaultLP != address(0), 1);
     _validate(_rdpxV2Core != address(0), 1);

-    addresses = Addresses({
-      optionPricing: _optionPricing,
-      assetPriceOracle: _assetPriceOracle,
-      volatilityOracle: _volatilityOracle,
-      feeDistributor: _feeDistributor,
-      rdpx: _rdpx,
-      perpetualAtlanticVaultLP: _perpetualAtlanticVaultLP,
-      rdpxV2Core: _rdpxV2Core
-    });
+    addresses.optionPricing = _optionPricing;
+    addresses.assetPriceOracle = _assetPriceOracle;
+    addresses.volatilityOracle = _volatilityOracle;
+    addresses.feeDistributor = _feeDistributor;
+    addresses.rdpx = _rdpx;
+    addresses.perpetualAtlanticVaultLP = _perpetualAtlanticVaultLP;
+    addresses.rdpxV2Core = _rdpxV2Core;
+
```


https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L296-L300

### [G-09-03]PerpetualAtlanticVault.sol.purchase(): Save 90 on Gas on average

*`Test: Purchase(uint256,address)(uint256)`*

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 312368    | 333657   | 333657 | 354946 |
| After  | 312278    | 333567   | 333567 | 354856 |

*`Test: Purchase(uint256,address)(uint256,uint256)`*

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 17877    | 256930   | 221256 | 359646 |
| After | 17877 | 256847 | 221166 | 359556 |


```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
296:    optionPositions[tokenId] = OptionPosition({
297:      strike: strike,
298:      amount: amount,
299:      positionId: tokenId
300:    });
```

```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
index 88ac2b3..83be049 100644
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
@@ -293,11 +293,11 @@ contract PerpetualAtlanticVault is

     // Mint the option tokens
     tokenId = _mintOptionToken();
-    optionPositions[tokenId] = OptionPosition({
-      strike: strike,
-      amount: amount,
-      positionId: tokenId
-    });
+
+    optionPositions[tokenId].strike = strike;
+    optionPositions[tokenId].amount = amount;
+    optionPositions[tokenId].positionId = tokenId;
+

```


https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L138-L148

### [G-09-04]ReLPContract.sol.setAddresses(Save 238 Gas) 

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 285281    | 285281   | 285281 | 285281 |
| After | 285043 | 285043 | 285043 | 285043 |
```solidity
File: /contracts/reLP/ReLPContract.sol
138:    addresses = Addresses({
139:      tokenA: _tokenA,
140:      tokenB: _tokenB,
141:      pair: _pair,
142:      rdpxV2Core: _rdpxV2Core,
143:      tokenAReserve: _tokenAReserve,
144:      amo: _amo,
145:      rdpxOracle: _rdpxOracle,
146:      ammFactory: _ammFactory,
147:      ammRouter: _ammRouter
148:    });
```

```diff
-    addresses = Addresses({
-      tokenA: _tokenA,
-      tokenB: _tokenB,
-      pair: _pair,
-      rdpxV2Core: _rdpxV2Core,
-      tokenAReserve: _tokenAReserve,
-      amo: _amo,
-      rdpxOracle: _rdpxOracle,
-      ammFactory: _ammFactory,
-      ammRouter: _ammRouter
-    });
+    addresses.tokenA = _tokenA;
+    addresses.tokenB = _tokenB;
+    addresses.pair = _pair;
+    addresses.rdpxV2Core = _rdpxV2Core;
+    addresses.tokenAReserve = _tokenAReserve;
+    addresses.amo = _amo;
+    addresses.rdpxOracle = _rdpxOracle;
+    addresses.ammFactory = _ammFactory;
+    addresses.ammRouter = _ammRouter;
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L327-L338

### [G-09-05]RdpxV2Core.sol:setAddresses(Save 268 Gas) 

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 33245    | 298196   | 330052 | 330052 |
| After | 33245 | 297928 | 329769 | 329769 |
```solidity
File: /contracts/core/RdpxV2Core.sol
327:    addresses = Addresses({
328:      dopexAMMRouter: _dopexAMMRouter,
329:      dpxEthCurvePool: _dpxEthCurvePool,
330:      rdpxDecayingBonds: _rdpxDecayingBonds,
331:      perpetualAtlanticVault: _perpetualAtlanticVault,
332:      perpetualAtlanticVaultLP: _perpetualAtlanticVaultLP,
333:      rdpxReserve: _rdpxReserve,
334:      rdpxV2ReceiptToken: _rdpxV2ReceiptToken,
335:      feeDistributor: _feeDistributor,
336:      reLPContract: _reLPContract,
337:      receiptTokenBonds: _receiptTokenBonds
338:    });
```


```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
index e28a65c..35a19e0 100644
--- a/contracts/core/RdpxV2Core.sol
+++ b/contracts/core/RdpxV2Core.sol
@@ -324,18 +324,17 @@ contract RdpxV2Core is
     _validate(_reLPContract != address(0), 17);
     _validate(_receiptTokenBonds != address(0), 17);

-    addresses = Addresses({
-      dopexAMMRouter: _dopexAMMRouter,
-      dpxEthCurvePool: _dpxEthCurvePool,
-      rdpxDecayingBonds: _rdpxDecayingBonds,
-      perpetualAtlanticVault: _perpetualAtlanticVault,
-      perpetualAtlanticVaultLP: _perpetualAtlanticVaultLP,
-      rdpxReserve: _rdpxReserve,
-      rdpxV2ReceiptToken: _rdpxV2ReceiptToken,
-      feeDistributor: _feeDistributor,
-      reLPContract: _reLPContract,
-      receiptTokenBonds: _receiptTokenBonds
-    });
+    addresses.dopexAMMRouter = _dopexAMMRouter;
+    addresses.dpxEthCurvePool = _dpxEthCurvePool;
+    addresses.rdpxDecayingBonds = _rdpxDecayingBonds;
+    addresses.perpetualAtlanticVault = _perpetualAtlanticVault;
+    addresses.perpetualAtlanticVaultLP = _perpetualAtlanticVaultLP;
+    addresses.rdpxReserve = _rdpxReserve;
+    addresses.rdpxV2ReceiptToken = _rdpxV2ReceiptToken;
+    addresses.feeDistributor = _feeDistributor;
+    addresses.reLPContract = _reLPContract;
+    addresses.receiptTokenBonds = _receiptTokenBonds;
+
     IERC20WithBurn(weth).approve(
       addresses.perpetualAtlanticVault,
       type(uint256).max
```

## [G-10]We can cache some gas intensive operations

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L372-L396

### [G-10-01]PerpetualAtlanticVault.sol.payFunding(): Cache the call `totalFundingForEpoch[latestFundingPaymentPointer]`(Save 669 Gas on average)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 3578    | 27125   | 34705 | 34705 |
| After | 3578 | 26456 | 33980 | 33980 |
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
372:  function payFunding() external onlyRole(RDPXV2CORE_ROLE) returns (uint256) {

382:    collateralToken.safeTransferFrom(
383:      addresses.rdpxV2Core,
384:      address(this),
385:      totalFundingForEpoch[latestFundingPaymentPointer]
386:    );
387:    _updateFundingRate(totalFundingForEpoch[latestFundingPaymentPointer]);


389:    emit PayFunding(
390:      msg.sender,
391:      totalFundingForEpoch[latestFundingPaymentPointer],
392:      latestFundingPaymentPointer
393:    );

395:    return (totalFundingForEpoch[latestFundingPaymentPointer]);
396:  }
```

```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
index 88ac2b3..16fb8a0 100644
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
@@ -378,21 +378,22 @@ contract PerpetualAtlanticVault is
         fundingPaymentsAccountedFor[latestFundingPaymentPointer],
       6
     );
+    uint256 _totalFundingForEpoch = totalFundingForEpoch[latestFundingPaymentPointer];

     collateralToken.safeTransferFrom(
       addresses.rdpxV2Core,
       address(this),
-      totalFundingForEpoch[latestFundingPaymentPointer]
+      _totalFundingForEpoch
     );
-    _updateFundingRate(totalFundingForEpoch[latestFundingPaymentPointer]);
+    _updateFundingRate(_totalFundingForEpoch);

     emit PayFunding(
       msg.sender,
-      totalFundingForEpoch[latestFundingPaymentPointer],
+      _totalFundingForEpoch,
       latestFundingPaymentPointer
     );

-    return (totalFundingForEpoch[latestFundingPaymentPointer]);
+    return (_totalFundingForEpoch);
   }

```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L502-L524

### [G-10-02]We can cache some operations instead of repeatedly doing them(Save 490 Gas on average)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 11472    | 33743   | 39946 | 53946 |
| After | 10982 | 33253 | 39456 | 53456 |
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
510:    collateralToken.safeTransfer(
511:      addresses.perpetualAtlanticVaultLP,
512:      (currentFundingRate * (block.timestamp - startTime)) / 1e18
513:    );

515:    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addProceeds(
516:      (currentFundingRate * (block.timestamp - startTime)) / 1e18
517:    );

519:    emit FundingPaid(
520:      msg.sender,
521:      ((currentFundingRate * (block.timestamp - startTime)) / 1e18),
522:      latestFundingPaymentPointer
523:    );
524:  }
```


```diff
+    uint256 _amount = (currentFundingRate * (block.timestamp - startTime)) / 1e18;
+
     collateralToken.safeTransfer(
       addresses.perpetualAtlanticVaultLP,
-      (currentFundingRate * (block.timestamp - startTime)) / 1e18
+      _amount
     );

     IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addProceeds(
-      (currentFundingRate * (block.timestamp - startTime)) / 1e18
+      _amount
     );

     emit FundingPaid(
       msg.sender,
-      ((currentFundingRate * (block.timestamp - startTime)) / 1e18),
+      _amount,
       latestFundingPaymentPointer
     );
   }
```



https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L372-L396

### [G-10-03]Cache `latestFundingPaymentPointer` in memory (Save  291 Gas)

**Not found by bot**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 3578    | 27125   | 34705 | 34705 |
| After | 3581 | 26834 | 34390 | 34390 |
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
372:  function payFunding() external onlyRole(RDPXV2CORE_ROLE) returns (uint256) {

376:    _validate(
377:      totalActiveOptions ==
378:        fundingPaymentsAccountedFor[latestFundingPaymentPointer],
379:      6
380:    );

382:    collateralToken.safeTransferFrom(
383:      addresses.rdpxV2Core,
384:      address(this),
385:      totalFundingForEpoch[latestFundingPaymentPointer]
386:    );
387:    _updateFundingRate(totalFundingForEpoch[latestFundingPaymentPointer]);

389:    emit PayFunding(
390:      msg.sender,
391:      totalFundingForEpoch[latestFundingPaymentPointer],
392:      latestFundingPaymentPointer
393:    );

395:    return (totalFundingForEpoch[latestFundingPaymentPointer]);
396:  }
```

```diff
+    uint256 _latestFundingPaymentPointer = latestFundingPaymentPointer;
     _validate(
       totalActiveOptions ==
-        fundingPaymentsAccountedFor[latestFundingPaymentPointer],
+        fundingPaymentsAccountedFor[_latestFundingPaymentPointer],
       6
     );

+    uint256 __totalFundingForEpoch = totalFundingForEpoch[_latestFundingPaymentPointer];
     collateralToken.safeTransferFrom(
       addresses.rdpxV2Core,
       address(this),
-      totalFundingForEpoch[latestFundingPaymentPointer]
+      __totalFundingForEpoch
     );
-    _updateFundingRate(totalFundingForEpoch[latestFundingPaymentPointer]);
+    _updateFundingRate(__totalFundingForEpoch);

     emit PayFunding(
       msg.sender,
-      totalFundingForEpoch[latestFundingPaymentPointer],
-      latestFundingPaymentPointer
+      __totalFundingForEpoch,
+      _latestFundingPaymentPointer
     );

-    return (totalFundingForEpoch[latestFundingPaymentPointer]);
+    return (__totalFundingForEpoch);
   }

```


https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L966-L967

### [G-10-04]We don't have to do `delegates.length - 1` twice especially because we are reading from storage(Save 136 Gas on average)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 648    | 85946   | 77526 | 158526 |
| After | 648 | 85810 | 77350 | 158350 |
```solidity
File: /contracts/core/RdpxV2Core.sol
966:    emit LogAddToDelegate(_amount, _fee, delegates.length - 1);
967:    return (delegates.length - 1);
```

```diff

-    emit LogAddToDelegate(_amount, _fee, delegates.length - 1);
-    return (delegates.length - 1);
+    uint256 _delegateLengthEmit = delegates.length - 1;
+
+    emit LogAddToDelegate(_amount, _fee, _delegateLengthEmit);
+    return (_delegateLengthEmit);
   }

```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L315-L369

### [G-10-05]PerpetualAtlanticVault.sol.settle(): `addresses.perpetualAtlanticVaultLP` should be cached (Save 139 Gas)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 17113    | 60186   | 56770 | 127485 |
| After | 17234 | 60047 | 56664 | 127152 |
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
315:  function settle(

328:    for (uint256 i = 0; i < optionIds.length; i++) {

346:    // Transfer collateral token from perpetual vault to rdpx rdpxV2Core
347:    collateralToken.safeTransferFrom(
348:      addresses.perpetualAtlanticVaultLP,
349:      addresses.rdpxV2Core,
350:      ethAmount
351:    );
352:    // Transfer rdpx from rdpx rdpxV2Core to perpetual vault
353:    IERC20WithBurn(addresses.rdpx).safeTransferFrom(
354:      addresses.rdpxV2Core,
355:      addresses.perpetualAtlanticVaultLP,
356:      rdpxAmount
357:    );

359:    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).subtractLoss(
360:      ethAmount
361:    );
362:    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP)
363:      .unlockLiquidity(ethAmount);
364:    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addRdpx(
365:      rdpxAmount
366:    );
```


```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
index 88ac2b3..941da6b 100644
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
@@ -324,6 +324,7 @@ contract PerpetualAtlanticVault is
     _isEligibleSender();

     updateFunding();
+    address _perpetualAtlanticVaultLP = addresses.perpetualAtlanticVaultLP;

     for (uint256 i = 0; i < optionIds.length; i++) {
       uint256 strike = optionPositions[optionIds[i]].strike;
@@ -345,23 +346,23 @@ contract PerpetualAtlanticVault is

     // Transfer collateral token from perpetual vault to rdpx rdpxV2Core
     collateralToken.safeTransferFrom(
-      addresses.perpetualAtlanticVaultLP,
+      _perpetualAtlanticVaultLP,
       addresses.rdpxV2Core,
       ethAmount
     );
     // Transfer rdpx from rdpx rdpxV2Core to perpetual vault
     IERC20WithBurn(addresses.rdpx).safeTransferFrom(
       addresses.rdpxV2Core,
-      addresses.perpetualAtlanticVaultLP,
+      _perpetualAtlanticVaultLP,
       rdpxAmount
     );

-    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).subtractLoss(
+    IPerpetualAtlanticVaultLP(_perpetualAtlanticVaultLP).subtractLoss(
       ethAmount
     );
-    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP)
+    IPerpetualAtlanticVaultLP(_perpetualAtlanticVaultLP)
       .unlockLiquidity(ethAmount);
-    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addRdpx(
+    IPerpetualAtlanticVaultLP(_perpetualAtlanticVaultLP).addRdpx(
       rdpxAmount
     );

```

## [G-11]We can avoid reading from state here

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L113-L124
**This is a different finding from the bot**
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
113:  constructor(
114:    string memory _name,
115:    string memory _symbol,
116:    address _collateralToken,
117:    uint256 _gensis
118:  ) ERC721(_name, _symbol) {
119:    _validate(_collateralToken != address(0), 1);

121:    collateralToken = IERC20WithBurn(_collateralToken);
122:    underlyingSymbol = collateralToken.symbol();
123:    collateralPrecision = 10 ** collateralToken.decimals();
124:    genesis = _gensis;
```


```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
index 88ac2b3..4a58e0e 100644
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
@@ -119,8 +119,8 @@ contract PerpetualAtlanticVault is
     _validate(_collateralToken != address(0), 1);

     collateralToken = IERC20WithBurn(_collateralToken);
-    underlyingSymbol = collateralToken.symbol();
-    collateralPrecision = 10 ** collateralToken.decimals();
+    underlyingSymbol = IERC20WithBurn(_collateralToken).symbol();
+    collateralPrecision = 10 ** IERC20WithBurn(_collateralToken).decimals();
     genesis = _gensis;
```


## [G-12]Since we have cached values, we should reference them instead of making a state read

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L101-L107
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVaultLP.sol
101:    rdpx = _rdpx;
102:    collateral = ERC20(_collateral);

106:    collateral.approve(_perpetualAtlanticVault, type(uint256).max);
107:    ERC20(rdpx).approve(_perpetualAtlanticVault, type(uint256).max);
```

Note, we have the values , `_rdpx and ERC20(_collateral)`, instead of calling their state counterparts `rpdx and collateral` we should utilize the local ones(Even better we could have just changed them into immutable)

```diff
-    collateral.approve(_perpetualAtlanticVault, type(uint256).max);
-    ERC20(rdpx).approve(_perpetualAtlanticVault, type(uint256).max);
+    ERC20(_collateral).approve(_perpetualAtlanticVault, type(uint256).max);
+    ERC20(_rdpx).approve(_perpetualAtlanticVault, type(uint256).max);
```


## [G-13]Unused libraries

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L10
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVaultLP.sol
10:import { Strings } from "@openzeppelin/contracts/utils/Strings.sol";
```

## [G-14]No need to initialize `state variables` with their default values

If a variable is not set/initialized, it is assumed to have the default value (0, false, 0x0 etc depending on the data type). If you explicitly initialize it with its default value, you increase deployment cost and size

Initializing variables with their default values requires additional bytecode instructions to set those values explicitly during deployment. By excluding these instructions, the overall size of the compiled contract's bytecode can be reduced.

The gas cost difference (remix and foundry) is 2206 gas per variable

We get 3 extra opcodes when we initialize it.
```
PUSH1 00 --> 3 gas
DUP1     --> 3 gas
SSTORE   --> 2200 gas
```
The above explains the `2206 gas` difference.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L89
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
89:  uint256 public latestFundingPaymentPointer = 0;
```


## Conclusion

It is important to emphasize that the provided recommendations aim to enhance the efficiency of the code without compromising its readability. We understand the value of maintainable and easily understandable code to both developers and auditors.

As you proceed with implementing the suggested optimizations, please exercise caution and be diligent in conducting thorough testing. It is crucial to ensure that the changes are not introducing any new vulnerabilities and that the desired performance improvements are achieved. Review code changes, and perform thorough testing to validate the effectiveness and security of the refactored code.

Should you have any questions or need further assistance, please don't hesitate to reach out.
