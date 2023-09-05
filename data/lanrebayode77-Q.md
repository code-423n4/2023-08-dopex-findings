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


## 4. updateFundingDuration() should have a zero check.
```fundingDuration``` is a vital state in the contract and extra caution must be taking when setting it, like preventing it from being set to zero. Add a zero check to the updateFundingDuration() function.
```
  function updateFundingDuration(
    uint256 _fundingDuration
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_fundingDuration > 0, 1);
    fundingDuration = _fundingDuration;
  }
```

## 5. No check for delegateId validity in RdpxV2Core.getDelegatePosition().
RdpxV2Core.getDelegatePosition() is a public function that returns the details of a delegate position.

However, it missis an important check to validate the existence of the delegate position, so if a delegated that does not exist is passed, it will return default values instead of reverting with a message. 

Add a validity check in the function.
```
_validate(_delegateId < delegates.length, 14);
 Delegate memory delegatePosition = delegates[_delegateId];
    return (
      delegatePosition.owner,
      delegatePosition.amount,
      delegatePosition.fee,
      delegatePosition.activeCollateral
    );
  }
```

## 6. Misleading natspec comment.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1047
The comment states: "* @notice Lets users mint DpxEth at a 1:1 ratio when DpxEth pegs above 1.01 of the ETH token", However, the implementation restricts the function to admin role. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1054
```
 function upperDepeg(
    uint256 _amount,
    uint256 minOut
  ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256 wethReceived) {
```
Natspec comment should describe exactly what the function is doing, if the function is meant to be called by only admin, the comment should state: "* @notice Lets admin* mint DpxEth at a 1:1 ratio when DpxEth pegs above 1.01 of the ETH token"
