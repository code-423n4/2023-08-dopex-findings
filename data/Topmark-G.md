1. https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L545
```solidity
      premium = ((IOptionPricing(addresses.optionPricing).getOptionPrice(
      _strike,
      _price > 0 ? _price : getUnderlyingPrice(),
      getVolatility(_strike),
      timeToExpiry
    ) * _amount) / 1e8);
```
reducing the gas usage from this getOptionPrice(...) function call in PerpetualAtlanticVault.sol contract is possible by removing some unnecessary code as seen below from
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/libraries/OptionPricingSimple.sol#L73-L77
```solidity
  function getOptionPrice(
  ...
  ) external view override returns (uint256) {
    ...
    --- bool isPut = true;

    uint256 optionPrice = BlackScholes
      .calculate(
    --- isPut ? 1 : 0, // 0 - Put, 1 - Call
    +++ 1, // 0 - Put, 1 - Call
        lastPrice,
        strike,
        timeToExpiry, // Number of days to expiry mul by 100
        0,
        volatility
      )
      .div(BlackScholes.DIVISOR);
   ...
}
```
isPut is set to true and only used once in the function in the calculate function call, a better way to save gas is to remove it and simply use 1 in the calculate function call