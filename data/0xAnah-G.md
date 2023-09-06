# DOPEX GAS OPTIMISATIONS

## INTRODUCTION 
The majority of optimizations underwent benchmarking using the protocol's tests, specifically employing the following configuration: `solc version 0.8.19`, optimizer enabled, and `200` runs. For optimizations that were not subjected to benchmarking, their rationale is elucidated through EVM gas expenses and opcodes.

Highlighted below are optimizations exclusively targeting state-mutating functions and view/pure functions invoked by state-mutating functions. In the discussion that follows, only runtime gas is emphasized, given its inevitable dominance over deployment gas costs throughout the protocol's lifetime. Additionally, certain code excerpts may be abbreviated for brevity, and comments within the code snippets may include @audit tags to facilitate the explanation of issues.


## TABLE OF FINDINGS
| Number |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| G-01| Using calldata instead of memory for read-only arguments in external functions saves gas | 7 | 2797 |
| G-02| Private functions only called once can be inlined to save gas | 1 | 46 |
| G-03| Redundant state variable getters | 1 |   |
| G-04| Use the existing Local variable/global variable when equal to a state variable to avoid reading from state | 4 | 1410 |
| G-05| Avoid using state variable in emit | 4 |  2341 |
| G-06| Structs can be modified to fit in fewer storage slots | 1 | 2000 |
| G-07| Unnecessary require statements | 1 |  317 |
| G-08| Declaring Unnecessary variables | 1 | 3 |
| G-09| Why emitting two events when you can emit one | 1 | 427  |
| G-10| Use constants for state variables whose value is known beforehand and is never changed | 1 | 2194  |
| G-11| State variables only set in the constructor should be declared immutable | 2 | 60964  |
| G-12| Unchecked block for addition that cannot overflow | 6 | 826  |
| G-13| Unbounded Gas Consumption Risk Due to External Call Recipients | 1 |   |
| G-14| Refactor external/internal function to avoid unnecessary SLOAD | 1 | 388  |
| G-15| Can transfer 0 | 13 |   |
| G-16| Use assembly to check for address(0) | 5 |  |



## [G-01] Using calldata instead of memory for read-only arguments in external functions saves gas
When you declare function arguments as memory, the data is copied from `calldata` to `memory`. This copying process consumes gas because it involves additional computation and storage. But in situations where you do not have to modify the data (i.e the data is read only) being passed to the function you can declare the arguments as `calldata` instead of `memory` and it avoids copying the data. Instead, it directly reads the data from the calldata area. This saves gas because it reduces the computational overhead associated with copying data.

### 7 Instances
1. #### Make the `_assetSymbol` parameter of `RdpxV2Core.addAssetTotokenReserves()` `calldata`
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L242
```solidity
file: contracts/core/RdpxV2Core.sol

240:  function addAssetTotokenReserves(
241:    address _asset,
242:    string memory _assetSymbol @audit change memory to calldata
243:  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
 .
 .
 .   
264:  }
```
Since the `_assetSymbol` parameter of the `addAssetTotokenReserves()` function in `RdpxV2Core` contract is never modified we could make the parameter `calldata` so as to reduce gas cost.
```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
index e28a65c..014abe8 100644
--- a/contracts/core/RdpxV2Core.sol
+++ b/contracts/core/RdpxV2Core.sol
@@ -239,7 +239,7 @@ contract RdpxV2Core is
    **/
   function addAssetTotokenReserves(
     address _asset,
-    string memory _assetSymbol
+    string calldata _assetSymbol
   ) external onlyRole(DEFAULT_ADMIN_ROLE) {
     require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");

```
#### Gas saving for `addAssetTotokenReserves()` function obtained via protocol test: Avg 318 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 96795 | 118125 | 104095 | 48 |
| After | 96477 | 117807 | 103777 | 48 |

2. #### Make the `_assetSymbol` parameter of `RdpxV2Core.removeAssetFromtokenReserves()` `calldata`
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L271
```solidity
file: contracts/core/RdpxV2Core.sol

270:  function removeAssetFromtokenReserves(
271:    string memory _assetSymbol     @audit change memory to calldata
272:  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
.
.
.
290:  }
```
Since the `_assetSymbol` parameter of the `removeAssetFromtokenReserves()` function in `RdpxV2Core` contract is never modified we could make the parameter `calldata` so as to reduce gas cost.
```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
index e28a65c..7fb6170 100644
--- a/contracts/core/RdpxV2Core.sol
+++ b/contracts/core/RdpxV2Core.sol
@@ -268,7 +268,7 @@ contract RdpxV2Core is
    * @dev    Can only be called by admin
    **/
   function removeAssetFromtokenReserves(
-    string memory _assetSymbol
+    string calldata _assetSymbol
   ) external onlyRole(DEFAULT_ADMIN_ROLE) {
     uint256 index = reservesIndex[_assetSymbol];
     _validate(index != 0, 18);
```

3. #### Make the `optionIds` parameter of `RdpxV2Core.settle()` `calldata`
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L765
```solidity
file: contracts/core/RdpxV2Core.sol

764:  function settle(
765:    uint256[] memory optionIds      @audit change memory to calldata
766:  )
767:    external
768:    onlyRole(DEFAULT_ADMIN_ROLE)
769:    returns (uint256 amountOfWeth, uint256 rdpxAmount)
770:  {
.
.
.
789:  }
```
Since the `optionIds` array parameter of the `settle()` function in `RdpxV2Core` contract is never modified we could make the parameter `calldata` so as to reduce gas cost.
```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol   
index e28a65c..84f618b 100644                                                
--- a/contracts/core/RdpxV2Core.sol                                          
+++ b/contracts/core/RdpxV2Core.sol                                          
@@ -762,7 +762,7 @@ contract RdpxV2Core is                                   
    * @return rdpxAmount the amount of rdpx sent out                         
    */                                                                       
   function settle(                                                          
-    uint256[] memory optionIds                                              
+    uint256[] calldata optionIds                                            
   )                                                                         
     external                                                                
     onlyRole(DEFAULT_ADMIN_ROLE)                                            
                                                                             
```
#### Gas saving for `settle()` function obtained via protocol test: Avg 646 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 19746 | 136328 | 64123 | 6 |
| After | 19070 | 135036 | 63477 | 6 |


4. #### Make the `_amounts` and `_delegateIds` parameters of `RdpxV2Core.bondWithDelegate()` `calldata`
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L821
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L822
```solidity
file: contracts/core/RdpxV2Core.sol

819:  function bondWithDelegate(
820:    address _to,
821:    uint256[] memory _amounts,      @audit change memory to calldata
822:    uint256[] memory _delegateIds,  @audit change memory to calldata
823:    uint256 rdpxBondId
824:  ) public returns (uint256 receiptTokenAmount, uint256[] memory) {
.
.
.
885:  }
```
Since the `_amounts` and `_delegateIds` array parameters of the `bondWithDelegate()` function in `RdpxV2Core` contract is never modified we could make the parameter `calldata` so as to reduce gas cost.
```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
index e28a65c..64d2849 100644
--- a/contracts/core/RdpxV2Core.sol
+++ b/contracts/core/RdpxV2Core.sol
@@ -818,8 +818,8 @@ contract RdpxV2Core is
    **/
   function bondWithDelegate(
     address _to,
-    uint256[] memory _amounts,
-    uint256[] memory _delegateIds,
+    uint256[] calldata _amounts,
+    uint256[] calldata _delegateIds,
     uint256 rdpxBondId
   ) public returns (uint256 receiptTokenAmount, uint256[] memory) {
     _whenNotPaused();

```
#### Gas saving for `bondWithDelegate()` function obtained via protocol test: Avg 768 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 2159 | 1705471 | 847375 | 7 |
| After | 1584 | 1704182 | 846607 | 7 |



5. #### Make the `_token` parameter of `RdpxV2Core.getReserveTokenInfo()` `calldata`
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1139
```solidity
file: contracts/core/RdpxV2Core.sol

1135:  function getReserveTokenInfo(
1136:    string memory _token      @audit change memory to calldata
1137:  ) public view returns (address, uint256, string memory) {
1138:    ReserveAsset memory asset = reserveAsset[reservesIndex[_token]];
1139:
1140:    _validate(
1141:      keccak256(abi.encodePacked(asset.tokenSymbol)) ==
1142:        keccak256(abi.encodePacked(_token)),
1143:      19
1144:    );
1145:
1146:    return (asset.tokenAddress, asset.tokenBalance, asset.tokenSymbol);
1147:  }
```
Since the `_token` parameter of the `getReserveTokenInfo()` function in `RdpxV2Core` contract is never modified we could make the parameter `calldata` so as to reduce gas cost.
```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
index e28a65c..739617f 100644
--- a/contracts/core/RdpxV2Core.sol
+++ b/contracts/core/RdpxV2Core.sol
@@ -1133,7 +1133,7 @@ contract RdpxV2Core is
    * @return tokenSymbol token symbol
    */
   function getReserveTokenInfo(
-    string memory _token
+    string calldata _token
   ) public view returns (address, uint256, string memory) {
     ReserveAsset memory asset = reserveAsset[reservesIndex[_token]];

```
#### Gas saving for `getReserveTokenInfo()` function obtained via protocol test: Avg 536 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 3699 | 13699 | 5099 | 30 |
| After | 3163 | 13163 | 4563 | 30 |

6. #### Make the `optionIds` parameter of `PerpetualAtlanticVault.settle()` `calldata`
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L316
```solidity
file: contracts/perp-vault/PerpetualAtlanticVault.sol

315:  function settle(
316:    uint256[] memory optionIds      @audit change memory to calldata
317:  )
318:    external
319:    nonReentrant
320:    onlyRole(RDPXV2CORE_ROLE)
321:    returns (uint256 ethAmount, uint256 rdpxAmount)
322:  {
.
.
.
369:  }
```
Since the `optionIds` parameter of the `settle()` function in `PerpetualAtlanticVault` contract is never modified we could make the parameter `calldata` so as to reduce gas cost.
```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
index 88ac2b3..8266e37 100644
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
@@ -313,7 +313,7 @@ contract PerpetualAtlanticVault is

   /// @inheritdoc      IPerpetualAtlanticVault
   function settle(
-    uint256[] memory optionIds
+    uint256[] calldata optionIds
   )
     external
     nonReentrant
```
#### Gas saving for `settle()` function obtained via protocol test: Avg 296 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 17113 | 127485 | 60186 | 14 |
| After | 16897 | 126616 | 59890 | 14 |


