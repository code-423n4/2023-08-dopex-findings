`totalWethDelegated` is not updated when user withdraws delegated WETH from contract.


```
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