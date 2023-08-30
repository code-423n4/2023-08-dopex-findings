# GAS OPTIMIZATION

##
## [G-1] Using ``calldata`` to optimize gas costs for ``Read-Only`` external function arguments 

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

###  ``optionIds``, ``strikes`` can be declared in ``calldata`` instead of ``memory``  : Saves ``568 GAS``, `` 1 Instance``

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

## [G-2] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a ``Gcoldsload (2100 gas)`` for each field of the ``struct/array``. If the fields are read from the new memory variable, they incur an additional ``MLOAD`` rather than a cheap stack read. Instead of declaring the variable with the ``memory`` keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct


###  ``delegatePosition`` variable can be stored in ``storage`` instead of ``memory`` : Saves ``4200 GAS``

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1272

```diff
FILE: 2023-08-dopex/contracts/core/RdpxV2Core.sol

- 1272: Delegate memory delegatePosition = delegates[_delegateId];
+ 1272: Delegate storage delegatePosition = delegates[_delegateId];

```
##

## [G-3] State variables can be packed into fewer storage slots

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

## [G-] State variables should not be ``initialized`` with ``default values``

If a state variable is not initialized with a default value, the default value of the type will be used. For example, a state variable of type uint256 will have the default value of 0 if it is not initialized with a default value. This will save gas because the default value of the type does not need to be stored in the blockchain. Saves ``2500 GAS``

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L89

```diff
FILE: Breadcrumbs2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol

- 89: uint256 public latestFundingPaymentPointer = 0;
+ 89: uint256 public latestFundingPaymentPointer;

```
##

## [G-] State variables should be cached in stack variables rather than re-reading them from storage

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

##

## [G-] When reading a ``state variable multiple times``, it is more gas-efficient to store the value in ``memory`` than in ``storage``




 



Struct can be packed in fewer slots 
Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct.



Multiple accesses of a mapping/array should use a local variable cache

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping’s value in a local storage or calldata variable when the value is accessed [multiple times](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0), saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory/calldata


Avoid emitting constants. Combine events 
A log topic (declared with indexed) has a gas cost of Glogtopic (375 gas). The Stake and Withdraw events’ second indexed parameter is a constant for a majority of events emitted (with the exception of the events emitted in the _stakeLP() and _withdrawLP() functions) and is unecessary to emit since the value will never change. Alternatively, you can avoid incurring the Glogtopic (375 gas) per call to any function that emits Stake/Withdraw (with the exception of _stakeLP() and _withdrawLP()) by creating separate events for each staking/withdraw function and opt out of using the current indexed asset topic in each event. This way you can still query the different staking/withdraw events and will save 375 gas for each staking/withdraw function (with the exception of _stakeLP() and _withdrawLP()).

Note that the events emitted in the _stakeLP() and withdrawLP() functions are not considered for this issue since the second indexed parameter is for the LP storage variable, which can be changed via the configureLP() function.

6. Is this possible to use costants for unchanged values 

7. If/Require checks should be top of the function 

8. Is this possible to avoid extra write ? 

struct names should be aligned way 

Unused state variables 

Cache the external function calls outside the loops

Result of the function call should be cached 

Less size uint128 incurs overhead 

Optimize names to save gas

public/external function names and public member variable names can be optimized to save gas. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, per [sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

USE A MORE RECENT VERSION OF SOLIDITY

Consider using bytes32 instead of string for known strings 



