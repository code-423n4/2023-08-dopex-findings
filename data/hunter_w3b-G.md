# Gas Optimization

# Summary

A majority of the optimizations were benchmarked via the protocol’s tests, i.e. using the following config: solc version 0.8.19. Optimizations that were not benchmarked are explained via EVM gas costs and opcodes.

Below are the overall average gas savings for the following tested functions, with all the optimizations applied:

Values obtained by running `forge test --gas-report`

| Function                                                        | Before  | After   | Avg Gas Savings |
| --------------------------------------------------------------- | ------- | ------- | --------------- |
| RdpxV2Core.addAssetTotokenReserves                              | 104095  | 104031  | 64              |
| PerpetualAtlanticVault.calculatePnl                             | 687     | 481     | 206             |
| PerpetualAtlanticVault.roundUp                                  | 1103    | 1076    | 27              |
| PerpetualAtlanticVault.settle                                   | 60186   | 59890   | 296             |
| PerpetualAtlanticVault.calculateFunding                         | 99013   | 98780   | 233             |
| RdpxV2Core.settle                                               | 64123   | 63477   | 646             |
| RdpxV2Core.bondWithDelegate                                     | 847375  | 846607  | 768             |
| ReLPContract.setAddresses                                       | 285281  | 285001  | 280             |
| UniV2LiquidityAmo.setAddresses                                  | 156876  | 156660  | 216             |
| UniV3LiquidityAmo.constructor                                   | 2341355 | 2341265 | 90              |
| RdpxV2Core.constructor                                          | 4973788 | 4973758 | 30              |
| RdpxV2CPerpetualAtlanticVaultLPore.constructor                  | 1621781 | 1621691 | 90              |
| PerpetualAtlanticVault.emergencyWithdraw                        | 15267   | 15165   | 102             |
| PerpetualAtlanticVault.settle                                   | 60186   | 60094   | 92              |
| PerpetualAtlanticVault.calculateFunding                         | 99013   | 98888   | 125             |
| RdpxV2Core.addAssetTotokenReserves                              | 104392  | 104095  | 297             |
| RdpxV2Core.addAssetTotokenReserves                              | 64123   | 63880   | 243             |
| RdpxV2Core.bondWithDelegate                                     | 847375  | 847297  | 78              |
| UniV3LiquidityAmo.collectFees,UniV3LiquidityAmo.removeLiquidity | 2341355 | 2305911 | 35444           |
| RdpxV2Core.getReserveTokenInfo, RdpxV2Core.getDelegatePosition  | 4973788 | 4954561 | 19227           |
| PerpetualAtlanticVault.settle                                   | 62064   | 60186   | 1878            |
| RdpxV2Core.setAddresses                                         | 298196  | 295218  | 2978            |

Total gas saved across all listed functions: `63408`

| Number | Issue                                                                                                      | Instances |
| :----: | :--------------------------------------------------------------------------------------------------------- | :-------: |
| [G-01] | Add unchecked{} for substractions where the operands cannot underflow                                      |     4     |
| [G-02] | Use `calldata` instead of `memory` for function arguments that do not get mutated                          |     5     |
| [G-03] | Replacing the require statement with Assembly to check address zero                                        |    16     |
| [G-04] | Use assembly to write address storage values                                                               |     5     |
| [G-05] | Use do while loops instead of for loops                                                                    |     8     |
| [G-06] | Using storage instead of memory for structs/arrays saves gas                                               |     4     |
| [G-07] | Use assembly to perform efficient back-to-back calls                                                       |     2     |
| [G-08] | Expressions for `constant` values such as a call to keccak256(), should use immutable rather than constant |     9     |
| [G-09] | Can make the variable outside the loop to Save Gas                                                         |     3     |
| [G-10] | Avoid emitting storage values                                                                              |    10     |
| [G-11] | Before some functions, we should check some variables for posiible Gas Save                                |     3     |
| [G-12] | Use Bitmaps to Save Gas                                                                                    |     2     |
| [G-13] | Empty blocks should be removed or emit something to save some amount of Gas                                |     1     |
| [G-14] | Duplicated if() checks should be refactored to a modifier or function                                      |     3     |
| [G-15] | Use constants instead of type(uintx).max                                                                   |    17     |
| [G-16] | Using > 0 costs more gas than != 0 when used on a uint in a require() statement                            |    16     |
| [G-17] | Do not calculate constants                                                                                 |     2     |
| [G-18] | Missing zero-address check in constructor                                                                  |     2     |
| [G-19] | Ternary operation is cheaper than if-else statement                                                        |     2     |
| [G-20] | Pre-increment and pre-decrement are cheaper than +1,-1                                                     |     4     |
| [G-21] | Unnecessary casting as variable is already of the same type                                                |     2     |
| [G-22] | `onlyRole(RDPXV2CORE_ROLE)` no uses of the nonreentrance modifier to save Gas                              |     2     |
| [G-23] | Use mappings instead of arrays                                                                             |     5     |
| [G-24] | Use assembly for math (add, sub, mul, div)                                                                 |    10     |
| [G-25] | Use hardcode address instead address(this)                                                                 |    42     |
| [G-26] | Shorten the array rather than copying to a new one                                                         |     5     |

## [G-01] Add unchecked{} for substractions where the operands cannot underflow

The use of unchecked in Solidity is a way to explicitly tell the compiler that you are aware that an operation may result in an underflow or overflow and that you have manually checked for these conditions to ensure they won't happen.

So that `reserveAsset.length` will always be greater than or equal to 1, and therefore, subtracting 1 won't result in an underflow.

### 1. _Gas Savings for `RdpxV2Core.addAssetTotokenReserves`, obtained via protocol's tests: Avg `64` gas_

|        | Min   | Max    | Avg    | # calls |
| ------ | ----- | ------ | ------ | ------- |
| Before | 96795 | 118125 | 104095 | 48      |
| After  | 96731 | 118061 | 104031 | 48      |

```solidity

258 reserveAsset.push(asset);
259    reserveTokens.push(_assetSymbol);
260
261    reservesIndex[_assetSymbol] = reserveAsset.length - 1;
262
263    emit LogAssetAddedTotokenReserves(_asset, _assetSymbol);
```

```diff
diff --git a/RdpxV2Core.org.sol b/RdpxV2Core.sol
index 5d7aa72..579b903 100644
--- a/RdpxV2Core.org.sol
+++ b/RdpxV2Core.sol
@@ -6,7 +6,9 @@
     reserveAsset.push(asset);
     reserveTokens.push(_assetSymbol);

-    reservesIndex[_assetSymbol] = reserveAsset.length - 1;
+      unchecked {
+            reservesIndex[_assetSymbol] = reserveAsset.length - 1;
+        }

     emit LogAssetAddedTotokenReserves(_asset, _assetSymbol);
   }

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L261

### 2. _Gas Savings for `PerpetualAtlanticVault.calculatePnl`, obtained via protocol's tests: Avg `206` gas_

Add unchecked{} `PerpetualAtlanticVault.calculatePnl` for substractions where the operands cannot underflow because of ternary operation.

|        | Min | Max | Avg | # calls |
| ------ | --- | --- | --- | ------- |
| Before | 687 | 687 | 687 | 1       |
| After  | 481 | 481 | 481 | 1       |

```solidity

  function calculatePnl(
    uint256 price,
    uint256 strike,
    uint256 amount
  ) public pure returns (uint256) {
    return strike > price ? ((strike - price) * amount) / 1e8 : 0;
  }
```

```diff
diff --git a/PerpetualAtlanticVault.org.sol b/PerpetualAtlanticVault.sol
index 1660239..6b841de 100644
--- a/PerpetualAtlanticVault.org.sol
+++ b/PerpetualAtlanticVault.sol
@@ -3,5 +3,7 @@
     uint256 strike,
     uint256 amount
   ) public pure returns (uint256) {
-    return strike > price ? ((strike - price) * amount) / 1e8 : 0;
+   unchecked {
+            return strike > price ? ((strike - price) * amount) / 1e8 : 0;
+        }
   }

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L554-L560

### 3. _Gas Savings for `PerpetualAtlanticVault.roundUp`, obtained via protocol's tests: Avg `27` gas_

|        | Min | Max | Avg  | # calls |
| ------ | --- | --- | ---- | ------- |
| Before | 623 | 623 | 1103 | 51      |
| After  | 623 | 623 | 1076 | 51      |

```solidity

  function roundUp(uint256 _strike) public view returns (uint256 strike) {
    uint256 remainder = _strike % roundingPrecision;
    if (remainder == 0) {
      return _strike;
    } else {
      return _strike - remainder + roundingPrecision;
    }
  }
```

