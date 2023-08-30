## [L-01] Block values as a proxy for times
In the case of block.timestamp, developers often attempt to use it to trigger time-dependent events. As Ethereum is decentralized, nodes can synchronize time only to some degree. Moreover, malicious miners can alter the timestamp of their blocks, especially if they can gain advantages by doing so. However, miners can't set a timestamp smaller than the previous one (otherwise the block will be rejected), nor can they set the timestamp too far ahead in the future. Taking all of the above into consideration, developers can't rely on the preciseness of the provided timestamp.

## Proof
```sol
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L231
        block.timestamp + 1
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L279
        block.timestamp + 1
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L342
        block.timestamp + 1
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L250
block.timestamp
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L502
      maturity: block.timestamp + bondMaturity,
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L503
      timestamp: block.timestamp
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L636
      _validate(expiry >= block.timestamp, 2);
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1023
    _validate(block.timestamp > bonds[id].maturity, 7);
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1103
          block.timestamp + 10
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1194
    ).nextFundingPaymentTimestamp() - block.timestamp;
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L283
    uint256 timeToExpiry = nextFundingPaymentTimestamp() - block.timestamp;
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L463
    while (block.timestamp >= nextFundingPaymentTimestamp()) {
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L508
    lastUpdateTime = block.timestamp;
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L512
      (currentFundingRate * (block.timestamp - startTime)) / 1e18
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L516
      (currentFundingRate * (block.timestamp - startTime)) / 1e18
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L521
      ((currentFundingRate * (block.timestamp - startTime)) / 1e18),
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L264
      block.timestamp + 10
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L283
        block.timestamp + 10
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L294
      block.timestamp + 10
```
## Tools Used
Manual Review.
## Recommendation
Use Oracles.

## [L-02] Out of bounds array access
The function called setLpAllowance posses an out of bounds array access.
So, the value in the function does not fall within the max length of the array.
## Proof
```sol
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L243-L250
  function setLpAllowance(bool increase) external onlyRole(DEFAULT_ADMIN_ROLE) {
    increase
      ? collateralToken.approve(
        addresses.perpetualAtlanticVaultLP,
        type(uint256).max
      )
      : collateralToken.approve(addresses.perpetualAtlanticVaultLP, 0);
  }
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L346-L351
    // Transfer collateral token from perpetual vault to rdpx rdpxV2Core
    collateralToken.safeTransferFrom(
      addresses.perpetualAtlanticVaultLP,
      addresses.rdpxV2Core,
      ethAmount
    );
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L429-L437
      uint256 premium = calculatePremium(
        strike,
        amount,
        timeToExpiry,
        getUnderlyingPrice()
      );


      latestFundingPerStrike[strike] = latestFundingPaymentPointer;
      fundingAmount += premium;
```
## Tools Used
Manual Review.
## Recommendation
Adjust the code so that all values fall within the range.