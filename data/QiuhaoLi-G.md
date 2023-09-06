## We can use a put option for all delegate bonds instead of one for every option

In `bondWithDelegate` we can delecate more than once, and for every delegate we buy the option for the bond. Since all the bonds are in the same transaction, the strike price will be the same. To save gas, we can buy a single option outside the for loop in `bondWithDelegate`. This can save gas in two ways:

1. When bonding with a delegate, we can save gas by only calling _purchaseOptions once.
2. When we settle, instead of calling multiple times on different options, we can do that in one call.

This can save considerable gas if there are many delegates when calling `bondWithDelegate`.


## When delegate.amount = 0, we can delete the slot or reuse it

Currently, we don't reuse the delegate ID (mapping) if a delegate is fully withdrawn by the owner, which brings a linear increase in storage used. This can be optimized to reuse slots or delete the mapping index for gas refund.

## When a bond is withdrawn from RdpxV2Core, we can delete bonds[id] for gas refund

Currently, we don't reuse the bonds[id] or delete them if a user redeem the bond, which brings a linear increase in storage used. This can be optimized to resue the slot or delete the slot for gas refund in redeem().

## Only calculate strike and timeToExpiry if putOptionsRequired

Currently, in calculateBondCost(), we always calculate the strike and timeToExpiry even put options is not required. This can be optimized to only calculated them if putOptionsRequired is set.

## owner field of struct Bond in RdpxDecayingBonds is not used

In contracts/decaying-bonds/RdpxDecayingBonds.sol, the struct Bond has an owner field, but this field is never used. For example, in RdpxV2Core.sol:_transfer, we only use the expiry and amount fields. Deleting this field can reduce storage/gas used.

## RdpxDecayingBonds:mint() has redundant hasRole check

In mint(), we have a onlyRole(MINTER_ROLE) modifier, but then we have a `require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");`. This is redundant and costs more gas.

## delete optionPositions[optionIds[i]] for more gas refund in settle()

Currently, we only set optionPositions[optionIds[i]].strike to zero at the end of settle, leaving amount and positionId unchanged. We can delete optionPositions[optionIds[i]] to clear them also, thus getting more gas refunded.

## Calculate (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) only once in updateFundingPaymentPointer()

In updateFundingPaymentPointer(), we calculated the time spent in one epoch multiple times:

```solidity
        collateralToken.safeTransfer(
          addresses.perpetualAtlanticVaultLP,
          (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) / // @audit-issue GAS this can be calculated only once
            1e18
        );

        IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP)
          .addProceeds(
            (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
              1e18
          );

        emit FundingPaid(
          msg.sender,
          ((currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
            1e18),
          latestFundingPaymentPointer
        );
```

The gas can be saved by calculating (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) only once.

## call nextFundingPaymentTimestamp only once in _updateFundingRate()

In _updateFundingRate(), we use nextFundingPaymentTimestamp()  multiple times. Since the latestFundingPaymentPointer and fundingDuration is not changed during the call, we can read the result to a local variable and use it later to save gas.

