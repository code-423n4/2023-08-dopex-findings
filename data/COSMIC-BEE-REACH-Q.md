## Summary
### [LOW]
1. Possible unhandled underflow in PerpetualAtlanticVaultLP::redeem()
2. No validations to verify that `rdpxAmount` is not 0
3. UniV2LiquidityAmo::liquidityInPool() needs validation to check for pool existence
4. The contract `RdpxV2Core` needs to add validations before using reserve assets
5. No validations to verify that expiry is at least `block.timestamp`
6. Adding of assets to the token reserve may DoS due to block gas limits
7. `UniV2LiquidityAmo::addLiquidity()` needs validation for token X amount min <= token X amount
8. Missing validation `UniV3LiquidityAmo::addLiquidity()` to verify that min amount is less than the desired

### [QA]
###### 1. Documentation is just placeholder
The documentation template was not updated to reflect the actual docs
```
  /// @dev the pointer to the lattest funding payment timestamp
  /// @notice Explain to an end user what this does
  /// @dev Explain to a developer any extra details
  /// @return Documents the return variables of a contract’s function state variable
  /// @inheritdoc IPerpetualAtlanticVault
```
Code: [link](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L84-L88)

###### 2. The 75% calculation can be made simplified
The code below calculates `// 25% below the current price` by subtracting 25% of value from the value. This can rather be made simpler by just doing (3*X)/4

##### Recommendation
`uint256 strike = roundUp(currentPrice - (currentPrice / 4)); // 25% below the current price`

Code: [link](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L270)

###### 3. `PerpetualAtlanticVaultLP::convertToShares()` can return assets directly if supply is 0, skipping all the extra calculations

In the final return statement of the function` PerpetualAtlanticVaultLP::convertToShares()`, it checks if the supply is 0, and just returns the assets. The code can be refactored to do this check early.
```
return
  supply == 0 ? assets : assets.mulDivDown(supply, totalVaultCollateral);
```
Code: [link](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L274)

##### Recommendation
Add the following to the function:
```
    if (supply == 0) {
        return assets;
    }
  }
```

## Vulnerability Details
###### 1. The PerpetualAtlanticVaultLP.redeem() function does not check if the allowed value is atleast shares, which can lead to allowed - shares to underflow, and then revert unexpectedly.
```
if (allowed != type(uint256).max) {
  allowance[owner][msg.sender] = allowed - shares;
}
```
 Code: [Link](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L153)

###### 2. No validations to verify that rdpxAmount is not 0. Consider adding checks to RdpxDecayingBonds::mint() function to make sure than `rdpxAmount > 0``
```
function mint(
  address to,
  uint256 expiry,
  uint256 rdpxAmount
)
```
Code: [Link](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L114)

###### 3. getPool() [code](https://docs.uniswap.org/contracts/v3/reference/core/interfaces/IUniswapV3Factory#getpool) "smartCard-inline") returns `address(0)` for pools that may not exist. If `getPool()` returns 0, then the `liquidityInPool()` function call can fail which can be hard to troubleshoot.
```univ3_factory.getPool(address(rdpx), _collateral_address, _fee```

Code: [Link](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L101)

###### 4. The contract `RdpxV2Core` relies on certain token to always be available in the `reserveAsset`, and many functions access the array directly with the assumption that it exists in the array.

**The same applies to all tokens that are referenced like reservesIndex[<TOKEN>]**

There are too many instances to mention them all

During contract creation, if the array was not initialized properly, or if it was removed using the remove function accidentally, then the functions in the contract will revert, and it will be hard to troubleshoot why.

**Below are the list of those initial tokens in the reserve:**
- index0: ZERO address
- index1: weth
- index2: rdpx
- index3: dpxEth
- index4: crv

Code: [link](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L52)

---
###### 5. Consider adding checks to RdpxDecayingBonds::mint() function to make sure than expiry >= block.timestamp

function mint(
  address to,
  uint256 expiry,
  uint256 rdpxAmount
)

Code: [link](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L114)

---
###### 6. The length of `reserveAsset` might become too large, which will lead to the failure of `RdpxV2Core::addAssetTotokenReserves()` function because of the block gas limits, as it performs a loop on the `reserveAsset`.
```
for (uint256 i = 1; i < reserveAsset.length; i++) {
  require(
    reserveAsset[i].tokenAddress != _asset,
    "RdpxV2Core: asset already exists"
  );
}
```
Code: [link](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L246)

###### 7. There are two inputs provided to `addLiquidity` for each token. A min amount for token, and the amount for the tokens. There is no validation to verify that the minimum amount is actually less that the amount.
```
  function addLiquidity(
    uint256 tokenAAmount,
    uint256 tokenBAmount,
    uint256 tokenAAmountMin,
    uint256 tokenBAmountMin
  )
```
Code: [Link](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L189)

Also, a call to UniswapV2 `addLiquidity` is done with the provided token values, and the Uniswap function requires the min amount to not exceed the desired amount.
Uniswap Documentation: [link](https://docs.uniswap.org/contracts/v2/reference/smart-contracts/router-02#addliquidity)

If the min value exceeds the desired value, then the transaction could revert without a clear indication of the failure.

###### 8. `function addLiquidity(`

Code: [link](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L155)

`AddLiquidityParams params` is passed to the `addLiquidity()` function. The struct has desired and min values for `amount0` and `amount1`. But, there are no checks performed to make sure that the min value is not greater than the desired value for both the `amount0` and `amount1`.
If incorrect values are passed, then the call can fail which can be hard to troubleshoot.

## Recommendation
1. Add the following validation before performing `allowed - shares`:
```require(allowed >= shares, "Not enough allowance");```
2. Add the required validation:
```require(rdpxAmount != 0);```
3. Verify that `getPool()` does not return `address(0)`
4. Add checks to validate the existence of the token in the `reserveAsset` array.
5. Add the required validation:
```require(expiry >= block.timestamp);```
6. Instead of looping over the entire array, the token address can be stored in a mapping. This mapping can be used to check if the token address already exists.
7. Add the required validations:
`require(tokenAAmount >= tokenAAmountMin);`
`require(tokenBAmount >= tokenBAmountMin);`
8. Add the required validations:
`require(params._amount0Desired >= params._amount0Min);`
`require(params._amount1Desired >= params._amount1Min);`