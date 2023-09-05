# Gas Optimizations

*Notes*: 

- most of the optimizations are benchmarkes via protocol's tests, since all functions were not included in the protocol's tests therefore some benchmarks are based on the deployment cost of protocol.
- for calculating the gas differences the `Max` column has been used, which stays the same in all tests.


# Summary

|   Number   |  Issue   |  Instances  |
|------|----------|------------|
|[G-01]| Use calldata instead of memory for function arguments that do not get mutated|7|
|[G-02]| Use assembly to write address storage values|5|
|[G-03]| >=/<= costs less gas than >/<|8|
|[G-04]| Change public state variable visibility to private|46|
|[G-05]| Remove or replace unused state variables|1|
|[G-06]| Use constants instead of type(uintx).max|17|
|[G-07]| Using storage instead of memory for structs/arrays saves gas|1|
|[G-08]| internal functions only called once can be inlined to save gas|1|
|[G-09]| Stack variable used as a cheaper cache for a state variable is only used once|2|
|[G-10]| Empty blocks should be removed or emit something|1|
|[G-11]| State variables only set in the constructor should be declared immutable|4|
|[G-12]| Gas savings can be achieved by changing the model for assigning value to the structure|7|
|[G-13]| Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if-statement|1|




## [G-01] Use calldata instead of memory for function arguments that do not get mutated
When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

Gas saving for `RdpxV2Core.sync`, obtained via protocol's tests: Avg 318 gas.

| | Max  |
|------|----------|
|Before|118125|
|After|117807|


```solidity
File: contracts/core/RdpxV2Core.sol
242       string memory _assetSymbol
```

