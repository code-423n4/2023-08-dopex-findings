QA1. Some ERC20 tokens do not support allowance approval from non-zero to non-zero. So the following RdpxV2Core.approveContractToSpend() code will not work. 

[https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/core/RdpxV2Core.sol#L403-L412](https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/core/RdpxV2Core.sol#L403-L412)

Correction: approve to zero first before approving to the designated ``amount`` for a safe approval:

```diff
function approveContractToSpend(
    address _token,
    address _spender,
    uint256 _amount
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_token != address(0), 17);
    _validate(_spender != address(0), 17);
    _validate(_amount > 0, 17);

+     uint256 allowance = IERC20WithBurn(_token).allowance(msg.sender, _spender);
+    if(allowance != 0) IERC20WithBurn(_token).approve(_spender, 0);


    IERC20WithBurn(_token).approve(_spender, _amount);
  }

```

QA2. Possible divide by zero error in ``perpetualAtlanticVault._updateFundingRate()`` due to lack of check of wether ``endTime == startTime`` for the case of ``fundingRates[latestFundingPaymentPointer] == 0``.

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L594-L614](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L594-L614)

Mitigation: we need to check ``endTime == startTime`` for the case of ``fundingRates[latestFundingPaymentPointer] == 0`` as well.

```diff
 function _updateFundingRate(uint256 amount) private {
    if (fundingRates[latestFundingPaymentPointer] == 0) {
      uint256 startTime;
      if (lastUpdateTime > nextFundingPaymentTimestamp() - fundingDuration) {
        startTime = lastUpdateTime;
      } else {
        startTime = nextFundingPaymentTimestamp() - fundingDuration;
      }
      uint256 endTime = nextFundingPaymentTimestamp();
+      if (endTime == startTime) return;
      fundingRates[latestFundingPaymentPointer] =
        (amount * 1e18) /
        (endTime - startTime);
    } else {
      uint256 startTime = lastUpdateTime;
      uint256 endTime = nextFundingPaymentTimestamp();
      if (endTime == startTime) return;
      fundingRates[latestFundingPaymentPointer] =
        fundingRates[latestFundingPaymentPointer] +
        ((amount * 1e18) / (endTime - startTime));
    }
  }
```