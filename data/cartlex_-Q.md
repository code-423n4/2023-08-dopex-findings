# L-01 `decreaseAmount()` can also increase amount.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L135-L145

## Impact
`decreaseAmount` not only decrease amount, but this function can also increase it.

```solidity
/// @notice Decreases the bond amount
  /// @dev Can only be called by the rdpxV2Core
  /// @param bondId id of the bond to decrease
  /// @param amount amount to decrease
  function decreaseAmount(
    uint256 bondId,
    uint256 amount
  ) public onlyRole(RDPXV2CORE_ROLE) {
    _whenNotPaused();
    bonds[bondId].rdpxAmount = amount;
  }
```

## Recommendation
Consider use require statement

```solidity
/// @notice Decreases the bond amount
  /// @dev Can only be called by the rdpxV2Core
  /// @param bondId id of the bond to decrease
  /// @param amount amount to decrease
  function decreaseAmount(
    uint256 bondId,
    uint256 amount
  ) public onlyRole(RDPXV2CORE_ROLE) {
    _whenNotPaused();
++++require(bonds[bondId].rdpxAmount > amount, "can't increase amount");
    bonds[bondId].rdpxAmount = amount;
  }
```

# L-02. `setRdpxBurnPercentage` function doesn't check the upper boundary.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L175-L186

## Impact
`setRdpxBurnPercentage` hasn't upper boundary, so the `DEFAULT_ADMIN_ROLE` can set it to a value that is not expected.

```solidity
function setRdpxBurnPercentage(
    uint256 _rdpxBurnPercentage
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_rdpxBurnPercentage > 0, 3);
    rdpxBurnPercentage = _rdpxBurnPercentage;
    emit LogSetRdpxBurnPercentage(_rdpxBurnPercentage);
  }
```

## Recommendation: 
Consider using require statement to check the upper boundary.


# L-03. `setRdpxFeePercentage` function doesn't check the upper boundary.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L188-L199

## Impact
`setRdpxFeePercentage` hasn't upper boundary, so the `DEFAULT_ADMIN_ROLE` can set it to a value that is not expected.

```solidity
function setRdpxFeePercentage(
    uint256 _rdpxFeePercentage
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_rdpxFeePercentage > 0, 3);
    rdpxFeePercentage = _rdpxFeePercentage;
    emit LogSetRdpxFeePercentage(_rdpxFeePercentage);
  }
```

## Recommendation: 
Consider using require statement to check the upper boundary.


# L-04. `setBondMaturity` function doesn't check the upper boundary.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L223-L234

## Impact
`setBondMaturity` hasn't upper boundary, so the `DEFAULT_ADMIN_ROLE` can set it to a value that is not expected.

```solidity
function setBondMaturity(
    uint256 _bondMaturity
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_bondMaturity > 0, 3);
    bondMaturity = _bondMaturity;
    emit LogSetBondMaturity(_bondMaturity);
  }
```

## Recommendation: 
Consider using require statement to check the upper boundary.


# L-05. `setBondDiscount` function doesn't check the upper boundary.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L436-L448

## Impact
`setBondDiscount` hasn't upper boundary, so the `DEFAULT_ADMIN_ROLE` can set it to a value that is not expected.

```solidity
function setBondDiscount(
    uint256 _bondDiscountFactor
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_bondDiscountFactor > 0, 3);
    bondDiscountFactor = _bondDiscountFactor;

    emit LogSetBondDiscountFactor(_bondDiscountFactor);
  }
```

## Recommendation: 
Consider using require statement to check the upper boundary.


# L-06. `setSlippageTolerance` function doesn't check the upper boundary.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L450-L462

## Impact
`setSlippageTolerance` hasn't upper boundary, so the `DEFAULT_ADMIN_ROLE` can set it to a value that is not expected.

```solidity
function setSlippageTolerance(
    uint256 _slippageTolerance
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_slippageTolerance > 0, 3);
    slippageTolerance = _slippageTolerance;

    emit LogSetSlippageTolerance(_slippageTolerance);
  }
```

## Recommendation: 
Consider using require statement to check the upper boundary.





# L-07 `setreLpFactor` function doesn't check the upper boundary.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L85-L100

## Impact
`setreLpFactor` not only decrease amount, but this function can also increase it.

```solidity
/**
   * @notice Set the re-LP factor
   * @dev    Can only be called by admin
   * @param  _reLPFactor the bond discount factor
   **/
  function setreLpFactor(
    uint256 _reLPFactor
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(
      _reLPFactor > 0,
      "reLPContract: reLP factor must be greater than 0"
    );
    reLPFactor = _reLPFactor;

    emit LogSetReLpFactor(_reLPFactor);
  }
```

## Recommendation: 
Consider using require statement to check the upper boundary.


# L-08 `setLiquiditySlippageTolerance` function doesn't check the upper boundary.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L166-L179

## Impact
`setLiquiditySlippageTolerance` not only decrease amount, but this function can also increase it.

```solidity
/**
   * @notice sets the liquidity slippage tolerance
   * @dev    Can only be called by admin
   * @param  _liquiditySlippageTolerance the liquidity slippage tolerance
   */
  function setLiquiditySlippageTolerance(
    uint256 _liquiditySlippageTolerance
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(
      _liquiditySlippageTolerance > 0,
      "reLPContract: liquidity slippage tolerance must be greater than 0"
    );
    liquiditySlippageTolerance = _liquiditySlippageTolerance;
  }
```

## Recommendation: 
Consider using require statement to check the upper boundary.


# L-09 `setSlippageTolerance` function doesn't check the upper boundary.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L181-L194

## Impact
`setSlippageTolerance` not only decrease amount, but this function can also increase it.

```solidity
/**
   * @notice sets the slippage tolerance
   * @dev    Can only be called by admin
   * @param  _slippageTolerance the slippage tolerance
   */
  function setSlippageTolerance(
    uint256 _slippageTolerance
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(
      _slippageTolerance > 0,
      "reLPContract: slippage tolerance must be greater than 0"
    );
    slippageTolerance = _slippageTolerance;
  }
```

## Recommendation: 
Consider using require statement to check the upper boundary.


# N-01. `bondMaturity` default value isn't 5 days.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L106

## Impact
In the documentation says that the 
```
After the bonding process is complete a bond (ERC721 token) will be minted to the user which can be redeemed after a variable maturity (initially to be set at `5 days`) for the receipt tokens.
```
It has the default value of zero.

## Recomendation: 
Set default value of `bondMaturity` to 5 days.
```solidity
 /// @notice Bond maturity
  uint256 public bondMaturity = 5 days;
```
