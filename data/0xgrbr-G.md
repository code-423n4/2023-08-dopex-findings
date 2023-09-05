## 1. Use calldata instead of memory for function parameters that don’t change
When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

Total Instances: 8

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L156

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L244

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L273

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L768

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L824

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L825

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L316

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L406

Change function arguments from memory to calldata.

## 2. Reorder structure layout
The following structures could be optimized moving the position of certain values in order to save a slot:

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L67
```
  /// @notice Token address that dpxEth is pegged to
  address public weth;
```
Move to:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L116
```
....
  /// @notice Token address that dpxEth is pegged to
  address public weth;

  /// @notice Whether reLP is active or not
  bool public isReLPActive;

  /// @notice Whether put options are requred
  bool public putOptionsRequired;
....
```

## 3. Fetch variable before the loop

The current code calls getUnderlyingPrice() inside the loop, which is inefficient and consumes more gas for each iteration of the loop.

Instances: 2 

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L333

Original Code:
```
....
    for (uint256 i = 0; i < optionIds.length; i++) {
      uint256 strike = optionPositions[optionIds[i]].strike;
      uint256 amount = optionPositions[optionIds[i]].amount;

      // check if strike is ITM
      _validate(strike >= getUnderlyingPrice(), 7);
....
```
Suggested Improvement:

Fetch the value of getUnderlyingPrice() outside of the loop to save on gas costs.

```
    uint256 underlyingPrice = getUnderlyingPrice()

    for (uint256 i = 0; i < optionIds.length; i++) {
      uint256 strike = optionPositions[optionIds[i]].strike;
      uint256 amount = optionPositions[optionIds[i]].amount;

      // check if strike is ITM
      _validate(strike >= underlyingPrice, 7);
....
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L433

Original Code:
```
....
    for (uint256 i = 0; i < strikes.length; i++) {
      _validate(optionsPerStrike[strikes[i]] > 0, 4);
      _validate(
        latestFundingPerStrike[strikes[i]] != latestFundingPaymentPointer,
        5
      );
      uint256 strike = strikes[i];

      uint256 amount = optionsPerStrike[strike] -
        fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
          strike
        ];

      uint256 timeToExpiry = nextFundingPaymentTimestamp() -
        (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration));

      uint256 premium = calculatePremium(
        strike,
        amount,
        timeToExpiry,
        getUnderlyingPrice()
      );
....
```
Suggested Improvement:

Fetch the value of getUnderlyingPrice() outside of the loop to save on gas costs.

```
....
    uint256 underlyingPrice = getUnderlyingPrice()

    for (uint256 i = 0; i < strikes.length; i++) {
      _validate(optionsPerStrike[strikes[i]] > 0, 4);
      _validate(
        latestFundingPerStrike[strikes[i]] != latestFundingPaymentPointer,
        5
      );
      uint256 strike = strikes[i];

      uint256 amount = optionsPerStrike[strike] -
        fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
          strike
        ];

      uint256 timeToExpiry = nextFundingPaymentTimestamp() -
        (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration));

      uint256 premium = calculatePremium(
        strike,
        amount,
        timeToExpiry,
        underlyingPrice
      );
....
```


## 4. Consider reduce the size and pack with other timestamps 

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L89

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L95

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L98

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L101

Consider changing the timestamp variables to uint64 and packing them together to save storage slots. This approach can optimize storage costs and make the contract more efficient.