```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
index 513a337..c93a210 100644
--- a/contracts/core/RdpxV2Core.sol
+++ b/contracts/core/RdpxV2Core.sol
@@ -241,7 +241,7 @@ contract RdpxV2Core is
      **/
     function addAssetTotokenReserves(
         address _asset,
-        string memory _assetSymbol
+        string calldata _assetSymbol
     ) external onlyRole(DEFAULT_ADMIN_ROLE) {
         require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L242

```solidity
File: contracts/core/RdpxV2Core.sol
271       string memory _assetSymbol
```

```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
index c93a210..76b0270 100644
--- a/contracts/core/RdpxV2Core.sol
+++ b/contracts/core/RdpxV2Core.sol
@@ -270,7 +270,7 @@ contract RdpxV2Core is
      * @dev    Can only be called by admin
      **/
     function removeAssetFromtokenReserves(
-        string memory _assetSymbol
+        string calldata _assetSymbol
     ) external onlyRole(DEFAULT_ADMIN_ROLE) {
         uint256 index = reservesIndex[_assetSymbol];
         _validate(index != 0, 18);
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L271

Gas saving for `RdpxV2Core.settle`, obtained via protocol's tests: Avg 389 gas.

| | Max  |
|------|----------|
|Before|122580|
|After|122191|


```solidity
File: contracts/core/RdpxV2Core.sol
765      uint256[] memory optionIds
```

```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
index 76b0270..f9afa4c 100644
--- a/contracts/core/RdpxV2Core.sol
+++ b/contracts/core/RdpxV2Core.sol
@@ -775,7 +775,7 @@ contract RdpxV2Core is
      * @return rdpxAmount the amount of rdpx sent out
      */
     function settle(
-        uint256[] memory optionIds
+        uint256[] calldata optionIds
     )
         external
         onlyRole(DEFAULT_ADMIN_ROLE)
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L765

Gas saving for `RdpxV2Core.bondWithDelegate`, obtained via protocol's tests: Avg 1289 gas.

| | Max  |
|------|----------|
|Before|1705471|
|After|1704182|


```solidity
File: contracts/core/RdpxV2Core.sol
821          uint256[] memory _amounts,
822          uint256[] memory _delegateIds,
```

```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
index f9afa4c..00bc4a1 100644
--- a/contracts/core/RdpxV2Core.sol
+++ b/contracts/core/RdpxV2Core.sol
@@ -833,8 +833,8 @@ contract RdpxV2Core is
      **/
     function bondWithDelegate(
         address _to,
-        uint256[] memory _amounts,
-        uint256[] memory _delegateIds,
+        uint256[] calldata _amounts,
+        uint256[] calldata _delegateIds,
         uint256 rdpxBondId
     ) public returns (uint256 receiptTokenAmount, uint256[] memory) {
         _whenNotPaused();
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L821C1-L822C35

Gas saving for `RdpxV2Core.getReserveTokenInfo`, obtained via protocol's tests: Avg 440 gas.

| | Max  |
|------|----------|
|Before|13654|
|After|13214|


```solidity
File: contracts/core/RdpxV2Core.sol
1136         string memory _token
```

```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
index 00bc4a1..7fb9143 100644
--- a/contracts/core/RdpxV2Core.sol
+++ b/contracts/core/RdpxV2Core.sol
@@ -1152,7 +1152,7 @@ contract RdpxV2Core is
      * @return tokenSymbol token symbol
      */
     function getReserveTokenInfo(
-        string memory _token
+        string calldata _token
     ) public view returns (address, uint256, string memory) {
         ReserveAsset memory asset = reserveAsset[reservesIndex[_token]];
 
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1136

Gas saving for `PerpetualAtlanticVault.settle`, obtained via protocol's tests: Avg 224 gas.

| | Max  |
|------|----------|
|Before|116128|
|After|115904|


```solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol
316        uint256[] memory optionIds
```

```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
index 3a575bf..fbbcd22 100644
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
@@ -316,7 +316,7 @@ contract PerpetualAtlanticVault is
 
     /// @inheritdoc    IPerpetualAtlanticVault
     function settle(
-        uint256[] memory optionIds
+        uint256[] calldata optionIds
     )
         external
         nonReentrant
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L316

Gas saving for `PerpetualAtlanticVault.calculateFunding`, obtained via protocol's tests: Avg 294 gas.

| | Max  |
|------|----------|
|Before|154379|
|After|154085|


```solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol
406       uint256[] memory strikes
```

```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
index fbbcd22..362e600 100644
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
@@ -405,7 +405,7 @@ contract PerpetualAtlanticVault is
      * @return fundingAmount the funding of options
      **/
     function calculateFunding(
-        uint256[] memory strikes
+        uint256[] calldata strikes
     ) external nonReentrant returns (uint256 fundingAmount) {
         _whenNotPaused();
         _isEligibleSender();
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L406


## [G-02] Use assembly to write address storage values

By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.

Gas saving for `PerpetualAtlanticVaultLP` contract, obtained via protocol's tests: Avg 90 gas in deployment cost.

| | Deployment cost  |
|------|----------|
|Before|1621781|
|After|1621691|


```solidity
File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
99       rdpxRdpxV2Core = _rdpxRdpxV2Core;

101      rdpx = _rdpx;
```

```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVaultLP.sol b/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
index 613d2da..a2be10a 100644
--- a/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
@@ -98,11 +98,14 @@ contract PerpetualAtlanticVaultLP is ERC20, IPerpetualAtlanticVaultLP {
         perpetualAtlanticVault = IPerpetualAtlanticVault(
             _perpetualAtlanticVault
         );
-        rdpxRdpxV2Core = _rdpxRdpxV2Core;
         collateralSymbol = _collateralSymbol;
-        rdpx = _rdpx;
         collateral = ERC20(_collateral);
 
+        assembly {
+            sstore(rdpxRdpxV2Core.slot, _rdpxRdpxV2Core)
+            sstore(rdpx.slot, _rdpx)
+        }
+
         symbol = string.concat(_collateralSymbol, "-LP");
 
         collateral.approve(_perpetualAtlanticVault, type(uint256).max);
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L99

Gas saving for `RdpxV2Core` contract, obtained via protocol's tests: Avg 30 gas in deployment cost.

| | Deployment cost  |
|------|----------|
|Before|5052495|
|After|5052465|


```solidity
File: contracts/core/RdpxV2Core.sol
126        weth = _weth;

```

```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
index 7fb9143..29c6d53 100644
--- a/contracts/core/RdpxV2Core.sol
+++ b/contracts/core/RdpxV2Core.sol
@@ -123,7 +123,10 @@ contract RdpxV2Core is
     // ================================ CONSTRUCTOR ================================ //
     constructor(address _weth) {
         _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
-        weth = _weth;
+
+        assembly {
+            sstore(weth.slot, _weth)
+        }
 
         // add Zero asset to reserveAsset
         ReserveAsset memory zeroAsset = ReserveAsset({
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L126

```solidity
File: contracts/amo/UniV3LiquidityAmo.sol
77          rdpx = _rdpx;
78            rdpxV2Core = _rdpxV2Core;
```

```diff
diff --git a/contracts/amo/UniV3LiquidityAmo.sol b/contracts/amo/UniV3LiquidityAmo.sol
index fe28b7b..d3594e4 100644
--- a/contracts/amo/UniV3LiquidityAmo.sol
+++ b/contracts/amo/UniV3LiquidityAmo.sol
@@ -74,8 +74,10 @@ contract UniV3LiquidityAMO is AccessControl, ERC721Holder {
     /* ========== CONSTRUCTOR ========== */
 
     constructor(address _rdpx, address _rdpxV2Core) {
-        rdpx = _rdpx;
-        rdpxV2Core = _rdpxV2Core;
+        assembly {
+            sstore(rdpx.slot, _rdpx)
+            sstore(rdpxV2Core.slot, _rdpxV2Core)
+        }
 
         _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L77C1-L78C30


## [G-03] >=/<= costs less gas than >/<
The compiler uses opcodes GT and ISZERO for code that uses >, but only requires LT for >=. A similar behaviour applies for <, which uses opcodes LT and ISZERO, but only requires GT for <= , which is more gas efficient and `3 gas` will be saved per instance.

Note: Bot race has pointed out some instances that are using >, but the instances below were missed by bot race and are not included in automated report.

```solidity
File:   contracts/amo/UniV2LiquidityAmo.sol
147        for (uint256 i = 0; i < tokens.length; i++) {
```

```diff
diff --git a/contracts/amo/UniV2LiquidityAmo.sol b/contracts/amo/UniV2LiquidityAmo.sol
index 4d20b0d..0c18030 100644
--- a/contracts/amo/UniV2LiquidityAmo.sol
+++ b/contracts/amo/UniV2LiquidityAmo.sol
@@ -144,7 +144,7 @@ contract UniV2LiquidityAMO is AccessControl {
     ) external onlyRole(DEFAULT_ADMIN_ROLE) {
         IERC20WithBurn token;
 
-        for (uint256 i = 0; i < tokens.length; i++) {
+        for (uint256 i = 0; i <= tokens.length; i++) {
             token = IERC20WithBurn(tokens[i]);
             token.safeTransfer(msg.sender, token.balanceOf(address(this)));
         }
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L147

```solidity
File:   contracts/amo/UniV3LiquidityAmo.sol
120       for (uint i = 0; i < positions_array.length; i++) {
```

```diff
diff --git a/contracts/amo/UniV3LiquidityAmo.sol b/contracts/amo/UniV3LiquidityAmo.sol
index d3594e4..40929d1 100644
--- a/contracts/amo/UniV3LiquidityAmo.sol
+++ b/contracts/amo/UniV3LiquidityAmo.sol
@@ -119,7 +119,7 @@ contract UniV3LiquidityAMO is AccessControl, ERC721Holder {
 
     // Iterate through all positions and collect fees accumulated
     function collectFees() external onlyRole(DEFAULT_ADMIN_ROLE) {
-        for (uint i = 0; i < positions_array.length; i++) {
+        for (uint i = 0; i <= positions_array.length; i++) {
             Position memory current_position = positions_array[i];
             INonfungiblePositionManager.CollectParams
                 memory collect_params = INonfungiblePositionManager
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L120


Note: In some instances in `RdpxV2Core` contract < was not swapped to <= to avoid `Index out of bond` exception.

```solidity
File:   contracts/core/RdpxV2Core.sol
167      for (uint256 i = 0; i < tokens.length; i++) {
	
391	 _validate(_index < amoAddresses.length, 18);
	     
1167	 _validate(bondDiscount < 100e8, 14);
```

```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
index 29c6d53..17d4fcc 100644
--- a/contracts/core/RdpxV2Core.sol
+++ b/contracts/core/RdpxV2Core.sol
@@ -167,7 +167,7 @@ contract RdpxV2Core is
         _whenPaused();
         IERC20WithBurn token;
 
-        for (uint256 i = 0; i < tokens.length; i++) {
+        for (uint256 i = 0; i <= tokens.length; i++) {
             token = IERC20WithBurn(tokens[i]);
             token.safeTransfer(msg.sender, token.balanceOf(address(this)));
         }

@@ -401,7 +401,7 @@ contract RdpxV2Core is
     function removeAMOAddress(
         uint256 _index
     ) external onlyRole(DEFAULT_ADMIN_ROLE) {
-        _validate(_index < amoAddresses.length, 18);
+        _validate(_index <= amoAddresses.length, 18);
         amoAddresses[_index] = amoAddresses[amoAddresses.length - 1];
         amoAddresses.pop();
     }
 
@@ -1186,7 +1186,7 @@ contract RdpxV2Core is
                 Math.sqrt(IRdpxReserve(addresses.rdpxReserve).rdpxReserve()) *
                 1e2) / (Math.sqrt(1e18)); // 1e8 precision
 
-            _validate(bondDiscount < 100e8, 14);
+            _validate(bondDiscount <= 100e8, 14);
 
             rdpxRequired =
                 ((RDPX_RATIO_PERCENTAGE - (bondDiscount / 2)) *
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L167
 
```solidity
File:    contracts/decaying-bonds/RdpxDecayingBonds.sol
103      for (uint256 i = 0; i < tokens.length; i++) {

156	 for (uint256 i; i < ownerTokenCount; i++) {
```

```diff
diff --git a/contracts/decaying-bonds/RdpxDecayingBonds.sol b/contracts/decaying-bonds/RdpxDecayingBonds.sol
index c47286e..0703ae5 100644
--- a/contracts/decaying-bonds/RdpxDecayingBonds.sol
+++ b/contracts/decaying-bonds/RdpxDecayingBonds.sol
@@ -100,7 +100,7 @@ contract RdpxDecayingBonds is
         }
         IERC20WithBurn token;
 
-        for (uint256 i = 0; i < tokens.length; i++) {
+        for (uint256 i = 0; i <= tokens.length; i++) {
             token = IERC20WithBurn(tokens[i]);
             token.safeTransfer(msg.sender, token.balanceOf(address(this)));
         }
         
@@ -153,7 +153,7 @@ contract RdpxDecayingBonds is
     ) external view returns (uint256[] memory) {
         uint256 ownerTokenCount = balanceOf(_address);
         uint256[] memory tokenIds = new uint256[](ownerTokenCount);
-        for (uint256 i; i < ownerTokenCount; i++) {
+        for (uint256 i; i <= ownerTokenCount; i++) {
             tokenIds[i] = tokenOfOwnerByIndex(_address, i);
         }
         return tokenIds;
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L103

Note: this rule was not applicable to all instances found in `PerpetualAtlanticVault` contract, due to encounter of 'Index out of bond' exception.
Instances followed the rule are listed below:

```solidity
File:    contracts/perp-vault/PerpetualAtlanticVault.sol
464       if (lastUpdateTime < nextFundingPaymentTimestamp()) {
```

```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
index 362e600..e603048 100644
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
@@ -465,7 +465,7 @@ contract PerpetualAtlanticVault is
     /// @dev Helper function that updates the latest funding payment pointer based on current timestamp
     function updateFundingPaymentPointer() public {
         while (block.timestamp >= nextFundingPaymentTimestamp()) {
-            if (lastUpdateTime < nextFundingPaymentTimestamp()) {
+            if (lastUpdateTime <= nextFundingPaymentTimestamp()) {
                 uint256 currentFundingRate = fundingRates[
                     latestFundingPaymentPointer
                 ];
         return tokenIds;
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L464


## [G-04] Change public state variable visibility to private
it's generally a good practice to limit the visibility of state variables to the minimum necessary level. This means that you should use the private visibility modifier for state variables whenever possible, and avoid using the public modifier unless it is absolutely necessary.

1.Gas costs: Public state variables can be more expensive to read and write than private state variables, since Solidity generates additional getter and setter functions for public variables. By using private state variables, you can reduce the gas cost of your contract and improve its efficiency.

3.Security: Public state variables can be read and modified by anyone on the blockchain, which can make your contract vulnerable to attacks. By using the private modifier, you can limit access to your state variables and reduce the risk of malicious actors exploiting them.

2.Encapsulation: Using private state variables can help to encapsulate the internal workings of your contract and make it easier to reason about and maintain. By only exposing the necessary interfaces to the outside world, you can reduce the complexity and potential for errors in your contract.

Gas saving for `UniV2LiquidityAmo` contract, obtained via protocol's tests: Avg 36442 gas in deployment cost. since all the functions included in this particular contract weren't covered in test cases, therefore gas saving was calculated based on deployment cost.

| | Deployment cost  |
|------|----------|
|Before|1623550|
|After|1587108|


```solidity
File:   contracts/amo/UniV2LiquidityAmo.sol
45        Addresses public addresses;

48       uint256 public constant DEFAULT_PRECISION = 1e8;

51       uint256 public slippageTolerance = 5e5; // 0.5%
```

```diff
diff --git a/contracts/amo/UniV2LiquidityAmo.sol b/contracts/amo/UniV2LiquidityAmo.sol
index 0c18030..2a6ad4a 100644
--- a/contracts/amo/UniV2LiquidityAmo.sol
+++ b/contracts/amo/UniV2LiquidityAmo.sol
@@ -42,13 +42,13 @@ contract UniV2LiquidityAMO is AccessControl {
     }
 
     /// @notice  addresses of the contracts
-    Addresses public addresses;
+    Addresses private addresses;
 
     /// @notice Precision used for prices, percentages and other calculations
-    uint256 public constant DEFAULT_PRECISION = 1e8;
+    uint256 private constant DEFAULT_PRECISION = 1e8;
 
     /// @notice The slippage tolernce in swaps in 1e8 precision
-    uint256 public slippageTolerance = 5e5; // 0.5%
+    uint256 private slippageTolerance = 5e5; // 0.5%
 
     /// @notice LP token Balance
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L45

Gas saving for `RdpxV2Core` contract, obtained via protocol's tests: Avg 295008 gas in deployment cost. since all the functions included in this particular contract weren't covered in test cases, therefore gas saving was calculated based on deployment cost.

| | Deployment cost  |
|------|----------|
|Before|5052665|
|After|4757657|


```solidity
File:     contracts/core/RdpxV2Core.sol
47          Addresses private addresses;

50          PricingOracleAddresses private pricingOracleAddresses;

61          ReserveAsset[] private reserveAsset;
  
64          address[] private amoAddresses;

67	    address private weth;

70	    string[] private reserveTokens;

73	    mapping(string => uint256) private reservesIndex;

76	    mapping(uint256 => Bond) private bonds;

79	    mapping(uint256 => bool) private optionsOwned;

82	    mapping(uint256 => bool) private fundingPaidFor;

85	    uint256 private constant DEFAULT_PRECISION = 1e8;

88	    uint256 private constant RDPX_RATIO_PERCENTAGE = 25 * DEFAULT_PRECISION;

91	    uint256 private constant ETH_RATIO_PERCENTAGE = 75 * DEFAULT_PRECISION;

94	    uint256 private rdpxBurnPercentage = 50 * DEFAULT_PRECISION;

97	    uint256 private rdpxFeePercentage = 50 * DEFAULT_PRECISION;

103	    uint256 private liquiditySlippageTolerance = 5e5; // 0.5%

106	    uint256 private bondMaturity;

112	    uint256 private totalWethDelegated;

115	    bool private isReLPActive;

118	    bool private putOptionsRequired;

```

```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
index b8b84f2..bea04a7 100644
--- a/contracts/core/RdpxV2Core.sol
+++ b/contracts/core/RdpxV2Core.sol
@@ -44,10 +44,10 @@ contract RdpxV2Core is
     // ================================ STATE VARIABLES ================================ //
 
     /// @notice Addresses used by the contract
-    Addresses public addresses;
+    Addresses private addresses;
 
     /// @notice The pricing oracle addresses
-    PricingOracleAddresses public pricingOracleAddresses;
+    PricingOracleAddresses private pricingOracleAddresses;
 
     /* Inital tokens in the reserve
      index0: ZERO address
@@ -58,64 +58,64 @@ contract RdpxV2Core is
   */
 
     /// @notice Array containg the reserve assets
-    ReserveAsset[] public reserveAsset;
+    ReserveAsset[] private reserveAsset;
 
     /// @notice Array that contains the addresses of the AMO
-    address[] public amoAddresses;
+    address[] private amoAddresses;
 
     /// @notice Token address that dpxEth is pegged to
-    address public weth;
+    address private weth;
 
     /// @notice Array that contains the symbol of the reserve tokens
-    string[] public reserveTokens;
+    string[] private reserveTokens;
 
     /// @notice Mapping that contains the index for a specific token in the reserves
-    mapping(string => uint256) public reservesIndex;
+    mapping(string => uint256) private reservesIndex;
 
     /// @dev Bond id => Bond
-    mapping(uint256 => Bond) public bonds;
+    mapping(uint256 => Bond) private bonds;
 
     /// @dev Option id => owned or not (boolean)
-    mapping(uint256 => bool) public optionsOwned;
+    mapping(uint256 => bool) private optionsOwned;
 
     /// @dev Funding paid for epoch
-    mapping(uint256 => bool) public fundingPaidFor;
+    mapping(uint256 => bool) private fundingPaidFor;
 
     /// @notice Precision used for prices, percentages and other calculations
-    uint256 public constant DEFAULT_PRECISION = 1e8;
+    uint256 private constant DEFAULT_PRECISION = 1e8;
 
     /// @notice rDPX Ratio required when bonding
-    uint256 public constant RDPX_RATIO_PERCENTAGE = 25 * DEFAULT_PRECISION;
+    uint256 private constant RDPX_RATIO_PERCENTAGE = 25 * DEFAULT_PRECISION;
 
     /// @notice ETH Ratio required when bonding
-    uint256 public constant ETH_RATIO_PERCENTAGE = 75 * DEFAULT_PRECISION;
+    uint256 private constant ETH_RATIO_PERCENTAGE = 75 * DEFAULT_PRECISION;
 
     /// @notice The % of rdpx to burn while bonding
-    uint256 public rdpxBurnPercentage = 50 * DEFAULT_PRECISION;
+    uint256 private rdpxBurnPercentage = 50 * DEFAULT_PRECISION;
 
     /// @notice The % of rdpx sent to fee distributor while bonding
-    uint256 public rdpxFeePercentage = 50 * DEFAULT_PRECISION;
+    uint256 private rdpxFeePercentage = 50 * DEFAULT_PRECISION;
 
     /// @notice The slippage tolernce in swaps in 1e8 precision
     uint256 public slippageTolerance = 5e5; // 0.5%
 
     /// @notice Liquidity slippage tolerance
-    uint256 public liquiditySlippageTolerance = 5e5; // 0.5%
+    uint256 private liquiditySlippageTolerance = 5e5; // 0.5%
 
     /// @notice Bond maturity
-    uint256 public bondMaturity;
+    uint256 private bondMaturity;
 
     /// @notice rDPX LP bond discount factor
     uint256 public bondDiscountFactor;
 
     /// @notice Total weth delegated
-    uint256 public totalWethDelegated;
+    uint256 private totalWethDelegated;
 
     /// @notice Whether reLP is active or not
-    bool public isReLPActive;
+    bool private isReLPActive;
 
     /// @notice Whether put options are requred
-    bool public putOptionsRequired;
+    bool private putOptionsRequired;
 
     /// @notice Delegates array
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L47

Gas saving for `DpxEthToken` contract, obtained via protocol's tests: Avg 13213 gas in deployment cost. since all the functions included in this particular contract weren't covered in test cases, therefore gas saving was calculated based on deployment cost.

Note: This rule was not applied to `MINTER_ROLE` and `BURNER_ROLE` state variables, since they were referenced somewhere else in the protocol and had to be public.

| | Deployment cost  |
|------|----------|
|Before|1150081|
|After|1136868|


```solidity
File:  contracts/dpxETH/DpxEthToken.sol
19            bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");

```

```diff
diff --git a/contracts/dpxETH/DpxEthToken.sol b/contracts/dpxETH/DpxEthToken.sol
index dafef17..57c100b 100644
--- a/contracts/dpxETH/DpxEthToken.sol
+++ b/contracts/dpxETH/DpxEthToken.sol
@@ -16,7 +16,7 @@ contract DpxEthToken is
     AccessControl,
     IDpxEthToken
 {
-    bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
+    bytes32 private constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
     bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
     bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
 
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/dpxETH/DpxEthToken.sol#L19

Gas saving for `PerpetualAtlanticVault` contract, obtained via protocol's tests: Avg 146186 gas in deployment cost. since all the functions included in this particular contract weren't covered in test cases, therefore gas saving was calculated based on deployment cost.

Note: This rule was not applied to `RDPXV2CORE_ROLE`, `optionPositions`, `totalFundingForEpoch`, `fundingRates`,   `latestFundingPaymentPointer`, `totalActiveOptions` and `fundingDuration` state variables in PerpetualAtlanticVault contract, since they were referenced somewhere else in the protocol and had to be public.

| | Deployment cost  |
|------|----------|
|Before|3763901|
|After|3617715|


```solidity
File:    contracts/perp-vault/PerpetualAtlanticVault.sol
45          bytes32 public constant MANAGER_ROLE = keccak256("MANAGER_ROLE");

51	    string public underlyingSymbol;

54	    Addresses public addresses;

57	    IERC20WithBurn public collateralToken;

60	    uint256 private collateralPrecision;

66	    mapping(uint256 => uint256) public fundingPaymentsAccountedFor;

69	    public fundingPaymentsAccountedForPerStrike;

76          mapping(uint256 => uint256) public optionsPerStrike;

79	    mapping(uint256 => uint256) public latestFundingPerStrike;

95	    uint256 public genesis;

98	    uint256 public lastUpdateTime;

104	    uint256 public roundingPrecision = 1e6;
   
```

```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
index e603048..763cf07 100644
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
@@ -42,41 +42,41 @@ contract PerpetualAtlanticVault is
     Counters.Counter private _tokenIdCounter;
 
     /// @dev Manager role which handles bootstrapping
-    bytes32 public constant MANAGER_ROLE = keccak256("MANAGER_ROLE");
+    bytes32 private constant MANAGER_ROLE = keccak256("MANAGER_ROLE");
 
     /// @dev Rdpx v2 core role which can purchase and settle options
     bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
 
     /// @dev Underlying assets symbol
-    string public underlyingSymbol;
+    string private underlyingSymbol;
 
     /// @dev Contract addresses
-    Addresses public addresses;
+    Addresses private addresses;
 
     /// @dev Collateral Token
-    IERC20WithBurn public collateralToken;
+    IERC20WithBurn private collateralToken;
 
     /// @dev The precision of the collateral token
-    uint256 public collateralPrecision;
+    uint256 private collateralPrecision;
 
     /// @dev tokenId => OptionPosition
     mapping(uint256 => OptionPosition) public optionPositions;
 
     /// @dev number of options funding has been accounted for the epoch
-    mapping(uint256 => uint256) public fundingPaymentsAccountedFor;
+    mapping(uint256 => uint256) private fundingPaymentsAccountedFor;
 
     /// @dev the funding accounted for the epoch and strike
     mapping(uint256 => mapping(uint256 => uint256))
-        public fundingPaymentsAccountedForPerStrike;
+        private fundingPaymentsAccountedForPerStrike;
 
     /// @dev the total funding for the epoch
     mapping(uint256 => uint256) public totalFundingForEpoch;
 
     /// @dev amount of options per strike
-    mapping(uint256 => uint256) public optionsPerStrike;
+    mapping(uint256 => uint256) private optionsPerStrike;
 
     /// @dev latest funding update per strike
-    mapping(uint256 => uint256) public latestFundingPerStrike;
+    mapping(uint256 => uint256) private latestFundingPerStrike;
 
     // @dev Funding rate for the epoch
     mapping(uint256 => uint256) public fundingRates;
@@ -92,16 +92,16 @@ contract PerpetualAtlanticVault is
     uint256 public totalActiveOptions;
 
     /// @dev genesis timestamp
-    uint256 public genesis;
+    uint256 private genesis;
 
     /// @dev the timestamp of the last update where funding was paid for
-    uint256 public lastUpdateTime;
+    uint256 private lastUpdateTime;
 
     /// @dev the duration between funding payments
     uint256 public fundingDuration = 7 days;
 
     /// @dev the precision to round up to
-    uint256 public roundingPrecision = 1e6;
+    uint256 private roundingPrecision = 1e6;
 
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L45

Gas saving for `PerpetualAtlanticVaultLP` contract, obtained via protocol's tests: Avg 47983 gas in deployment cost. since all the functions included in this particular contract weren't covered in test cases, therefore gas saving was calculated based on deployment cost.

| | Deployment cost  |
|------|----------|
|Before|1621619|
|After|1573636|


```solidity
File:    contracts/perp-vault/PerpetualAtlanticVaultLP.sol
46          IPerpetualAtlanticVault public perpetualAtlanticVault;

49	     ERC20 public collateral;

52	    string public underlyingSymbol;

55	    string public collateralSymbol;

67	     address public rdpx;

70	    address public rdpxRdpxV2Core;

```

```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVaultLP.sol b/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
index a2be10a..07d3af6 100644
--- a/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
@@ -43,16 +43,16 @@ contract PerpetualAtlanticVaultLP is ERC20, IPerpetualAtlanticVaultLP {
     // ================================ STATE VARIABLES ================================ //
 
     /// @dev The address of the perpetual Atlantic Vault contract creating the lp token
-    IPerpetualAtlanticVault public perpetualAtlanticVault;
+    IPerpetualAtlanticVault private perpetualAtlanticVault;
 
     /// @dev The collateral token
-    ERC20 public collateral;
+    ERC20 private collateral;
 
     /// @dev The symbol reperesenting the underlying asset of the perpetualatlanticvault lp
-    string public underlyingSymbol;
+    string private underlyingSymbol;
 
     /// @dev The symbol representing the collateral token of the perpetualatlanticvault lp
-    string public collateralSymbol;
+    string private collateralSymbol;
 
     /// @dev Total collateral available
     uint256 private _totalCollateral;
@@ -64,10 +64,10 @@ contract PerpetualAtlanticVaultLP is ERC20, IPerpetualAtlanticVaultLP {
     uint256 private _rdpxCollateral;
 
     /// @dev address of rdpx token
-    address public rdpx;
+    address private rdpx;
 
     /// @dev address of the rdpx rdpxV2Core contract
-    address public rdpxRdpxV2Core;
+    address private rdpxRdpxV2Core;
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L46

Gas saving for `ReLPContract` contract, obtained via protocol's tests: Avg 53456 gas in deployment cost. since all the functions included in this particular contract weren't covered in test cases, therefore gas saving was calculated based on deployment cost.

Note: This rule was not applied to `RDPXV2CORE_ROLE` in ReLPContract  state variables, since it was referenced somewhere else in the protocol and had to be public.

| | Deployment cost  |
|------|----------|
|Before|1768141|
|After|1714685|


```solidity
File:      contracts/reLP/ReLPContract.sol
61        Addresses public addresses;

64	  uint256 public reLPFactor;

67	  uint256 public constant DEFAULT_PRECISION = 1e8;

73	  uint256 public liquiditySlippageTolerance = 5e5; // 0.5%

76	  uint256 public slippageTolerance = 5e5; // 0.5%

```

```diff
diff --git a/contracts/reLP/ReLPContract.sol b/contracts/reLP/ReLPContract.sol
index 38fc733..a104540 100644
--- a/contracts/reLP/ReLPContract.sol
+++ b/contracts/reLP/ReLPContract.sol
@@ -58,22 +58,22 @@ contract ReLPContract is AccessControl {
     }
 
     /// @notice  addresses of the contracts
-    Addresses public addresses;
+    Addresses private addresses;
 
     /// @notice reLP factor
-    uint256 public reLPFactor;
+    uint256 private reLPFactor;
 
     /// @notice Precision used for prices, percentages and other calculations
-    uint256 public constant DEFAULT_PRECISION = 1e8;
+    uint256 private constant DEFAULT_PRECISION = 1e8;
 
     /// @notice rdpxV2Core role
     bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
 
     /// @notice liquidity slippage tolerance
-    uint256 public liquiditySlippageTolerance = 5e5; // 0.5%
+    uint256 private liquiditySlippageTolerance = 5e5; // 0.5%
 
     /// @notice The slippage tolernce in swaps in 1e8 precision
-    uint256 public slippageTolerance = 5e5; // 0.5%
+    uint256 private slippageTolerance = 5e5; // 0.5%
 
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L61


## [G-05] Remove or replace unused state variables
Note: These instances were missed by bot race  and are not included in automated report.

Saves a storage slot. If the variable is assigned a non-zero value, saves Gsset (20000 gas). If it's assigned a zero value, saves Gsreset (2900 gas). If the variable remains unassigned, there is no gas savings unless the variable is public, in which case the compiler-generated non-payable getter deployment cost is saved. 


```solidity
File: contracts/amo/UniV2LiquidityAmo.sol
48       uint256 public constant DEFAULT_PRECISION = 1e8;
```

```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
index 513a337..c93a210 100644
--- a/contracts/core/RdpxV2Core.sol
+++ b/contracts/core/RdpxV2Core.sol
@@ -241,7 +241,7 @@ contract RdpxV2Core is
      **/
     function addAssetTotokenReserves(
         address _asset,
-        string memory _assetSymbol
+        string calldata _assetSymbol
     ) external onlyRole(DEFAULT_ADMIN_ROLE) {
         require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L48


## [G-06] Use constants instead of type(uintx).max
it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.
The reason for this is that the type(uintX).max expression uses more gas in the distribution process and also for each transaction than constant usage and involves a computation at runtime, whereas a constant is evaluated at compile-time.
This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.
By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

```solidity
File: contracts/amo/UniV3LiquidityAmo.sol
126         type(uint128).max,

127         type(uint128).max,

190         type(uint256).max,

223         type(uint128).max,

224         type(uint128).max,

```

```diff
diff --git a/contracts/amo/UniV3LiquidityAmo.sol b/contracts/amo/UniV3LiquidityAmo.sol
index ad9402f..a572c88 100644
--- a/contracts/amo/UniV3LiquidityAmo.sol
+++ b/contracts/amo/UniV3LiquidityAmo.sol
@@ -71,6 +71,10 @@ contract UniV3LiquidityAMO is AccessControl, ERC721Holder {
     // RdpxV2Core address
     address private rdpxV2Core;
 
+    uint256 private constant MAX_VALUE_256 = 2 ** 256 - 1;
+
+    uint128 private constant MAX_VALUE_128 = 2 ** 128 - 1;
+
     /* ========== CONSTRUCTOR ========== */
 
     constructor(address _rdpx, address _rdpxV2Core) {
@@ -126,8 +130,8 @@ contract UniV3LiquidityAMO is AccessControl, ERC721Holder {
                     .CollectParams(
                         current_position.token_id,
                         rdpxV2Core,
-                        type(uint128).max,
-                        type(uint128).max
+                        MAX_VALUE_128,
+                        MAX_VALUE_128
                     );
 
             // Send to custodian address
@@ -190,7 +194,7 @@ contract UniV3LiquidityAMO is AccessControl, ERC721Holder {
                 params._amount0Min,
                 params._amount1Min,
                 address(this),
-                type(uint256).max
+                MAX_VALUE_256
             );
 
         (uint256 tokenId, uint128 amountLiquidity, , ) = univ3_positions.mint(
@@ -223,8 +227,8 @@ contract UniV3LiquidityAMO is AccessControl, ERC721Holder {
             memory collect_params = INonfungiblePositionManager.CollectParams(
                 pos.token_id,
                 rdpxV2Core,
-                type(uint128).max,
-                type(uint128).max
+                MAX_VALUE_128,
+                MAX_VALUE_128
             );
 
         (
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L126

```solidity
File:   contracts/core/RdpxV2Core.sol
341       type(uint256).max

343       type(uint256).max

344       type(uint256).max

347       type(uint256).max

```

```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
index bea04a7..a2e1908 100644
--- a/contracts/core/RdpxV2Core.sol
+++ b/contracts/core/RdpxV2Core.sol
@@ -120,6 +120,8 @@ contract RdpxV2Core is
     /// @notice Delegates array
     Delegate[] public delegates;
 
+    uint256 private constant MAX_VALUE = 2 ** 256 - 1;
+
     // ================================ CONSTRUCTOR ================================ //
     constructor(address _weth) {
         _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
@@ -343,20 +345,11 @@ contract RdpxV2Core is
         });
         IERC20WithBurn(weth).approve(
             addresses.perpetualAtlanticVault,
-            type(uint256).max
-        );
-        IERC20WithBurn(weth).approve(
-            addresses.dopexAMMRouter,
-            type(uint256).max
-        );
-        IERC20WithBurn(weth).approve(
-            addresses.dpxEthCurvePool,
-            type(uint256).max
-        );
-        IERC20WithBurn(weth).approve(
-            addresses.rdpxV2ReceiptToken,
-            type(uint256).max
+            MAX_VALUE
         );
+        IERC20WithBurn(weth).approve(addresses.dopexAMMRouter, MAX_VALUE);
+        IERC20WithBurn(weth).approve(addresses.dpxEthCurvePool, MAX_VALUE);
+        IERC20WithBurn(weth).approve(addresses.rdpxV2ReceiptToken, MAX_VALUE);
         emit LogSetAddresses(addresses);
     }
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L341

```solidity
File:   contracts/perp-vault/PerpetualAtlanticVault.sol
209       type(uint256).max

247       type(uint256).max

```

```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
index 763cf07..ed0fd16 100644
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
@@ -103,6 +103,8 @@ contract PerpetualAtlanticVault is
     /// @dev the precision to round up to
     uint256 private roundingPrecision = 1e6;
 
+    uint256 private constant MAX_VALUE = 2 ** 256 - 1;
+
     // ================================ CONSTRUCTOR ================================ //
 
     /// @notice Contract constructor
@@ -206,7 +208,7 @@ contract PerpetualAtlanticVault is
         });
         collateralToken.safeApprove(
             addresses.perpetualAtlanticVaultLP,
-            type(uint256).max
+            MAX_VALUE
         );
         emit AddressesSet(addresses);
     }
@@ -246,7 +248,7 @@ contract PerpetualAtlanticVault is
         increase
             ? collateralToken.approve(
                 addresses.perpetualAtlanticVaultLP,
-                type(uint256).max
+                MAX_VALUE
             )
             : collateralToken.approve(addresses.perpetualAtlanticVaultLP, 0);
     }
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L209

```solidity
File:   contracts/perp-vault/PerpetualAtlanticVaultLP.sol
106      collateral.approve(_perpetualAtlanticVault, type(uint256).max);

107      ERC20(rdpx).approve(_perpetualAtlanticVault, type(uint256).max);

155      if (allowed != type(uint256).max) {

```

```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVaultLP.sol b/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
index a2be10a..a24a55e 100644
--- a/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
@@ -69,6 +69,8 @@ contract PerpetualAtlanticVaultLP is ERC20, IPerpetualAtlanticVaultLP {
     /// @dev address of the rdpx rdpxV2Core contract
     address public rdpxRdpxV2Core;
 
+    uint256 private constant MAX_VALUE = 2 ** 256 - 1;
+
     // ================================ CONSTRUCTOR ================================ //
 
     /**
@@ -108,8 +110,8 @@ contract PerpetualAtlanticVaultLP is ERC20, IPerpetualAtlanticVaultLP {
 
         symbol = string.concat(_collateralSymbol, "-LP");
 
-        collateral.approve(_perpetualAtlanticVault, type(uint256).max);
-        ERC20(rdpx).approve(_perpetualAtlanticVault, type(uint256).max);
+        collateral.approve(_perpetualAtlanticVault, MAX_VALUE);
+        ERC20(rdpx).approve(_perpetualAtlanticVault, MAX_VALUE);
     }
 
     // ================================ PUBLIC FUNCTIONS ================================ //
@@ -157,7 +159,7 @@ contract PerpetualAtlanticVaultLP is ERC20, IPerpetualAtlanticVaultLP {
         if (msg.sender != owner) {
             uint256 allowed = allowance[owner][msg.sender]; // Saves gas for limited approvals.
 
-            if (allowed != type(uint256).max) {
+            if (allowed != MAX_VALUE) {
                 allowance[owner][msg.sender] = allowed - shares;
             }
         }
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L106

```solidity
File:   contracts/reLP/ReLPContract.sol
152      type(uint256).max

157      type(uint256).max

162      type(uint256).max

```

```diff
diff --git a/contracts/reLP/ReLPContract.sol b/contracts/reLP/ReLPContract.sol
index a104540..7fedbe5 100644
--- a/contracts/reLP/ReLPContract.sol
+++ b/contracts/reLP/ReLPContract.sol
@@ -75,6 +75,8 @@ contract ReLPContract is AccessControl {
     /// @notice The slippage tolernce in swaps in 1e8 precision
     uint256 private slippageTolerance = 5e5; // 0.5%
 
+    uint256 private constant MAX_VALUE = 2 ** 256 - 1;
+
     // ================================ CONSTRUCTOR ================================ //
     constructor() {
         _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
@@ -149,17 +151,17 @@ contract ReLPContract is AccessControl {
 
         IERC20WithBurn(addresses.pair).safeApprove(
             addresses.ammRouter,
-            type(uint256).max
+            MAX_VALUE
         );
 
         IERC20WithBurn(addresses.tokenA).safeApprove(
             addresses.ammRouter,
-            type(uint256).max
+            MAX_VALUE
         );
 
         IERC20WithBurn(addresses.tokenB).safeApprove(
             addresses.ammRouter,
-            type(uint256).max
+            MAX_VALUE
         );
     }
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L152


## [G-07] Using storage instead of memory for structs/arrays saves gas
Note: These instances were missed by bot race and are not included in automated report.

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

Gas saving for `RdpxV2Core.getDelegatePosition`, obtained via protocol's tests: Avg 105 gas.

| | Max  |
|------|----------|
|Before|1215|
|After|1110|

```solidity
File:     contracts/core/RdpxV2Core.sol
1272       Delegate memory delegatePosition = delegates[_delegateId];
```

```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
index a2e1908..6476c3e 100644
--- a/contracts/core/RdpxV2Core.sol
+++ b/contracts/core/RdpxV2Core.sol
@@ -1287,7 +1287,7 @@ contract RdpxV2Core is
             uint256 activeCollateral
         )
     {
-        Delegate memory delegatePosition = delegates[_delegateId];
+        Delegate storage delegatePosition = delegates[_delegateId];
         return (
             delegatePosition.owner,
             delegatePosition.amount,
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1272


## [G‑08] internal functions only called once can be inlined to save gas
Note: instances below were missed by bot race  and are not included in automated report.

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

approximately `20 to 40 gas` saved by inlining the function below in `PerpetualAtlanticVaultLP` contract

```solidity
File:     contracts/perp-vault/PerpetualAtlanticVaultLP.sol
218        function _convertToAssets(
    			uint256 shares
  		) internal view virtual returns (uint256 assets, uint256 rdpxAmount) {
   		 uint256 supply = totalSupply;
   		 return
     		 (supply == 0)
    	    ? (shares, 0)
     		   : (
     	  	   shares.mulDivDown(totalCollateral(), supply),
         	 shares.mulDivDown(_rdpxCollateral, supply)
       	 );
  }
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L218C1-L229C4


## [G‑09] Stack variable used as a cheaper cache for a state variable is only used once
If the variable is only accessed once, it's cheaper to use the state variable directly that one time, 
and **save the 3 gas** the extra stack assignment would spend.

```solidity
File:     contracts/core/RdpxV2Core.sol
524        address coin0 = dpxEthCurvePool.coins(0);
```

```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
index 6476c3e..6e54041 100644
--- a/contracts/core/RdpxV2Core.sol
+++ b/contracts/core/RdpxV2Core.sol
@@ -527,8 +527,9 @@ contract RdpxV2Core is
         IStableSwap dpxEthCurvePool = IStableSwap(addresses.dpxEthCurvePool);
 
         // First compute a reverse swapping of dpxETH to ETH to compute the amount of ETH required
-        address coin0 = dpxEthCurvePool.coins(0);
-        (uint256 a, uint256 b) = coin0 == weth ? (0, 1) : (1, 0);
+        (uint256 a, uint256 b) = dpxEthCurvePool.coins(0) == weth
+            ? (0, 1)
+            : (1, 0);
 
         // validate the swap for peg functions
         if (validate) {
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L524

Gas saving for `ReLPContract.reLP`, obtained via protocol's tests: Avg 13 gas.

| | Max  |
|------|----------|
|Before|229217|
|After|229204|

```solidity
File:    contracts/reLP/ReLPContract.sol
237	uint256 totalLpSupply = IUniswapV2Pair(addresses.pair).totalSupply();
```

```diff
diff --git a/contracts/reLP/ReLPContract.sol b/contracts/reLP/ReLPContract.sol
index 7fedbe5..83300c4 100644
--- a/contracts/reLP/ReLPContract.sol
+++ b/contracts/reLP/ReLPContract.sol
@@ -234,9 +234,8 @@ contract ReLPContract is AccessControl {
             tokenAInfo.tokenALpReserve *
             baseReLpRatio) / (1e18 * DEFAULT_PRECISION * 1e2);
 
-        uint256 totalLpSupply = IUniswapV2Pair(addresses.pair).totalSupply();
-
-        uint256 lpToRemove = (tokenAToRemove * totalLpSupply) /
+        uint256 lpToRemove = (tokenAToRemove *
+            IUniswapV2Pair(addresses.pair).totalSupply()) /
             tokenAInfo.tokenALpReserve;
 
         // transfer LP tokens from the AMO
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L237


## [G‑10] Empty blocks should be removed or emit something
The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.
If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation.

If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if( x ) {}else if(y){ . . . }else{ . . . } => if ( !x) {if(y) { . . . }else{ . . .}}) . Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas.

```solidity
File:     contracts/perp-vault/PerpetualAtlanticVaultLP.sol
235        ) internal virtual {}
```

```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVaultLP.sol b/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
index a24a55e..b826367 100644
--- a/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
@@ -236,12 +236,6 @@ contract PerpetualAtlanticVaultLP is ERC20, IPerpetualAtlanticVaultLP {
                 );
     }
 
-    function _beforeTokenTransfer(
-        address from,
-        address,
-        uint256
-    ) internal virtual {}
-
     // ================================ VIEWS ================================ //
 
     /// @notice Returns the total active collateral
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L235


## [G‑11] State variables only set in the constructor should be declared immutable
Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

Note: Instances below were missed by bot race and are not included in automated report.

total of **`4169 gas`** is saved in `UniV2LiquidityAmo` contract

```solidity
File:     contracts/amo/UniV3LiquidityAmo.sol
77        rdpx = _rdpx;

78	  rdpxV2Core = _rdpxV2Core;
```

```diff
diff --git a/contracts/amo/UniV3LiquidityAmo.sol b/contracts/amo/UniV3LiquidityAmo.sol
index 457986a..a4f1fac 100644
--- a/contracts/amo/UniV3LiquidityAmo.sol
+++ b/contracts/amo/UniV3LiquidityAmo.sol
@@ -66,10 +66,10 @@ contract UniV3LiquidityAMO is AccessControl, ERC721Holder {
     mapping(uint256 => Position) private positions_mapping;
 
     // Rdpx address
-    address private rdpx;
+    address private immutable rdpx;
 
     // RdpxV2Core address
-    address private rdpxV2Core;
+    address private immutable rdpxV2Core;
 
     uint256 private constant MAX_VALUE_256 = 2 ** 256 - 1;
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L77

total of **`4169 gas`** is saved in `PerpetualAtlanticVaultLP` contract

```solidity
File:     contracts/perp-vault/PerpetualAtlanticVaultLP.sol
98        perpetualAtlanticVault = IPerpetualAtlanticVault(_perpetualAtlanticVault);

101	   rdpx = _rdpx;
```

```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVaultLP.sol b/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
index 1582dab..1beff76 100644
--- a/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
@@ -43,7 +43,7 @@ contract PerpetualAtlanticVaultLP is ERC20, IPerpetualAtlanticVaultLP {
     // ================================ STATE VARIABLES ================================ //
 
     /// @dev The address of the perpetual Atlantic Vault contract creating the lp token
-    IPerpetualAtlanticVault public perpetualAtlanticVault;
+    IPerpetualAtlanticVault public immutable perpetualAtlanticVault;
 
     /// @dev The collateral token
     ERC20 public collateral;
@@ -64,7 +64,7 @@ contract PerpetualAtlanticVaultLP is ERC20, IPerpetualAtlanticVaultLP {
     uint256 private _rdpxCollateral;
 
     /// @dev address of rdpx token
-    address public rdpx;
+    address public immutable rdpx;
 
     /// @dev address of the rdpx rdpxV2Core contract
     address public rdpxRdpxV2Core;
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L98
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L101


## [G-12] Gas savings can be achieved by changing the model for assigning value to the structure
To optimize gas usage in Solidity smart contracts, consider altering the method used to assign values to data structures particularly structs, as this can lead to significant gas savings.
instead of  
example = Example({key1: value1, key2: value2}),   we can assign it this way:
example.key1 = value1;
example.key2 = value2;

Gas saving for `UniV2LiquidityAmo.setAddresses`, obtained via protocol's tests: Avg 159 gas.

| | Max  |
|------|----------|
|Before|156854|
|After|156695|


```solidity
File: contracts/amo/UniV2LiquidityAmo.sol
93       addresses = Addresses({
      		tokenA: _tokenA,
     		tokenB: _tokenB,
      		pair: _pair,
      		rdpxV2Core: _rdpxV2Core,
      		rdpxOracle: _rdpxOracle,
      		ammFactory: _ammFactory,
      		ammRouter: _ammRouter
    	});
```

```diff
diff --git a/contracts/amo/UniV2LiquidityAmo.sol b/contracts/amo/UniV2LiquidityAmo.sol
index 7804559..d28c4f1 100644
--- a/contracts/amo/UniV2LiquidityAmo.sol
+++ b/contracts/amo/UniV2LiquidityAmo.sol
@@ -89,15 +89,14 @@ contract UniV2LiquidityAMO is AccessControl {
                 _ammRouter != address(0),
             "reLPContract: address cannot be 0"
         );
-        addresses = Addresses({
-            tokenA: _tokenA,
-            tokenB: _tokenB,
-            pair: _pair,
-            rdpxV2Core: _rdpxV2Core,
-            rdpxOracle: _rdpxOracle,
-            ammFactory: _ammFactory,
-            ammRouter: _ammRouter
-        });
+
+        addresses.tokenA = _tokenA;
+        addresses.tokenB = _tokenB;
+        addresses.pair = _pair;
+        addresses.rdpxV2Core = _rdpxV2Core;
+        addresses.rdpxOracle = _rdpxOracle;
+        addresses.ammFactory = _ammFactory;
+        addresses.ammRouter = _ammRouter;
     }
 
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L93C1-L101C8

Gas saving for `RdpxV2Core.setAddresses`, obtained via protocol's tests: Avg 283 gas.

| | Max  |
|------|----------|
|Before|329943|
|After|329660|


```solidity
File:   contracts/core/RdpxV2Core.sol
327    addresses = Addresses({
      	dopexAMMRouter: _dopexAMMRouter,
     	dpxEthCurvePool: _dpxEthCurvePool,
      	rdpxDecayingBonds: _rdpxDecayingBonds,
      	perpetualAtlanticVault: _perpetualAtlanticVault,
      	perpetualAtlanticVaultLP: _perpetualAtlanticVaultLP,
      	rdpxReserve: _rdpxReserve,
      	rdpxV2ReceiptToken: _rdpxV2ReceiptToken,
      	feeDistributor: _feeDistributor,
      	reLPContract: _reLPContract,
      	receiptTokenBonds: _receiptTokenBonds
    });
```

```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
index 6e54041..6be9146 100644
--- a/contracts/core/RdpxV2Core.sol
+++ b/contracts/core/RdpxV2Core.sol
@@ -331,18 +331,17 @@ contract RdpxV2Core is
         _validate(_reLPContract != address(0), 17);
         _validate(_receiptTokenBonds != address(0), 17);
 
-        addresses = Addresses({
-            dopexAMMRouter: _dopexAMMRouter,
-            dpxEthCurvePool: _dpxEthCurvePool,
-            rdpxDecayingBonds: _rdpxDecayingBonds,
-            perpetualAtlanticVault: _perpetualAtlanticVault,
-            perpetualAtlanticVaultLP: _perpetualAtlanticVaultLP,
-            rdpxReserve: _rdpxReserve,
-            rdpxV2ReceiptToken: _rdpxV2ReceiptToken,
-            feeDistributor: _feeDistributor,
-            reLPContract: _reLPContract,
-            receiptTokenBonds: _receiptTokenBonds
-        });
+        addresses.dopexAMMRouter = _dopexAMMRouter;
+        addresses.dpxEthCurvePool = _dpxEthCurvePool;
+        addresses.rdpxDecayingBonds = _rdpxDecayingBonds;
+        addresses.perpetualAtlanticVault = _perpetualAtlanticVault;
+        addresses.perpetualAtlanticVaultLP = _perpetualAtlanticVaultLP;
+        addresses.rdpxReserve = _rdpxReserve;
+        addresses.rdpxV2ReceiptToken = _rdpxV2ReceiptToken;
+        addresses.feeDistributor = _feeDistributor;
+        addresses.reLPContract = _reLPContract;
+        addresses.receiptTokenBonds = _receiptTokenBonds;
+
         IERC20WithBurn(weth).approve(
             addresses.perpetualAtlanticVault,
             MAX_VALUE
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L327C3-L338C8

Gas saving for `RdpxV2Core.setPricingOracleAddresses`, obtained via protocol's tests: Avg 51 gas.

| | Max  |
|------|----------|
|Before|46718|
|After|46667|


```solidity
File:   contracts/core/RdpxV2Core.sol
365     pricingOracleAddresses = PricingOracleAddresses({
           rdpxPriceOracle: _rdpxPriceOracle,
           dpxEthPriceOracle: _dpxEthPriceOracle
        });
 
```

```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
index 6be9146..9795d20 100644
--- a/contracts/core/RdpxV2Core.sol
+++ b/contracts/core/RdpxV2Core.sol
@@ -365,10 +365,8 @@ contract RdpxV2Core is
         _validate(_rdpxPriceOracle != address(0), 17);
         _validate(_dpxEthPriceOracle != address(0), 17);
 
-        pricingOracleAddresses = PricingOracleAddresses({
-            rdpxPriceOracle: _rdpxPriceOracle,
-            dpxEthPriceOracle: _dpxEthPriceOracle
-        });
+        pricingOracleAddresses.rdpxPriceOracle = _rdpxPriceOracle;
+        pricingOracleAddresses.dpxEthPriceOracle = _dpxEthPriceOracle;
 
         emit LogSetPricingOracleAddresses(pricingOracleAddresses);
     }
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L365C1-L368C8

Note: `_issueBond` function in `RdpxV2Core` contract wasn't included in test provided in protocol, therefore providing benchmark for saving gas wasn't possible.

```solidity
File:   contracts/core/RdpxV2Core.sol
500     bonds[bondId] = Bond({
      		amount: _amount,
      		maturity: block.timestamp + bondMaturity,
      		timestamp: block.timestamp
    	});
 
```

```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
index 9795d20..6324530 100644
--- a/contracts/core/RdpxV2Core.sol
+++ b/contracts/core/RdpxV2Core.sol
@@ -500,11 +500,10 @@ contract RdpxV2Core is
         uint256 _amount
     ) internal returns (uint256 bondId) {
         bondId = RdpxV2Bond(addresses.receiptTokenBonds).mint(_to);
-        bonds[bondId] = Bond({
-            amount: _amount,
-            maturity: block.timestamp + bondMaturity,
-            timestamp: block.timestamp
-        });
+
+        bonds[bondId].amount = _amount;
+        bonds[bondId].maturity = block.timestamp + bondMaturity;
+        bonds[bondId].timestamp = block.timestamp;
     }
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L500C1-L504C8

Gas saving for `PerpetualAtlanticVault.setAddresses`, obtained via protocol's tests: Avg 199 gas.

| | Max  |
|------|----------|
|Before|187895|
|After|187696|


```solidity
File:   contracts/perp-vault/PerpetualAtlanticVault.sol
198     addresses = Addresses({
      		optionPricing: _optionPricing,
      		assetPriceOracle: _assetPriceOracle,
      		volatilityOracle: _volatilityOracle,
     		feeDistributor: _feeDistributor,
      		rdpx: _rdpx,
      		perpetualAtlanticVaultLP: _perpetualAtlanticVaultLP,
      		rdpxV2Core: _rdpxV2Core
        });
 
```

```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
index bf23c79..ea6798f 100644
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
@@ -197,15 +197,14 @@ contract PerpetualAtlanticVault is
         _validate(_perpetualAtlanticVaultLP != address(0), 1);
         _validate(_rdpxV2Core != address(0), 1);
 
-        addresses = Addresses({
-            optionPricing: _optionPricing,
-            assetPriceOracle: _assetPriceOracle,
-            volatilityOracle: _volatilityOracle,
-            feeDistributor: _feeDistributor,
-            rdpx: _rdpx,
-            perpetualAtlanticVaultLP: _perpetualAtlanticVaultLP,
-            rdpxV2Core: _rdpxV2Core
-        });
+        addresses.optionPricing = _optionPricing;
+        addresses.assetPriceOracle = _assetPriceOracle;
+        addresses.volatilityOracle = _volatilityOracle;
+        addresses.feeDistributor = _feeDistributor;
+        addresses.rdpx = _rdpx;
+        addresses.perpetualAtlanticVaultLP = _perpetualAtlanticVaultLP;
+        addresses.rdpxV2Core = _rdpxV2Core;
+
         collateralToken.safeApprove(
             addresses.perpetualAtlanticVaultLP,
             MAX_VALUE
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L198C5-L206C8

Gas saving for `PerpetualAtlanticVault.purchase(uint256,address)(uint256,uint256)`, obtained via protocol's tests: Avg 90 gas.

| | Max  |
|------|----------|
|Before|355220|
|After|355130|


```solidity
File:   contracts/perp-vault/PerpetualAtlanticVault.sol
296      optionPositions[tokenId] = OptionPosition({
      		strike: strike,
      		amount: amount,
      		positionId: tokenId
         });
 
```

```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
index ea6798f..ca4bbe3 100644
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
@@ -297,11 +297,10 @@ contract PerpetualAtlanticVault is
 
         // Mint the option tokens
         tokenId = _mintOptionToken();
-        optionPositions[tokenId] = OptionPosition({
-            strike: strike,
-            amount: amount,
-            positionId: tokenId
-        });
+
+        optionPositions[tokenId].strike = strike;
+        optionPositions[tokenId].amount = amount;
+        optionPositions[tokenId].positionId = tokenId;
 
         totalActiveOptions += amount;
         fundingPaymentsAccountedFor[latestFundingPaymentPointer] += amount;
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L296C2-L300C8

Gas saving for `ReLPContract.setAddresses`, obtained via protocol's tests: Avg 238 gas.

| | Max  |
|------|----------|
|Before|285303|
|After|285065|


```solidity
File:    contracts/reLP/ReLPContract.sol
138       addresses = Addresses({
      		tokenA: _tokenA,
      		tokenB: _tokenB,
     		pair: _pair,
      		rdpxV2Core: _rdpxV2Core,
      		tokenAReserve: _tokenAReserve,
      		amo: _amo,
      		rdpxOracle: _rdpxOracle,
      		ammFactory: _ammFactory,
      		ammRouter: _ammRouter
    	});
 
```

```diff
diff --git a/contracts/reLP/ReLPContract.sol b/contracts/reLP/ReLPContract.sol
index 83300c4..b39213c 100644
--- a/contracts/reLP/ReLPContract.sol
+++ b/contracts/reLP/ReLPContract.sol
@@ -137,17 +137,16 @@ contract ReLPContract is AccessControl {
                 _ammRouter != address(0),
             "reLPContract: address cannot be 0"
         );
-        addresses = Addresses({
-            tokenA: _tokenA,
-            tokenB: _tokenB,
-            pair: _pair,
-            rdpxV2Core: _rdpxV2Core,
-            tokenAReserve: _tokenAReserve,
-            amo: _amo,
-            rdpxOracle: _rdpxOracle,
-            ammFactory: _ammFactory,
-            ammRouter: _ammRouter
-        });
+
+        addresses.tokenA = _tokenA;
+        addresses.tokenB = _tokenB;
+        addresses.pair = _pair;
+        addresses.rdpxV2Core = _rdpxV2Core;
+        addresses.tokenAReserve = _tokenAReserve;
+        addresses.amo = _amo;
+        addresses.rdpxOracle = _rdpxOracle;
+        addresses.ammFactory = _ammFactory;
+        addresses.ammRouter = _ammRouter;
 
         IERC20WithBurn(addresses.pair).safeApprove(
             addresses.ammRouter,
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L138C2-L148C8


## [G‑13] Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if-statement
Adding unchecked {} for subtractions where underflow is guaranteed to be prevented by prior require() or if-statement checks is a gas-saving optimization in Solidity. It allows you to skip redundant underflow checks, reducing gas costs by eliminating unnecessary operations when you are confident that arithmetic underflow cannot occur due to prior conditionals.
require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }

Note: since the `beforeWithdraw` function(where the rule was applied) in `PerpectualAtlanticVaultLP` was not included in protocol's tests, we will be calculating the gas differences using the `redeem` function which is calling this particular function.

Gas saving for `PerpectualAtlanticVaultLP.redeem`, obtained via protocol's tests: Avg 65 gas.

| | Max  |
|------|----------|
|Before|57092|
|After|57027|


```solidity
File:    contracts/perp-vault/PerpetualAtlanticVaultLP.sol
291           _totalCollateral -= assets;
```

```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVaultLP.sol b/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
index 1beff76..9b2c502 100644
--- a/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
@@ -290,7 +290,10 @@ contract PerpetualAtlanticVaultLP is ERC20, IPerpetualAtlanticVaultLP {
             assets <= totalAvailableCollateral(),
             "Not enough available assets to satisfy withdrawal"
         );
-        _totalCollateral -= assets;
+
+        unchecked {
+            _totalCollateral -= assets;
+        }
     }
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L291
