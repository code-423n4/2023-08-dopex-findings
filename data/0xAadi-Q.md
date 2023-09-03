## [L] Checking `MINTER` access control twice in `mint()` function

The `mint()` function performs a double verification of the MINTER access control. This is by through the use of both the `onlyRole(MINTER_ROLE)` modifier and the `require(hasRole(MINTER_ROLE, msg.sender))` statement. Consequently, the message "Caller is not a minter" will never be triggered. The require statement can be removed.

There is 1 instances of this issue:

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L114C3-L125C4

```solidity
  function mint(
    address to,
    uint256 expiry,
    uint256 rdpxAmount
  ) external onlyRole(MINTER_ROLE) {
    _whenNotPaused();
    require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
    uint256 bondId = _mintToken(to);
    bonds[bondId] = Bond(to, expiry, rdpxAmount);


    emit BondMinted(to, bondId, expiry, rdpxAmount);
  }
```