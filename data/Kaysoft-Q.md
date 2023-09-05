## [NC-01] `approveContractToSpend` will return wrong error message when `_amount` does not pass the validation check.
File: https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L410

- Wrong error code was supplied to the third validation in this code. The right error code that is supposed to be passed is `4` instead of `17`

```
function approveContractToSpend(
    address _token,
    address _spender,
    uint256 _amount
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_token != address(0), 17);
    _validate(_spender != address(0), 17);
@>    _validate(_amount > 0, 17);//@audit-info 17 error code wrong.
    IERC20WithBurn(_token).approve(_spender, _amount);
  }
```
```
// ERROR CODES
// E1: "Insufficient bond amount",
// E2: "Bond has expired",
// E3: "Invalid parameters",
// E4: "Invalid amount",
// E5: "Not enough ETH available from the delegate",
// E6: "Invalid bond ID",
// E7: "Bond has not matured",
// E8: "Fee cannot be more than 100",
// E9: "msg.sender is not the owner",
// E10: "Price is not above the upper peg",
// E11: "Amount greater that max permissible amount",
// E12: "Price is not lower than first lower peg",
// E13: "Amount greater than max permissible amount"
// E14: "Invalid delegate Id"
// E15: "Invalid amount"
// E16: "Funding already paid for the epoch"
// E17: "Zero address"
// E18: "Asset not found"
// E19: "Token not found"
```

#### Recommended Mitigation Steps
Replace the `17` on the `amount` validation here with `4` like this.
```
function approveContractToSpend(
    address _token,
    address _spender,
    uint256 _amount
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_token != address(0), 17);
    _validate(_spender != address(0), 17);
-   _validate(_amount > 0, 17);//@audit-info 17 error code wrong.
+   _validate(_amount > 0, 4);
    IERC20WithBurn(_token).approve(_spender, _amount);
  }
```

## [NC-02] Double access control check for `mint()` function of `RdpxDecayingBonds.sol`.
File: https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L118-L120

The `mint()` function has double access control checks.
1. An `onlyRole(MINTER_ROLE)` modifier access check
2. And a `require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter")` access check

```
function mint(
    address to,
    uint256 expiry,
    uint256 rdpxAmount
  ) external onlyRole(MINTER_ROLE) {
    _whenNotPaused();
    require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");//@audit-info same modifier already used.
    uint256 bondId = _mintToken(to);
    bonds[bondId] = Bond(to, expiry, rdpxAmount);

    emit BondMinted(to, bondId, expiry, rdpxAmount);
  }

```
#### Recommended Mitigation Steps
Remove the require statement to avoid double access control check.

