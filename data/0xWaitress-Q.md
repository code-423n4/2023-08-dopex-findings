[L-1] there is no validation that rdpxBurnPercentage and rdpxFeePercentage adds up to 100%.

rdpxBurnPercentage and rdpxFeePercentage represents the two destinations of RDPX during transfer, 1 is burnt and 1 is transferred to the fee dsitributor. however they can be set separately and arbitrarily. 

```solidity
  /// @notice The % of rdpx to burn while bonding
  uint256 public rdpxBurnPercentage = 50 * DEFAULT_PRECISION;

  /// @notice The % of rdpx sent to fee distributor while bonding
  uint256 public rdpxFeePercentage = 50 * DEFAULT_PRECISION;

...

  function setRdpxBurnPercentage(
    uint256 _rdpxBurnPercentage
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_rdpxBurnPercentage > 0, 3);
    rdpxBurnPercentage = _rdpxBurnPercentage;
    emit LogSetRdpxBurnPercentage(_rdpxBurnPercentage);
  }

  /**
   * @notice Sets the rdpx fee percentage
   * @dev    Can only be called by admin
   * @param  _rdpxFeePercentage the fee percentage to set in 1e8 precision
   **/
  function setRdpxFeePercentage(
    uint256 _rdpxFeePercentage
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_rdpxFeePercentage > 0, 3);
    rdpxFeePercentage = _rdpxFeePercentage;
    emit LogSetRdpxFeePercentage(_rdpxFeePercentage);
  }

```

Recommendation:
combine them into a setter and the other one gets updated automically.