7. #### Make the `strikes` parameter of `PerpetualAtlanticVault.calculateFunding()` `calldata`
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L406
```solidity
file: contracts/perp-vault/PerpetualAtlanticVault.sol

405:  function calculateFunding(
406:    uint256[] memory strikes   @audit change memory to calldata
407:  ) external nonReentrant returns (uint256 fundingAmount) {
.
.
.
459:  }
```
Since the `strikes` parameter of the `strikes()` function in `PerpetualAtlanticVault` contract is never modified we could make the parameter `calldata` so as to reduce gas cost.
```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
index 88ac2b3..226f869 100644
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
@@ -403,7 +403,7 @@ contract PerpetualAtlanticVault is
    * @return fundingAmount the funding of options
    **/
   function calculateFunding(
-    uint256[] memory strikes
+    uint256[] calldata strikes
   ) external nonReentrant returns (uint256 fundingAmount) {
     _whenNotPaused();
     _isEligibleSender();
```
#### Gas saving for `calculateFunding()` function obtained via protocol test: Avg 233 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 64779 | 154379 | 99013 | 13 |
| After | 64485 | 154379 | 98780 | 13 |




## [G-02] Private functions only called once can be inlined to save gas

### 1 Instance
1. #### The operations of the `_mintToken()` function can be inlined in the `mint()` function
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L129
```solidity
file: RdpxDecayingBonds.sol

114:  function mint(
115:    address to,
116:    uint256 expiry,
117:    uint256 rdpxAmount
118:  ) external onlyRole(MINTER_ROLE) {
119:    _whenNotPaused();
120:    require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
121:    uint256 bondId = _mintToken(to);
122:    bonds[bondId] = Bond(to, expiry, rdpxAmount);
123:
124:    emit BondMinted(to, bondId, expiry, rdpxAmount);
125:  }
.
.
.
129:  function _mintToken(address to) private returns (uint256 tokenId) {
130:    tokenId = _tokenIdCounter.current();
131:    _tokenIdCounter.increment();
132:    _mint(to, tokenId);
133:  }
```
The `RdpxDecayingBonds` contract defined a private function `_mintToken()` which was only invoked once in the `mint()` function of the contract. We could save up to `40` gas units (which is as a result of two extra `JUMP` instructions and additional stack operations needed for `_mintToken()` function invocation) if we inline the operations of the `_mintToken()` function in the `mint()` function. The diff below shows how it could be done.
```diff
diff --git a/contracts/decaying-bonds/RdpxDecayingBonds.sol b/contracts/decaying-bonds/RdpxDecayingBonds.sol       
index e89349e..f059958 100644                                                                                      
--- a/contracts/decaying-bonds/RdpxDecayingBonds.sol                                                               
+++ b/contracts/decaying-bonds/RdpxDecayingBonds.sol                                                               
@@ -118,20 +118,14 @@ contract RdpxDecayingBonds is                                                                
   ) external onlyRole(MINTER_ROLE) {                                                                              
     _whenNotPaused();                                                                                             
     require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");                                          
-    uint256 bondId = _mintToken(to);                                                                              
+    uint256 bondId = _tokenIdCounter.current();                                                                   
+    _tokenIdCounter.increment();                                                                                  
+    _mint(to, bondId);                                                                                            
     bonds[bondId] = Bond(to, expiry, rdpxAmount);                                                                 
                                                                                                                   
     emit BondMinted(to, bondId, expiry, rdpxAmount);                                                              
   }                                                                                                               
                                                                                                                   
-  /// @dev Internal function to mint a bond position token                                                        
-  /// @param to the address to mint the position to                                                               
-  function _mintToken(address to) private returns (uint256 tokenId) {                                             
-    tokenId = _tokenIdCounter.current();                                                                          
-    _tokenIdCounter.increment();                                                                                  
-    _mint(to, tokenId);                                                                                           
-  }                                                                                                               
-                                                                                                                  
```
#### Gas saving for `mint()` function obtained via protocol test: Avg 46 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 184661 | 197461 | 192194 | 6 |
| After |  184615 | 197415 |  192148 | 6 |



## [G-03] Redundant state variable getters
Getters for public state variables are automatically generated so there is no need to code them manually as this increases deployment cost.
### 1 Instance
1. #### Make the `delegates` state variable private since the `getDelegatePosition()` function could represent a getter function for the state variable.
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L121
```solidity
file: contracts/core/RdpxV2Core.sol

121:  Delegate[] public delegates;  
.
.
.
1260:  function getDelegatePosition( @audit getter function for public state variable delegates
1261:    uint256 _delegateId
1262:  )
1263:    public
1264:    view
1265:    returns (
1266:      address delegate,
1267:      uint256 amount,
1268:      uint256 fee,
1269:      uint256 activeCollateral
1270:    )
1271:  {
1272:    Delegate memory delegatePosition = delegates[_delegateId];
1273:    return (
1274:      delegatePosition.owner,
1275:      delegatePosition.amount,
1276:      delegatePosition.fee,
1277:      delegatePosition.activeCollateral
1278:    );
1279:  }
```
Getter functions are automatically generated by the compiler for public state variables so it is redundant and unnecessary for you to manually define a getter function for a public state variable as this would increase deployment cost. You could either make the state variable `delegates` private or refactor the code to remove the getter function you wrote.
```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
index e28a65c..83f9ff5 100644
--- a/contracts/core/RdpxV2Core.sol
+++ b/contracts/core/RdpxV2Core.sol
@@ -118,7 +118,7 @@ contract RdpxV2Core is
   bool public putOptionsRequired;

   /// @notice Delegates array
-  Delegate[] public delegates;
+  Delegate[] private delegates;
```

## [G-04] Use the existing Local variable/global variable when equal to a state variable to avoid reading from state

### 4 Instances
1. #### Use stack variables `_perpetualAtlanticVault`, `_dopexAMMRouter`, `_dpxEthCurvePool` and `_rdpxV2ReceiptToken` in place of `addresses.perpetualAtlanticVault`, `addresses.dopexAMMRouter`, `addresses.dpxEthCurvePool` and `addresses.rdpxV2ReceiptToken` respectively to avoid reading from state.
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L327
```solidity
file: contracts/core/RdpxV2Core.sol

304:  function setAddresses(
305:    address _dopexAMMRouter,
306:    address _dpxEthCurvePool,
307:    address _rdpxDecayingBonds,
308:    address _perpetualAtlanticVault,
309:    address _perpetualAtlanticVaultLP,
310:    address _rdpxReserve,
311:    address _rdpxV2ReceiptToken,
312:    address _feeDistributor,
313:    address _reLPContract,
314:    address _receiptTokenBonds
315:  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
.
.
.
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
339:    IERC20WithBurn(weth).approve(
340:      addresses.perpetualAtlanticVault, @audit use _perpetualAtlanticVault variable instead
341:      type(uint256).max
342:    );
343:    IERC20WithBurn(weth).approve(addresses.dopexAMMRouter, type(uint256).max); @audit use _dopexAMMRouter variable instead
344:    IERC20WithBurn(weth).approve(addresses.dpxEthCurvePool, type(uint256).max); @audit use _dpxEthCurvePool variable instead
345:    IERC20WithBurn(weth).approve(
346:      addresses.rdpxV2ReceiptToken,  @audit use _rdpxV2ReceiptToken variable instead
347:      type(uint256).max
348:    );
349:    emit LogSetAddresses(addresses);
350:  }
```
In the `setAddresses()` function above we could use the stack variables `_perpetualAtlanticVault`, `_dopexAMMRouter`, `_dpxEthCurvePool` and `_rdpxV2ReceiptToken` in place of `addresses.perpetualAtlanticVault`, `addresses.dopexAMMRouter`, `addresses.dpxEthCurvePool` and `addresses.rdpxV2ReceiptToken` respectively to avoid the gas expensive operation of reading from state. In implementing this change a cheaper stack read `3 gas` would be used in place of 1 `SLOAD Gcoldaccess` `2100` gas units and 3 `SLOAD Gwarmaccess` `100` gas units. The diff below shows how the code should be refactored:
```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
index e28a65c..2feca15 100644
--- a/contracts/core/RdpxV2Core.sol
+++ b/contracts/core/RdpxV2Core.sol
@@ -337,13 +337,13 @@ contract RdpxV2Core is
       receiptTokenBonds: _receiptTokenBonds
     });
     IERC20WithBurn(weth).approve(
-      addresses.perpetualAtlanticVault,
+      _perpetualAtlanticVault,
       type(uint256).max
     );
-    IERC20WithBurn(weth).approve(addresses.dopexAMMRouter, type(uint256).max);
-    IERC20WithBurn(weth).approve(addresses.dpxEthCurvePool, type(uint256).max);
+    IERC20WithBurn(weth).approve(_dopexAMMRouter, type(uint256).max);
+    IERC20WithBurn(weth).approve(_dpxEthCurvePool, type(uint256).max);
     IERC20WithBurn(weth).approve(
-      addresses.rdpxV2ReceiptToken,
+      _rdpxV2ReceiptToken,
       type(uint256).max
     );
     emit LogSetAddresses(addresses);

```
#### Gas saving for `setAddresses()` function obtained via protocol test: Avg 298 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 35245 | 330052 | 298196 | 18 |
| After | 33245 | 329737 | 297898 | 18 |


