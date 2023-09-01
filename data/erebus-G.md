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

## [Gas-02] Comment does not adhere to implementation
In [RdpxDecayingBonds::decreaseAmount](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L139C1-L145C4), the `@notice` and the name of the function suggests that the new amount shall be less than the current one. However, it does not check that, indeed, is less nor it is different, being misleading and doing unnecessary opcodes respectively. Consider changing the function corpse to 

```
_whenNotPaused();
uint256 temp = bonds[bondId].rdpxAmount; // sload
require(amount < temp, "...");
bonds[bondId].rdpxAmount = amount; // sstore
```

or changing the comment and the function name if your intention for the function is to behave like a setter. 

NOTE -> I will put it as gas too