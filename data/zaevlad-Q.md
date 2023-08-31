`decreaseAmount` in the RdpxDecayingBonds contract is used when admin plan to decreases the bond amount:

```
  function decreaseAmount(
    uint256 bondId,
    uint256 amount
  ) public onlyRole(RDPXV2CORE_ROLE) {
    _whenNotPaused();
    bonds[bondId].rdpxAmount = amount;
  }
```

However there is no check that amount is lower than previous one and it can be set up to a higher amount undesirably. 

Consider providing a check that the amount is lower than previuos one or rename the functions to, for example, `changeAmount()` if it is planned to change the amount at any direction.