2. #### Use stack variable `_perpetualAtlanticVaultLP` in place of `addresses.perpetualAtlanticVaultLP` to avoid reading from state.
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L198
```solidity
file: contracts/perp-vault/PerpetualAtlanticVault.sol

181:  function setAddresses(
182:    address _optionPricing,
183:    address _assetPriceOracle,
184:    address _volatilityOracle,
185:    address _feeDistributor,
186:    address _rdpx,
187:    address _perpetualAtlanticVaultLP,
188:    address _rdpxV2Core
189:  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
.
.
.
198:    addresses = Addresses({
199:      optionPricing: _optionPricing,
200:      assetPriceOracle: _assetPriceOracle,
201:      volatilityOracle: _volatilityOracle,
202:      feeDistributor: _feeDistributor,
203:      rdpx: _rdpx,
204:      perpetualAtlanticVaultLP: _perpetualAtlanticVaultLP,
205:      rdpxV2Core: _rdpxV2Core
206:    });
207:    collateralToken.safeApprove(
208:      addresses.perpetualAtlanticVaultLP, @audit use _perpetualAtlanticVaultLP variable instead
209:      type(uint256).max
210:    );
211:    emit AddressesSet(addresses);
212:  }
```
In the `setAddresses()` function above we could use the stack variable `_perpetualAtlanticVaultLP` in place of `addresses.perpetualAtlanticVaultLP` to avoid the gas expensive operation of reading from state. In implementing this change a cheaper stack read `3 gas` would be used in place of 1 `SLOAD Gcoldaccess` `2100` gas units. The diff below shows how the code should be done:
```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
index 88ac2b3..73c6d62 100644
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
@@ -205,7 +205,7 @@ contract PerpetualAtlanticVault is
       rdpxV2Core: _rdpxV2Core
     });
     collateralToken.safeApprove(
-      addresses.perpetualAtlanticVaultLP,
+      _perpetualAtlanticVaultLP,
       type(uint256).max
     );
     emit AddressesSet(addresses);
```
#### Gas saving for `setAddresses()` function obtained via protocol test: Avg 97 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 71992 | 187992 | 185092 | 40 |
| After | 71967 | 187895 | 184995 | 40 |


3. #### Define a stack variable assign `IERC20WithBurn(_collateralToken)` to the variable and use the stack variable in  place of `collateralToken` to avoid reading from state.
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L121
```solidity
file: contracts/perp-vault/PerpetualAtlanticVault.sol

113:  constructor(
114:    string memory _name,
115:    string memory _symbol,
116:    address _collateralToken,
117:    uint256 _gensis
118:  ) ERC721(_name, _symbol) {
119:    _validate(_collateralToken != address(0), 1);
120:
121:    collateralToken = IERC20WithBurn(_collateralToken);
122:    underlyingSymbol = collateralToken.symbol();
123:    collateralPrecision = 10 ** collateralToken.decimals();
124:    genesis = _gensis;
125:
126:    _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
127:    _setupRole(MANAGER_ROLE, msg.sender);
128:  }
```
In the `PerpetualAtlanticVault` contract constructor we could avoid reading from state in the `collateralToken.symbol()` and `collateralToken.decimals()` expressions by declaring a variable of type `IERC20WithBurn` assign the `IERC20WithBurn(_collateralToken)` expression to the newly created variable, assign the newly created variable to the state variable `collateralToken` and use the newly created variable in place of `collateralToken` in the `collateralToken.symbol()` and `collateralToken.decimals()` expressions. Implementing this change would help avoid 1 `SLOAD Gcoldaccess` ` 2100 gas units` and 1 `SLOAD Gwarmaccess` ` 100 gas units` and replace these opcodes with cheaper stack reads `3 gas units`. The diff below shows how this could be done:
```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
index 88ac2b3..4d659c7 100644
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
@@ -118,9 +118,10 @@ contract PerpetualAtlanticVault is
   ) ERC721(_name, _symbol) {
     _validate(_collateralToken != address(0), 1);

-    collateralToken = IERC20WithBurn(_collateralToken);
-    underlyingSymbol = collateralToken.symbol();
-    collateralPrecision = 10 ** collateralToken.decimals();
+    IERC20WithBurn IercCollteralToken = IERC20WithBurn(_collateralToken);
+    collateralToken = IercCollteralToken;
+    underlyingSymbol = IercCollteralToken.symbol();
+    collateralPrecision = 10 ** IercCollteralToken.decimals();
     genesis = _gensis;

     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
```

4. #### Use stack variables `_pair`, `_ammRouter`, `_tokenA` and `_tokenB` in place of `addresses.pair`, `addresses.ammRouter`, `addresses.tokenA` and `addresses.tokenB` respectively to avoid reading from state.
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L138
```solidity
file: contracts/reLP/ReLPContract.sol

115:  function setAddresses(
116:    address _tokenA,
117:    address _tokenB,
118:    address _pair,
119:    address _rdpxV2Core,
120:    address _tokenAReserve,
121:    address _amo,
122:    address _rdpxOracle,
123:    address _ammFactory,
124:    address _ammRouter
125:  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
.
.
.
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
149:
150:    IERC20WithBurn(addresses.pair).safeApprove(    @audit use _pair instead of addresses.pair
151:      addresses.ammRouter,     @audit use _ammRouter instead of addresses.ammRouter
152:      type(uint256).max
153:    );
154:
155:    IERC20WithBurn(addresses.tokenA).safeApprove(  @audit use _tokenA instead of addresses.tokenA
156:      addresses.ammRouter,     @audit use _ammRouter instead of addresses.ammRouter
157:      type(uint256).max
158:    );
159:
160:    IERC20WithBurn(addresses.tokenB).safeApprove(  @audit use _tokenB instead of addresses.tokenB
161:      addresses.ammRouter,     @audit use _ammRouter instead of addresses.ammRouter
162:      type(uint256).max
163:    );
164:  }
```
In the `setAddresses()` function above we could use stack variables `_pair`, `_ammRouter`, `_tokenA` and `_tokenB` in place of `addresses.pair`, `addresses.ammRouter`, `addresses.tokenA` and `addresses.tokenB` respectively to avoid the gas expensive operation of reading from state. In implementing this change a cheaper stack read `3 gas` would be used in place of 1 `SLOAD Gcoldaccess` `2100` gas units and 3 `SLOAD Gwarmaccess` `100` gas units. The diff below shows how the code should be refactored:
```diff
diff --git a/contracts/reLP/ReLPContract.sol b/contracts/reLP/ReLPContract.sol
index bc57fbd..d5d9664 100644
--- a/contracts/reLP/ReLPContract.sol
+++ b/contracts/reLP/ReLPContract.sol
@@ -147,18 +147,18 @@ contract ReLPContract is AccessControl {
       ammRouter: _ammRouter
     });

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
#### Gas saving for `setAddresses()` function obtained via protocol test: Avg 427 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 285281 | 285281 | 285281 | 1 |
| After |  284854 |  284854 |  284854 | 1 |



## [G-05] Avoid using state variable in emit
In scenarios where you have a memory, calldata variable or parameter with the same value as the state variable you should use the memory, calldata variable or parameter in the emit statement rather than state variable.

### 4 Instances
1. #### The `setAddresses()` function of `PerpetualAtlanticVault` contract can be refactored to avoid reading from state in the emit statement.
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L211
```solidity
file: contracts/perp-vault/PerpetualAtlanticVault.sol

181:  function setAddresses(
182:    address _optionPricing,
183:    address _assetPriceOracle,
184:    address _volatilityOracle,
185:    address _feeDistributor,
186:    address _rdpx,
187:    address _perpetualAtlanticVaultLP,
188:    address _rdpxV2Core
189:  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
.
.
.
198:    addresses = Addresses({
199:      optionPricing: _optionPricing,
200:      assetPriceOracle: _assetPriceOracle,
201:      volatilityOracle: _volatilityOracle,
202:      feeDistributor: _feeDistributor,
203:      rdpx: _rdpx,
204:      perpetualAtlanticVaultLP: _perpetualAtlanticVaultLP,
205:      rdpxV2Core: _rdpxV2Core
206:    });
207:    collateralToken.safeApprove(
208:      addresses.perpetualAtlanticVaultLP,
209:      type(uint256).max
210:    );
211:    emit AddressesSet(addresses);
212:  }
```
We can refactor `PerpetualAtlanticVault.setAddresses()` function to avoid reading form state in the emit statement. The diff below shows how the code could be refactored: 
```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol 
index 88ac2b3..51cb1c0 100644                                                                                  
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol                                                          
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol                                                          
@@ -195,7 +195,7 @@ contract PerpetualAtlanticVault is                                                         
     _validate(_perpetualAtlanticVaultLP != address(0), 1);                                                    
     _validate(_rdpxV2Core != address(0), 1);                                                                  
                                                                                                               
-    addresses = Addresses({                                                                                   
+    Addresses memory _addresses = Addresses({                                                                 
       optionPricing: _optionPricing,                                                                          
       assetPriceOracle: _assetPriceOracle,                                                                    
       volatilityOracle: _volatilityOracle,                                                                    
@@ -204,11 +204,12 @@ contract PerpetualAtlanticVault is                                                       
       perpetualAtlanticVaultLP: _perpetualAtlanticVaultLP,                                                    
       rdpxV2Core: _rdpxV2Core                                                                                 
     });                                                                                                       
+    addresses = _addresses;                                                                                   
     collateralToken.safeApprove(                                                                              
       _perpetualAtlanticVaultLP,                                                                     
       type(uint256).max                                                                                       
     );                                                                                                        
-    emit AddressesSet(addresses);                                                                             
+    emit AddressesSet(_addresses);                                                                            
   }
```
#### Gas saving for `setAddresses()` function obtained via protocol test: Avg 590 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 71992 | 187992 | 185092 | 1 |
| After |  71402 |  187402 | 184502 | 1 |


2. #### The `payFunding()` function of `PerpetualAtlanticVault` contract can be refactored to avoid reading from state in the emit statement.
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L389-#L393
```solidity
file: contracts/perp-vault/PerpetualAtlanticVault.sol

372:  function payFunding() external onlyRole(RDPXV2CORE_ROLE) returns (uint256) {
373:    _whenNotPaused();
374:    _isEligibleSender();
375:
376:    _validate(
377:      totalActiveOptions ==
378:        fundingPaymentsAccountedFor[latestFundingPaymentPointer],
379:      6
380:    );
381:
382:    collateralToken.safeTransferFrom(
383:      addresses.rdpxV2Core,
384:      address(this),
385:      totalFundingForEpoch[latestFundingPaymentPointer]
386:    );
387:    _updateFundingRate(totalFundingForEpoch[latestFundingPaymentPointer]);
388:
389:    emit PayFunding(
390:      msg.sender,
391:      totalFundingForEpoch[latestFundingPaymentPointer],
392:      latestFundingPaymentPointer
393:    );
394:
395:    return (totalFundingForEpoch[latestFundingPaymentPointer]);
396:  }
```
We can refactor `PerpetualAtlanticVault.payFunding()` function to avoid reading form state in the emit statement. The diff below shows how the code could be refactored: 
```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
index 88ac2b3..fdbf05e 100644                                                                                 
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol                                                         
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol                                                         
@@ -373,26 +373,29 @@ contract PerpetualAtlanticVault is                                                      
     _whenNotPaused();                                                                                        
     _isEligibleSender();                                                                                     
                                                                                                              
+    uint256 _latestFundingPaymentPointer = latestFundingPaymentPointer;                                      
     _validate(                                                                                               
       totalActiveOptions ==                                                                                  
-        fundingPaymentsAccountedFor[latestFundingPaymentPointer],                                            
+        fundingPaymentsAccountedFor[_latestFundingPaymentPointer],                                           
       6                                                                                                      
     );                                                                                                       
                                                                                                              
+    uint256 _totalFundingForEpochPointer = totalFundingForEpoch[latestFundingPaymentPointer];                
+                                                                                                             
     collateralToken.safeTransferFrom(                                                                        
       addresses.rdpxV2Core,                                                                                  
       address(this),                                                                                         
-      totalFundingForEpoch[latestFundingPaymentPointer]                                                      
+      _totalFundingForEpochPointer                                                                           
     );                                                                                                       
-    _updateFundingRate(totalFundingForEpoch[latestFundingPaymentPointer]);                                   
+    _updateFundingRate(_totalFundingForEpochPointer);                                                        
                                                                                                              
     emit PayFunding(                                                                                         
       msg.sender,                                                                                            
-      totalFundingForEpoch[latestFundingPaymentPointer],                                                     
-      latestFundingPaymentPointer                                                                            
+      _totalFundingForEpochPointer,                                                                          
+      _latestFundingPaymentPointer                                                                           
     );                                                                                                       
                                                                                                              
-    return (totalFundingForEpoch[latestFundingPaymentPointer]);                                              
+    return (_totalFundingForEpochPointer);                                                                   
   }                                                                                                          
