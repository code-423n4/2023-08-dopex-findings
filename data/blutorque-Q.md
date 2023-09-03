### [NC-01] `RdpxDecayingBonds#decreaseAmount` not necessarily decreases the bond amount

`RdpxDecayingBonds#decreaseAmount` is called by the `RdpxV2Core` to decrease the bond amount for a bondId. The code below doesn't include any logic to decrease the `rdpxAmount` in the `bonds` mapping; instead, it sets the `rdpxAmount` to a new value. 

The function naming wrongly suggest its functioning, consider changing that.

```solidity
  function decreaseAmount(
    uint256 bondId,
    uint256 amount
  ) public onlyRole(RDPXV2CORE_ROLE) {
    _whenNotPaused();
    bonds[bondId].rdpxAmount = amount;
  }
```
**Recommendation**
Change the function name to `setRdpxAmount`
