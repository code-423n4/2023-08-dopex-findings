# GAS OPTIMIZATION

The final gas calculations are determined based on the ``OPCODEs`` and the ``sample tests``. The ``OPCODEs`` are the basic instructions that are used to execute code in Solidity. The sample test is a piece of code that is used to test the gas consumption of a function.

The gas costs of each ``OPCODE`` are defined in the Solidity documentation. The gas costs of the sample test can be calculated by ``running`` the ``test`` and ``measuring`` the ``amount of gas`` that is used.

- [Struct can be packed in fewer slots - Saves ``4000 GAS``](#g-1-struct-can-be-packed-in-fewer-slots)
  - [``owner``,``expiry`` can be packed within same slot : Saves ``2000 GAS``, ``1 SLOT``](#ownerexpiry-can-be-packed-within-same-slot--saves-2000-gas-1-slot)
  - [``_amount0Min`` and ``_amount1Min`` can be packed to same slot : Saves ``2000 GAS`` , ``1 SLOT``](#_amount0min-and-_amount1min-can-be-packed-to-same-slot--saves-2000-gas--1-slot)

- [The result of a function calls should be cached rather than re-calling the function - Saves ``27912 GAS``](#g-2-the-result-of-a-function-calls-should-be-cached-rather-than-re-calling-the-function)
  - [``getEthPrice()``,``getDpxEthPrice()``, ``getRdpxPrice()`` functions should be cached : Saves ``300 GAS``](#getethpricegetdpxethprice-getrdpxprice-functions-should-be-cached--saves-300-gas)
  - [``nextFundingPaymentTimestamp()`` function should be cached : Saves ``27612 GAS``, ``9 Instances``](#nextfundingpaymenttimestamp-function-should-be-cached--saves-27612-gas-9-instances)
- [Using ``calldata`` to optimize gas costs for ``Read-Only`` external function arguments - Saves ``1420 GAS`` ](#g-3-using-calldata-to-optimize-gas-costs-for-read-only-external-function-arguments)
  - [``_assetSymbol``, ``_assetSymbol``, and ``optionIds``can be declared in ``calldata`` instead of ``memory`` : Saves ``852 GAS``, ``3 Instances``](#_assetsymbol-_assetsymbol-and-optionidscan-be-declared-in-calldata-instead-of-memory--saves-852-gas-3-instances)
  - [``optionIds``, ``strikes`` can be declared in ``calldata`` instead of ``memory``  : Saves ``568 GAS``, `` 2 Instance``](#optionids-strikes-can-be-declared-in-calldata-instead-of-memory---saves-568-gas--2-instance)
- [Using storage instead of memory for structs/arrays saves gas - Saves ``4200 GAS``](#g-4-using-storage-instead-of-memory-for-structsarrays-saves-gas)
  - [``delegatePosition`` variable can be stored in ``storage`` instead of ``memory`` : Saves ``4200 GAS``](#delegateposition-variable-can-be-stored-in-storage-instead-of-memory--saves-4200-gas)
- [State variables can be packed into fewer storage slots - Saves ``8000 GAS``](#g-5-state-variables-can-be-packed-into-fewer-storage-slots)
  - [``slippageTolerance `` variable can be uint96 instead of uint256 : Saves ``2000 GAS``, ``1 SLOT``](#slippagetolerance--variable-can-be-uint96-instead-of-uint256--saves-2000-gas-1-slot)
  - [``liquiditySlippageTolerance ``,``slippageTolerance``  variable can be uint128 instead of uint256 : Saves ``2000 GAS``, ``1 SLOT``](#liquidityslippagetolerance-slippagetolerance--variable-can-be-uint128-instead-of-uint256--saves-2000-gas-1-slot)
  - [``rdpxBurnPercentage ``,``rdpxFeePercentage ``,``slippageTolerance ``,``liquiditySlippageTolerance `` variable can be uint128 instead of uint256 : Saves ``4000 GAS``, ``2 SLOT``](#rdpxburnpercentage-rdpxfeepercentage-slippagetolerance-liquidityslippagetolerance--variable-can-be-uint128-instead-of-uint256--saves-4000-gas-2-slot)
- [State variables should not be ``initialized`` with ``default values`` Saves 2500 GAS](#g-6-state-variables-should-not-be-initialized-with-default-values)
- [State variables should be cached in stack variables rather than re-reading them from storage - Saves ``5500 GAS`` , ``55 SLOD```](#g-7-state-variables-should-be-cached-in-stack-variables-rather-than-re-reading-them-from-storage)
  - [``addresses.dpxEthCurvePool``, ``addresses.rdpxReserve ``,``addresses.rdpxDecayingBonds``,``reserveAsset[reservesIndex["RDPX"]].tokenAddress``, ``addresses.perpetualAtlanticVault``,``delegates.length``,``delegate.activeCollateral``,``reserveAsset[i].tokenAddress``,``addresses.receiptTokenBonds``, ``addresses.perpetualAtlanticVault``   should be cached : Saves ``1300 GAS`` , ``13 SLOD``](#addressesdpxethcurvepool-addressesrdpxreserve-addressesrdpxdecayingbondsreserveassetreservesindexrdpxtokenaddress-addressesperpetualatlanticvaultdelegateslengthdelegateactivecollateralreserveassetitokenaddressaddressesreceipttokenbonds-addressesperpetualatlanticvault---should-be-cached--saves-1300-gas--13-slod)
  - [``addresses.perpetualAtlanticVaultLP`` ,``latestFundingPaymentPointer``, ``totalFundingForEpoch[latestFundingPaymentPointer]``, ``latestFundingPaymentPointer`` ,``lastUpdateTime`` ,``addresses.perpetualAtlanticVaultLP``, ``roundingPrecision`` , ``fundDuration`` variables should be cached : Saves ``1800 GAS`` , ``18 SLOD``](#addressesperpetualatlanticvaultlp-latestfundingpaymentpointer-totalfundingforepochlatestfundingpaymentpointer-latestfundingpaymentpointer-lastupdatetime-addressesperpetualatlanticvaultlp-roundingprecision--fundduration-variables-should-be-cached--saves-1800-gas--18-slod)
  - [``_totalCollateral ``, ``_rdpxCollateral `` should be cached : Saves ``300 GAS``, ``3 SLOD``](#_totalcollateral--_rdpxcollateral--should-be-cached--saves-300-gas-3-slod)
  - [``addresses.ammRouter``,``liquiditySlippageTolerance`` , ``addresses.tokenA`` , ``addresses.tokenB``,``addresses.pair`` Should be cached : Saves ``2100 GAS``, ``21 SLOD``](#addressesammrouterliquidityslippagetolerance--addressestokena--addressestokenbaddressespair-should-be-cached--saves-2100-gas-21-slod)
- [Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead](#g-8-usage-of-uintsints-smaller-than-32-bytes-256-bits-incurs-overhead)
- [With assembly, ``.call (bool success)`` transfer can be done gas-optimized](#g-9-with-assembly-call-bool-success-transfer-can-be-done-gas-optimized)
- [Use assembly to check for address(0)](#g--use-assembly-to-check-for-address0)
- [State variables only set in the constructor should be declared immutable - Saves ``6000 GAS``](#g-10-state-variables-only-set-in-the-constructor-should-be-declared-immutable)
- [Normal ``state variables`` are more gas efficient than ``Counters`` library]()

##

##

## [G-1] Struct can be packed in fewer slots 

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct.

### ``owner``,``expiry`` can be packed within same slot : Saves ``2000 GAS``, ``1 SLOT``

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L40-L44

``uint96`` can store epoch time for approximately ``792281625142`` years. So ``uitn96`` is more than enough to store ``timestamps`` 

```diff
FILE: Breadcrumbs2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol

40: struct Bond {
41:    address owner;
- 42:    uint256 expiry;
+ 42:    uint96 expiry;
43:    uint256 rdpxAmount;
44:  }

```

### ``_amount0Min`` and ``_amount1Min`` can be packed to same slot : Saves ``2000 GAS`` , ``1 SLOT``

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L50-L60

``_amount0Min`` and ``_amount1Min`` store the minimum allowed values, which are not very large. Therefore, a ``uint128`` is more than enough to store these values accurately.

```diff
FILE: 2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol

50: struct AddLiquidityParams {
51:    address _tokenA;
52:    address _tokenB;
53:    int24 _tickLower;
54:    int24 _tickUpper;
55:    uint24 _fee;
56:    uint256 _amount0Desired;
57:    uint256 _amount1Desired;
- 58:    uint256 _amount0Min;
+ 58:    uint128 _amount0Min;
- 59:    uint256 _amount1Min;
+ 59:    uint128 _amount1Min;

60:  }

```

##

## [G-2] The result of a function calls should be cached rather than re-calling the function

External calls are expensive

### ``getEthPrice()``,``getDpxEthPrice()``, ``getRdpxPrice()`` functions should be cached : Saves ``300 GAS``


https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L546-L549

```diff
FILE: 2023-08-dopex/contracts/core/RdpxV2Core.sol

+ uint256 DpxEthPrice_ = getDpxEthPrice();
+ uint256 ethPrice_ = getEthPrice() ;
544: // calculate minimum amount out
545:    uint256 minOut = _ethToDpxEth
- 546:      ? (((_amount * getDpxEthPrice()) / 1e8) -
+ 546:      ? (((_amount * DpxEthPrice_ ) / 1e8) -
- 547:        (((_amount * getDpxEthPrice()) * slippageTolerance) / 1e16))
+ 547:        (((_amount * DpxEthPrice_) * slippageTolerance) / 1e16))
- 548:      : (((_amount * getEthPrice()) / 1e8) -
+ 548:      : (((_amount * ethPrice_ ) / 1e8) -
- 549:        (((_amount * getEthPrice()) * slippageTolerance) / 1e16));
+ 549:        (((_amount * ethPrice_ ) * slippageTolerance) / 1e16));


+ uint256 rdpxPrice_ = getEthPrice() ;
668: // calculate extra rdpx to withdraw to compensate for discount
- 669:      uint256 rdpxAmountInWeth = (_rdpxAmount * getRdpxPrice()) / 1e8;
+ 669:      uint256 rdpxAmountInWeth = (_rdpxAmount * rdpxPrice_) / 1e8;
670:      uint256 discountReceivedInWeth = _bondAmount -
671:        _wethAmount -
672:        rdpxAmountInWeth;
673:      uint256 extraRdpxToWithdraw = (discountReceivedInWeth * 1e8) /
- 674:        getRdpxPrice();
+ 674:        rdpxPrice_ ;

```

### ``nextFundingPaymentTimestamp()`` function should be cached : Saves ``27612 GAS``, ``9 Instances``

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L463

``nextFundingPaymentTimestamp()`` function fetching ``genesis``,``latestFundingPaymentPointer``,``fundingDuration`` storage variables and doing some calculations this will cost  huge volume of gas. Gas cost as per test ``3068 GAS`` per instance 

```diff
FILE: Breadcrumbs2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol

+ uint256 nextFundingPaymentTimestamp_ = nextFundingPaymentTimestamp() ;
- 463: while (block.timestamp >= nextFundingPaymentTimestamp()) {
+ 463: while (block.timestamp >= nextFundingPaymentTimestamp_ ) {
-      if (lastUpdateTime < nextFundingPaymentTimestamp()) {
+      if (lastUpdateTime < nextFundingPaymentTimestamp_) {
        uint256 currentFundingRate = fundingRates[latestFundingPaymentPointer];

        uint256 startTime = lastUpdateTime == 0
-          ? (nextFundingPaymentTimestamp() - fundingDuration)
+          ? (nextFundingPaymentTimestamp_ - fundingDuration)
          : lastUpdateTime;

-        lastUpdateTime = nextFundingPaymentTimestamp();
+        lastUpdateTime = nextFundingPaymentTimestamp_;

        collateralToken.safeTransfer(
          addresses.perpetualAtlanticVaultLP,
-          (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
+          (currentFundingRate * (nextFundingPaymentTimestamp_ - startTime)) /
            1e18
        );

        IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP)
          .addProceeds(
-            (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
+            (currentFundingRate * (nextFundingPaymentTimestamp_ - startTime)) /
              1e18
          );

        emit FundingPaid(
          msg.sender,
          ((currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
            1e18),
          latestFundingPaymentPointer
        );
      }



+ uint256 nextFundingPaymentTimestamp_ = nextFundingPaymentTimestamp() ;
- 597:  if (lastUpdateTime > nextFundingPaymentTimestamp() - fundingDuration) {
+ 597:  if (lastUpdateTime > nextFundingPaymentTimestamp_ - fundingDuration) {
        startTime = lastUpdateTime;
      } else {
-        startTime = nextFundingPaymentTimestamp() - fundingDuration;
+        startTime = nextFundingPaymentTimestamp_ - fundingDuration;
      }
-      uint256 endTime = nextFundingPaymentTimestamp();
+      uint256 endTime = nextFundingPaymentTimestamp_;
      fundingRates[latestFundingPaymentPointer] =
        (amount * 1e18) /
        (endTime - startTime);
    } else {
      uint256 startTime = lastUpdateTime;
-      uint256 endTime = nextFundingPaymentTimestamp();
+      uint256 endTime = nextFundingPaymentTimestamp_;

```



##
## [G-3] Using ``calldata`` to optimize gas costs for ``Read-Only`` external function arguments 

``Calldata`` can be used for read-only arguments in external functions 

When a function with a ``memory`` array is called externally, the ``abi.decode ()``  step has to use a for-loop to copy each index of the ``calldata`` to the ``memory`` index. Each iteration of this for-loop costs at least ``60 gas`` (i.e. 60 * <mem_array>.length). Using ``calldata`` directly, obliviates the need for such a loop in the contract code and runtime execution. 

### ``_assetSymbol``, ``_assetSymbol``, and ``optionIds``can be declared in ``calldata`` instead of ``memory`` : Saves ``852 GAS``, ``3 Instances``

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L242

```diff
FILE: 2023-08-dopex/contracts/core/RdpxV2Core.sol

240: function addAssetTotokenReserves(
241:    address _asset,
- 242:    string memory _assetSymbol
+ 242:    string calldata _assetSymbol
243:  ) external onlyRole(DEFAULT_ADMIN_ROLE) {

270: function removeAssetFromtokenReserves(
- 271:    string memory _assetSymbol
+ 271:    string calldata _assetSymbol
272:   ) external onlyRole(DEFAULT_ADMIN_ROLE) {

764: function settle(
- 765:    uint256[] memory optionIds
+ 765:    uint256[] calldata optionIds
766:  )
767:    external

```

###  ``optionIds``, ``strikes`` can be declared in ``calldata`` instead of ``memory``  : Saves ``568 GAS``, `` 2 Instance``

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L315-L317

```diff
FILE: 2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol

315: function settle(
- 316:    uint256[] memory optionIds
+ 316:    uint256[] calldata optionIds
317:  )
318:    external

405:  function calculateFunding(
- 406:    uint256[] memory strikes
+ 406:    uint256[] calldata strikes
407:  ) external nonReentrant returns (uint256 fundingAmount) {



```
##

## [G-4] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a ``Gcoldsload (2100 gas)`` for each field of the ``struct/array``. If the fields are read from the new memory variable, they incur an additional ``MLOAD`` rather than a cheap stack read. Instead of declaring the variable with the ``memory`` keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct


###  ``delegatePosition`` variable can be stored in ``storage`` instead of ``memory`` : Saves ``4200 GAS``

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1272

```diff
FILE: 2023-08-dopex/contracts/core/RdpxV2Core.sol

- 1272: Delegate memory delegatePosition = delegates[_delegateId];
+ 1272: Delegate storage delegatePosition = delegates[_delegateId];

```
##

## [G-5] State variables can be packed into fewer storage slots

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper

### ``slippageTolerance `` variable can be uint96 instead of uint256 : Saves ``2000 GAS``, ``1 SLOT``

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L44-L54

``slippageTolerance`` is the maximum price difference a sender is willing to accept in order to execute their swap. Slippage tolerance typically does not exceed ``0.5% to 1%``. Therefore, a ``uint96`` is sufficient to store slippage tolerance


```diff
FILE: Breadcrumbs2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol

44: /// @notice  addresses of the contracts
45:  Addresses public addresses;
+ 51:  uint96 public slippageTolerance = 5e5; // 0.5%
46:
47:  /// @notice Precision used for prices, percentages and other calculations
48:  uint256 public constant DEFAULT_PRECISION = 1e8;
49:
50:  /// @notice The slippage tolernce in swaps in 1e8 precision
- 51:  uint256 public slippageTolerance = 5e5; // 0.5%

```

### ``liquiditySlippageTolerance ``,``slippageTolerance``  variable can be uint128 instead of uint256 : Saves ``2000 GAS``, ``1 SLOT``

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L72-L76

``slippageTolerance``,``liquiditySlippageTolerance `` is the maximum price difference a sender is willing to accept in order to execute their swap. Slippage tolerance typically does not exceed ``0.5% to 1%``. Therefore, a ``uint128`` is sufficient to store slippage tolerance and liquiditySlippageTolerance .


```diff
FILE: 2023-08-dopex/contracts/reLP/ReLPContract.sol

72: /// @notice liquidity slippage tolerance
- 73:  uint256 public liquiditySlippageTolerance = 5e5; // 0.5%
+ 73:  uint128 public liquiditySlippageTolerance = 5e5; // 0.5%
74:
75:  /// @notice The slippage tolernce in swaps in 1e8 precision
- 76:  uint256 public slippageTolerance = 5e5; // 0.5%
+ 76:  uint256 public slippageTolerance = 5e5; // 0.5%

```

### ``rdpxBurnPercentage ``,``rdpxFeePercentage ``,``slippageTolerance ``,``liquiditySlippageTolerance `` variable can be uint128 instead of uint256 : Saves ``4000 GAS``, ``2 SLOT``

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L93-L103

```diff
FILE: 2023-08-dopex/contracts/core/RdpxV2Core.sol

93: /// @notice The % of rdpx to burn while bonding
- 94:  uint256 public rdpxBurnPercentage = 50 * DEFAULT_PRECISION;
+ 94:  uint128 public rdpxBurnPercentage = 50 * DEFAULT_PRECISION;
95:
96:  /// @notice The % of rdpx sent to fee distributor while bonding
- 97:  uint256 public rdpxFeePercentage = 50 * DEFAULT_PRECISION;
+ 97:  uint128 public rdpxFeePercentage = 50 * DEFAULT_PRECISION;
98:
99:  /// @notice The slippage tolernce in swaps in 1e8 precision
- 100:  uint256 public slippageTolerance = 5e5; // 0.5%
+ 100:  uint128 public slippageTolerance = 5e5; // 0.5%
101:
102:  /// @notice Liquidity slippage tolerance
- 103:  uint256 public liquiditySlippageTolerance = 5e5; // 0.5%
+ 103:  uint128 public liquiditySlippageTolerance = 5e5; // 0.5%

```
##

## [G-6] State variables should not be ``initialized`` with ``default values``

If a state variable is not initialized with a default value, the default value of the type will be used. For example, a state variable of type uint256 will have the default value of 0 if it is not initialized with a default value. This will save gas because the default value of the type does not need to be stored in the blockchain. Saves ``2500 GAS``

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L89

```diff
FILE: Breadcrumbs2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol

- 89: uint256 public latestFundingPaymentPointer = 0;
+ 89: uint256 public latestFundingPaymentPointer;

```
##

## [G-7] State variables should be cached in stack variables rather than re-reading them from storage

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

### ``addresses.dpxEthCurvePool``, ``addresses.rdpxReserve ``,``addresses.rdpxDecayingBonds``,``reserveAsset[reservesIndex["RDPX"]].tokenAddress``, ``addresses.perpetualAtlanticVault``,``delegates.length``,``delegate.activeCollateral``,``reserveAsset[i].tokenAddress``,``addresses.receiptTokenBonds``, ``addresses.perpetualAtlanticVault``   should be cached : Saves ``1300 GAS`` , ``13 SLOD``

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L521-L530

```diff
FILE: 2023-08-dopex/contracts/core/RdpxV2Core.sol


+ address dpxEthCurvePool_ = addresses.dpxEthCurvePool ;
521: - IStableSwap dpxEthCurvePool = IStableSwap(addresses.dpxEthCurvePool);
+ IStableSwap dpxEthCurvePool = IStableSwap(dpxEthCurvePool_ );
    // First compute a reverse swapping of dpxETH to ETH to compute the amount of ETH required
    address coin0 = dpxEthCurvePool.coins(0);
    (uint256 a, uint256 b) = coin0 == weth ? (0, 1) : (1, 0);

    // validate the swap for peg functions
    if (validate) {
-      uint256 ethBalance = IStableSwap(addresses.dpxEthCurvePool).balances(a);
+      uint256 ethBalance = IStableSwap(dpxEthCurvePool_).balances(a);
-      uint256 dpxEthBalance = IStableSwap(addresses.dpxEthCurvePool).balances(
+      uint256 dpxEthBalance = IStableSwap(dpxEthCurvePool_).balances(


+   address rdpxReserve_ = addresses.rdpxReserve ;
630: if (_bondId != 0) {
+  address rdpxDecayingBonds_ = addresses.rdpxDecayingBonds ; 
 (, uint256 expiry, uint256 amount) = IRdpxDecayingBonds(
-        addresses.rdpxDecayingBonds
+        rdpxDecayingBonds_
      ).bonds(_bondId);

      _validate(amount >= _rdpxAmount, 1);
      _validate(expiry >= block.timestamp, 2);
      _validate(
-        IRdpxDecayingBonds(addresses.rdpxDecayingBonds).ownerOf(_bondId) ==
+        IRdpxDecayingBonds(rdpxDecayingBonds_).ownerOf(_bondId) ==
          msg.sender,
        9
      );

-      IRdpxDecayingBonds(addresses.rdpxDecayingBonds).decreaseAmount(
+      IRdpxDecayingBonds(rdpxDecayingBonds_).decreaseAmount(
        _bondId,
        amount - _rdpxAmount
      );

-      IRdpxReserve(addresses.rdpxReserve).withdraw(_rdpxAmount);
+      IRdpxReserve(rdpxReserve_).withdraw(_rdpxAmount);

      reserveAsset[reservesIndex["RDPX"]].tokenBalance += _rdpxAmount;
    } else {
+  address tokenAddress_ = reserveAsset[reservesIndex["RDPX"]].tokenAddress ;
      // Transfer rDPX and ETH token from user
-      IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress)
+      IERC20WithBurn(tokenAddress_)
        .safeTransferFrom(msg.sender, address(this), _rdpxAmount);

      // burn the rdpx
-      IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress).burn(
+      IERC20WithBurn(tokenAddress_).burn(
        (_rdpxAmount * rdpxBurnPercentage) / 1e10
      );

      // transfer the rdpx to the fee distributor
-      IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress)
+      IERC20WithBurn(tokenAddress_)
        .safeTransfer(
          addresses.feeDistributor,
          (_rdpxAmount * rdpxFeePercentage) / 1e10
        );

      // calculate extra rdpx to withdraw to compensate for discount
      uint256 rdpxAmountInWeth = (_rdpxAmount * getRdpxPrice()) / 1e8;
      uint256 discountReceivedInWeth = _bondAmount -
        _wethAmount -
        rdpxAmountInWeth;
      uint256 extraRdpxToWithdraw = (discountReceivedInWeth * 1e8) /
        getRdpxPrice();

      // withdraw the rdpx
-       IRdpxReserve(addresses.rdpxReserve).withdraw(
+       IRdpxReserve(rdpxReserve_).withdraw(
        _rdpxAmount + extraRdpxToWithdraw
      );

      reserveAsset[reservesIndex["RDPX"]].tokenBalance +=
        _rdpxAmount +
        extraRdpxToWithdraw;
    }
  }


795:  _whenNotPaused();
+  address perpetualAtlanticVault_ = addresses.perpetualAtlanticVault ;
-    uint256 pointer = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
+    uint256 pointer = IPerpetualAtlanticVault(perpetualAtlanticVault_)
      .latestFundingPaymentPointer();
    _validate(fundingPaidFor[pointer] == false, 16);

-     fundingAmount = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
+     fundingAmount = IPerpetualAtlanticVault(perpetualAtlanticVault_)



+ uint256 length_ = delegates.length ;
- 966: emit LogAddToDelegate(_amount, _fee, delegates.length - 1);
+ 966: emit LogAddToDelegate(_amount, _fee, length_ - 1);
-    return (delegates.length - 1);
+    return (length_ - 1);



+  activeCollateral_ = delegate.activeCollateral ;
- 983: amountWithdrawn = delegate.amount - delegate.activeCollateral;
+ 983: amountWithdrawn = delegate.amount - activeCollateral_;
984:    _validate(amountWithdrawn > 0, 15);
- 985:    delegate.amount = delegate.activeCollateral;
+ 985:    delegate.amount =  activeCollateral_;



+  address tokenAddress_ = reserveAsset[i].tokenAddress ;
- 997: uint256 balance = IERC20WithBurn(reserveAsset[i].tokenAddress).balanceOf(
+ 997: uint256 balance = IERC20WithBurn(tokenAddress_).balanceOf(
998:        address(this)
999:      );
1000:
- 1001:      if (weth == reserveAsset[i].tokenAddress) {
+ 1001:      if (weth == tokenAddress_ ) {



+  address receiptTokenBonds_ = addresses.receiptTokenBonds;
1025: _validate(
-       msg.sender == RdpxV2Bond(addresses.receiptTokenBonds).ownerOf(id),
+       msg.sender == RdpxV2Bond(receiptTokenBonds_).ownerOf(id),
      9
    );

    // Burn the bond token
    // Note requires an approval of the bond token to this contract
-    RdpxV2Bond(addresses.receiptTokenBonds).burn(id);
+    RdpxV2Bond(receiptTokenBonds_ ).burn(id);



+ address perpetualAtlanticVault_ = addresses.perpetualAtlanticVault ;
- 1189: uint256 strike = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
+ 1189: uint256 strike = IPerpetualAtlanticVault(perpetualAtlanticVault_)
      .roundUp(rdpxPrice - (rdpxPrice / 4)); // 25% below the current price

    uint256 timeToExpiry = IPerpetualAtlanticVault(
-      addresses.perpetualAtlanticVault
+      perpetualAtlanticVault_
    ).nextFundingPaymentTimestamp() - block.timestamp;
    if (putOptionsRequired) {
-      wethRequired += IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
+      wethRequired += IPerpetualAtlanticVault(perpetualAtlanticVault_)

```

### ``addresses.perpetualAtlanticVaultLP`` ,``latestFundingPaymentPointer``, ``totalFundingForEpoch[latestFundingPaymentPointer]``, ``latestFundingPaymentPointer`` ,``lastUpdateTime`` ,``addresses.perpetualAtlanticVaultLP``, ``roundingPrecision`` , ``fundDuration`` variables should be cached : Saves ``1800 GAS`` , ``18 SLOD``

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L359

```diff
FILE: Breadcrumbs2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol

+ address perpetualAtlanticVaultLP_ = addresses.perpetualAtlanticVaultLP ;
- 359: IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).subtractLoss(
+ 359: IPerpetualAtlanticVaultLP(perpetualAtlanticVaultLP_).subtractLoss(
360:      ethAmount
361:    );
- 362:    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP)
+ 362:    IPerpetualAtlanticVaultLP(aperpetualAtlanticVaultLP_)
363:      .unlockLiquidity(ethAmount);
- 364:    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addRdpx(
+ 364:    IPerpetualAtlanticVaultLP(aperpetualAtlanticVaultLP_).addRdpx(
365:      rdpxAmount
366:    );



+ uint256 latestFundingPaymentPointer_ = latestFundingPaymentPointer;
+ uint256 totalFundingForEpoch_ = totalFundingForEpoch[latestFundingPaymentPointer_] ;
382: collateralToken.safeTransferFrom(
      addresses.rdpxV2Core,
      address(this),
-      totalFundingForEpoch[latestFundingPaymentPointer]
+      totalFundingForEpoch_
    );
-    _updateFundingRate(totalFundingForEpoch[latestFundingPaymentPointer]);
+    _updateFundingRate(totalFundingForEpoch_);
    emit PayFunding(
      msg.sender,
-      totalFundingForEpoch[latestFundingPaymentPointer],
+      totalFundingForEpoch_,
-      latestFundingPaymentPointer
+      latestFundingPaymentPointer_
    );

-    return (totalFundingForEpoch[latestFundingPaymentPointer]);
+    return (totalFundingForEpoch_);


+ uint256 latestFundingPaymentPointer_ = latestFundingPaymentPointer ;
+ uint256 lastUpdateTime_ = lastUpdateTime ;
+ address perpetualAtlanticVaultLP_ = addresses.perpetualAtlanticVaultLP ;
- uint256 currentFundingRate = fundingRates[latestFundingPaymentPointer];
+ uint256 currentFundingRate = fundingRates[latestFundingPaymentPointer_];
-    uint256 startTime = lastUpdateTime == 0
+    uint256 startTime = lastUpdateTime_ == 0
      ? (nextFundingPaymentTimestamp() - fundingDuration)
-      : lastUpdateTime;
+      : lastUpdateTime_;
    lastUpdateTime = block.timestamp;

    collateralToken.safeTransfer(
-      addresses.perpetualAtlanticVaultLP,
+      perpetualAtlanticVaultLP_,
      (currentFundingRate * (block.timestamp - startTime)) / 1e18
    );

-    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addProceeds(
+    IPerpetualAtlanticVaultLP(perpetualAtlanticVaultLP_).addProceeds(
      (currentFundingRate * (block.timestamp - startTime)) / 1e18
    );

    emit FundingPaid(
      msg.sender,
      ((currentFundingRate * (block.timestamp - startTime)) / 1e18),
-       latestFundingPaymentPointer
+       latestFundingPaymentPointer_ 
    );


+ uint256 roundingPrecision_ = roundingPrecision ; 
- 577:  uint256 remainder = _strike % roundingPrecision;\
+ 577:  uint256 remainder = _strike % roundingPrecision_;
    if (remainder == 0) {
      return _strike;
    } else {
-      return _strike - remainder + roundingPrecision;
+      return _strike - remainder + roundingPrecision_ ;


+ uint256 latestFundingPaymentPointer_ = latestFundingPaymentPointer ;
+ uint256 fundingDuration_ = fundingDuration ;
- 595: if (fundingRates[latestFundingPaymentPointer] == 0) {
+ 595: if (fundingRates[latestFundingPaymentPointer_ ] == 0) {
      uint256 startTime;
-      if (lastUpdateTime > nextFundingPaymentTimestamp() - fundingDuration) {
+      if (lastUpdateTime > nextFundingPaymentTimestamp() - fundingDuration_) {
        startTime = lastUpdateTime;
      } else {
-        startTime = nextFundingPaymentTimestamp() - fundingDuration;
+        startTime = nextFundingPaymentTimestamp() - fundingDuration_ ;
      }
      uint256 endTime = nextFundingPaymentTimestamp();
-      fundingRates[latestFundingPaymentPointer] =
+      fundingRates[latestFundingPaymentPointer_ ] =
        (amount * 1e18) /
        (endTime - startTime);
    } else {
      uint256 startTime = lastUpdateTime;
      uint256 endTime = nextFundingPaymentTimestamp();
      if (endTime == startTime) return;
-      fundingRates[latestFundingPaymentPointer] =
+      fundingRates[latestFundingPaymentPointer_ ] =
-        fundingRates[latestFundingPaymentPointer] +
+        fundingRates[latestFundingPaymentPointer_ ] +
        ((amount * 1e18) / (endTime - startTime));
    }

```

### ``_totalCollateral ``, ``_rdpxCollateral `` should be cached : Saves ``300 GAS``, ``3 SLOD``

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L192


```diff
FILE: 2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVaultLP.sol

190: function addProceeds(uint256 proceeds) public onlyPerpVault {
+  uint256 _totalCollateralcache =  _totalCollateral  ;
191:    require(
- 192:      collateral.balanceOf(address(this)) >= _totalCollateral + proceeds,
+ 192:      collateral.balanceOf(address(this)) >= _totalCollateralcache + proceeds,
193:      "Not enough collateral token was sent"
194:    );
- 195:    _totalCollateral += proceeds;
+ 195:    _totalCollateral =_totalCollateralcache + proceeds;
196:  }


199: function subtractLoss(uint256 loss) public onlyPerpVault {
+  uint256 _totalCollateralcache =  _totalCollateral  ;
200:    require(
- 201:      collateral.balanceOf(address(this)) == _totalCollateral - loss,
+ 201:      collateral.balanceOf(address(this)) == _totalCollateralcache - loss,
202:      "Not enough collateral was sent out"
203:    );
- 204:    _totalCollateral -= loss;
+ 204:    _totalCollateral =_totalCollateralcache - loss;
205:   }

208: function addRdpx(uint256 amount) public onlyPerpVault {
+ uint256 _rdpxCollateralCache = _rdpxCollateral ;
209:    require(
- 210:      IERC20WithBurn(rdpx).balanceOf(address(this)) >= _rdpxCollateral + amount,
+ 210:      IERC20WithBurn(rdpx).balanceOf(address(this)) >= _rdpxCollateralCache + amount,
211:      "Not enough rdpx token was sent"
212:    );
- 213:    _rdpxCollateral += amount;
+ 213:    _rdpxCollateral =_rdpxCollateralCache + amount;
214:  }

```

###  ``addresses.ammRouter``,``liquiditySlippageTolerance`` , ``addresses.tokenA`` , ``addresses.tokenB``,``addresses.pair`` Should be cached : Saves ``2100 GAS``, ``21 SLOD``

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L151


```diff
FILE: Breadcrumbs2023-08-dopex/contracts/reLP/ReLPContract.sol

+  address ammRouter_ = addresses.ammRouter ; 
151: IERC20WithBurn(addresses.pair).safeApprove( //@audit is this possible to cache local ?
-       addresses.ammRouter,
+       ammRouter_,
      type(uint256).max
    );

    IERC20WithBurn(addresses.tokenA).safeApprove(
-      addresses.ammRouter,
+      ammRouter_,
      type(uint256).max
    );

    IERC20WithBurn(addresses.tokenB).safeApprove(
-      addresses.ammRouter,
+      ammRouter_,
      type(uint256).max
    );



+ uint256 liquiditySlippageTolerance_ = liquiditySlippageTolerance ; 
249: // calculate min amounts to remove
250:    uint256 mintokenAAmount = tokenAToRemove -
- 251:      ((tokenAToRemove * liquiditySlippageTolerance) / 1e8);
+ 251:      ((tokenAToRemove * liquiditySlippageTolerance_ ) / 1e8);
252:    uint256 mintokenBAmount = ((tokenAToRemove * tokenAInfo.tokenAPrice) /
253:      1e8) -
- 254:      ((tokenAToRemove * tokenAInfo.tokenAPrice) * liquiditySlippageTolerance) /
+ 254:      ((tokenAToRemove * tokenAInfo.tokenAPrice) * liquiditySlippageTolerance_) /
256:      1e16;


+ address addressestokenA_ = addresses.tokenA ;
+ address addressestokenB_ = addresses.tokenB ;
205: (address tokenASorted, address tokenBSorted) = UniswapV2Library.sortTokens(
-       addresses.tokenA,
+       addressestokenA_  ,
-      addresses.tokenB
+      addressestokenB_
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

-    tokenAInfo.tokenALpReserve = addresses.tokenA == tokenASorted
+    tokenAInfo.tokenALpReserve = addressestokenA_ == tokenASorted
      ? reserveA
      : reserveB;

    uint256 baseReLpRatio = (reLPFactor *
      Math.sqrt(tokenAInfo.tokenAReserve) *
      1e2) / (Math.sqrt(1e18)); // 1e6 precision

    uint256 tokenAToRemove = ((((_amount * 4) * 1e18) /
      tokenAInfo.tokenAReserve) *
      tokenAInfo.tokenALpReserve *
      baseReLpRatio) / (1e18 * DEFAULT_PRECISION * 1e2);
+  address addressesPair_ = addresses.pair ;
-    uint256 totalLpSupply = IUniswapV2Pair(addresses.pair).totalSupply();
+    uint256 totalLpSupply = IUniswapV2Pair(addressesPair_).totalSupply();
    uint256 lpToRemove = (tokenAToRemove * totalLpSupply) /
      tokenAInfo.tokenALpReserve;

    // transfer LP tokens from the AMO
-    IERC20WithBurn(addresses.pair).transferFrom(
+    IERC20WithBurn(addressesPair_).transferFrom(
      addresses.amo,
      address(this),
      lpToRemove
    );
+  uint256 liquiditySlippageTolerance_ = liquiditySlippageTolerance ;
    // calculate min amounts to remove
    uint256 mintokenAAmount = tokenAToRemove -
-      ((tokenAToRemove * liquiditySlippageTolerance) / 1e8);
+      ((tokenAToRemove * liquiditySlippageTolerance_) / 1e8);
    uint256 mintokenBAmount = ((tokenAToRemove * tokenAInfo.tokenAPrice) /
      1e8) -
-      ((tokenAToRemove * tokenAInfo.tokenAPrice) * liquiditySlippageTolerance) /
+      ((tokenAToRemove * tokenAInfo.tokenAPrice) * liquiditySlippageTolerance_) /
      1e16;
+  address addressesAmmRouter_= addresses.ammRouter ;
-     (, uint256 amountB) = IUniswapV2Router(addresses.ammRouter).removeLiquidity(
+     (, uint256 amountB) = IUniswapV2Router(addressesAmmRouter_).removeLiquidity(
-      addresses.tokenA,
+      addressestokenA_,
-      addresses.tokenB,
+      addressestokenB_,
      lpToRemove,
      mintokenAAmount,
      mintokenBAmount,
      address(this),
      block.timestamp + 10
    );

    address[] memory path;
    path = new address[](2);
-    path[0] = addresses.tokenB;
+    path[0] = addressestokenB_;
-    path[1] = addresses.tokenA;
+    path[1] = addressestokenA_;
 
    // calculate min amount of tokenA to be received
    mintokenAAmount =
      (((amountB / 2) * tokenAInfo.tokenAPrice) / 1e8) -
      (((amountB / 2) * tokenAInfo.tokenAPrice * slippageTolerance) / 1e16);

-    uint256 tokenAAmountOut = IUniswapV2Router(addresses.ammRouter)
+    uint256 tokenAAmountOut = IUniswapV2Router(addressesAmmRouter_)
      .swapExactTokensForTokens(
        amountB / 2,
        mintokenAAmount,
        path,
        address(this),
        block.timestamp + 10
      )[path.length - 1];

-    (, , uint256 lp) = IUniswapV2Router(addresses.ammRouter).addLiquidity(
+    (, , uint256 lp) = IUniswapV2Router(addressesAmmRouter_).addLiquidity(
-      addresses.tokenA,
+      addressestokenA_ ,
-      addresses.tokenB,
+      addressestokenB_,
      tokenAAmountOut,
      amountB / 2,
      0,
      0,
      address(this),
      block.timestamp + 10
    );

    // transfer the lp to the amo
-    IERC20WithBurn(addresses.pair).safeTransfer(addresses.amo, lp);
+    IERC20WithBurn(addressespair_).safeTransfer(addresses.amo, lp);
    IUniV2LiquidityAmo(addresses.amo).sync();

    // transfer rdpx to rdpxV2Core
-    IERC20WithBurn(addresses.tokenA).safeTransfer(
+    IERC20WithBurn(addressestokenA_).safeTransfer(
      addresses.rdpxV2Core,
-      IERC20WithBurn(addresses.tokenA).balanceOf(address(this))
+      IERC20WithBurn(addressestokenA_).balanceOf(address(this))
    );
    IRdpxV2Core(addresses.rdpxV2Core).sync

```

##

## [G-8] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contracts gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size..

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html

```solidity
FILE: 2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol

193: (uint256 tokenId, uint128 amountLiquidity, , ) = univ3_positions.mint(

235: uint128 liquidity,

```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L235

##

## [G-9] With assembly, ``.call (bool success)`` transfer can be done gas-optimized

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.

https://twitter.com/pashovkrum/status/1607024043718316032?t=xs30iD6ORWtE2bTTYsCFIQ&s=19

```diff
FILE: 2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol

- 344: (bool success, bytes memory result) = _to.call{ value: _value }(_data);

+ bool success;
+ bytes memory result;

+ assembly {
+    let dataSize := 0
+    let dataOffset := add(result, 0x20)
+
+    success := call(gas(), _to, _value, dataOffset, dataSize, dataOffset, dataSize)
+
+    switch success
+    case 0 {
+        // Call failed, revert with the provided data
+        revert(dataOffset, dataSize)
+    }
+    default {
+        // Call succeeded, set the actual output size
+        mstore(result, dataSize)
+    }
+ }

```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L343C5-L343C73

```diff
FILE: 2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol

- 98:  (bool success, ) = to.call{ value: amount, gas: gas }("");

+ bool success;
+ assembly {
+    success := call(gas, to, amount, 0, 0, 0, 0)
+ }

```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L98

##

## [G-10] Use assembly to check for ``address(0)``

```solidity
FILE: 2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol

83: require(
      _tokenA != address(0) &&
        _tokenB != address(0) &&
        _pair != address(0) &&
        _rdpxV2Core != address(0) &&
        _rdpxOracle != address(0) &&
        _ammFactory != address(0) &&
        _ammRouter != address(0),
      "reLPContract: address cannot be 0"
    );

131: require(_token != address(0), "reLPContract: token cannot be 0");
132: require(_spender != address(0), "reLPContract: spender cannot be 0");

FILE: 2023-08-dopex/contracts/core/RdpxV2Bond.sol

```
##

## [G-11] State variables only set in the constructor should be declared immutable

> Missing Instances from Bot Race

Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

```solidity
FILE: 2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol

77: rdpx = _rdpx;
78: rdpxV2Core = _rdpxV2Core;

```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L77-L78

```solidity
FILE: Breadcrumbs2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVaultLP.sol

98: perpetualAtlanticVault = IPerpetualAtlanticVault(_perpetualAtlanticVault);

```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L98

##

## [G-12] Normal ``state variables`` are more gas efficient than ``Counters`` library 

[Discussions](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/4233)

There are a few issues though. This "guarantee" was always a soft guarantee, because the inner struct members are directly modifiable bypassing the struct interface. Additionally, a Counter being a uint256 value occupies an entire storage slot whereas in most occasions packing a counter in storage with other values will be a far larger optimization than removing overflow checks; the significance of this has also changed since the introduction of Counters due to changes in the EVM gas prices.

```solidity
FILE: 2023-08-dopex/contracts/reLP/ReLPContract.sol

29: Counters.Counter private _tokenIdCounter;

```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L20

### Mitigation 

```solidity

uint256 private _tokenIdCounter;

 _safeMint(_receiver, _tokenIdCounter++)

```