```
#### Gas saving for `payFunding()` function obtained via protocol test: Avg 754 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 3578 | 34705 | 27125 | 13 |
| After | 3581 |  33888 | 26371 | 13 |


3. #### The `calculateFunding()` function of `PerpetualAtlanticVault` contract can be refactored to avoid reading from state in the emit statement.
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L451-#L457
```solidity
file: 

405:  function calculateFunding(
406:    uint256[] memory strikes
407:  ) external nonReentrant returns (uint256 fundingAmount) {
408:    _whenNotPaused();
409:    _isEligibleSender();
410:
411:    updateFundingPaymentPointer();
412:
413:    for (uint256 i = 0; i < strikes.length; i++) {
414:      _validate(optionsPerStrike[strikes[i]] > 0, 4);
415:      _validate(
416:        latestFundingPerStrike[strikes[i]] != latestFundingPaymentPointer,
417:        5
418:      );
419:      uint256 strike = strikes[i];
420:
421:      uint256 amount = optionsPerStrike[strike] -
422:        fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
423:          strike
424:        ];
425:
426:      uint256 timeToExpiry = nextFundingPaymentTimestamp() -
427:        (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration));
428:
429:      uint256 premium = calculatePremium(
430:        strike,
431:        amount,
432:        timeToExpiry,
433:        getUnderlyingPrice()
434:      );
435:
436:      latestFundingPerStrike[strike] = latestFundingPaymentPointer;
437:      fundingAmount += premium;
438:
439:      // Record number of options that funding payments were accounted for, for this epoch
440:      fundingPaymentsAccountedFor[latestFundingPaymentPointer] += amount;
441:
442:      // record the number of options funding has been accounted for the epoch and strike
443:      fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
444:        strike
445:      ] += amount;
446:
447:      // Record total funding for this epoch
448:      // This does not need to be done in purchase() since it's already accounted for using `addProceeds()`
449:      totalFundingForEpoch[latestFundingPaymentPointer] += premium;
450:
451:      emit CalculateFunding(
452:        msg.sender,
453:        amount,
454:        strike,
455:        premium,
456:        latestFundingPaymentPointer
457:      );
458:    }
459:  }
```
We can refactor `PerpetualAtlanticVault.calculateFunding()` function to avoid reading from state in the emit statement. The diff below shows how the code could be refactored: 

```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
index 88ac2b3..20abf24 100644                                                                                 
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol                                                         
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol                                                         
@@ -410,21 +410,22 @@ contract PerpetualAtlanticVault is                                                      
                                                                                                              
     updateFundingPaymentPointer();                                                                           
                                                                                                              
+    uint256 _latestFundingPaymentPointer = latestFundingPaymentPointer;                                      
     for (uint256 i = 0; i < strikes.length; i++) {                                                           
       _validate(optionsPerStrike[strikes[i]] > 0, 4);                                                        
       _validate(                                                                                             
-        latestFundingPerStrike[strikes[i]] != latestFundingPaymentPointer,                                   
+        latestFundingPerStrike[strikes[i]] != _latestFundingPaymentPointer,                                  
         5                                                                                                    
       );                                                                                                     
       uint256 strike = strikes[i];                                                                           
                                                                                                              
       uint256 amount = optionsPerStrike[strike] -                                                            
-        fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][                                   
+        fundingPaymentsAccountedForPerStrike[_latestFundingPaymentPointer][                                  
           strike                                                                                             
         ];                                                                                                   
                                                                                                              
       uint256 timeToExpiry = nextFundingPaymentTimestamp() -                                                 
-        (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration));                                   
+        (genesis + ((_latestFundingPaymentPointer - 1) * fundingDuration));                                  
                                                                                                              
       uint256 premium = calculatePremium(                                                                    
         strike,                                                                                              
@@ -433,27 +434,27 @@ contract PerpetualAtlanticVault is                                                      
         getUnderlyingPrice()                                                                                 
       );                                                                                                     
                                                                                                              
-      latestFundingPerStrike[strike] = latestFundingPaymentPointer;                                          
+      latestFundingPerStrike[strike] = _latestFundingPaymentPointer;                                         
       fundingAmount += premium;                                                                              
                                                                                                              
       // Record number of options that funding payments were accounted for, for this epoch                   
-      fundingPaymentsAccountedFor[latestFundingPaymentPointer] += amount;                                    
+      fundingPaymentsAccountedFor[_latestFundingPaymentPointer] += amount;                                   
                                                                                                              
       // record the number of options funding has been accounted for the epoch and strike
       -      fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
+      fundingPaymentsAccountedForPerStrike[_latestFundingPaymentPointer][
         strike
       ] += amount;

       // Record total funding for this epoch
       // This does not need to be done in purchase() since it's already accounted for using `addProceeds()`
-      totalFundingForEpoch[latestFundingPaymentPointer] += premium;
+      totalFundingForEpoch[_latestFundingPaymentPointer] += premium;

       emit CalculateFunding(
         msg.sender,
         amount,
         strike,
         premium,
-        latestFundingPaymentPointer
+        _latestFundingPaymentPointer
       );
     }
   }                    
```
#### Gas saving for `calculateFunding()` function obtained via protocol test: Avg 902 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 64779 | 154379 | 99013 | 13 |
| After |  63248 |  152848 | 98111 | 13 |


4. #### The `updateFunding()` function of `PerpetualAtlanticVault` contract can be refactored to avoid reading from state in the emit statement.
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L519-#L523
```solidity
file: contracts/perp-vault/PerpetualAtlanticVault.sol

502:  function updateFunding() public {
503:    updateFundingPaymentPointer();
504:    uint256 currentFundingRate = fundingRates[latestFundingPaymentPointer];
505:    uint256 startTime = lastUpdateTime == 0
506:      ? (nextFundingPaymentTimestamp() - fundingDuration)
507:      : lastUpdateTime;
508:    lastUpdateTime = block.timestamp;
509:
510:    collateralToken.safeTransfer(
512:      addresses.perpetualAtlanticVaultLP,
513:      (currentFundingRate * (block.timestamp - startTime)) / 1e18
514:    );
515:
516:    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addProceeds(
517:      (currentFundingRate * (block.timestamp - startTime)) / 1e18
518:    );
519:
520:    emit FundingPaid(
521:      msg.sender,
522:      ((currentFundingRate * (block.timestamp - startTime)) / 1e18),
523:      latestFundingPaymentPointer
524:    );
525:  }
```

```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
index 88ac2b3..20abf24 100644                                                                                 
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol                                                         
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol      
@@ -501,7 +502,8 @@ contract PerpetualAtlanticVault is                          
    **/                                                                         
   function updateFunding() public {                                            
     updateFundingPaymentPointer();                                             
-    uint256 currentFundingRate = fundingRates[latestFundingPaymentPointer];    
+    uint _latestFundingPaymentPointer = latestFundingPaymentPointer;           
+    uint256 currentFundingRate = fundingRates[_latestFundingPaymentPointer];   
     uint256 startTime = lastUpdateTime == 0                                    
       ? (nextFundingPaymentTimestamp() - fundingDuration)                      
       : lastUpdateTime;                                                        
@@ -519,7 +521,7 @@ contract PerpetualAtlanticVault is                          
     emit FundingPaid(                                                          
       msg.sender,                                                              
       ((currentFundingRate * (block.timestamp - startTime)) / 1e18),           
-      latestFundingPaymentPointer                                              
+      _latestFundingPaymentPointer                                             
     );                                                                         
   }                                                                            
                                                                                
```
#### Gas saving for `updateFunding()` function obtained via protocol test: Avg 95 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 11472 | 53946 | 33743 | 42 |
| After |  11377 |  53851 | 33648 | 42 |


## [G-06] Structs can be modified to fit in fewer storage slots
The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if values combined are <= 32 bytes). If the variables packed together are retrieved together in functions (more likely with structs) we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

Some member types of the struct can be safely modified, and as result, these struct will fit in fewer storage slots. Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

### 1 Instance
1. #### The `token_id` member of the `Position` struct could be safely modified to type `uint96` therefore we could pack `token_id` and `collateral_address` into one storage slot to save 1 slot (~2000 gas).
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L40-#L47
```solidity
file: contracts/amo/UniV3LiquidityAmo.sol