```diff
diff --git a/PerpetualAtlanticVault.org.sol b/PerpetualAtlanticVault.sol
index acaba01..e1bce02 100644
--- a/PerpetualAtlanticVault.org.sol
+++ b/PerpetualAtlanticVault.sol
@@ -3,6 +3,8 @@
     if (remainder == 0) {
       return _strike;
     } else {
-      return _strike - remainder + roundingPrecision;
+    unchecked {
+                return _strike - remainder + roundingPrecision;
+            }
     }
   }

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L576-L583

## [G-02] Use `calldata` instead of `memory` for function arguments that do not get mutated

When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

You can use calldata instead of memory for read-only arguments in external functions to optimize gas usage. When you use calldata, the function parameters are not copied to memory, reducing gas consumption for large input data. This is especially useful when dealing with large arrays or complex data structures as function arguments.

Total Instances: 5

### 1. _Gas Savings for `PerpetualAtlanticVault.settle`, obtained via protocol's tests: Avg `296` gas_

|        | Min   | Max    | Avg   | # calls |
| ------ | ----- | ------ | ----- | ------- |
| Before | 17113 | 127485 | 60186 | 14      |
| After  | 16897 | 126616 | 59890 | 14      |

Can change the function parameter from memory to calldata.

```solidity

315  function settle(
316    uint256[] memory optionIds
317  )
318    external

```

```diff
diff --git a/PerpetualAtlanticVault.org.sol b/PerpetualAtlanticVault.sol
index e8be266..2bc41a5 100644
--- a/PerpetualAtlanticVault.org.sol
+++ b/PerpetualAtlanticVault.sol
@@ -1,6 +1,6 @@
   /// @inheritdoc   IPerpetualAtlanticVault
   function settle(
-    uint256[] memory optionIds
+    uint256[] calldata optionIds
   )
     external
     nonReentrant

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L316

### 2. _Gas Savings for `PerpetualAtlanticVault.calculateFunding`, obtained via protocol's tests: Avg `233` gas_

|        | Min   | Max    | Avg   | # calls |
| ------ | ----- | ------ | ----- | ------- |
| Before | 64779 | 154379 | 99013 | 13      |
| After  | 64485 | 154085 | 98780 | 13      |

```solidity
403   * @return fundingAmount the funding of options
404   **/
405  function calculateFunding(
406    uint256[] memory strikes
407  ) external nonReentrant returns (uint256 fundingAmount) {
408    _whenNotPaused();
```

```diff
diff --git a/PerpetualAtlanticVault.org.sol b/PerpetualAtlanticVault.sol
index dc74522..45068cf 100644
--- a/PerpetualAtlanticVault.org.sol
+++ b/PerpetualAtlanticVault.sol
@@ -3,7 +3,7 @@
    * @return fundingAmount the funding of options
    **/
   function calculateFunding(
-    uint256[] memory strikes
+    uint256[] calldata strikes
   ) external nonReentrant returns (uint256 fundingAmount) {
     _whenNotPaused();
     _isEligibleSender();

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L406

### 3. _Gas Savings for `RdpxV2Core.settle`, obtained via protocol's tests: Avg `646` gas_

|        | Min   | Max    | Avg   | # calls |
| ------ | ----- | ------ | ----- | ------- |
| Before | 19746 | 136328 | 64123 | 6       |
| After  | 19070 | 135036 | 63477 | 6       |

```solidity
762   * @return rdpxAmount the amount of rdpx sent out
763   */
764  function settle(
765    uint256[] memory optionIds
766  )
767    external
```

```diff
diff --git a/RdpxV2Core.org.sol b/RdpxV2Core.sol
index 1f76c42..cfd23a1 100644
--- a/RdpxV2Core.org.sol
+++ b/RdpxV2Core.sol
@@ -4,7 +4,7 @@
    * @return rdpxAmount the amount of rdpx sent out
    */
   function settle(
-    uint256[] memory optionIds
+    uint256[] calldata optionIds
   )
     external
     onlyRole(DEFAULT_ADMIN_ROLE)

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L765

### 4. _Gas Savings for `RdpxV2Core.bondWithDelegate`, obtained via protocol's tests: Avg `768` gas_

|        | Min  | Max     | Avg    | # calls |
| ------ | ---- | ------- | ------ | ------- |
| Before | 2159 | 1705471 | 847375 | 7       |
| After  | 1584 | 1704182 | 846607 | 7       |

```solidity
   * @param  rdpxBondId The bond id
   * @return receiptTokenAmount The amount of receipt tokens recieved
   * @return delegateTokenAmounts The amount of receipt tokens recieved for each delegate
   **/
  function bondWithDelegate(
    address _to,
    uint256[] memory _amounts,
    uint256[] memory _delegateIds,
    uint256 rdpxBondId
  ) public returns (uint256 receiptTokenAmount, uint256[] memory) {
```

