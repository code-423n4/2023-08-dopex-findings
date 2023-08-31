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
isPut is set to true and only used once in the function in the calculate function call, a better way to save gas is to remove it and simply use 1 in the calculate function call.

2. https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1189-L1198
```solidity
  function calculateBondCost(
    ...
  ) public view returns (uint256 rdpxRequired, uint256 wethRequired) {
    ...
 +++ if (putOptionsRequired) {
       uint256 strike = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
      .roundUp(rdpxPrice - (rdpxPrice / 4)); // 25% below the current price
       uint256 timeToExpiry = IPerpetualAtlanticVault(
      addresses.perpetualAtlanticVault
    ).nextFundingPaymentTimestamp() - block.timestamp;
 --- if (putOptionsRequired) {
      wethRequired += IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
        .calculatePremium(strike, rdpxRequired, timeToExpiry, 0);
    }
}
```
since strike and timeToExpiry are declared and used solely be because of the proceeding "if" condition, a more direct way to save gas is it validate the condition first before carrying out any other computational process.