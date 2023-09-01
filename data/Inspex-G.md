## Gas Issues

| Number | Issues  | Instances |
| ------ | ------- | --------- |
| [G-01] |  Calculate strike and timetoExpiry only if putOptionsRequired value is true | 1         |

### [G-01] Calculate strike and timetoExpiry only if putOptionsRequired value is true

#### Line References

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1189-L1198

#### Description
The `strike` and `timeToExpiry` are local variables in the `calculateBondCost()` function. These variables are only used when `putOptionsRequired` is `true`. So if the `putOptionsRequired` is `false` this function call would be wasting gas to compute the value of `strike` and `timeToExpiry`.

**RdpxV2Core.sol**

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1156-L1199

```solidity=1156
function calculateBondCost(
  uint256 _amount,
  uint256 _rdpxBondId
) public view returns (uint256 rdpxRequired, uint256 wethRequired) {
  uint256 rdpxPrice = getRdpxPrice();

  if (_rdpxBondId == 0) {
    uint256 bondDiscount = (bondDiscountFactor *
      Math.sqrt(IRdpxReserve(addresses.rdpxReserve).rdpxReserve()) *
      1e2) / (Math.sqrt(1e18)); // 1e8 precision

    _validate(bondDiscount < 100e8, 14);

    rdpxRequired =
      ((RDPX_RATIO_PERCENTAGE - (bondDiscount / 2)) *
        _amount *
        DEFAULT_PRECISION) /
      (DEFAULT_PRECISION * rdpxPrice * 1e2);

    wethRequired =
      ((ETH_RATIO_PERCENTAGE - (bondDiscount / 2)) * _amount) /
      (DEFAULT_PRECISION * 1e2);
  } else {
    // if rdpx decaying bonds are being used there is no discount
    rdpxRequired =
      (RDPX_RATIO_PERCENTAGE * _amount * DEFAULT_PRECISION) /
      (DEFAULT_PRECISION * rdpxPrice * 1e2);

    wethRequired =
      (ETH_RATIO_PERCENTAGE * _amount) /
      (DEFAULT_PRECISION * 1e2);
  }

  uint256 strike = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
    .roundUp(rdpxPrice - (rdpxPrice / 4)); // 25% below the current price

  uint256 timeToExpiry = IPerpetualAtlanticVault(
    addresses.perpetualAtlanticVault
  ).nextFundingPaymentTimestamp() - block.timestamp;
  if (putOptionsRequired) {
    wethRequired += IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
      .calculatePremium(strike, rdpxRequired, timeToExpiry, 0);
  }
}
```

#### Recommended Mitigation Steps
We suggest computing the `strike` and `timeToExpiry` only if `putOptionsRequired` value is `true`. For example:

**RdpxV2Core.sol**

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1189-L1198

```diff=1189
+ if (putOptionsRequired) {
  uint256 strike = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
    .roundUp(rdpxPrice - (rdpxPrice / 4)); // 25% below the current price

  uint256 timeToExpiry = IPerpetualAtlanticVault(
    addresses.perpetualAtlanticVault
  ).nextFundingPaymentTimestamp() - block.timestamp;
- if (putOptionsRequired) {
    wethRequired += IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
      .calculatePremium(strike, rdpxRequired, timeToExpiry, 0);
  }
```