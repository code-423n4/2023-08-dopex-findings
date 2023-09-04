# Hardcoded strings to reference mapping values

## Impact
L-01: The use of magic numbers and hardcoded strings is spread out in several contracts, making it error-prone.
In RdpxV2Core.sol:657, the usage of magic numbers and hardcoded strings in several places. This can lead to potential issues and make the code difficult to maintain and understand.

Tools Used
Manual Review

## Mitigation
To mitigate this issue, it is recommended to replace the magic numbers and hardcoded strings with constants that are named in a way that clearly expresses their intended use. For example:

```solidity
string constant public RESERVE_INDEX_RDPX = "RDPX";
```
By using constants like RESERVE_INDEX_RDPX, the code becomes more readable and less error-prone. The constant can be used to reference the item in the reservesIndex mapping, as shown in the code snippet.

```solidity
IERC20WithBurn(reserveAsset[reservesIndex[RESERVE_INDEX_RDPX]].tokenAddress).burn(
    (_rdpxAmount * rdpxBurnPercentage) / 1e10
);
```
This improves code maintainability and reduces the risk of introducing bugs due to the use of magic numbers and hardcoded strings.


# Magic Numbers Usage in Funding Rate Calculation

## Impact
In `PerpetualAtlanticVault.sol:527`, the contract uses magic numbers to calculate the funding rate. This can lead to potential errors and make the code less maintainable.

## Proof of Concept
The vulnerability can be observed in the following code snippet from the contract:

```solidity
function _updateFundingRate(uint256 amount) private {
    if (fundingRates[latestFundingPaymentPointer] == 0) {
        uint256 startTime;
        if (lastUpdateTime > nextFundingPaymentTimestamp() - fundingDuration) {
            startTime = lastUpdateTime;
        } else {
            startTime = nextFundingPaymentTimestamp() - fundingDuration;
        }
        uint256 endTime = nextFundingPaymentTimestamp();
        fundingRates[latestFundingPaymentPointer] = (amount * 1e18) / (endTime - startTime); // @audit-issue L-02 Using magic numbers to calculate funding rate
    } else {
        uint256 startTime = lastUpdateTime;
        uint256 endTime = nextFundingPaymentTimestamp();
        if (endTime == startTime) return;
        fundingRates[latestFundingPaymentPointer] =
            fundingRates[latestFundingPaymentPointer] + ((amount * 1e18) / (endTime - startTime));
    }
}
```

## Tools Used
Manual Review

## Recommended Mitigation Steps
Replace the magic number in the funding rate calculation with named constants or variables that clearly express their intended purpose. This will make the code more readable and reduce the risk of errors.

## Additional Info
The issue can be seen in other places of the `PerpetualAtlanticVault` contract such as the functions:
- updateFundingPaymentPointer
- updateFunding
- calculatePremium

## Issue Type
Decimal