40:  struct Position {
41:    uint256 token_id;    @audit could be modified to type uint96
42:    address collateral_address;
43:    uint128 liquidity; // the liquidity of the position
44:    int24 tickLower; // the tick range of the position
45:    int24 tickUpper;
46:    uint24 fee_tier;
47:  }
```
The `token_id` member of the `Position` struct could be safely modified to type `uint96` because the `uint96` type have 2^96 different values which would be enough to repesent the id's of the different tokens. This then allows us pack the `token_id` and `collateral_address` into one storage slot and we save 1 slot. 
```diff
diff --git a/contracts/amo/UniV3LiquidityAmo.sol b/contracts/amo/UniV3LiquidityAmo.sol
index 9129094..10dd2b7 100644
--- a/contracts/amo/UniV3LiquidityAmo.sol
+++ b/contracts/amo/UniV3LiquidityAmo.sol
@@ -38,7 +38,7 @@ contract UniV3LiquidityAMO is AccessControl, ERC721Holder {

   // Details about the AMO's uniswap positions
   struct Position {
-    uint256 token_id;
+    uint96 token_id;
     address collateral_address;
     uint128 liquidity; // the liquidity of the position
     int24 tickLower; // the tick range of the position
```
```
Estimated gas saved: 2000
```


## [G-07]  Unnecessary require statements

### 1 Instance 
1. #### The `require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter")` statement is unnecessary as the function has `onlyRole(MINTER_ROLE)` modifier in its header.
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L120
```solidity
file: RdpxDecayingBonds.sol

114:  function mint(
115:    address to,
116:    uint256 expiry,
117:    uint256 rdpxAmount
118:  ) external onlyRole(MINTER_ROLE) {
119:    _whenNotPaused();
120:    require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");  //@audit there's the onlyRole modifier:
.
.
.
.  }
```
The `mint()` function above defines an `onlyRole()` modifier in its header and still proceeds to perform a require check in which it calls the `hasRole()` function. This require statement is unnecessary as this is the job of the `onlyRole()` modifier defined in the functions header.
```diff
diff --git a/contracts/decaying-bonds/RdpxDecayingBonds.sol b/contracts/decaying-bonds/RdpxDecayingBonds.sol
index e89349e..983f88c 100644
--- a/contracts/decaying-bonds/RdpxDecayingBonds.sol
+++ b/contracts/decaying-bonds/RdpxDecayingBonds.sol
@@ -117,7 +117,6 @@ contract RdpxDecayingBonds is
     uint256 rdpxAmount
   ) external onlyRole(MINTER_ROLE) {
     _whenNotPaused();
-    require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
     uint256 bondId = _tokenIdCounter.current();
    _tokenIdCounter.increment();
    _mint(to, bondId);
    bonds[bondId] = Bond(to, expiry, rdpxAmount);
```
#### Gas saving for `mint()` function obtained via protocol test: Avg 317 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 184615 | 197415 | 192148 | 6 |
| After | 184298 |  197098 | 191831  | 6 |



## [G-08] Declaring Unnecessary variables
Some varibles were defined even though they are used once. Not defining variables can reduce gas cost and contract size.
### 1 Instance
1. #### Declaring the `current_position` variable is unnecessary
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L121
```solidity
file: UniV3LiquidityAmo.sol

119:  function collectFees() external onlyRole(DEFAULT_ADMIN_ROLE) {
120:    for (uint i = 0; i < positions_array.length; i++) {
121:      Position memory current_position = positions_array[i];
122:      INonfungiblePositionManager.CollectParams
123:        memory collect_params = INonfungiblePositionManager.CollectParams(
124:          current_position.token_id,
125:          rdpxV2Core,
126:          type(uint128).max,
127:          type(uint128).max
128:        );
129:
130:      // Send to custodian address
131:      univ3_positions.collect(collect_params);
132:    }
133:  }
```

In the `collectFees()` functions above declaring the `current_position` variable is unnecessary as it was only used once in the function we could use the `positions_array[i]` directly instead and save `3` gas units. The diff below shows how it could be refactored:  
```diff
diff --git a/contracts/amo/UniV3LiquidityAmo.sol b/contracts/amo/UniV3LiquidityAmo.sol      
index 9129094..8f0d624 100644                                                               
--- a/contracts/amo/UniV3LiquidityAmo.sol                                                   
+++ b/contracts/amo/UniV3LiquidityAmo.sol                                                   
@@ -118,10 +118,9 @@ contract UniV3LiquidityAMO is AccessControl, ERC721Holder {            
   // Iterate through all positions and collect fees accumulated                            
   function collectFees() external onlyRole(DEFAULT_ADMIN_ROLE) {                           
     for (uint i = 0; i < positions_array.length; i++) {                                    
-      Position memory current_position = positions_array[i];                               
       INonfungiblePositionManager.CollectParams                                            
         memory collect_params = INonfungiblePositionManager.CollectParams(                 
-          current_position.token_id,                                                       
+          positions_array[i].token_id,                                                     
           rdpxV2Core,                                                                      
           type(uint128).max,                                                               
           type(uint128).max                                                                
```
```
Estimated gas saved: 3 gas units
```


## [G-09] Why emitting two events when you can emit one

### 1 Instance
1. #### Rather than emitting `emit log(positions_array.length)` and `emit log(positions_mapping[pos.token_id].token_id)` we can emit a single event that contain both parameters.
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L268-#L268
```solidity
file: contracts/amo/UniV3LiquidityAmo.sol

213:  function removeLiquidity(
214:    uint256 positionIndex,
215:    uint256 minAmount0,
216:    uint256 minAmount1
217:  ) public onlyRole(DEFAULT_ADMIN_ROLE) {
.
.
.
268:    emit log(positions_array.length);
269:    emit log(positions_mapping[pos.token_id].token_id);
270:  }
```
The `log` event of `UniV3LiquidityAMO` contract could be refactored to having 2 parameters rather than 1 this way we would avoid having to emit the log event twice in the  `removeLiquidity()` function. The diff below shows how this could be done. 
```diff
diff --git a/contracts/amo/UniV3LiquidityAmo.sol b/contracts/amo/UniV3LiquidityAmo.sol
index 9129094..3e58e92 100644
--- a/contracts/amo/UniV3LiquidityAmo.sol
+++ b/contracts/amo/UniV3LiquidityAmo.sol
@@ -265,8 +265,7 @@ contract UniV3LiquidityAMO is AccessControl, ERC721Holder {
     // send tokens to rdpxV2Core
     _sendTokensToRdpxV2Core(tokenA, tokenB);

-    emit log(positions_array.length);
-    emit log(positions_mapping[pos.token_id].token_id);
+    emit log(positions_array.length, positions_mapping[pos.token_id].token_id);
   }

   // Swap tokenA into tokenB using univ3_router.ExactInputSingle()
@@ -376,5 +375,5 @@ contract UniV3LiquidityAMO is AccessControl, ERC721Holder {
   /*
    **  burn tokenAmount from the recipient and send tokens to the receipient
    */
-  event log(uint);
+  event log(uint indexed, uint indexed);
 }
```
#### Gas saving for `removeLiquidity()` function obtained via protocol test: Avg 472 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 179258 | 179258 | 179258 | 1 |
| After |  178786 | 178786 |  178786 | 1 |



## [G-10] Use constants for state variables whose value is known beforehand and is never changed
When you declare a state variable as a constant, you're telling the Solidity compiler that this variable's value is known at compile time and will never change during the contract's lifetime. Because of this:
The value of the constant state variable is computed and set at the time of contract deployment, not during runtime.

Since the value is known beforehand and cannot change, there is no need to store this value on the Ethereum blockchain, as it can be computed directly whenever needed.
When you access the constant state variable in your contract functions, it consume very little gas because the value is readily available (saved in the contracts bytecode) and does not require any computation.

So, by using constants for state variables whose value is known beforehand and is static (never changed), you save gas because you avoid unnecessary storage and computation costs associated with regular state variables.

### 1 Instance
1. #### The `roundingPrecision` state variable should be made a constatnt since its value is known before hand and never modified.
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L104
```solidity
file: contracts/perp-vault/PerpetualAtlanticVault.sol

104:   uint256 public roundingPrecision = 1e6;  @audit should be made constant
```
Making the `roundingPrecision` state variable constant would help avoid `SLOAD` cold access (2100) and `SLOAD` warm access (100) and replace them with cheaper stack read `3` gas units. The diff below shows how the code could be refactored:
```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
index 88ac2b3..34978ce 100644
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
@@ -101,7 +101,7 @@ contract PerpetualAtlanticVault is
   uint256 public fundingDuration = 7 days;

   /// @dev the precision to round up to
-  uint256 public roundingPrecision = 1e6;
+  uint256 public constant roundingPrecision = 1e6;

```
```
Estimated gas saved: 2194 gas units
```


## [G-11] State variables only set in the constructor should be declared immutable
Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

### 2 Instances
1. #### The `perpetualAtlanticVault` state variable could be made `immutable` 
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L46
```solidity
file: contracts/perp-vault/PerpetualAtlanticVaultLP.sol

46:  IPerpetualAtlanticVault public perpetualAtlanticVault;
```
Making the `perpetualAtlanticVault` state variable immutable would help avoid `Gsset` (20000 gas units) in the constructor during deployment. In implementing this change a cheaper stack read `3 gas` would be used in place of 1 `SLOAD` (cold access 2100 gas units) in the functions `deposit()`, `redeem()`, `convertToShares()` which reads the variable `perpetualAtlanticVault` directly and also in the functions `lockCollateral()`, `unlockLiquidity()`, `addProceeds()`, `subtractLoss()`, `addRdpx()` which have the modifier `onlyPerpVault` that reads the variable `perpetualAtlanticVault`. The diff below shows how the code should be refactored:
```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVaultLP.sol b/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
index 5758161..2dc8637 100644
--- a/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
@@ -43,7 +43,7 @@ contract PerpetualAtlanticVaultLP is ERC20, IPerpetualAtlanticVaultLP {
   // ================================ STATE VARIABLES ================================ //

   /// @dev The address of the perpetual Atlantic Vault contract creating the lp token
-  IPerpetualAtlanticVault public perpetualAtlanticVault;
+  IPerpetualAtlanticVault public immutable perpetualAtlanticVault;

   /// @dev The collateral token
   ERC20 public collateral;
```
```
Estimated gas saved: 36773 gas units
```

2. #### The `rdpx` state variable could be made `immutable` 
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L67
```solidity
file: contracts/perp-vault/PerpetualAtlanticVaultLP.sol

67:  address public rdpx;
```
Making the `rdpx` state variable immutable would help avoid `Gsset` (20000 gas units) in the constructor during deployment. In implementing this change a cheaper stack read `3 gas` would be used in place of 1 `SLOAD` (cold access 2100 gas units) in the functions `redeem()` and `addRdpx()` which reads the variable `rdpx`. The diff below shows how the code should be refactored:
```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVaultLP.sol b/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
index 5758161..6f361f8 100644
--- a/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
@@ -64,7 +64,7 @@ contract PerpetualAtlanticVaultLP is ERC20, IPerpetualAtlanticVaultLP {
   uint256 private _rdpxCollateral;

   /// @dev address of rdpx token
-  address public rdpx;
+  address public immutable rdpx;

   /// @dev address of the rdpx rdpxV2Core contract
   address public rdpxRdpxV2Core;
```
```
Estimated gas saved: 24191 gas units
```


## [G-12] Unchecked block for addition that cannot overflow

### 6 Instances
1. #### `latestFundingPaymentPointer += 1` statement could be in an `unchecked` block since it cannot overflow
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L493
```solidity
file: contracts/perp-vault/PerpetualAtlanticVault.sol

426:  function updateFundingPaymentPointer() public {
427:    while (block.timestamp >= nextFundingPaymentTimestamp()) {
.
.
.
439:      latestFundingPaymentPointer += 1; @audit increment can be in unchecked block
440:      emit FundingPaymentPointerUpdated(latestFundingPaymentPointer);
441    }
442:  }
```
In the `PerpetualAtlanticVault` contract the value of the state variable `latestFundingPaymentPointer` is only updated in the `updateFundingPaymentPointer()` function in the statement `latestFundingPaymentPointer += 1` which is just incrementing its previous value by 1. The type of `latestFundingPaymentPointer` is `uint256` which have 2^256 different values so it would take almost enternity for an overflow to occur so we could safely put the `latestFundingPaymentPointer += 1` in an `unchecked` block. The diff below shows how the code could be refactored:
```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol 
index 88ac2b3..517b330 100644                                                                                  
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol                                                          
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol                                                          
@@ -489,8 +489,10 @@ contract PerpetualAtlanticVault is                                                        
           latestFundingPaymentPointer                                                                         
         );                                                                                                    
       }                                                                                                       
-                                                                                                              
-      latestFundingPaymentPointer += 1;                                                                       
+      unchecked {                                                                                             
+        latestFundingPaymentPointer += 1;                                                                     
+      }                                                                                                       
+                                                                                                              
       emit FundingPaymentPointerUpdated(latestFundingPaymentPointer);                                         
     }                                                                                                         
   }                                                                                                           
                                                                                                               
```
#### Gas saving for `updateFundingPaymentPointer()` function obtained via protocol test: Avg 161 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 741 | 85407 |  27932 | 14 |
| After | 741 | 85219 |  27771 | 14 |


2. #### `block.timestamp + 1` statement of `UniV2LiquidityAMO.addLiquidity()` could be in an `unchecked` block since it cannot overflow
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L231
```solidity
file: contracts/amo/UniV2LiquidityAmo.sol

189:  function addLiquidity(
190:    uint256 tokenAAmount,
191:    uint256 tokenBAmount,
192:    uint256 tokenAAmountMin,
193:    uint256 tokenBAmountMin
194:  )
195:    external
196:    onlyRole(DEFAULT_ADMIN_ROLE)
197:    returns (uint256 tokenAUsed, uint256 tokenBUsed, uint256 lpReceived)
198:  {
199:    // approve the AMM Router
.
.
.
221:    // add Liquidity
222:    (tokenAUsed, tokenBUsed, lpReceived) = IUniswapV2Router(addresses.ammRouter)
223:      .addLiquidity(
224:        addresses.tokenA,
225:        addresses.tokenB,
226:        tokenAAmount,
227:        tokenBAmount,
228:        tokenAAmountMin,
229:        tokenBAmountMin,
230:        address(this),
231:        block.timestamp + 1    @audit can be unchecked
232:      );
.
.
.
250:  }
```
In the `addLiquidity` function of `UniV2LiquidityAMO` contract the expression `block.timestamp + 1` was passed as an argument to `addLiquidity()` function of `IUniswapV2Router`. This argument represents the value of the `deadline` parameter of the `addLiquidity()` function and this `deadline` paramenter is of type `uint256`. 2^256 as a unix timestamp represents approximately 3.6658949e+69 years into the future. This is an incredibly large number and far beyond the age of the universe therefore it would take enternity for `block.timestamp + 1` expression to overflow so we could safely refactor the code such that this expression is in an `unchecked` block. The diff below shows the code could be refactored:
```diff
diff --git a/contracts/amo/UniV2LiquidityAmo.sol b/contracts/amo/UniV2LiquidityAmo.sol
index ce92d43..ecba957 100644
--- a/contracts/amo/UniV2LiquidityAmo.sol
+++ b/contracts/amo/UniV2LiquidityAmo.sol
@@ -218,6 +218,11 @@ contract UniV2LiquidityAMO is AccessControl {
       tokenBAmount
     );

+    uint256 deadline;
+    unchecked {
+      deadline = block.timestamp + 1;
+    }
+
     // add Liquidity
     (tokenAUsed, tokenBUsed, lpReceived) = IUniswapV2Router(addresses.ammRouter)
       .addLiquidity(
@@ -228,7 +233,7 @@ contract UniV2LiquidityAMO is AccessControl {
         tokenAAmountMin,
         tokenBAmountMin,
         address(this),
-        block.timestamp + 1
+        deadline
       );

     // update LP token Balance
```
#### Gas saving for `addLiquidity()` function obtained via protocol test: Avg 136 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 192860 | 214708 |  207425 | 3 |
| After | 192725 | 214572 |  207289 | 3 |


3. #### `block.timestamp + 1` statement of `UniV2LiquidityAMO.removeLiquidity()` could be in an `unchecked` block since it cannot overflow
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L279
```solidity
file: contracts/amo/UniV2LiquidityAmo.sol

258:  function removeLiquidity(
259:    uint256 lpAmount,
260:    uint256 tokenAAmountMin,
261:    uint256 tokenBAmountMin
262:  )
263:    external
264:    onlyRole(DEFAULT_ADMIN_ROLE)
265:    returns (uint256 tokenAReceived, uint256 tokenBReceived)
266:  {
267:    // approve the AMM Router
268:    IERC20WithBurn(addresses.pair).safeApprove(addresses.ammRouter, lpAmount);
269:
270:    // remove liquidity
271:    (tokenAReceived, tokenBReceived) = IUniswapV2Router(addresses.ammRouter)
272:      .removeLiquidity(
273:        addresses.tokenA,
274:        addresses.tokenB,
275:        lpAmount,
276:        tokenAAmountMin,
277:        tokenBAmountMin,
278:        address(this),
279:        block.timestamp + 1    @audit can be unchecked
280:      );
.
.
.
296:  }
```
In the `removeLiquidity()` function of `UniV2LiquidityAMO` contract the expression `block.timestamp + 1` was passed as an argument to `removeLiquidity()` function of `IUniswapV2Router`. This argument represents the value of the `deadline` parameter of the `removeLiquidity()` function and this `deadline` paramenter is of type `uint256`. 2^256 as a unix timestamp represents approximately 3.6658949e+69 years into the future. This is an incredibly large number and far beyond the age of the universe therefore it would take enternity for `block.timestamp + 1` expression to overflow so we could safely refactor the code such that this expression is in an `unchecked` block. The diff below shows the code could be refactored:
```diff
diff --git a/contracts/amo/UniV2LiquidityAmo.sol b/contracts/amo/UniV2LiquidityAmo.sol
index ce92d43..2bb12fb 100644
--- a/contracts/amo/UniV2LiquidityAmo.sol
+++ b/contracts/amo/UniV2LiquidityAmo.sol
@@ -267,6 +267,10 @@ contract UniV2LiquidityAMO is AccessControl {
     // approve the AMM Router
     IERC20WithBurn(addresses.pair).safeApprove(addresses.ammRouter, lpAmount);

+    uint256 deadline;
+    unchecked {
+      deadline = block.timestamp + 1
+    }
     // remove liquidity
     (tokenAReceived, tokenBReceived) = IUniswapV2Router(addresses.ammRouter)
       .removeLiquidity(
@@ -276,7 +280,7 @@ contract UniV2LiquidityAMO is AccessControl {
         tokenAAmountMin,
         tokenBAmountMin,
         address(this),
-        block.timestamp + 1
+        deadline
       );

     // update LP token Balance
```
#### Gas saving for `removeLiquidity()` function obtained via protocol test: Avg 128 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 125632 | 125632 |  125632 | 2 |
| After | 125504 | 125504 |  125504 | 2 |


4. #### `block.timestamp + 1` statement of `UniV2LiquidityAMO.swap()` could be in an `unchecked` block since it cannot overflow
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L342
```solidity
file: contracts/amo/UniV2LiquidityAmo.sol

304:  function swap(
305:    uint256 token1Amount,
306:    uint256 token2AmountOutMin,
307:    bool swapTokenAForTokenB
308:  ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256 token2Amount) {
309:    address token1;
310:    address token2;
.
.
.
335:    // swap token A for token B
336:    token2Amount = IUniswapV2Router(addresses.ammRouter)
337:      .swapExactTokensForTokens(
338:        token1Amount,
339:        token2AmountOutMin,
340:        path,
341:        address(this),
342:        block.timestamp + 1    @audit can be unchecked
343:      )[path.length - 1];
.
.
.
355:  }
```
In the `swap()` function of `UniV2LiquidityAMO` contract the expression `block.timestamp + 1` was passed as an argument to `swapExactTokensForTokens()` function of `IUniswapV2Router`. This argument represents the value of the `deadline` parameter of the `swapExactTokensForTokens()` function and this `deadline` paramenter is of type `uint256`. 2^256 as a unix timestamp represents approximately 3.6658949e+69 years into the future. This is an incredibly large number and far beyond the age of the universe therefore it would take enternity for `block.timestamp + 1` expression to overflow so we could safely refactor the code such that this expression is in an `unchecked` block. The diff below shows the code could be refactored:
```diff
diff --git a/contracts/amo/UniV2LiquidityAmo.sol b/contracts/amo/UniV2LiquidityAmo.sol
index ce92d43..579341a 100644
--- a/contracts/amo/UniV2LiquidityAmo.sol
+++ b/contracts/amo/UniV2LiquidityAmo.sol
@@ -332,6 +332,10 @@ contract UniV2LiquidityAMO is AccessControl {
     path[0] = token1;
     path[1] = token2;

+    uint256 deadline;
+    unchecked {
+      deadline = block.timestamp + 1;
+    }
     // swap token A for token B
     token2Amount = IUniswapV2Router(addresses.ammRouter)
       .swapExactTokensForTokens(
@@ -339,7 +343,7 @@ contract UniV2LiquidityAMO is AccessControl {
         token2AmountOutMin,
         path,
         address(this),
-        block.timestamp + 1
+        deadline
       )[path.length - 1];

     // send tokens back to rdpxV2Core
```
#### Gas saving for `swap()` function obtained via protocol test: Avg 128 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 97326 | 97374 |  97350 | 4 |
| After | 97287 | 97335 |  97311 | 4 |

 
5. #### `block.timestamp + 10` statement of `RdpxV2Core.lowerDepeg()` could be in an `unchecked` block since it cannot overflow
 https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1103
```solidity
file: contracts/core/RdpxV2Core.sol

1080:  function lowerDepeg(
1081:    uint256 _rdpxAmount,
1082:    uint256 _wethAmount,
1083:    uint256 minamountOfWeth,
1084:    uint256 minOut
1085:  ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256 dpxEthReceived) {
1086:    _isEligibleSender();
.
.
.
1097:      amountOfWethOut = IUniswapV2Router(addresses.dopexAMMRouter)
1098:        .swapExactTokensForTokens(
1099:          _rdpxAmount,
1100:          minamountOfWeth,
1101:          path,
1102:          address(this),
1103:          block.timestamp + 10     @audit can be unchecked
1104:        )[path.length - 1];
.
.
.
1124:  }
```
In the `lowerDepeg()` function of `RdpxV2Core` contract the expression `block.timestamp + 10` was passed as an argument to `swapExactTokensForTokens()` function of `IUniswapV2Router`. This argument represents the value of the `deadline` parameter of the `swapExactTokensForTokens()` function and this `deadline` paramenter is of type `uint256`. 2^256 as a unix timestamp represents approximately 3.6658949e+69 years into the future. This is an incredibly large number and far beyond the age of the universe therefore it would take enternity for `block.timestamp + 10` expression to overflow so we could safely refactor the code such that this expression is in an `unchecked` block. The diff below shows the code could be refactored:
```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
index e28a65c..5426e9d 100644
--- a/contracts/core/RdpxV2Core.sol
+++ b/contracts/core/RdpxV2Core.sol
@@ -1094,13 +1094,17 @@ contract RdpxV2Core is
       path[0] = reserveAsset[reservesIndex["RDPX"]].tokenAddress;
       path[1] = weth;

+      uint256 deadline;
+      unchecked {
+        deadline = block.timestamp + 10;
+      }
       amountOfWethOut = IUniswapV2Router(addresses.dopexAMMRouter)
         .swapExactTokensForTokens(
           _rdpxAmount,
           minamountOfWeth,
           path,
           address(this),
-          block.timestamp + 10
+          deadline
         )[path.length - 1];

       reserveAsset[reservesIndex["RDPX"]].tokenBalance -= _rdpxAmount;
```
#### Gas saving for `lowerDepeg()` function obtained via protocol test: Avg 24 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 12438 | 130625 |  73833 | 5 |
| After | 12438 | 130586 |  73809 | 5 |


6. #### `block.timestamp + 10` statements of `ReLPContract.reLP()` could be in `unchecked` blocks since they cannot overflow
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L264
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L283
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L294

```solidity
file: contracts/reLP/ReLPContract.sol

202:  function reLP(uint256 _amount) external onlyRole(RDPXV2CORE_ROLE) {
203:    // get the pool reserves
204:    (address tokenASorted, address tokenBSorted) = UniswapV2Library.sortTokens(
205:      addresses.tokenA,
206:      addresses.tokenB
207:    );
208:    (uint256 reserveA, uint256 reserveB) = UniswapV2Library.getReserves(
209:      addresses.ammFactory,
210:      tokenASorted,
211:      tokenBSorted
212:    );
.
.
.
257:    (, uint256 amountB) = IUniswapV2Router(addresses.ammRouter).removeLiquidity(
258:      addresses.tokenA,
259:      addresses.tokenB,
260:      lpToRemove,
261:      mintokenAAmount,
262:      mintokenBAmount,
263:      address(this),
264:      block.timestamp + 10     @audit can be unchecked
265:    );
.
.
.
277:    uint256 tokenAAmountOut = IUniswapV2Router(addresses.ammRouter)
278:      .swapExactTokensForTokens(
279:        amountB / 2,
280:        mintokenAAmount,
281:        path,
282:        address(this),
283:        block.timestamp + 10   @audit can be unchecked
284:      )[path.length - 1];
285:
286:    (, , uint256 lp) = IUniswapV2Router(addresses.ammRouter).addLiquidity(
287:      addresses.tokenA,
288:      addresses.tokenB,
289:      tokenAAmountOut,
290:      amountB / 2,
291:      0,
292:      0,
293:      address(this),
294:      block.timestamp + 10    @audit can be unchecked 
295:    );
.
.
.
307:  }
```
In the `reLP()` function of `ReLPContract` contract the expression `block.timestamp + 10` was passed as an argument to `swapExactTokensForTokens()`, `removeLiquidity()` and `addLiquidity()` functions of `IUniswapV2Router`. This argument represents the value of the `deadline` parameter of the `swapExactTokensForTokens()`, `removeLiquidity()` and `addLiquidity()` functions and this `deadline` paramenter is of type `uint256`. 2^256 as a unix timestamp represents approximately 3.6658949e+69 years into the future. This is an incredibly large number and far beyond the age of the universe therefore it would take enternity for `block.timestamp + 10` expression to overflow so we could safely refactor the code such that this expression is in an `unchecked` block. The diff below shows the code could be refactored:
```diff
diff --git a/contracts/reLP/ReLPContract.sol b/contracts/reLP/ReLPContract.sol
index bc57fbd..1a1e267 100644
--- a/contracts/reLP/ReLPContract.sol
+++ b/contracts/reLP/ReLPContract.sol
@@ -254,6 +254,10 @@ contract ReLPContract is AccessControl {
       ((tokenAToRemove * tokenAInfo.tokenAPrice) * liquiditySlippageTolerance) /
       1e16;

+      uint256 deadline;
+      unchecked {
+        deadline = block.timestamp + 10;
+      }
     (, uint256 amountB) = IUniswapV2Router(addresses.ammRouter).removeLiquidity(
       addresses.tokenA,
       addresses.tokenB,
@@ -261,7 +265,7 @@ contract ReLPContract is AccessControl {
       mintokenAAmount,
       mintokenBAmount,
       address(this),
-      block.timestamp + 10
+      deadline
     );

     address[] memory path;
@@ -280,7 +284,7 @@ contract ReLPContract is AccessControl {
         mintokenAAmount,
         path,
         address(this),
-        block.timestamp + 10
+        deadline
       )[path.length - 1];

     (, , uint256 lp) = IUniswapV2Router(addresses.ammRouter).addLiquidity(
@@ -291,7 +295,7 @@ contract ReLPContract is AccessControl {
       0,
       0,
       address(this),
-      block.timestamp + 10
+      deadline
     );
```
#### Gas saving for `reLP()` function obtained via protocol test: Avg 249 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 229271 | 229271 |  229271 | 1 |
| After | 229022 | 229022 |  229022 | 1 |



## [G-13] Unbounded Gas Consumption Risk Due to External Call Recipients
In the context of Solidity, external function calls without a specified gas limit present a significant risk. The callee contract has the potential to consume all the gas allocated to the transaction, causing an undesired revert and disrupt the function's execution. To mitigate this, it's recommended to explicitly set a gas limit when performing external calls using addr.call{gas: }. This limits the gas forwarded to the callee, preventing potential pitfalls and offering better control over transaction execution.

### 1 Instance
1. #### Specify gas for a low-level external call
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L344
```solidity
file: contracts/amo/UniV3LiquidityAmo.sol

344:  function execute(
345:    address _to,
346:    uint256 _value,
347:    bytes calldata _data
348:  ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (bool, bytes memory) {
349:    (bool success, bytes memory result) = _to.call{ value: _value }(_data); @audit specify gas for external call
350:    return (success, result);
351:  }
```
In the `execute()` function a low-level call was made without specifing the amount of gas to be used in the low-level call. This is bad practice as the called contract could possibly use up all the gas thereby causing the `execute()` function to revert and the transaction fails. This scenario could lead to bad consumer experience, you should specify the amount of gas to be used in a low-level call as this would prevent such occurences as only a predefined amount of gas can be used by the called contract. The code could be refactored to:
```diff
diff --git a/contracts/amo/UniV3LiquidityAmo.sol b/contracts/amo/UniV3LiquidityAmo.sol
index 9129094..2ba2e23 100644
--- a/contracts/amo/UniV3LiquidityAmo.sol
+++ b/contracts/amo/UniV3LiquidityAmo.sol
@@ -339,9 +339,10 @@ contract UniV3LiquidityAMO is AccessControl, ERC721Holder {
   function execute(
     address _to,
     uint256 _value,
-    bytes calldata _data
+    bytes calldata _data,
+    uint256 _gas
   ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (bool, bytes memory) {
-    (bool success, bytes memory result) = _to.call{ value: _value }(_data);
+    (bool success, bytes memory result) = _to.call{ value: _value, gas: _gas }(_data);
     return (success, result);
   }
```

## [G-14] Refactor external/internal function to avoid unnecessary SLOAD
The function below read storage slots that are previously read in the functions that invoke them. We can refactor the external/internal functions to pass cached storage variables as stack variables and avoid the extra storage reads that would otherwise take place in the internal functions.

## 1 Instance
- #### The `_sendTokensToRdpxV2Core()` internal function read storage slots that were read in functions that invokes it.
 https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L160

```solidity
file: contracts/amo/UniV2LiquidityAmo.sol

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
176:
177:    emit LogAssetsTransfered(msg.sender, tokenABalance, tokenBBalance);
178:  }
.
.
.
189:  function addLiquidity(
190:    uint256 tokenAAmount,
191:    uint256 tokenBAmount,
192:    uint256 tokenAAmountMin,
193:    uint256 tokenBAmountMin
194:  )
195:    external
196:    onlyRole(DEFAULT_ADMIN_ROLE)
197:    returns (uint256 tokenAUsed, uint256 tokenBUsed, uint256 lpReceived)
198:  {
199:    // approve the AMM Router
200:    IERC20WithBurn(addresses.tokenA).safeApprove(
201:      addresses.ammRouter,
202:      tokenAAmount
203:    );
.
.
.
238:    _sendTokensToRdpxV2Core();
.
.
.

250:  }
.
.
.
258:  function removeLiquidity(
259:    uint256 lpAmount,
260:    uint256 tokenAAmountMin,
261:    uint256 tokenBAmountMin
262:  )
263:    external
264:    onlyRole(DEFAULT_ADMIN_ROLE)
265:    returns (uint256 tokenAReceived, uint256 tokenBReceived)
266:  {
.
.
.
286:    _sendTokensToRdpxV2Core();
.
.
.
296:  }
.
.
.
304:  function swap(
305:    uint256 token1Amount,
306:    uint256 token2AmountOutMin,
307:    bool swapTokenAForTokenB
308:  ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256 token2Amount) {
.
.
.
346:    _sendTokensToRdpxV2Core();
.
.
.
355:  }
```
The `_sendTokensToRdpxV2Core()` function above reads storage for the values of `addresses.tokenA` and `addresses.tokenB` multiple times but these storage values were also read in all the functions (`addLiquidity()`, `removeLiquidity()` and `swap()`) that invokes `_sendTokensToRdpxV2Core()` so we could refactor the code such that we replace the storage reads (4 SLOAD 400 gas units) for these variables with cheaper stack reads. The diff below shows how the code could be refactored:
```diff
diff --git a/contracts/amo/UniV2LiquidityAmo.sol b/contracts/amo/UniV2LiquidityAmo.sol        
index ce92d43..1bfcfa2 100644                                                                 
--- a/contracts/amo/UniV2LiquidityAmo.sol                                                     
+++ b/contracts/amo/UniV2LiquidityAmo.sol                                                     
@@ -157,19 +157,19 @@ contract UniV2LiquidityAMO is AccessControl {                           
   /**                                                                                        
    * @dev sends token A and B to the rdpxV2Core                                              
    */                                                                                        
-  function _sendTokensToRdpxV2Core() internal {                                              
-    uint256 tokenABalance = IERC20WithBurn(addresses.tokenA).balanceOf(                      
+  function _sendTokensToRdpxV2Core(address tokenA, address tokenB) internal {                
+    uint256 tokenABalance = IERC20WithBurn(tokenA).balanceOf(                                
       address(this)                                                                          
     );                                                                                       
-    uint256 tokenBBalance = IERC20WithBurn(addresses.tokenB).balanceOf(                      
+    uint256 tokenBBalance = IERC20WithBurn(tokenB).balanceOf(                                
       address(this)                                                                          
     );                                                                                       
     // transfer token A and B from this contract to the rdpxV2Core                           
-    IERC20WithBurn(addresses.tokenA).safeTransfer(                                           
+    IERC20WithBurn(tokenA).safeTransfer(                                                     
       addresses.rdpxV2Core,                                                                  
       tokenABalance                                                                          
     );                                                                                       
-    IERC20WithBurn(addresses.tokenB).safeTransfer(                                           
+    IERC20WithBurn(tokenB).safeTransfer(                                                     
       addresses.rdpxV2Core,                                                                  
       tokenBBalance                                                                          
     );                                                                                       
@@ -196,23 +196,26 @@ contract UniV2LiquidityAMO is AccessControl {                           
     onlyRole(DEFAULT_ADMIN_ROLE)                                                             
     returns (uint256 tokenAUsed, uint256 tokenBUsed, uint256 lpReceived)                     
   {                                                                                          
+                                                                                             
+    address tokenA = addresses.tokenA;                                                       
+    address tokenB = addresses.tokenB;                                                       
     // approve the AMM Router                                                                
-    IERC20WithBurn(addresses.tokenA).safeApprove(                                            
+    IERC20WithBurn(tokenA).safeApprove(                                                      
       addresses.ammRouter,                                                                   
       tokenAAmount                                                                           
     );                                                                                       
-    IERC20WithBurn(addresses.tokenB).safeApprove(
+    IERC20WithBurn(tokenB).safeApprove(
       addresses.ammRouter,
       tokenBAmount
     );

     // transfer token A and B from the rdpxV2Core to this contract
-    IERC20WithBurn(addresses.tokenA).safeTransferFrom(
+    IERC20WithBurn(tokenA).safeTransferFrom(
       addresses.rdpxV2Core,
       address(this),
       tokenAAmount
     );
-    IERC20WithBurn(addresses.tokenB).safeTransferFrom(
+    IERC20WithBurn(tokenB).safeTransferFrom(
       addresses.rdpxV2Core,
       address(this),
       tokenBAmount
@@ -221,8 +224,8 @@ contract UniV2LiquidityAMO is AccessControl {
     // add Liquidity
     (tokenAUsed, tokenBUsed, lpReceived) = IUniswapV2Router(addresses.ammRouter)
       .addLiquidity(
-        addresses.tokenA,
-        addresses.tokenB,
+        tokenA,
+        tokenB,
         tokenAAmount,
         tokenBAmount,
         tokenAAmountMin,
@@ -235,7 +238,7 @@ contract UniV2LiquidityAMO is AccessControl {
     lpTokenBalance += lpReceived;

     // send unused token A and token B back to rdpxV2Core
-    _sendTokensToRdpxV2Core();
+    _sendTokensToRdpxV2Core(tokenA, tokenB);

     emit LogAddLiquidity(
       msg.sender,
@@ -267,11 +270,13 @@ contract UniV2LiquidityAMO is AccessControl {
     // approve the AMM Router
     IERC20WithBurn(addresses.pair).safeApprove(addresses.ammRouter, lpAmount);

+    address tokenA = addresses.tokenA;
+    address tokenB = addresses.tokenB;
     // remove liquidity
     (tokenAReceived, tokenBReceived) = IUniswapV2Router(addresses.ammRouter)
       .removeLiquidity(
-        addresses.tokenA,
-        addresses.tokenB,
+        tokenA,
+        tokenB,
         lpAmount,
         tokenAAmountMin,
         tokenBAmountMin,
@@ -283,7 +288,7 @@ contract UniV2LiquidityAMO is AccessControl {
     lpTokenBalance -= lpAmount;

     // send unused token A and token B back to rdpxV2Core
-    _sendTokensToRdpxV2Core();
+    _sendTokensToRdpxV2Core(tokenA, tokenB);

     emit LogRemoveLiquidity(
       msg.sender,
@@ -309,13 +314,15 @@ contract UniV2LiquidityAMO is AccessControl {
     address token1;
     address token2;

+    address tokenA = addresses.tokenA;
+    address tokenB = addresses.tokenB;
     // check to see if we are swapping token A for token B
     if (swapTokenAForTokenB) {
-      token1 = addresses.tokenA;
-      token2 = addresses.tokenB;
+      token1 = tokenA;
+      token2 = tokenB;
     } else {
-      token1 = addresses.tokenB;
-      token2 = addresses.tokenA;
+      token1 = tokenB;
+      token2 = tokenA;
     }
     // transfer token A from the rdpxV2Core to this contract
     IERC20WithBurn(token1).safeTransferFrom(
@@ -343,7 +350,7 @@ contract UniV2LiquidityAMO is AccessControl {
       )[path.length - 1];

     // send tokens back to rdpxV2Core
+    address tokenA = addresses.tokenA;                                
+    address tokenB = addresses.tokenB;                                
     // check to see if we are swapping token A for token B            
     if (swapTokenAForTokenB) {                                        
-      token1 = addresses.tokenA;                                      
-      token2 = addresses.tokenB;                                      
+      token1 = tokenA;                                                
+      token2 = tokenB;                                                
     } else {                                                          
-      token1 = addresses.tokenB;                                      
-      token2 = addresses.tokenA;                                      
+      token1 = tokenB;                                                
+      token2 = tokenA;                                                
     }                                                                 
     // transfer token A from the rdpxV2Core to this contract          
     IERC20WithBurn(token1).safeTransferFrom(                          
@@ -343,7 +350,7 @@ contract UniV2LiquidityAMO is AccessControl {      
       )[path.length - 1];                                             
                                                                       
     // send tokens back to rdpxV2Core                                 
-    _sendTokensToRdpxV2Core();                                        
+    _sendTokensToRdpxV2Core(tokenA, tokenB);                          
                                                                                                                   
```
```
Estimated gas saved: 388 gas units
```

## [G-15]  Can transfer 0
In Solidity, performing unnecessary operations can consume more gas than needed, leading to cost inefficiencies. For instance, if a transfer function doesn't have a zero amount check and someone calls it with a zero amount, unnecessary gas will be consumed in executing the function, even though the state of the contract remains the same. By implementing a zero amount check, such unnecessary function calls can be avoided, thereby saving gas and making the contract more efficient.

### 13 Instances

- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L149
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L210
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L215
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L283
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L319
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L344
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L354
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L355
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L169
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L653
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L662
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L98
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L105


## [G-16] Use assembly to check for address(0)
A simple zero address check can be written in assembly to save some gas. Using assembly for address comparisons in Solidity can save gas because it allows for more direct access to the Ethereum Virtual Machine (EVM), reducing the overhead of higher-level operations. Solidity's high-level abstraction simplifies coding but can introduce additional gas costs. Using assembly for simple operations like address comparisons can be more gas-efficient.

### 5 Instances
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L84-#L90
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L131-#L132
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L244
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L94-#L97
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L126-#L137


## CONCLUSION
As you embark on the journey of incorporating the recommended optimizations, we want to emphasize the utmost importance of proceeding with vigilance and dedicating thorough efforts to comprehensive testing. It is of paramount significance to ensure that the proposed alterations do not inadvertently introduce fresh vulnerabilities, while also successfully achieving the anticipated enhancements in performance.

We strongly advise conducting a meticulous and exhaustive evaluation of the modifications made to the codebase. This rigorous scrutiny and exhaustive assessment will play a pivotal role in affirming both the security and efficacy of the refactored code. Your careful attention to detail, coupled with the implementation of a robust testing framework, will provide the necessary assurance that the refined code aligns with your security objectives and effectively fulfills the intended performance optimizations.