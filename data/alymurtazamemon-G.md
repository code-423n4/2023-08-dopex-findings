# Dopex - Gas Optimization Report

## Table Of Content

| Number                                                                                        | Issue                                                                           | Instances |
| :-------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------ | --------: |
| [G-01](#g-01---use-custom-errors-to-save-users-gas-cost)                                      | Use `custom errors` to save users gas cost                                      |         5 |
| [G-02](#g-02---use-do-while-loop-instead-of-for-loop-to-save-users-gas-cost)                  | Use `do-while` loop instead of `for-loop` to save users gas cost                |         3 |
| [G-03](#g-03---multiple-reads-of-storage-variable-which-can-be-cached-to-save-users-gas-cost) | Multiple reads of `storage` variable which can be cached to save users gas cost |         6 |
| [G-04](#g-04---use-calldata-data-location-for-functions-parameters-to-save-gas)               | Use `calldata` data location for functions parameters to save gas               |         4 |
| [G-05](#g-05---variables-contains-keccak256-expression-should-be-immutable)                   | Variables contains `keccak256` expression should be `immutable`                 |         2 |
| [G-06](#g-06---do-not-emit-the-information-in-the-events-which-is-already-included-in-the-tx) | Do not emit the information in the events which is already included in the `tx` |         1 |
| [G-07](#g-07---use-unchecked-where-possible-to-save-users-gas)                                | Use `unchecked` where possible to save users gas                                |         1 |
| [G-08](#g-08---reorder-modifers-to-save-gas-cost)                                             | Reorder `modifers` to save gas cost                                             |         1 |

### [G-01] - Use `custom errors` to save users gas cost.

**Details**

Custom errors are very efficient gas vice and should be used to save users gas costs.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[UniV2LiquidityAmo.sol - Line 83 - 92](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L83-L92)

| Calculation Type | Before | After |          Gas Saved |
| :--------------- | :----- | :---- | -----------------: |
| Avg              | 1681   | 1573  | 108 uints per call |

```diff
    // add this in errors section
+   error UniV2LiquidityAMO_AddressCannotBeZero();

+   // * @audit use custom error
-   require(
-       _tokenA != address(0) &&
-           _tokenB != address(0) &&
-           _pair != address(0) &&
-           _rdpxV2Core != address(0) &&
-           _rdpxOracle != address(0) &&
-           _ammFactory != address(0) &&
-           _ammRouter != address(0),
-       "reLPContract: address cannot be 0"
-   );
+   if (
+       _tokenA == address(0) ||
+       _tokenB == address(0) ||
+       _pair == address(0) ||
+       _rdpxV2Core == address(0) ||
+       _rdpxOracle == address(0) ||
+       _ammFactory == address(0) ||
+       _ammRouter == address(0)
+   ) {
+       revert UniV2LiquidityAMO_AddressCannotBeZero();
+   }
```

[UniV2LiquidityAmo.sol - Line 112 - 115](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L112-L115)

| Calculation Type | Before | After |         Gas Saved |
| :--------------- | :----- | :---- | ----------------: |
| Avg              | 828    | 755   | 73 uints per call |

```diff
    // define this in errors section
+   error UniV2LiquidityAMO_ToleranceMustBeGreaterThanZero();

+   // * @audit use custom error
-   require(
-       _slippageTolerance > 0,
-       "reLPContract: slippage tolerance must be greater than 0"
-   );
+   if (_slippageTolerance == 0) {
+       revert UniV2LiquidityAMO_ToleranceMustBeGreaterThanZero();
+   }

```

[UniV2LiquidityAmo.sol - Line 131 - 133](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L131-L133)

| Calculation Type | Before | After |         Gas Saved |
| :--------------- | :----- | :---- | ----------------: |
| Avg              | 1012   | 969   | 43 uints per call |

```diff
    // define this in the erros section
+   error UniV2LiquidityAMO_AmountMustBeGreaterThanZero();

+   // * @audit use custom error
-   require(_token != address(0), "reLPContract: token cannot be 0");
-   require(_spender != address(0), "reLPContract: spender cannot be 0");
-   require(_amount > 0, "reLPContract: amount must be greater than 0");
+   if (_token == address(0) || _spender == address(0)) {
+       revert UniV2LiquidityAMO_AddressCannotBeZero();
+   }
+
+   if (_amount == 0) {
+       revert UniV2LiquidityAMO_AmountMustBeGreaterThanZero();
+   }
```

[RdpxV2Core.sol - Line 244](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L244)

| Calculation Type | Before | After |         Gas Saved |
| :--------------- | :----- | :---- | ----------------: |
| Avg              | 1172   | 1093  | 79 uints per call |

```diff
+   // * @audit use custom error
-   require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");
+   if (_asset == address(0)) {
+       revert RdpxV2Core_ZeroAddress();
+   }
```

[RdpxV2Core.sol - Line 271](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L271)

| Calculation Type | Before | After |          Gas Saved |
| :--------------- | :----- | :---- | -----------------: |
| Avg              | 51556  | 51120 | 436 uints per call |

```diff
+   // * @audit gas - use calldata parameter
   function removeAssetFromtokenReserves(
-       string memory _assetSymbol
+       string calldata _assetSymbol
   ) external onlyRole(DEFAULT_ADMIN_ROLE) {
```

</details>

### [G-02] - Use `do-while` loop instead of `for-loop` to save users gas cost.

**Details**

do-while does not check the first condition and prevent assembly from executing losts of opcodes need for conditions checks and all these places are right for it because these all always execute the code inside loops on the first condition.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[UniV2LiquidityAmo.sol - Line 147 - 150](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L147-L150)

| Calculation Type | Before | After |          Gas Saved |
| :--------------- | :----- | :---- | -----------------: |
| Avg              | 4943   | 4811  | 132 uints per call |

```diff
+   // * @audit use do while loop
+   // * @audit use unchecked and ++i

-   for (uint256 i = 0; i < tokens.length; i++) {
       token = IERC20WithBurn(tokens[i]);
       token.safeTransfer(msg.sender, token.balanceOf(address(this)));
-   }

+   uint256 i;
+   do {
       token = IERC20WithBurn(tokens[i]);
       token.safeTransfer(msg.sender, token.balanceOf(address(this)));
+
+       unchecked {
+           ++i;
+       }
+   } while (i < tokens.length);
```

[RdpxV2Core.sol - Lines 167 - 170](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L167-L170)

| Calculation Type | Before | After |          Gas Saved |
| :--------------- | :----- | :---- | -----------------: |
| Avg              | 4943   | 4811  | 132 uints per call |

```diff
+   // * @audit use do while loop
+   // * @audit use unchecked and ++i
-   for (uint256 i = 0; i < tokens.length; i++) {
       token = IERC20WithBurn(tokens[i]);
       token.safeTransfer(msg.sender, token.balanceOf(address(this)));
-   }

+   uint256 i;
+   do {
       token = IERC20WithBurn(tokens[i]);
       token.safeTransfer(msg.sender, token.balanceOf(address(this)));
+
+       unchecked {
+           ++i;
+       }
+   } while (i < tokens.length);
```

[RdpxV2Core.sol - Line 775 - 777](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L775-L777)

| Calculation Type | Before | After |         Gas Saved |
| :--------------- | :----- | :---- | ----------------: |
| Avg              | 49165  | 49096 | 69 units per call |

```diff
+   // * @audit use do while loop
+   // * @audit use unchecked
-   for (uint256 i = 0; i < optionIds.length; i++) {
        optionsOwned[optionIds[i]] = false;
-   }

+   uint256 i;
+   do {
       optionsOwned[optionIds[i]] = false;
+
+       unchecked {
+           ++i;
+       }
+   } while (i < optionIds.length);
```

</details>

### [G-03] - Multiple reads of `storage` variable which can be cached to save users gas cost.

**Details**

SLOAD (is the opcode used of read storage variable) cost 100 units of gas and MLOAD & MSTORE (are the opcodes used to read and write to memory respectivily) cost 3, 3 units of gas respectivily.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[UniV3LiquidityAmo.sol - Line 120 - 132](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L120-L132)

| Calculation Type | Before | After |          Gas Saved |
| :--------------- | :----- | :---- | -----------------: |
| Avg              | 5860   | 5255  | 605 uints per call |

```diff
+   // * @audit storage array should be cached
+   // * @audit use unchecked and ++i

+   Position[] memory positionsArray = positions_array;

-   for (uint i = 0; i < positions_array.length; i++) {
+   for (uint i = 0; i < positionsArray.length;) {
-       Position memory current_position = positions_array[i];
+       Position memory current_position = positionsArray[i];

       INonfungiblePositionManager.CollectParams
           memory collect_params = INonfungiblePositionManager
               .CollectParams(
                   current_position.token_id,
                   rdpxV2Core,
                   type(uint128).max,
                   type(uint128).max
               );

       // Send to custodian address
       univ3_positions.collect(collect_params);
+
+       unchecked {
+           ++i;
+       }
+   }
```

[UniV3LiquidityAmo.sol - Line 260](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L260)

| Calculation Type | Before | After |          Gas Saved |
| :--------------- | :----- | :---- | -----------------: |
| Avg              | 2837   | 2734  | 103 uints per call |

```diff
+   // * @audit `positions_array.length` should be cached
+   uint256 positionsArrayLength = positions_array.length;

   positions_array[positionIndex] = positions_array[
-       positions_array.length - 1
+       positionsArrayLength - 1
   ];

   positions_array.pop();

   delete positions_mapping[pos.token_id];

   // send tokens to rdpxV2Core
   _sendTokensToRdpxV2Core(tokenA, tokenB);

-   emit log(positions_array.length);
+   emit log(positionsArrayLength);
   emit log(positions_mapping[pos.token_id].token_id);
```

[RdpxV2Core.sol - Link 246](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L246)

| Calculation Type | Before | After  |          Gas Saved |
| :--------------- | :----- | :----- | -----------------: |
| Avg              | 103709 | 103436 | 273 uints per call |

```diff
+   // * @audit `reserveAsset` length should be cached
+   uint256 reserveAssetsLength = reserveAsset.length;
-   // for (uint256 i = 1; i < reserveAsset.length; ) {
+   for (uint256 i = 1; i < reserveAssetsLength; ) {
        require(
            reserveAsset[i].tokenAddress != _asset,
            "RdpxV2Core: asset already exists"
        );

        unchecked {
            ++i;
        }
    }

    ReserveAsset memory asset = ReserveAsset({
        tokenAddress: _asset,
        tokenBalance: 0,
        tokenSymbol: _assetSymbol
    });

    reserveAsset.push(asset);

    reserveTokens.push(_assetSymbol);

-    reservesIndex[_assetSymbol] = reserveAsset.length - 1;
+    // here we did not remove 1 because we push another element after storing length into this variable
+    reservesIndex[_assetSymbol] = reserveAssetsLength;

    emit LogAssetAddedTotokenReserves(_asset, _assetSymbol);
```

[RdpxV2Core.sol - Line 280](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L280)

| Calculation Type | Before | After |         Gas Saved |
| :--------------- | :----- | :---- | ----------------: |
| Avg              | 51120  | 51035 | 85 uints per call |

```diff
+   // * @audit `reserveTokens.length` should be cached
+   uint256 reserveTokensLength = reserveTokens.length;

-   reservesIndex[reserveTokens[reserveTokens.length - 1]] = index;
+   reservesIndex[reserveTokens[reserveTokensLength - 1]] = index;

// update the index of reserveAsset with the last element
-   reserveAsset[index] = reserveAsset[reserveTokens.length - 1];
+   reserveAsset[index] = reserveAsset[reserveTokensLength - 1];

// remove the last element
reserveAsset.pop();
reserveTokens.pop();

emit LogAssetRemovedFromtokenReserves(_assetSymbol, index);
```

[RdpxV2Core.sol - Lines 339 - 345](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L339-L345)

| Calculation Type | Before | After  |          Gas Saved |
| :--------------- | :----- | :----- | -----------------: |
| Avg              | 329730 | 329432 | 298 uints per call |

```diff
+   // * @audit `weth` should be cached
+   address _weth = weth;

-   IERC20WithBurn(weth).approve(
+   IERC20WithBurn(_weth).approve(
    addresses.perpetualAtlanticVault,
    type(uint256).max
);

-   IERC20WithBurn(weth).approve(
+   IERC20WithBurn(_weth).approve(
    addresses.dopexAMMRouter,
    type(uint256).max
);

-   IERC20WithBurn(weth).approve(
+   IERC20WithBurn(_weth).approve(
    addresses.dpxEthCurvePool,
    type(uint256).max
);

-   IERC20WithBurn(weth).approve(
+   IERC20WithBurn(_weth).approve(
    addresses.rdpxV2ReceiptToken,
    type(uint256).max
);
```

[RdpxV2Core.sol - Lines 388 - 394](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L388-L394)

| Calculation Type | Before | After |         Gas Saved |
| :--------------- | :----- | :---- | ----------------: |
| Avg              | 1868   | 1787  | 81 units per call |

```diff
    function removeAMOAddress(
        uint256 _index
    ) external onlyRole(DEFAULT_ADMIN_ROLE) {
        // _validate(_index < amoAddresses.length, 18);
+        // * @audit `amoAddresses.length` should be cached
+        uint256 amoAddressesLength = amoAddresses.length;
-        if (_index >= amoAddresses.length) revert RdpxV2Core_AssetNotFound();
+        if (_index >= amoAddressesLength) revert RdpxV2Core_AssetNotFound();

-        amoAddresses[_index] = amoAddresses[amoAddresses.length - 1];
+        amoAddresses[_index] = amoAddresses[amoAddressesLength - 1];

        amoAddresses.pop();
    }
```

</details>

### [G-04] - Use `calldata` data location for functions parameters to save gas.

**Details**

-   Solidity docs suggests that `If you can, try to use calldata as data location because it will avoid copies and also makes sure that the data cannot be modified.`
-   And it also save gas when compare to memory data location.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[UniV3LiquidityAmo.sol - Line 156](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L156)

| Calculation Type | Before  | After   |          Gas Saved |
| :--------------- | :------ | :------ | -----------------: |
| Avg              | 3737309 | 3736785 | 524 units per call |

```diff
+   // * @audit should use the calldata parameter
    function addLiquidity(
-        AddLiquidityParams memory params
+        AddLiquidityParams calldata params
    ) public onlyRole(DEFAULT_ADMIN_ROLE) {
```

[RdpxV2Core.sol - Line 242](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L242)

| Calculation Type | Before | After  |          Gas Saved |
| :--------------- | :----- | :----- | -----------------: |
| Avg              | 104095 | 103777 | 318 units per call |

```diff
+   // * @audit use calldata parameters
   function addAssetTotokenReserves(
       address _asset,
-       string memory _assetSymbol
+       string calldata _assetSymbol
   ) external onlyRole(DEFAULT_ADMIN_ROLE) {
```

[RdpxV2Core.sol - Line 765](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L765)

| Calculation Type | Before | After |          Gas Saved |
| :--------------- | :----- | :---- | -----------------: |
| Avg              | 49682  | 49165 | 517 units per call |

```diff
function settle(
-    uint256[] memory optionIds
+    uint256[] calldata optionIds
)
```

[RdpxV2Core.sol - Lines 821 - 822](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L821-L822)

| Calculation Type | Before  | After   |          Gas Saved |
| :--------------- | :------ | :------ | -----------------: |
| Avg              | 1092177 | 1091492 | 685 units per call |

```diff
+   // * @audit-ok should use calldata parameters
    function bondWithDelegate(
        address _to,
-       uint256[] memory _amounts,
-       uint256[] memory _delegateIds,
+       uint256[] calldata _amounts,
+       uint256[] calldata _delegateIds,
        uint256 rdpxBondId
    ) public returns (uint256 receiptTokenAmount, uint256[] memory) {
```

</details>

### [G-05] - Variables contains `keccak256` expression should be `immutable`.

**Details**

Expressions are not constant they execute everytime the variable calls, using them with `immutable` variables will only compute them on deployment time.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[RdpxV2Bond.sol - Line 22](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L22)

```diff
+   // * @audit keccak256 variable should be immutable
-   bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
+   bytes32 public immutable MINTER_ROLE;

    constructor() ERC721("rDPX V2 Bond", "rDPXV2Bond") {
+       MINTER_ROLE = keccak256("MINTER_ROLE");
```

[DpxEthToken.sol - Line 19 - 21](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L19-L21)

```diff
+   // * @audit gas - keccak256 variables should be immutable
-   bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
-   bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
-   bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
+   bytes32 public immutable PAUSER_ROLE;
+   bytes32 public immutable MINTER_ROLE;
+   bytes32 public immutable BURNER_ROLE;

    constructor() ERC20("Dopex Synthetic ETH", "dpxETH") {
+       PAUSER_ROLE = keccak256("PAUSER_ROLE");
+       MINTER_ROLE = keccak256("MINTER_ROLE");
+       BURNER_ROLE = keccak256("BURNER_ROLE");
```

</details>

### [G-06] - Do not emit the information in the events which is already included in the `tx`.

**Details**

Each event log parameter almost cost `375` units of gas. Reading them outside is completely free but emitting them from inside the contract cost gas. Due to that, we should only emit the information that is necessary and information that is already available such as block.timestamp, msg.sender, etc should not be emitted to reduce gas costs.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[RdpxV2Core.sol - Line 172](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L172)

If we remove it then we can save `375` per parameter.

```diff
-   emit LogEmergencyWithdraw(msg.sender, tokens);
+   emit LogEmergencyWithdraw(tokens);
```

</details>

### [G-07] - Use `unchecked` where possible to save users gas.

**Details**

When `unchecked` is used variable then compiler do not execute opcodes which check over/underflow errors, it can be risky some times but we should use it when we are sure it will not exceed the limits.

And here we have a huge limit for it.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[RdpxV2Core.sol - Line 246](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L246)

| Calculation Type | Before | After  | Gas Saved |
| :--------------- | :----- | :----- | --------: |
| Avg              | 103777 | 103709 |        68 |

```diff
+   // * @audit use unchecked
-   for (uint256 i = 1; i < reserveAsset.length; i++) {
+   for (uint256 i = 1; i < reserveAsset.length; ) {
    require(
        reserveAsset[i].tokenAddress != _asset,
        "RdpxV2Core: asset already exists"
    );

+    unchecked {
+        ++i;
+    }
}
```

</details>

### [G-08] - Reorder `modifers` to save gas cost.

**Details**

If we reoder the modifiers in the below mentioned instances then the gas cost will be efficient.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[PerpetualAtlanticVault.sol - Line 260 - 261](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L260-L261)

```diff
+    // * @audit reorder modifiers
    function purchase(
        uint256 amount,
        address to
    )
        external
-        nonReentrant
-        onlyRole(RDPXV2CORE_ROLE)
+        onlyRole(RDPXV2CORE_ROLE)
+        nonReentrant
        returns (uint256 premium, uint256 tokenId)
```

</details>