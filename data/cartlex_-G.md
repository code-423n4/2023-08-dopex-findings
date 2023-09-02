# [G-01] Consider make variable constant since it doesn't change anywhere.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L104

```solidity
uint256 public roundingPrecision = 1e6; // @audit make constant and private to save gas
```

Deployment Cost `3802550` <-- `roundingPrecision` is public.

Deployment Cost `3776436` <-- `roundingPrecision` is private constant.

Difference: `26114`

## Recommendation:
Consider use private constant instead of public.

```solidity
--  uint256 public roundingPrecision = 1e6;
++  uint256 private constant roundingPrecision = 1e6;
```


# [G-02] Caller role checks twice in `mint()` function.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L114-L125

In `mint()` function there are two checks that caller has `minter` role. 

```solidity
function mint(
    address to,
    uint256 expiry,
    uint256 rdpxAmount
  ) external onlyRole(MINTER_ROLE) { // @audit first check
    _whenNotPaused();
    require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter"); // @audit second check
    uint256 bondId = _mintToken(to);
    bonds[bondId] = Bond(to, expiry, rdpxAmount);

    emit BondMinted(to, bondId, expiry, rdpxAmount);
  }
```
By eliminating one check, you can win on gas cost.

Difference in gas cost:
```
// avg    | median | max   
// 197461 | 197461 | 197461  <-- with require and modifier
// 197144 | 197144 | 197144  <-- without require
// 197076 | 197076 | 197076  <-- without modifier
```

Using require statement without modifier can save caller of the function 385 gas per call.

## Recommendation:
Consider use only require statement since it more gas efficient.
```solidity
function mint(
    address to,
    uint256 expiry,
    uint256 rdpxAmount
  ) external { // @audit modifier removed since require is cheaper
    _whenNotPaused();
    require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
    uint256 bondId = _mintToken(to);
    bonds[bondId] = Bond(to, expiry, rdpxAmount);

    emit BondMinted(to, bondId, expiry, rdpxAmount);
  }
```

