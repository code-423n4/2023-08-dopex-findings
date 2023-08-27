RdpxCoreV2::calculateBondCost

Unless putOptionsRequired flag is true, the below calls are meaningless. So. add an if condition and do the below only if putOptionsRequired is true. This will save some gas.


``` Modified code
  if (putOptionsRequired) {
    uint256 strike = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
      .roundUp(rdpxPrice - (rdpxPrice / 4)); // 25% below the current price

    uint256 timeToExpiry = IPerpetualAtlanticVault(
      addresses.perpetualAtlanticVault
    ).nextFundingPaymentTimestamp() - block.timestamp;
   
      wethRequired += IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
        .calculatePremium(strike, rdpxRequired, timeToExpiry, 0);
    }
```