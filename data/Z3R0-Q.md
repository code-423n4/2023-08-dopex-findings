### Improper accounting of delegated WETH on withdrawals

`totalWethDelegated` is used to keep track of delegated WETH across the RdpxV2Core contract. Delegators call `addToDelegate()` to provide WETH, on execution - the amount of WETH deposited by the caller is added to `totalWethDelegate` as shown below:

```solidity
    // add amount to total weth delegated
    totalWethDelegated += _amount;
```

In `withdraw()`, the amount of WETH withdrawn by a previous delegator is not subtracted from `totalWethDelegated`:

```solidity
  function withdraw(
    uint256 delegateId
  ) external returns (uint256 amountWithdrawn) {
    _whenNotPaused();
    _validate(delegateId < delegates.length, 14);
    Delegate storage delegate = delegates[delegateId];
    _validate(delegate.owner == msg.sender, 9);

    amountWithdrawn = delegate.amount - delegate.activeCollateral;
    _validate(amountWithdrawn > 0, 15);
    delegate.amount = delegate.activeCollateral;

    IERC20WithBurn(weth).safeTransfer(msg.sender, amountWithdrawn);

    emit LogDelegateWithdraw(delegateId, amountWithdrawn);
  }
```

## Impact
Since this variable is not used in critical accounting operations across the RdpxV2Core contract, it's impact is Low/Informational.

## Recommendation
Add the following line to `withdraw()`:

```solidity
    // subtract amount withdrawn by delegate
    totalWethDelegated -= amountWithdrawn;
```