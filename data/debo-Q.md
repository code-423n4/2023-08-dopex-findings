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

## [L-03] Arithmetic Underflow and Overflow
```txt
I've identified an instance of integer overflow within some functions. 
This flaw enables me to manipulate the uint256 value in a way that results in its reduction to zero. Consequently, I can exploit this situation to make a transfer exceeding my actual balance.
I've discovered an integer underflow within the some functions. 
This vulnerability enables me to manipulate the uint256 value in a manner that causes it to wrap around to the maximum value. 
As a result, I can exploit this situation to withdraw an amount greater than what I actually possess.
```
## Proof
```sol
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L270
    uint256 strike = roundUp(currentPrice - (currentPrice / 4)); // 25% below the current price
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L283
    uint256 timeToExpiry = nextFundingPaymentTimestamp() - block.timestamp;
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L427
        (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration));
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L468
          ? (nextFundingPaymentTimestamp() - fundingDuration)
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L475
          (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L481-L482
            (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
              1e18
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L487-L488
          ((currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
            1e18),
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L506
      ? (nextFundingPaymentTimestamp() - fundingDuration)
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L512
      (currentFundingRate * (block.timestamp - startTime)) / 1e18
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L516
      (currentFundingRate * (block.timestamp - startTime)) / 1e18
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L521
      ((currentFundingRate * (block.timestamp - startTime)) / 1e18),
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L559
    return strike > price ? ((strike - price) * amount) / 1e8 : 0;
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L581
      return _strike - remainder + roundingPrecision;
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L597
      if (lastUpdateTime > nextFundingPaymentTimestamp() - fundingDuration) {
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L600
        startTime = nextFundingPaymentTimestamp() - fundingDuration;
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L605
        (endTime - startTime);
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L612
        ((amount * 1e18) / (endTime - startTime));
```
## Tools Used
Manual Review.
## Recommendation
Use safe math.