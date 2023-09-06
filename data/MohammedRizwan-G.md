## Summary

### Gas Optimizations
| |Issue|Instances| |
|-|:-|:-:|:-:|
| [G&#x2011;01] | Use assembly to emit events | 15 |
| [G&#x2011;02] | Use assembly to write address storage values | 4 |
| [G&#x2011;03] | Use custom errors instead of require/assert | 21 |
| [G&#x2011;04] | Avoid double access control checking in `mint()` to save gas by removing extra bytes | 1 |
| [G&#x2011;05] | Amounts should be checked for 0 before calling a transfer | contracts |




### [G&#x2011;01]  Use assembly to emit events
Assembly can be used to emit events efficiently by utilizing scratch space and the free memory pointer. This will allow us to potentially avoid memory expansion costs. 
Note: In order to do this optimization safely, we will need to cache and restore the free memory pointer.

There are 15 instances of this issue in contracts,

1) `UniV2LiquidityAmo.sol` has 5 event.
2) `UniV3LiquidityAmo.sol` has 5 event.
3) `RdpxDecayingBonds.sol` has 2 event.
4) `PerpetualAtlanticVaultLP.sol` has 2 event.
5) `ReLPContract.sol` has 1 event.

### [G&#x2011;02]  Use assembly to write address storage values
There are 4 instances of this issue:

1) In `UniV2LiquidityAmo.sol`, [`setAddresses()`](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L74-L102) function.

2) In `RdpxV2Core.sol`, [`setAddresses()`](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L304-L350) function

3) In `PerpetualAtlanticVault.sol`, [`setAddresses()`](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L181-L212)

4) In `ReLPContract.sol`, [`setAddresses()`](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L115-L148)

### [G&#x2011;03]  Use custom errors instead of require/assert
There are 21 instances of this issue:

1) In `UniV2LiquidityAmo.sol`, 5 instances
2) In `RdpxV2Core.sol`, 2 instances
3) In `RdpxDecayingBonds.sol`, 2 instances
4) In `PerpetualAtlanticVaultLP.sol`, 8 instances
5) In `ReLPContract.sol`, 4 instances

### [G&#x2011;04]  Avoid double access control checking in `mint()` to save gas by removing extra bytes
In `RdpxDecayingBonds.sol`, `mint()` is checking the access to function by both `onlyRole(MINTER_ROLE)` and `hasRole(MINTER_ROLE, msg.sender)`, However the modifier is enough to check the access control. The extra require can be removed here to save gas which will reduce contract bytes resulting less cost while deploying.

```Solidity
File: contracts/decaying-bonds/RdpxDecayingBonds.sol

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

### Recommended Mitigation steps
```diff

  function mint(
    address to,
    uint256 expiry,
    uint256 rdpxAmount
  ) external onlyRole(MINTER_ROLE) {
    _whenNotPaused();
-   require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
    uint256 bondId = _mintToken(to);
    bonds[bondId] = Bond(to, expiry, rdpxAmount);

    emit BondMinted(to, bondId, expiry, rdpxAmount);
  }
```

### [G&#x2011;05]  Amounts should be checked for 0 before calling a transfer
This should be taken care in all contracts where amount is transferred by ERC20 functions. Zero amount check should be added.
