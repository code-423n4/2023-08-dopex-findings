### Funding payment mechanism is at risk of being delayed or missed by ‘growing pains’ of the market
The funding payment mechanism is at the core of PerpetualAtlanticVault.sol. The payment amount and funding rates are supposed to be updated at every epoch. The risk here is the volume of state changes required to bring funding payments and funding rates up to date at every epoch might grow too extensive to function promptly within a short period of time relative to the contract lifespan. 

Although in the context of static analysis the mechanism functions as intended, when taking into account of a longer timespan where the protocol grows and gains more traction, the state changes in every epoch might grow exponentially as more users deposit and purchase options into the vault. 

(1) The underlying issue:
The underlying issue might be the accounting structure which relies on the accounting of individual strike prices in PerpetualAtlanticVault.sol. There are multiple state mapping variables using individual strike prices as key such as `fundingPaymentsAccountedForPerStrike` , `optionsPerStrike` , `latestFundingPerStrike`.

(2) Observation walkthrough:
Looking at two key functions `calculateFunding()` and `payFunding()` on PerpetualAtlanticVault.sol, every single strike needs to be accounted for in every epoch for funding payments to be sent to LP. 

Specifically in `calculateFunding()`  a public function can be called by anyone, strikes are sent as an array and then iterated inside the function in a for loop to update states related to each strike price passed. There are two main hurdles I see here: 

- The strike passed needs to be valid, or the whole transaction will revert. If a user or anyone from the protocol passes through one invalid strike price where optionsPerStrike[strikes[i]]>0 validation is violated, the caller would have to inspect each strike price, pinpoint the failed strike price, and resubmit the whole transaction. This is potentially a  labor-intensive process when done manually or at least an unpleasant user experience if called by a user. (See an attack vector detailed in one of my medium issue report)

  In addition, this might still happen when the correct strike prices are submitted. The caller might have submitted the correct strike price array to `calculateFunding()`, but still has their transaction reverted due to `settle()` which settles options and deduct amount from `optionsPerStrike` . The `settle()` transaction might be submitted after `calculateFunding()` but settled before `calculateFunding()` -  perhaps due to gas forwarded, in which case, a previously valid strike price might become invalid when `calculateFunding()` settles. To avoid this scenario, `settle()` and `calculteFundinng()` need to be coordinated. However, the coordination cannot be guaranteed due to `calcualteFunding()` is intended to be not access controlled and can be contributed by anyone.

  Recommendation: Consider removing `_validate(optionsPerStrike[strikes[i]] > 0, 4);` and even `_validate(latestFundingPerStrike[strikes[i]] != latestFundingPaymentPointer, 5)`. Instead of revert, simply add checks on these two conditions, skip the instance, and continue to the next iteration.  Allowing skip and continue instead of revert in the for loop, mitigates the edge cases.

- There might be too many strike prices to update and the effort becomes extensive and exhausting, in which case funding payments for a certain epoch might be missed. This can happen either due to malicious user dust bonds or due to the natural growth of the market overtime.

  For the first scenario, I detailed the attack vector of dust bonds in one of my high issue report. 

  For the second scenario, it's harder to mitigate and prevent. Since strike prices organically fluctuate, the longer the market runs, the more diverse active strike prices are, one can argue there will be an eventual breaking point where there are too many strike prices to be updated in `calculateFunding()` and funding payments will be missed or greatly reduced from then on. 

(3) Recommendations:
Apart from the recommendation mentioned above to mitigate for loop reverts, more efforts seem needed to mitigate the second hurdle. The dependency on strike prices, in my opinion, boils down to how funding payments are calculated.

`calculatePremium()` uses both strike price and current price to calculate the premium , which then got added to `totalFundingForEpoch[latestFundingPaymentPointer] += premium;` which converts to funding rate for a given epoch. It might be the protocol's discretion to see if optimization can be made in calculating funding payments for options purchased in older epochs in a way that doesn't require `calculatePremium()`. 





### Time spent:
30 hours