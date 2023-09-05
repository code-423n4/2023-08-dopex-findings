# [G-01] Redundant required

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L120

This:
```solidity
require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
```
already handled in modifier: 
```solidity
  ) external onlyRole(MINTER_ROLE) {
```
so, remove it

# [G-02] Make variable immutable

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L57
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L95
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L49
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L46
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L67

Make these variable immutable to save gas.

# [G-03] Pass currentPrice to save gas
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L286

```solidity
premium = calculatePremium(strike, amount, timeToExpiry, 0);
```
Instead of 0 you may pass `currentPrice`, so you get same behaviour, due to implementation details
```solidity
  function calculatePremium(
    uint256 _strike,
    uint256 _amount,
    uint256 timeToExpiry,
    uint256 _price
  ) public view returns (uint256 premium) {
    premium = ((IOptionPricing(addresses.optionPricing).getOptionPrice(
      _strike,
      _price > 0 ? _price : getUnderlyingPrice(),
      getVolatility(_strike),
      timeToExpiry
    ) * _amount) / 1e8);
  }
```

# [G-04] Redundant check

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L324

`_isEligibleSender` covered by `onlyRole(RDPXV2CORE_ROLE)`, so remove `_isEligibleSender()` check.

# [G-05] Move out transfers to save gas

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L473-L483

```solidity
collateralToken.safeTransfer(
          addresses.perpetualAtlanticVaultLP,
          (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
            1e18
        );

        IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP)
          .addProceeds(
            (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
              1e18
          );
```

You may move out this from while-loop, because it's cheaper to transfer funds only once, neither do it every time.

# [G-06] Unused variable

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L70

Remove this unused variable.
`address public rdpxRdpxV2Core;`