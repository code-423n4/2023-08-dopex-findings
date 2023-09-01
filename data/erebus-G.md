# Gas
## [Gas-01] Redundant code
In [RdpxDecayingBonds::mint](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L114C1-L125C4), there is the modifier `onlyRole` which calls `hasRole(MINTER_ROLE,  _msgSender())` under the hood, which is called again in [line 120](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L120). Remove that, as it is the job of the modifier to check the role of the caller.

```
  function mint(
    address to,
    uint256 expiry,
    uint256 rdpxAmount
  ) external onlyRole(MINTER_ROLE) { <=========================== hasRole(MINTER_ROLE, _msgSender()) under the hood
    _whenNotPaused();
    require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter"); <========= Redundant
    uint256 bondId = _mintToken(to);
    bonds[bondId] = Bond(to, expiry, rdpxAmount);

    emit BondMinted(to, bondId, expiry, rdpxAmount);
  }
```