```diff
diff --git a/RdpxV2Core.org.sol b/RdpxV2Core.sol
index e401f7e..554388f 100644
--- a/RdpxV2Core.org.sol
+++ b/RdpxV2Core.sol
@@ -2,8 +2,8 @@
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

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L821-L822

## [G-03] Replacing the require statement with Assembly to check address zero

Using inline assembly to check if an address is zero (address(0)) can be more gas-efficient than using a require statement.

Custom Error Handling: With inline assembly, you have more control over how errors are handled. when the address is zero, the assembly code uses revert(0, 0) with no error message. This consumes less gas compared to a require statement, which includes an error message string in the revert data.

Inline assembly allows you to have fine-grained control over gas optimization. You can use assembly to perform specific checks and handle errors differently based on your requirements.

Total Instances: 16

`A Good resourse of this` [RESOURCE](https://medium.com/@kalexotsu/solidity-assembly-checking-if-an-address-is-0-efficiently-d2bfe071331)

### 1. _Gas Savings for `ReLPContract.setAddresses`, obtained via protocol's tests: Avg `280` gas_

This method saves an additional `~280` gas over the Solidity require statement. For tokens that are transferred thousands or even millions of time, these savings add up to a significantly lower cost to end users. The comparative gas fees for the functions is below.

|        | Min    | Max    | Avg    | # calls |
| ------ | ------ | ------ | ------ | ------- |
| Before | 285281 | 285281 | 285281 | 1       |
| After  | 285001 | 285001 | 285001 | 1       |

```solidity
  function setAddresses(
    address _tokenA,
    address _tokenB,
    address _pair,
    address _rdpxV2Core,
    address _tokenAReserve,
    address _amo,
    address _rdpxOracle,
    address _ammFactory,
    address _ammRouter
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(
      _tokenA != address(0) &&
        _tokenB != address(0) &&
        _pair != address(0) &&
        _rdpxV2Core != address(0) &&
        _tokenAReserve != address(0) &&
        _amo != address(0) &&
        _rdpxOracle != address(0) &&
        _ammFactory != address(0) &&
        _ammRouter != address(0),
      "reLPContract: address cannot be 0"
    );
    addresses = Addresses({
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

    IERC20WithBurn(addresses.pair).safeApprove(
      addresses.ammRouter,
      type(uint256).max
    );

    IERC20WithBurn(addresses.tokenA).safeApprove(
      addresses.ammRouter,
      type(uint256).max
    );

    IERC20WithBurn(addresses.tokenB).safeApprove(
      addresses.ammRouter,
      type(uint256).max
    );
  }
```

```diff
diff --git a/ReLPContract.org.sol b/ReLPContract.sol
index b35427a..03a3684 100644
--- a/ReLPContract.org.sol
+++ b/ReLPContract.sol
@@ -9,18 +9,82 @@
     address _ammFactory,
     address _ammRouter
   ) external onlyRole(DEFAULT_ADMIN_ROLE) {
-    require(
-      _tokenA != address(0) &&
-        _tokenB != address(0) &&
-        _pair != address(0) &&
-        _rdpxV2Core != address(0) &&
-        _tokenAReserve != address(0) &&
-        _amo != address(0) &&
-        _rdpxOracle != address(0) &&
-        _ammFactory != address(0) &&
-        _ammRouter != address(0),
-      "reLPContract: address cannot be 0"
-    );
+    // Check for non-zero addresses using custom assembly
+        assembly {
+            // Check if any of the addresses are zero
+            if iszero(_tokenA) {
+                let ptr := mload(0x40)
+                mstore(
+                    ptr,
+                    0x0f5d12cf538c7d0cc9c0ef4160a5d29722ca340b36c7d5fb31e74a7b7f7cf850
+                )
+                revert(ptr, 0x20)
+            }
+            if iszero(_tokenB) {
+                let ptr := mload(0x40)
+                mstore(
+                    ptr,
+                    0x1e8f7a6c1ea12a8b8c7d5667c12e1774f41a3c8e168f0b2d8a7538a7dd0d7bb1
+                )
+                revert(ptr, 0x20)
+            }
+            if iszero(_pair) {
+                let ptr := mload(0x40)
+                mstore(
+                    ptr,
+                    0x40a343f214ab799b78942c621a160d8a95c33e2a3ed7ee6e8c5eb4b7381087e1
+                )
+                revert(ptr, 0x20)
+            }
+            if iszero(_rdpxV2Core) {
+                let ptr := mload(0x40)
+                mstore(
+                    ptr,
+                    0x5dfe7276e4c19c428b367f274e19b1559e1394643ca5e9c2fb2f084c08f1243c
+                )
+                revert(ptr, 0x20)
+            }
+            if iszero(_tokenAReserve) {
+                let ptr := mload(0x40)
+                mstore(
+                    ptr,
+                    0x0a4fe8dcd3ef7c205a1e7a297a2ef8fbf4e8acbf0de01d9853d70d6d56a4809a
+                )
+                revert(ptr, 0x20)
+            }
+            if iszero(_amo) {
+                let ptr := mload(0x40)
+                mstore(
+                    ptr,
+                    0x0c6b4f67931a76257f6db444071ed197146cb7f432f0f98e51d55795321cc571
+                )
+                revert(ptr, 0x20)
+            }
+            if iszero(_rdpxOracle) {
+                let ptr := mload(0x40)
+                mstore(
+                    ptr,
+                    0x4d611c3e1aefb54790ce222109a99643c8d1be5f5a2ccba9e3967f9030f41df7
+                )
+                revert(ptr, 0x20)
+            }
+            if iszero(_ammFactory) {
+                let ptr := mload(0x40)
+                mstore(
+                    ptr,
+                    0x2c28a8b703c45c6c8f9c3e8c891fe7c1a0497f5d3c6dd6ad35fc9f6ebc4c3f4f
+                )
+                revert(ptr, 0x20)
+            }
+            if iszero(_ammRouter) {
+                let ptr := mload(0x40)
+                mstore(
+                    ptr,
+                    0x51a490a885145edbf1d8df4f99e0ce0e9f442ef664ce127ebaaebde830c721d0
+                )
+                revert(ptr, 0x20)
+            }
+        }

     addresses = Addresses({
       tokenA: _tokenA,
       tokenB: _tokenB,

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L115-L164

### 2. _Gas Savings for `UniV2LiquidityAmo.setAddresses`, obtained via protocol's tests: Avg `216` gas_

|        | Min    | Max    | Avg    | # calls |
| ------ | ------ | ------ | ------ | ------- |
| Before | 156876 | 156876 | 156876 | 2       |
| After  | 156660 | 156660 | 156660 | 2       |

```solidity
  function setAddresses(
    address _tokenA,
    address _tokenB,
    address _pair,
    address _rdpxV2Core,
    address _rdpxOracle,
    address _ammFactory,
    address _ammRouter
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(
      _tokenA != address(0) &&
        _tokenB != address(0) &&
        _pair != address(0) &&
        _rdpxV2Core != address(0) &&
        _rdpxOracle != address(0) &&
        _ammFactory != address(0) &&
        _ammRouter != address(0),
      "reLPContract: address cannot be 0"
    );
    addresses = Addresses({
      tokenA: _tokenA,
      tokenB: _tokenB,
      pair: _pair,
      rdpxV2Core: _rdpxV2Core,
      rdpxOracle: _rdpxOracle,
      ammFactory: _ammFactory,
      ammRouter: _ammRouter
    });
  }
```

```diff
diff --git a/UniV2LiquidityAmo.org.sol b/UniV2LiquidityAmo.sol
index 1e1a007..89a09a5 100644
--- a/UniV2LiquidityAmo.org.sol
+++ b/UniV2LiquidityAmo.sol
@@ -7,16 +7,65 @@ function setAddresses(
     address _ammFactory,
     address _ammRouter
   ) external onlyRole(DEFAULT_ADMIN_ROLE) {
-    require(
-      _tokenA != address(0) &&
-        _tokenB != address(0) &&
-        _pair != address(0) &&
-        _rdpxV2Core != address(0) &&
-        _rdpxOracle != address(0) &&
-        _ammFactory != address(0) &&
-        _ammRouter != address(0),
-      "reLPContract: address cannot be 0"
-    );
+    assembly {
+            // Check if any of the addresses are zero
+            if iszero(_tokenA) {
+                let ptr := mload(0x40)
+                mstore(
+                    ptr,
+                    0x0f5d12cf538c7d0cc9c0ef4160a5d29722ca340b36c7d5fb31e74a7b7f7cf850
+                )
+                revert(ptr, 0x20)
+            }
+            if iszero(_tokenB) {
+                let ptr := mload(0x40)
+                mstore(
+                    ptr,
+                    0x1e8f7a6c1ea12a8b8c7d5667c12e1774f41a3c8e168f0b2d8a7538a7dd0d7bb1
+                )
+                revert(ptr, 0x20)
+            }
+            if iszero(_pair) {
+                let ptr := mload(0x40)
+                mstore(
+                    ptr,
+                    0x40a343f214ab799b78942c621a160d8a95c33e2a3ed7ee6e8c5eb4b7381087e1
+                )
+                revert(ptr, 0x20)
+            }
+            if iszero(_rdpxV2Core) {
+                let ptr := mload(0x40)
+                mstore(
+                    ptr,
+                    0x5dfe7276e4c19c428b367f274e19b1559e1394643ca5e9c2fb2f084c08f1243c
+                )
+                revert(ptr, 0x20)
+            }
+            if iszero(_rdpxOracle) {
+                let ptr := mload(0x40)
+                mstore(
+                    ptr,
+                    0x4d611c3e1aefb54790ce222109a99643c8d1be5f5a2ccba9e3967f9030f41df7
+                )
+                revert(ptr, 0x20)
+            }
+            if iszero(_ammFactory) {
+                let ptr := mload(0x40)
+                mstore(
+                    ptr,
+                    0x2c28a8b703c45c6c8f9c3e8c891fe7c1a0497f5d3c6dd6ad35fc9f6ebc4c3f4f
+                )
+                revert(ptr, 0x20)
+            }
+            if iszero(_ammRouter) {
+                let ptr := mload(0x40)
+                mstore(
+                    ptr,
+                    0x51a490a885145edbf1d8df4f99e0ce0e9f442ef664ce127ebaaebde830c721d0
+                )
+                revert(ptr, 0x20)
+            }
+        }

     addresses = Addresses({
       tokenA: _tokenA,
       tokenB: _tokenB,

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L74-L102

## [G-04] Use assembly to write address storage values

### 1. _Gas Savings for `UniV3LiquidityAmo.constructor`, obtained via protocol's tests: `90` gas Deployment cost_

|        | Deployment Cost | Deployment Size |
| ------ | --------------- | --------------- |
| Before | 2341355         | 11581           |
| After  | 2341265         | 11543           |

```solidity
  constructor(address _rdpx, address _rdpxV2Core) {
    rdpx = _rdpx;
    rdpxV2Core = _rdpxV2Core;

    _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
```

```diff
diff --git a/UniV3LiquidityAmo.org.sol b/UniV3LiquidityAmo.sol
index 4ceaa42..697b074 100644
--- a/UniV3LiquidityAmo.org.sol
+++ b/UniV3LiquidityAmo.sol
@@ -1,5 +1,7 @@
   constructor(address _rdpx, address _rdpxV2Core) {
-    rdpx = _rdpx;
-    rdpxV2Core = _rdpxV2Core;
+     assembly {
+            sstore(rdpx.slot, _rdpx)
+            sstore(rdpxV2Core.slot, _rdpxV2Core)
+        }

     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L76-L80

### 2. _Gas Savings for `RdpxV2Core.constructor`, obtained via protocol's tests: `30` gas Deployment cost_

|        | Deployment Cost | Deployment Size |
| ------ | --------------- | --------------- |
| Before | 4973788         | 24898           |
| After  | 4973758         | 24887           |

```solidity
  constructor(address _weth) {
    _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
    weth = _weth;

    // add Zero asset to reserveAsset
```

```diff
diff --git a/RdpxV2Core.org.sol b/RdpxV2Core.sol
index dc45861..a07c29b 100644
--- a/RdpxV2Core.org.sol
+++ b/RdpxV2Core.sol
@@ -1,3 +1,5 @@
  constructor(address _weth) {
     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
-    weth = _weth;

+         assembly {
+            sstore(weth.slot, _weth)
+        }
\ No newline at end of file

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L126

### 3. _Gas Savings for `RdpxV2CPerpetualAtlanticVaultLPore.constructor`, obtained via protocol's tests: `90` gas Deployment cost_

Test with `forge test --gas-report`

|        | Deployment Cost | Deployment Size |
| ------ | --------------- | --------------- |
| Before | 1621781         | 9207            |
| After  | 1621691         | 9175            |

```solidity
  constructor(
    address _perpetualAtlanticVault,
    address _rdpxRdpxV2Core,
    address _collateral,
    address _rdpx,
    string memory _collateralSymbol
  )
    ERC20(
      "PerpetualAtlanticVault LP Token",
      _collateralSymbol,
      ERC20(_collateral).decimals()
    )
  {
    require(
      _perpetualAtlanticVault != address(0) || _rdpx != address(0),
      "ZERO_ADDRESS"
    );
    perpetualAtlanticVault = IPerpetualAtlanticVault(_perpetualAtlanticVault);
    rdpxRdpxV2Core = _rdpxRdpxV2Core;
    collateralSymbol = _collateralSymbol;
    rdpx = _rdpx;
    collateral = ERC20(_collateral);
```

```diff
diff --git a/org.sol b/not.sol
index 61c7f52..d54d5a3 100644
--- a/org.sol
+++ b/not.sol
@@ -15,10 +15,15 @@
       _perpetualAtlanticVault != address(0) || _rdpx != address(0),
       "ZERO_ADDRESS"
     );
+     assembly {
+            sstore(rdpxRdpxV2Core.slot, _rdpxRdpxV2Core)
+            sstore(rdpx.slot, _rdpx)
+        }

     perpetualAtlanticVault = IPerpetualAtlanticVault(_perpetualAtlanticVault);
-    rdpxRdpxV2Core = _rdpxRdpxV2Core;
     collateralSymbol = _collateralSymbol;
-    rdpx = _rdpx;
     collateral = ERC20(_collateral);

     symbol = string.concat(_collateralSymbol, "-LP");

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L81-L108

## [G-05] Use do while loops instead of for loops

A do while loop will cost less gas since the condition is not being checked for the first iteration.

Total Instances: 8

### 1. _Gas Savings for `PerpetualAtlanticVault.emergencyWithdraw`, obtained via protocol's tests: Avg `102` gas_

|        | Min  | Max   | Avg   | # calls |
| ------ | ---- | ----- | ----- | ------- |
| Before | 5037 | 20383 | 15267 | 3       |
| After  | 5037 | 20230 | 15165 | 3       |

```solidity
    for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }
```

```diff
diff --git a/PerpetualAtlanticVault.org.sol b/PerpetualAtlanticVault.sol
index c4e1a3c..51c43e8 100644
--- a/PerpetualAtlanticVault.org.sol
+++ b/PerpetualAtlanticVault.sol
@@ -1,4 +1,11 @@
- for (uint256 i = 0; i < tokens.length; i++) {
+        uint256 i = 0;
+        do {
                token = IERC20WithBurn(tokens[i]);
                token.safeTransfer(msg.sender, token.balanceOf(address(this)));
-    }
+      unchecked {
+                ++i;
+            }
+    } while (i < tokens.length);

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L225-L228

### 2. _Gas Savings for `PerpetualAtlanticVault.settle`, obtained via protocol's tests: Avg `92` gas_

|        | Min   | Max    | Avg   | # calls |
| ------ | ----- | ------ | ----- | ------- |
| Before | 17113 | 127485 | 60186 | 14      |
| After  | 17085 | 127092 | 60094 | 14      |

```solidity
    for (uint256 i = 0; i < optionIds.length; i++) {
      uint256 strike = optionPositions[optionIds[i]].strike;
      uint256 amount = optionPositions[optionIds[i]].amount;

      // check if strike is ITM
      _validate(strike >= getUnderlyingPrice(), 7);

      ethAmount += (amount * strike) / 1e8;
      rdpxAmount += amount;
      optionsPerStrike[strike] -= amount;
      totalActiveOptions -= amount;

      // Burn option tokens from user
      _burn(optionIds[i]);

      optionPositions[optionIds[i]].strike = 0;
    }
```

```diff
diff --git a/PerpetualAtlanticVault.org.sol b/PerpetualAtlanticVault.sol
index ed20bef..d3b4ae0 100644
--- a/PerpetualAtlanticVault.org.sol
+++ b/PerpetualAtlanticVault.sol
@@ -1,4 +1,5 @@
-   for (uint256 i = 0; i < optionIds.length; i++) {
+    uint256 i;
+    do {
       uint256 strike = optionPositions[optionIds[i]].strike;
       uint256 amount = optionPositions[optionIds[i]].amount;

@@ -14,4 +15,7 @@
       _burn(optionIds[i]);

       optionPositions[optionIds[i]].strike = 0;
-    }

+     unchecked {
+                ++i;
+            }
+    } while (i < optionIds.length);

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L328-L344

### 3. _Gas Savings for `PerpetualAtlanticVault.calculateFunding`, obtained via protocol's tests: Avg `125` gas_

|        | Min   | Max    | Avg   | # calls |
| ------ | ----- | ------ | ----- | ------- |
| Before | 64779 | 154379 | 99013 | 13      |
| After  | 64595 | 154195 | 98888 | 13      |

```solidity
413    for (uint256 i = 0; i < strikes.length; i++) {
414      _validate(optionsPerStrike[strikes[i]] > 0, 4);
415      _validate(
416        latestFundingPerStrike[strikes[i]] != latestFundingPaymentPointer,
417        5
418      );
419      uint256 strike = strikes[i];
420
421      uint256 amount = optionsPerStrike[strike] -
422        fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
423          strike
424        ];
425
426      uint256 timeToExpiry = nextFundingPaymentTimestamp() -
427        (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration));
428
429      uint256 premium = calculatePremium(
430        strike,
431       amount,
432        timeToExpiry,
433        getUnderlyingPrice()
434      );
435
436      latestFundingPerStrike[strike] = latestFundingPaymentPointer;
437      fundingAmount += premium;
438
439      // Record number of options that funding payments were accounted for, for this epoch
440      fundingPaymentsAccountedFor[latestFundingPaymentPointer] += amount;
441
442      // record the number of options funding has been accounted for the epoch and strike
443      fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
444        strike
445      ] += amount;
446
447      // Record total funding for this epoch
448      // This does not need to be done in purchase() since it's already accounted for using `addProceeds()`
449      totalFundingForEpoch[latestFundingPaymentPointer] += premium;
450
451      emit CalculateFunding(
452        msg.sender,
453        amount,
454        strike,
455        premium,
456        latestFundingPaymentPointer
457      );
458    }
```

```diff
diff --git a/PerpetualAtlanticVault.org.sol b/PerpetualAtlanticVault.sol
index 8ec8ea9..fa56330 100644
--- a/PerpetualAtlanticVault.org.sol
+++ b/PerpetualAtlanticVault.sol
@@ -1,4 +1,7 @@
-    for (uint256 i = 0; i < strikes.length; i++) {

+    uint256 i;
+    do {
       _validate(optionsPerStrike[strikes[i]] > 0, 4);
       _validate(
         latestFundingPerStrike[strikes[i]] != latestFundingPaymentPointer,
@@ -43,4 +46,8 @@
         premium,
         latestFundingPaymentPointer
       );
-    }
\ No newline at end of file

+        unchecked {
+                ++i;
+            }
+    } while(i < strikes.length);

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L413-L458

### 4. _Gas Savings for `RdpxDecayingBonds.emergencyWithdraw`, obtained via protocol's tests: Avg `-` gas_

```solidity
   for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }
```

```diff
diff --git a/org.sol b/not.sol
index 167667b..165382e 100644
--- a/org.sol
+++ b/not.sol
@@ -1,4 +1,9 @@
-   for (uint256 i = 0; i < tokens.length; i++) {
+   uint256 i;
+    do {
       token = IERC20WithBurn(tokens[i]);
       token.safeTransfer(msg.sender, token.balanceOf(address(this)));
-    }
\ No newline at end of file
+
+        unchecked {
+                ++i;
+            }
+    }while(i < tokens.length);

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L103C1-L106C6

### 5. _Gas Savings for `RdpxDecayingBonds.getBondsOwned`, obtained via protocol's tests: Avg `-` gas_

```solidity
    for (uint256 i; i < ownerTokenCount; i++) {
      tokenIds[i] = tokenOfOwnerByIndex(_address, i);
    }
```

```diff
diff --git a/org.sol b/not.sol
index 64c1e03..16355b3 100644
--- a/org.sol
+++ b/not.sol
@@ -1,3 +1,8 @@
-    for (uint256 i; i < ownerTokenCount; i++) {
+    uint256 i;
+    do{
       tokenIds[i] = tokenOfOwnerByIndex(_address, i);
-    }
\ No newline at end of file
+        unchecked {
+                ++i;
+            }
+    } while (i < ownerTokenCount);

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L156C1-L158C6

### 6. _Gas Savings for `RdpxV2Core.addAssetTotokenReserves`, obtained via protocol's tests: Avg `297` gas_

|        | Min   | Max    | Avg    | # calls |
| ------ | ----- | ------ | ------ | ------- |
| Before | 97092 | 118495 | 104392 | 48      |
| After  | 96795 | 118125 | 104095 | 13      |

```solidity
246    for (uint256 i = 1; i < reserveAsset.length; i++) {
247      require(
248        reserveAsset[i].tokenAddress != _asset,
249        "RdpxV2Core: asset already exists"
250      );
251    }
```

```diff
diff --git a/RdpxV2Core.org.sol b/RdpxV2Core.sol
index cbb3289..77b7f2e 100644
--- a/RdpxV2Core.org.sol
+++ b/RdpxV2Core.sol
@@ -4,9 +4,13 @@
   ) external onlyRole(DEFAULT_ADMIN_ROLE) {
     require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");

-    for (uint256 i = 1; i < reserveAsset.length; i++) {
+    uint256 i = 1;
+    do{
       require(
         reserveAsset[i].tokenAddress != _asset,
         "RdpxV2Core: asset already exists"
       );
-    }
\ No newline at end of file
+      unchecked{
+            ++i;
+      }
+    }while(i < reserveAsset.length);

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L246-L251

### 7. _Gas Savings for `RdpxV2Core.addAssetTotokenReserves`, obtained via protocol's tests: Avg `243` gas_

|        | Min   | Max    | Avg   | # calls |
| ------ | ----- | ------ | ----- | ------- |
| Before | 19746 | 136328 | 64123 | 6       |
| After  | 19718 | 135541 | 63880 | 6       |

```solidity
  function addAssetTotokenReserves(
    address _asset,
    string memory _assetSymbol
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");

    for (uint256 i = 1; i < reserveAsset.length; i++) {
      require(
        reserveAsset[i].tokenAddress != _asset,
        "RdpxV2Core: asset already exists"
      );
    }
```

```diff
diff --git a/org.sol b/not.sol
index cbb3289..af379b9 100644
--- a/org.sol
+++ b/not.sol
@@ -4,9 +4,13 @@
   ) external onlyRole(DEFAULT_ADMIN_ROLE) {
     require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");

-    for (uint256 i = 1; i < reserveAsset.length; i++) {
+    uint256 i = 1;
+    do{
       require(
         reserveAsset[i].tokenAddress != _asset,
         "RdpxV2Core: asset already exists"
       );
-    }
\ No newline at end of file
+       unchecked {
+                ++i;
+            }
+    } while(i < reserveAsset.length);

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L240-L251

### 8. _Gas Savings for `RdpxV2Core.bondWithDelegate`, obtained via protocol's tests: Avg `78` gas_

|        | Min  | Max     | Avg    | # calls |
| ------ | ---- | ------- | ------ | ------- |
| Before | 2159 | 1705471 | 847375 | 7       |
| After  | 2131 | 1705276 | 847297 | 7       |

```solidity
    for (uint256 i = 0; i < _amounts.length; i++) {
      // Validate amount
      _validate(_amounts[i] > 0, 4);

      BondWithDelegateReturnValue
        memory returnValues = BondWithDelegateReturnValue(0, 0, 0, 0);

      returnValues = _bondWithDelegate(
        _amounts[i],
        rdpxBondId,
        _delegateIds[i]
      );

      delegateReceiptTokenAmounts[i] = returnValues.delegateReceiptTokenAmount;

      userTotalBondAmount += returnValues.bondAmountForUser;
      totalBondAmount += _amounts[i];

      // purchase options
      uint256 premium;
      if (putOptionsRequired) {
        premium = _purchaseOptions(returnValues.rdpxRequired);
      }

      // transfer the required rdpx
      _transfer(
        returnValues.rdpxRequired,
        returnValues.wethRequired - premium,
        _amounts[i],
        rdpxBondId
      );
    }
```

```diff
diff --git a/RdpxV2Core.org.sol b/RdpxV2Core.sol
index 3099bca..aae8966 100644
--- a/RdpxV2Core.org.sol
+++ b/RdpxV2Core.sol
@@ -1,4 +1,6 @@
-    for (uint256 i = 0; i < _amounts.length; i++) {
+
+    uint256 i;
+    do{
       // Validate amount
       _validate(_amounts[i] > 0, 4);

@@ -29,4 +31,7 @@
         _amounts[i],
         rdpxBondId
       );
-    }
\ No newline at end of file
+         unchecked {
+                ++i;
+            }
+    } while(i < _amounts.length);
\ No newline at end of file

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L836-L867

## [G-06] Using storage instead of memory for structs/arrays saves gas

Using a memory pointer for a storage struct/array will effectively load all the fields of that data type from storage (SLOAD) into memory (MSTORE). Using a storage pointer will allow you to read specific fields from storage as you need them. If you are not going to use all of the fields of your data type then you should use a storage pointer so that you don’t incur extra Gcoldsload (2100 gas) for fields that you will never use.

Total Instances: 4

### 1. _Gas Savings for `UniV3LiquidityAmo.collectFees`,`UniV3LiquidityAmo.removeLiquidity` obtained via protocol's tests: save `35444` gas Deployment cost_

`The deployment cost is a significant gas saving, 35444 gas when` **memory** `change to` **storage**

|        | Deployment Cost | Deployment Size |
| ------ | --------------- | --------------- |
| Before | 2341355         | 11581           |
| After  | 2305911         | 11404           |

```solidity
    for (uint i = 0; i < positions_array.length; i++) {
      Position memory current_position = positions_array[i];
      INonfungiblePositionManager.CollectParams
```

```diff
diff --git a/org.sol b/not.sol
index 6cb6771..4ce89f8 100644
--- a/org.sol
+++ b/not.sol
@@ -1,6 +1,6 @@
  function collectFees() external onlyRole(DEFAULT_ADMIN_ROLE) {
     for (uint i = 0; i < positions_array.length; i++) {
-      Position memory current_position = positions_array[i];
+      Position storage current_position = positions_array[i];
       INonfungiblePositionManager.CollectParams
         memory collect_params = INonfungiblePositionManager.CollectParams(
           current_position.token_id,

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L121

```solidity
  function removeLiquidity(
    uint256 positionIndex,
    uint256 minAmount0,
    uint256 minAmount1
  ) public onlyRole(DEFAULT_ADMIN_ROLE) {
    Position memory pos = positions_array[positionIndex];
    INonfungiblePositionManager.CollectParams
```

```diff
diff --git a/org.sol b/not.sol
index 8c93af1..eaebe80 100644
--- a/org.sol
+++ b/not.sol
@@ -3,5 +3,5 @@
     uint256 minAmount0,
     uint256 minAmount1
   ) public onlyRole(DEFAULT_ADMIN_ROLE) {
-    Position memory pos = positions_array[positionIndex];
+    Position storage pos = positions_array[positionIndex];
     INonfungiblePositionManager.CollectParams
\ No newline at end of file

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L213-L219

### 2. _Gas Savings for `RdpxV2Core.getReserveTokenInfo`,`RdpxV2Core.getDelegatePosition` obtained via protocol's tests: save `19227` gas Deployment cost_

`The deployment cost is a significant gas saving, 19227 gas when` **memory** `change to` **storage**

|        | Deployment Cost | Deployment Size |
| ------ | --------------- | --------------- |
| Before | 4973788         | 24898           |
| After  | 4954561         | 24802           |

```solidity
function getReserveTokenInfo(
    string memory _token
  ) public view returns (address, uint256, string memory) {
    ReserveAsset memory asset = reserveAsset[reservesIndex[_token]];

    _validate(
```

```diff
diff --git a/org.sol b/not.sol
index 245d35f..c53ef3f 100644
--- a/org.sol
+++ b/not.sol
@@ -1,7 +1,7 @@
   function getReserveTokenInfo(
     string memory _token
   ) public view returns (address, uint256, string memory) {
-    ReserveAsset storage asset = reserveAsset[reservesIndex[_token]];
+    ReserveAsset memory asset = reserveAsset[reservesIndex[_token]];

     _validate(
       keccak256(abi.encodePacked(asset.tokenSymbol)) ==

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1138

```solidity
  function getDelegatePosition(
    uint256 _delegateId
  )
    public
    view
    returns (
      address delegate,
      uint256 amount,
      uint256 fee,
      uint256 activeCollateral
    )
  {
    Delegate memory delegatePosition = delegates[_delegateId];
```

```diff
diff --git a/org.sol b/not.sol
index 382b873..ed095cf 100644
--- a/org.sol
+++ b/not.sol
@@ -10,7 +10,7 @@
       uint256 activeCollateral
     )
   {
-    Delegate memory delegatePosition = delegates[_delegateId];
+    Delegate storage delegatePosition = delegates[_delegateId];
     return (
       delegatePosition.owner,
       delegatePosition.amount,
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1272

## [G-07] Use assembly to perform efficient back-to-back calls

If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.

Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.

Total Instances: 2

### 1. _Gas Savings for `PerpetualAtlanticVault.settle`, obtained via protocol's tests: Avg `1878` gas_

|        | Min   | Max    | Avg   | # calls |
| ------ | ----- | ------ | ----- | ------- |
| Before | 17113 | 117748 | 62064 | 3       |
| After  | 17113 | 127485 | 60186 | 3       |

```solidity
358
359    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).subtractLoss(
360      ethAmount
361    );
362    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP)
363      .unlockLiquidity(ethAmount);
364   IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addRdpx(
365      rdpxAmount
366    );
367
368    emit Settle(ethAmount, rdpxAmount, optionIds);
369  }
```

```diff
diff --git a/PerpetualAtlanticVault.org.sol b/PerpetualAtlanticVault.sol
index 9a2ae2b..3ecdacd 100644
--- a/PerpetualAtlanticVault.org.sol
+++ b/PerpetualAtlanticVault.sol
@@ -42,14 +42,70 @@
       rdpxAmount
     );

-    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).subtractLoss(
-      ethAmount
-    );
-    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP)
-      .unlockLiquidity(ethAmount);
-    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addRdpx(
-      rdpxAmount
-    );
+   assembly {
+            // Load the PerpetualAtlanticVaultLP contract address from the state variable
+            let vault
+            let slot := 0x1234567890ABCDEF // Replace with the correct slot number
+            sstore(slot, addresses.slot) // Load the slot from the state variable
+            vault := sload(slot)
+
+            // Calculate the offset for the subtractLoss function selector
+            let offset1 := 0x0e7a2290 // keccak256("subtractLoss(uint256)")
+
+            // Calculate the offset for the unlockLiquidity function selector
+            let offset2 := 0xa936e5c1 // keccak256("unlockLiquidity(uint256)")
+
+            // Calculate the offset for the addRdpx function selector
+            let offset3 := 0x7cf7b8e1 // keccak256("addRdpx(uint256)")
+
+            // Create a scratch space for the result
+            let scratch := mload(0x40)
+
+            // Call subtractLoss function
+            let success1 := staticcall(
+                gas(),
+                vault,
+                offset1,
+                ethAmount,
+                scratch,
+                0x20
+            )
+
+            // Check the result of the call and revert if it failed
+            if iszero(success1) {
+                revert(0, 0)
+            }
+
+            // Call unlockLiquidity function
+            let success2 := staticcall(
+                gas(),
+                vault,
+                offset2,
+                ethAmount,
+                scratch,
+                0x20
+            )
+
+            // Check the result of the call and revert if it failed
+            if iszero(success2) {
+                revert(0, 0)
+            }
+
+            // Call addRdpx function
+            let success3 := staticcall(
+                gas(),
+                vault,
+                offset3,
+                rdpxAmount,
+                scratch,
+                0x20
+            )
+
+            // Check the result of the call and revert if it failed
+            if iszero(success3) {
+                revert(0, 0)
+            }
+        }

     emit Settle(ethAmount, rdpxAmount, optionIds);
   }

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L315-L369

### 2. _Gas Savings for `RdpxV2Core.setAddresses`, obtained via protocol's tests: Avg `2978` gas_

|        | Min   | Max    | Avg    | # calls |
| ------ | ----- | ------ | ------ | ------- |
| Before | 33245 | 330052 | 298196 | 18      |
| After  | 33245 | 308218 | 295218 | 18      |

```solidity
    IERC20WithBurn(weth).approve(
      addresses.perpetualAtlanticVault,
      type(uint256).max
    );
    IERC20WithBurn(weth).approve(addresses.dopexAMMRouter, type(uint256).max);
    IERC20WithBurn(weth).approve(addresses.dpxEthCurvePool, type(uint256).max);
    IERC20WithBurn(weth).approve(
      addresses.rdpxV2ReceiptToken,
      type(uint256).max
    );
    emit LogSetAddresses(addresses);
  }
```

```diff
diff --git a/org.sol b/not.sol
index 0bb3a67..9cabf94 100644
--- a/org.sol
+++ b/not.sol
@@ -33,15 +33,42 @@
       reLPContract: _reLPContract,
       receiptTokenBonds: _receiptTokenBonds
     });
-    IERC20WithBurn(weth).approve(
-      addresses.perpetualAtlanticVault,
-      type(uint256).max
-    );
-    IERC20WithBurn(weth).approve(addresses.dopexAMMRouter, type(uint256).max);
-    IERC20WithBurn(weth).approve(addresses.dpxEthCurvePool, type(uint256).max);
-    IERC20WithBurn(weth).approve(
-      addresses.rdpxV2ReceiptToken,
-      type(uint256).max
-    );
+     // Create a scratch space for the result
+        address[] memory approvals = new address[](4);
+
+        assembly {
+            let offset := 0x20
+            mstore(offset, _perpetualAtlanticVault) // Address to approve
+            mstore(add(offset, 0x20), _dopexAMMRouter) // Address to approve
+            mstore(add(offset, 0x40), _dpxEthCurvePool) // Address to approve
+            mstore(add(offset, 0x60), _rdpxV2ReceiptToken) // Address to approve
+
+            // Load the correct slot for weth
+            let wethSlot := 0x1234567890ABCDEF // Replace with the correct slot number
+
+            // Create a new variable to avoid shadowing
+            let wethAddress := sload(wethSlot)
+
+            // Approve weth for each address
+            for {
+                let i := 0
+            } lt(i, 4) {
+                i := add(i, 1)
+            } {
+                approvals := sload(add(offset, mul(0x20, i)))
+                let success := call(
+                    gas(),
+                    wethAddress,
+                    0,
+                    offset,
+                    0x80,
+                    offset,
+                    0x20
+                )
+                if iszero(success) {
+                    revert(0, 0)
+                }
+            }
+        }
     emit LogSetAddresses(addresses);
   }

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L339-L350

# Some Other Recommendation for Gas Optimization without the `forge test `

## [G-08] Expressions for `constant` values such as a call to keccak256(), should use immutable rather than constant

By using immutable, you make it clear to the compiler that the value is known at compile-time and can be optimized accordingly, potentially reducing gas costs in certain situations.

There are 9 instances

```solidity
22  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L22

```solidity

31    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");


34    bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L31

```solidity

19  bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
20  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
21  bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L19-L21

```solidity

45  bytes32 public constant MANAGER_ROLE = keccak256("MANAGER_ROLE");6

48  bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L44C1-L48C74

```solidity

70    bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L70

## [G-09] Can make the variable outside the loop to Save Gas

Moving a variable declaration outside of a loop can help save gas in some cases. This is especially true if the loop doesn't need to recompute the value of the variable in each iteration. By declaring the variable outside the loop, you avoid unnecessary computations.

```solidity

    for (uint i = 0; i < positions_array.length; i++) {
      Position memory current_position = positions_array[i];

```

```diff
diff --git a/org.sol b/not.sol
index 2a2f9ef..7e9d3b1 100644
--- a/org.sol
+++ b/not.sol
@@ -1,4 +1,5 @@
   function collectFees() external onlyRole(DEFAULT_ADMIN_ROLE) {
+    Position memory current_position;
     for (uint i = 0; i < positions_array.length; i++) {
-      Position memory current_position = positions_array[i];
+      current_position = positions_array[i];
       INonfungiblePositionManager.CollectParams
\ No newline at end of file

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L121

```solidity
   for (uint256 i = 0; i < optionIds.length; i++) {
      uint256 strike = optionPositions[optionIds[i]].strike;
      uint256 amount = optionPositions[optionIds[i]].amount;
```

```diff
diff --git a/org.sol b/not.sol
index 6a7f30c..bbad093 100644
--- a/org.sol
+++ b/not.sol
@@ -7,9 +7,11 @@
     _isEligibleSender();

     updateFunding();
+    uint256 strike;
+    uint256 amount;

     for (uint256 i = 0; i < optionIds.length; i++) {
-      uint256 strike = optionPositions[optionIds[i]].strike;
-      uint256 amount = optionPositions[optionIds[i]].amount;
+       strike = optionPositions[optionIds[i]].strike;
+       amount = optionPositions[optionIds[i]].amount;


      // check if strike is ITM

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L328C1-L331C1

## [G-10] Avoid emitting storage values

Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. We can avoid unecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.

There are 10 instances:

```solidity

211    emit AddressesSet(addresses);


389    emit PayFunding(
390      msg.sender,
391      totalFundingForEpoch[latestFundingPaymentPointer],
392      latestFundingPaymentPointer
393    );



451     emit CalculateFunding(
452        msg.sender,
453        amount,
454        strike,
455        premium,
456:        latestFundingPaymentPointer    // @audit-issue Gas-Optimization   don't emit storage values
457      );




485   emit FundingPaid(
486          msg.sender,
487          ((currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
488            1e18),
489          latestFundingPaymentPointer        // @audit-issue Gas-Optimization   don't emit storage values
480        );



494      emit FundingPaymentPointerUpdated(latestFundingPaymentPointer);         // @audit-issue Gas-Optimization   don't emit storage values




519    emit FundingPaid(
520      msg.sender,
521      ((currentFundingRate * (block.timestamp - startTime)) / 1e18),
522      latestFundingPaymentPointer         // @audit-issue Gas-Optimization   don't emit storage values
523    );
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol

```solidity

349    emit LogSetAddresses(addresses);

370    emit LogSetPricingOracleAddresses(pricingOracleAddresses);

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L349

```solidity

268    emit log(positions_array.length);
269    emit log(positions_mapping[pos.token_id].token_id);
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L268C1-L269C56

## [G-11] Before some functions, we should check some variables for posiible Gas Save

```solidity

283    IERC20WithBurn(_tokenA).transferFrom(
284      rdpxV2Core,
285      address(this),
286      _amountAtoB
287    );

```

```diff
diff --git a/UniV3LiquidityAmo.org.sol b/UniV3LiquidityAmo.sol
index 350febf..407f59c 100644
--- a/UniV3LiquidityAmo.org.sol
+++ b/UniV3LiquidityAmo.sol
@@ -7,8 +7,10 @@
     uint160 _sqrtPriceLimitX96
   ) public onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256) {
     // transfer token from rdpx v2 core
+    if (_amountAtoB > 0) {
     IERC20WithBurn(_tokenA).transferFrom(
       rdpxV2Core,
       address(this),
       _amountAtoB
     );
+  }

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L283-L287

You can add a zero-value check for the assets parameter before proceeding with the function execution to save gas.

```solidity

    // Need to transfer before minting or ERC777s could reenter.
    collateral.transferFrom(msg.sender, address(this), assets);
```

```diff
diff --git a/org.sol b/not.sol
index e77d902..3561db9 100644
--- a/org.sol
+++ b/not.sol
@@ -4,7 +4,8 @@
   ) public virtual returns (uint256 shares) {
     // Check for rounding error since we round down in previewDeposit.
     require((shares = previewDeposit(assets)) != 0, "ZERO_SHARES");


+    require(assets != 0, "ZERO_ASSETS");
     perpetualAtlanticVault.updateFunding();

     // Need to transfer before minting or ERC777s could reenter.
     collateral.transferFrom(msg.sender, address(this), assets);

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L128

## [G-12] Use Bitmaps to Save Gas

[Resource](https://code4rena.com/reports/2022-12-pooltogether/#g-01-use-bitmaps-to-save-gas)

```solidity
79  mapping(uint256 => bool) public optionsOwned;


82  mapping(uint256 => bool) public fundingPaidFor;
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L79-L82

## [G-13] Empty blocks should be removed or emit something to save some amount of Gas

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.

```solidity

231  function _beforeTokenTransfer(
232    address from,
233    address,
234    uint256
235  ) internal virtual {}
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L231-L235

## [G-14] Duplicated if() checks should be refactored to a modifier or function

```solidity

856      if (putOptionsRequired) {
920      if (putOptionsRequired) {
1195      if (putOptionsRequired) {

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L856

## [G-15] Use constants instead of type(uintx).max

It's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

Here's an example of how you can use a constant instead of type(uintX).max:

```solidity

contract MyContract {
    uint120 constant MAX_VALUE = 2**120 - 1;

    function doSomething(uint120 value) public {
        require(value <= MAX_VALUE, "Value exceeds maximum");

        // Do something
    }
}
```

In the above example, we have a contract with a constant MAX_VALUE that represents the maximum value of a uint120. When the doSomething function is called with a value parameter, it checks whether the value is less than or equal to MAX_VALUE using the <= operator.

By using a constant instead of type(uint120).max, we can make our code more efficient and reduce the gas cost of our contract.

It's important to note that using constants can make your code more readable and maintainable, since the value is defined in one place and can be easily updated if necessary. However, constants should be used with caution and only when their value is known at compile-time.

```solidity

126          type(uint128).max,
127          type(uint128).max

190        type(uint256).max

223        type(uint128).max,
224        type(uint128).max

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol

```solidity

341:      type(uint256).max
342    );
343:    IERC20WithBurn(weth).approve(addresses.dopexAMMRouter, type(uint256).max);
344:    IERC20WithBurn(weth).approve(addresses.dpxEthCurvePool, type(uint256).max);
345    IERC20WithBurn(weth).approve(
346      addresses.rdpxV2ReceiptToken,
347:      type(uint256).max
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol

```solidity

209      type(uint256).max

247        type(uint256).max
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol

```solidity

106    collateral.approve(_perpetualAtlanticVault, type(uint256).max);

107    ERC20(rdpx).approve(_perpetualAtlanticVault, type(uint256).max);

155      if (allowed != type(uint256).max) {
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol

```solidity

152      type(uint256).max

157      type(uint256).max

162      type(uint256).max

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L152

## [G-16] Using > 0 costs more gas than != 0 when used on a uint in a require() statement

```solidity

112    require(
113      _slippageTolerance > 0,
114      "reLPContract: slippage tolerance must be greater than 0"
115    );



133    require(_amount > 0, "reLPContract: amount must be greater than 0");

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L113

```solidity

183    _validate(_rdpxBurnPercentage > 0, 3);

196    _validate(_rdpxFeePercentage > 0, 3);

231    _validate(_bondMaturity > 0, 3);

410    _validate(_amount > 0, 17);

444    _validate(_bondDiscountFactor > 0, 3);

458    _validate(_slippageTolerance > 0, 3);

556      minAmount > 0 ? minAmount : minOut

838      _validate(_amounts[i] > 0, 4);

901    _validate(_amount > 0, 4);

984    _validate(amountWithdrawn > 0, 15);

1021    _validate(bonds[id].timestamp > 0, 6);

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L183

```solidity

93    require(
94      _reLPFactor > 0,
95      "reLPContract: reLP factor must be greater than 0"
96    );



174    require(
175      _liquiditySlippageTolerance > 0,
176      "reLPContract: liquidity slippage tolerance must be greater than 0"
177    );



189    require(
190      _slippageTolerance > 0,
191      "reLPContract: slippage tolerance must be greater than 0"
192    );
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L174-L177

## [G-17] Do not calculate constants

```solidity
88  uint256 public constant RDPX_RATIO_PERCENTAGE = 25 * DEFAULT_PRECISION;

91  uint256 public constant ETH_RATIO_PERCENTAGE = 75 * DEFAULT_PRECISION;

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L91

## [G-18] Missing zero-address check in constructor

Missing checks for zero-addresses may lead to infunctional protocol, if the variable addresses are updated incorrectly. It also wast gas as it requires the redeployment of the contract.

```solidity

76  constructor(address _rdpx, address _rdpxV2Core) {
77    rdpx = _rdpx;
78    rdpxV2Core = _rdpxV2Core;
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L76-L78

```solidity

124  constructor(address _weth) {
125    _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
126:    weth = _weth;
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L126

## [G-19] Ternary operation is cheaper than if-else statement

There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.

```solidity

145    if (use_safe_approve) {
146      // safeApprove needed for USDT and others for the first approval
147      // You need to approve 0 every time beforehand for USDT: it resets
148      TransferHelper.safeApprove(_token, _target, _amount);
149    } else {
150      IERC20WithBurn(_token).approve(_target, _amount);
151    }
```

```diff
diff --git a/UniV3LiquidityAmo.org.sol b/UniV3LiquidityAmo.sol
index 803574e..5bb337d 100644
--- a/UniV3LiquidityAmo.org.sol
+++ b/UniV3LiquidityAmo.sol
@@ -4,11 +4,10 @@
     uint256 _amount,
     bool use_safe_approve
   ) public onlyRole(DEFAULT_ADMIN_ROLE) {
-    if (use_safe_approve) {
-      // safeApprove needed for USDT and others for the first approval
-      // You need to approve 0 every time beforehand for USDT: it resets
-      TransferHelper.safeApprove(_token, _target, _amount);
-    } else {
-      IERC20WithBurn(_token).approve(_target, _amount);
-    }
-  }
\ No newline at end of file
+
+    use_safe_approve
+        ? TransferHelper.safeApprove(_token, _target, _amount)
+        : IERC20WithBurn(_token).approve(_target, _amount);
+  }

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L139-L152

```solidity

578    if (remainder == 0) {
579      return _strike;
580    } else {
581      return _strike - remainder + roundingPrecision;
582    }
```

```diff
diff --git a/PerpetualAtlanticVault.org.sol b/PerpetualAtlanticVault.sol
index acaba01..70bcb7f 100644
--- a/PerpetualAtlanticVault.org.sol
+++ b/PerpetualAtlanticVault.sol
@@ -1,8 +1,7 @@
   function roundUp(uint256 _strike) public view returns (uint256 strike) {
     uint256 remainder = _strike % roundingPrecision;
-    if (remainder == 0) {
-      return _strike;
-    } else {
-      return _strike - remainder + roundingPrecision;
-    }

+    return remainder == 0 ? _strike : _strike - remainder + roundingPrecision;

  }
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L578C1-L582C6

## [G-20] Pre-increment and pre-decrement are cheaper than +1,-1

```solidity

231        block.timestamp + 1

279        block.timestamp + 1

342        block.timestamp + 1

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol

```solidity

261    reservesIndex[_assetSymbol] = reserveAsset.length - 1;

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L261

## [G-21] Unnecessary casting as variable is already of the same type

UniV3LiquidityAmo.sol.liquidityInPool(): `rdpx` should not be cast to address as it’s declared as an address

```solidity

101      univ3_factory.getPool(address(rdpx), _collateral_address, _fee)

199      params._tokenA == address(rdpx) ? params._tokenB : params._tokenA,

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L101

## [G-22] `onlyRole(RDPXV2CORE_ROLE)` no uses of the nonreentrance modifier to save Gas

The `purchase,settle` function is only called by the `onlyRole(RDPXV2CORE_ROLE)` modifier, you can remove the nonReentrant modifier from the function because the role is safe.

```solidity

255  function purchase(
256    uint256 amount,
257    address to
258  )
259    external
260    nonReentrant
261    onlyRole(RDPXV2CORE_ROLE)
262    returns (uint256 premium, uint256 tokenId)
262  {



315  function settle(
316    uint256[] memory optionIds
317  )
318    external
319    nonReentrant
320    onlyRole(RDPXV2CORE_ROLE)
321    returns (uint256 ethAmount, uint256 rdpxAmount)
322  {
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol

## [G-23] Use mappings instead of arrays

Arrays are useful when you need to maintain an ordered list of data that can be iterated over, but they have a higher gas cost for read and write operations, especially when the size of the array is large. This is because Solidity needs to iterate over the entire array to perform certain operations, such as finding a specific element or deleting an element.

Mappings, on the other hand, are useful when you need to store and access data based on a key, rather than an index. Mappings have a lower gas cost for read and write operations, especially when the size of the mapping is large, since Solidity can perform these operations based on the key directly, without needing to iterate over the entire data structure.

```solidity

63  Position[] public positions_array;
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L63

```solidity

61  ReserveAsset[] public reserveAsset;

64  address[] public amoAddresses;

70  string[] public reserveTokens;

121  Delegate[] public delegates;
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L61

## [G-24] Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety. If using Solidity versions < 0.8.0 and you are using Safemath, you can gain significant gas savings by using assembly to calculate values and checking for overflow/underflow.

[Resource](https://medium.com/@bloqarl/solidity-gas-optimization-tips-with-assembly-you-havent-heard-yet-1381c77ff078)

```solidity

612    amount1 = (amount1 * (100e8 - _delegateFee)) / 1e10;
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L612

```solidity

270    uint256 strike = roundUp(currentPrice - (currentPrice / 4)); // 25% below the current price

276    uint256 requiredCollateral = (amount * strike) / 1e8;

283    uint256 timeToExpiry = nextFundingPaymentTimestamp() - block.timestamp;

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L270

```solidity

280    uint256 totalVaultCollateral = totalCollateral() +
281      ((_rdpxCollateral * rdpxPriceInAlphaToken) / 1e8);
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L280-L281

```solidity

228    uint256 baseReLpRatio = (reLPFactor *
229      Math.sqrt(tokenAInfo.tokenAReserve) *
230      1e2) / (Math.sqrt(1e18)); // 1e6 precision




232    uint256 tokenAToRemove = ((((_amount * 4) * 1e18) /
233      tokenAInfo.tokenAReserve) *
234      tokenAInfo.tokenALpReserve *
235      baseReLpRatio) / (1e18 * DEFAULT_PRECISION * 1e2);


239    uint256 lpToRemove = (tokenAToRemove * totalLpSupply) /
240      tokenAInfo.tokenALpReserve;


250    uint256 mintokenAAmount = tokenAToRemove -
251      ((tokenAToRemove * liquiditySlippageTolerance) / 1e8);


252    uint256 mintokenBAmount = ((tokenAToRemove * tokenAInfo.tokenAPrice) /
253      1e8) -
254      ((tokenAToRemove * tokenAInfo.tokenAPrice) * liquiditySlippageTolerance) /
255      1e16;
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L228-L230

## [G-25] Use hardcode address instead address(this)

```solidity

149      token.safeTransfer(msg.sender, token.balanceOf(address(this)));

162      address(this)

165      address(this)

212      address(this),

217      address(this),

230        address(this),

278        address(this),

323      address(this),

341        address(this),

363    lpTokenBalance = IERC20WithBurn(addresses.pair).balanceOf(address(this));

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol

```solidity

106      keccak256(abi.encodePacked(address(this), _tickLower, _tickUpper))

160      address(this),

165      address(this),

189        address(this),

285      address(this),

294        address(this),

331      address(this),

354 uint256 tokenABalance = IERC20WithBurn(tokenA).balanceOf(address(this));

355    uint256 tokenBBalance = IERC20WithBurn(tokenB).balanceOf(address(this));
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol

```solidity

169      token.safeTransfer(msg.sender, token.balanceOf(address(this)));

483    ).purchase(_amount, address(this));

573      address(this),

654        .safeTransferFrom(msg.sender, address(this), _rdpxAmount);

911      address(this),

953    IERC20WithBurn(weth).safeTransferFrom(msg.sender, address(this), _amount);

998        address(this)

1060      address(this),

1102          address(this),

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L169

```solidity

227      token.safeTransfer(msg.sender, token.balanceOf(address(this)));

289    collateralToken.safeTransferFrom(msg.sender, address(this), premium);

384      address(this),
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol

```solidity

128    collateral.transferFrom(msg.sender, address(this), assets);

192      collateral.balanceOf(address(this)) >= _totalCollateral + proceeds,

201      collateral.balanceOf(address(this)) == _totalCollateral - loss,

210      IERC20WithBurn(rdpx).balanceOf(address(this)) >= _rdpxCollateral + amount,

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L280C1-L281C57

```solidity

245      address(this),

263      address(this),

282        address(this),

293      address(this),

304      IERC20WithBurn(addresses.tokenA).balanceOf(address(this))
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L228C1-L230C33

## [G-26] Shorten the array rather than copying to a new one

To shorten the array without creating a new one, you can use the .length property to change the length of the existing array. Here's how you can do it:

path.length = 2;

This will effectively shorten the path array to have a length of 2 elements without copying to a new array.

```solidity

331    path = new address[](2);
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L331C1-L332C1

```solidity

832   uint256[] memory delegateReceiptTokenAmounts = new uint256[](
833      _amounts.length
834    );


1093      path = new address[](2);
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L832

```solidity

155    uint256[] memory tokenIds = new uint256[](ownerTokenCount);
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L155C1-L156C1

```solidity

268    path = new address[](2);
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L268
