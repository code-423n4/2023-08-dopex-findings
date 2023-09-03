The elements in bonds are not deleted

"RdpxV2Core.redeem" will burn the token corresponding to the id in bonds.
But it didn't remove the elements from bonds.

It is recommended to delete bonds[id] at redeem after burning token.

```
function redeem(
    uint256 id,
    address to
  ) external returns (uint256 receiptTokenAmount) {
    ......

    RdpxV2Bond(addresses.receiptTokenBonds).burn(id);

    // transfer receipt tokens to user
    receiptTokenAmount = bonds[id].amount;
    IERC20WithBurn(addresses.rdpxV2ReceiptToken).safeTransfer(
      to,
      receiptTokenAmount
    );

    emit LogRedeem(to, receiptTokenAmount);
  }
```