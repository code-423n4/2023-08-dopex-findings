
L-01: rdpxBurnPercentage + rdpxFeePercentage Could be more than 100%

The current protocol requires `rdpxBurnPercentage + rdpxFeePercentage = 100%`. 
But it is set separately, and there is no check on what the sum of the two is, which may add up to more or less than 100%

Suggest removing `setRdpxBurnPercentage()` and `setRdpxFeePercentage()` and combining them into one method and checking that the sum needs to be 100%.


```solidity
- function setRdpxBurnPercentage(
-    uint256 _rdpxBurnPercentage
-  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
-    _validate(_rdpxBurnPercentage > 0, 3);
-    rdpxBurnPercentage = _rdpxBurnPercentage;
-    emit LogSetRdpxBurnPercentage(_rdpxBurnPercentage);
-  }

-  function setRdpxFeePercentage(
-    uint256 _rdpxFeePercentage
-  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
-    _validate(_rdpxFeePercentage > 0, 3);
-    rdpxFeePercentage = _rdpxFeePercentage;
-    emit LogSetRdpxFeePercentage(_rdpxFeePercentage);
-  }



+  function setRdpxBurnAndFeePercentage(
+    uint256 _rdpxBurnPercentage,
+    uint256 _rdpxFeePercentage
+  )external onlyRole(DEFAULT_ADMIN_ROLE) {
+    _validate(_rdpxBurnPercentage > 0, 3);
+    rdpxBurnPercentage = _rdpxBurnPercentage;
+    emit LogSetRdpxBurnPercentage(_rdpxBurnPercentage);

+    _validate(_rdpxFeePercentage > 0, 3);
+    rdpxFeePercentage = _rdpxFeePercentage;
+    emit LogSetRdpxFeePercentage(_rdpxFeePercentage);    

+    require(_rdpxBurnPercentage + _rdpxFeePercentage == 1e10,"invalid input");
+  }   
```



L-02: payFunding() Without clearing totalFundingForEpoch[], a malicious `RDPXV2CORE_ROLE` can repeatedly execute transfer `weth` to `PerpetualAtlanticVault`.

Currently `payFunding()` does not clear `totalFundingForEpoch[latestFundingPaymentPointer]` upon execution
If there is another address with `RDPXV2CORE_ROLE` privilege, this method can be executed repeatedly for token transfer.
It is recommended to clear `totalFundingForEpoch[latestFundingPaymentPointer]` after execution.

```solidity
  function payFunding() external onlyRole(RDPXV2CORE_ROLE) returns (uint256) {
    _whenNotPaused();
    _isEligibleSender();

    _validate(
      totalActiveOptions ==
        fundingPaymentsAccountedFor[latestFundingPaymentPointer],
      6
    );

    collateralToken.safeTransferFrom(
      addresses.rdpxV2Core,
      address(this),
      totalFundingForEpoch[latestFundingPaymentPointer]
    );
    _updateFundingRate(totalFundingForEpoch[latestFundingPaymentPointer]);

    emit PayFunding(
      msg.sender,
      totalFundingForEpoch[latestFundingPaymentPointer],
      latestFundingPaymentPointer
    );
-   return (totalFundingForEpoch[latestFundingPaymentPointer]);    
+   uint256 result = totalFundingForEpoch[latestFundingPaymentPointer];
+   delete totalFundingForEpoch[latestFundingPaymentPointer];
+   return result;
  }

```


L-03: removeLiquidity() emit log Use the cleared positions_mapping[pos.token_id].token_id

`removeLiquidity()` Use already cleared `positions_mapping[pos.token_id].token_id`
It is recommended to emit before clearing

```diff
  function removeLiquidity(
    uint256 positionIndex,
    uint256 minAmount0,
    uint256 minAmount1
  ) public onlyRole(DEFAULT_ADMIN_ROLE) {
...

    positions_array[positionIndex] = positions_array[
      positions_array.length - 1
    ];
    positions_array.pop();
+   emit log(positions_mapping[pos.token_id].token_id);    
    delete positions_mapping[pos.token_id];

    // send tokens to rdpxV2Core
    _sendTokensToRdpxV2Core(tokenA, tokenB);

    emit log(positions_array.length);
-   emit log(positions_mapping[pos.token_id].token_id);
  }  
```

L-03: UniV3LiquidityAMO.recoverERC20() Missing Trigger sync()

in `UniV3LiquidityAMO.recoverERC20()` The `token` is transferred to `rdpxV2Core`, without triggering `sync()`.

```diff
  function recoverERC20(
    address tokenAddress,
    uint256 tokenAmount
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    // Can only be triggered by owner or governance, not custodian
    // Tokens are sent to the custodian, as a sort of safeguard
    TransferHelper.safeTransfer(tokenAddress, rdpxV2Core, tokenAmount);
+   rdpxV2Core.sync();

    emit RecoveredERC20(tokenAddress, tokenAmount);
  }
```


L-04: bondWithDelegate() There is a risk of blockchain reorganization

`bondWithDelegate()` passes in `_delegateIds[]`, and the Fee is different for each delegate
If the blockchain is reorganized, the user can use the unintended Fee.

Example.
delegate[10].fee = 1%

user submit `bondWithDelegate(_delegateIds[10]) `


Due to reorganization, some other user submitted `addToDelegate(fee=20%)`.
His ID became 10 i.e.
delegate[10].fee = 20%
delegate[11].fee = 1%  (old)

 This way when executing `bondWithDelegate(_delegateIds[10]) ` again, the user may get unexpected fees (20%)

 Suggest to add fees for checking
