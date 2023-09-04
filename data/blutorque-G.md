### [G-01] Calculate premium only if the `putOptionRequired` 
RdpxV2Core#calculateBondCost() is used to calculate the cost of bonding, which returns the rdpx and weth required corresponding to given bonding amount. If the `putOptionRequired` check true, we add the premium(in weth) to the `wethRequired`. 

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1195-L1198

There can be saved some gas by bounding the strike price and timeToExpiry calculation within the `putOptionRequired` check, as this allow us to calculate the parameters if and only if the `putOptionRequired` is true.

**Recommended**
Change to below.

```solidity
  function calculateBondCost(
    uint256 _amount,
    uint256 _rdpxBondId
  ) public view returns (uint256 rdpxRequired, uint256 wethRequired) {
    // ...SNIP...
    if (putOptionsRequired) {
        uint256 strike = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
        .roundUp(rdpxPrice - (rdpxPrice / 4)); // 25% below the current price

        uint256 timeToExpiry = IPerpetualAtlanticVault(
        addresses.perpetualAtlanticVault
        ).nextFundingPaymentTimestamp() - block.timestamp;
    
        wethRequired += IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
            .calculatePremium(strike, rdpxRequired, timeToExpiry, 0);
   }
}
```
### [G-02] MINTER_ROLE validate twice against caller

RdpxDecayingBonds#mint() is used to mints decaying $rDPX bonds, called only by those who have assigned MINTER_ROLE. 

The `onlyRole(MINTER_ROLE)` modifier ensures the caller can only be the minter. However, it re-check the same validation in the second line of mint function which is totally redundant. 

```solidity 
  function mint(
    address to,
    uint256 expiry,
    uint256 rdpxAmount
  ) external onlyRole(MINTER_ROLE) {
    _whenNotPaused();
    require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter"); // Redundant check
    uint256 bondId = _mintToken(to);
    bonds[bondId] = Bond(to, expiry, rdpxAmount);

    emit BondMinted(to, bondId, expiry, rdpxAmount);
  }
```

**Recommended**
Remove the extra validation to saves the gas. 


### [G-03] Cache the storage variable used in loops
Its a helper function that updates the latest funding payment pointer based on current timestamp, it uses loops which run over n over until the current timestamp eq `nextFundingPaymentTimestamp()`. Caching the reused expression inside the loop can really saves the large sum of gas. 

The function fetch the `emissionRate` and multiply it with the `timeElasped` since the last update, which returns the collateral amount that need to be add to the lpVault. Currently, the `updateFundingPaymentPointer()` computes collateralAmount multiple times by itself, which means the accessing storage slots also multiple times.

The calculated collateral amount is `(currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) / 1e18`

```solidity
  function nextFundingPaymentTimestamp()
    public
    view
    returns (uint256 timestamp)
  {
    return genesis + (latestFundingPaymentPointer * fundingDuration);
  }

```

SLOADs(100 gas after the 1st one) are expensive compared to the MLOADs(3 gas).
- you can save 3 SLOADs per computation of collateral amount,
- each computation is called thrice in single loop, means we can avoid 2 extra computation by caching 1 in local variable


**Recommended**

```solidity
  function updateFundingPaymentPointer() public {
    while (block.timestamp >= nextFundingPaymentTimestamp()) {
      if (lastUpdateTime < nextFundingPaymentTimestamp()) {
        uint256 currentFundingRate = fundingRates[latestFundingPaymentPointer];

        uint256 startTime = lastUpdateTime == 0
          ? (nextFundingPaymentTimestamp() - fundingDuration)
          : lastUpdateTime;

        lastUpdateTime = nextFundingPaymentTimestamp();

+       uint256 collateralAmount = (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) / 1e18;

        collateralToken.safeTransfer(
          addresses.perpetualAtlanticVaultLP,
+         collateralAmount
-         (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
            1e18
        );

        IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP)
          .addProceeds(
+           collateralAmount
-           (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
              1e18
          );

        emit FundingPaid(
          msg.sender,
+         collateralAmount,
-         ((currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
            1e18),
          latestFundingPaymentPointer
        );
      }

      latestFundingPaymentPointer += 1;
      emit FundingPaymentPointerUpdated(latestFundingPaymentPointer);
    }
  }
```

```solidity
  function updateFunding() public {
    updateFundingPaymentPointer();
    uint256 currentFundingRate = fundingRates[latestFundingPaymentPointer];
    uint256 startTime = lastUpdateTime == 0
      ? (nextFundingPaymentTimestamp() - fundingDuration)
      : lastUpdateTime;
    lastUpdateTime = block.timestamp;

+   uint256 collateralAmount = (currentFundingRate * (block.timestamp - startTime)) / 1e18;

    collateralToken.safeTransfer(
      addresses.perpetualAtlanticVaultLP,
+     collateralAmount
-     (currentFundingRate * (block.timestamp - startTime)) / 1e18
    );

    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addProceeds(
+     collateralAmount
-     (currentFundingRate * (block.timestamp - startTime)) / 1e18
    );

    emit FundingPaid(
      msg.sender,
+     collateralAmount,
-     ((currentFundingRate * (block.timestamp - startTime)) / 1e18),
      latestFundingPaymentPointer
    );
  }
```


