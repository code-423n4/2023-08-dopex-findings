### [GAS-01] Calculate premium only if the `putOptionRequired` 
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

