## 1. No zero Check in PerpetualAtlanticVault.updateFundingDuration
```
function updateFundingDuration(
    uint256 _fundingDuration
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    fundingDuration = _fundingDuration;
  }
```
Consider adding a zero check in this function to prevent a denial of service when admins erroneously changes the funding duration to zero.