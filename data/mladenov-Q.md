# [L-01] Double check for MINTER_ROLE

### Description

[mint](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L120) function includes double check for MINTER_ROLE.
There is no need to use requre statement since there is attached onlyRole modifier to the function.

Under the hood **onlyRole** modifier uses **hasRole** function which checks if user has specific role.

### Proof Of Concept

RdpxDecayingBonds.sol

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L120

```solidity
function mint(
    address to,
    uint256 expiry,
    uint256 rdpxAmount
  ) external onlyRole(MINTER_ROLE) {
    _whenNotPaused();
    require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter"); <--- second check
    uint256 bondId = _mintToken(to);
    bonds[bondId] = Bond(to, expiry, rdpxAmount);

    emit BondMinted(to, bondId, expiry, rdpxAmount);
  }
```

[**onlyRole**](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/AccessControl.sol#L63) modifier calls **_checkRole** function which uses **hasRole** function to check if user has the role.

### Tools used
Manual review

### Recommended Mitigation Steps

Remove require statement

```solidity
function mint(
    address to,
    uint256 expiry,
    uint256 rdpxAmount
  ) external onlyRole(MINTER_ROLE) {
    _whenNotPaused();
    require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter"); <--- remove this line
    uint256 bondId = _mintToken(to);
    bonds[bondId] = Bond(to, expiry, rdpxAmount);

    emit BondMinted(to, bondId, expiry, rdpxAmount);
  }
```
