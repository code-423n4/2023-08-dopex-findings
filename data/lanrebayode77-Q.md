## 1. No zero Check in PerpetualAtlanticVault.updateFundingDuration
```
function updateFundingDuration(
    uint256 _fundingDuration
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    fundingDuration = _fundingDuration;
  }
```
Consider adding a zero check in this function to prevent a denial of service when admins erroneously changes the funding duration to zero.

## 2. State Update before Validiation Check
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L710-L717
```
 // update ETH token reserve
    reserveAsset[reservesIndex["WETH"]].tokenBalance += wethRequired;

    Delegate storage delegate = delegates[delegateId];

    // update delegate active collateral
    _validate(delegate.amount - delegate.activeCollateral >= wethRequired, 5);
    delegate.activeCollateral += wethRequired;
```
In RdpxV2Core._bondWithDelegate(), the validation check that delegate has enough available weth should come before changing the state of reserveAsset.


## 3. Ensure Genesis time is set to the future.
No Validation to ensure that the ```genesis``` time was set to the future from the constructor
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L124
Add a validation check to ensure genesis is never in the past.
```
_validate(_genesis >= block.timestamp, "can't be in the past");
genesis = _gensis;
```