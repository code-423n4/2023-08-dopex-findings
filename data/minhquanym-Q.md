# Summary

| Id | Title |
| --- | --- |
| L-1 | `reserveAsset` and `reserveTokens` are not matched since the constructor only pushes to `reserveAsset` |
| L-2 | Not resetting allowance when setting new addresses in `setAddresses()` |
| L-3 | `emergencyWithdraw()` should transfer tokens to `to` instead of `msg.sender` |
| L-4 | Duplicate access control check in function `mint()` |
| NC-1 | `_beforeTokenTransfer()` is removed in newer versions of OpenZeppelin |

# L-1. `reserveAsset` and `reserveTokens` are not matched since the constructor only pushes to `reserveAsset`

https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/core/RdpxV2Core.sol#L134

## Detail

There are two lists in `RdpxV2Core` that track assets: `reserveAsset` and `reserveTokens`. Each asset should be tracked with the same index in both lists.

However, in the constructor, only the zero asset is pushed to `reserveAsset`, while nothing is pushed to `reserveTokens`. This causes assets to not be tracked by the same index.

```solidity
constructor(address _weth) {
  _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
  weth = _weth;

  // add Zero asset to reserveAsset
  ReserveAsset memory zeroAsset = ReserveAsset({
    tokenAddress: address(0),
    tokenBalance: 0,
    tokenSymbol: "ZERO"
  });
  reserveAsset.push(zeroAsset); // @audit only reserveAsset is updated
  putOptionsRequired = true;
}

```

## Recommendation

Consider making the index of each asset match in both lists.

# L-2. Not resetting allowance when setting new addresses in `setAddresses()`

https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/core/RdpxV2Core.sol#L339

## Detail

When setting new addresses by calling the `setAddresses()` function, the new addresses of Vault, Router, and Pool are given the maximum allowance. However, the allowance given to the old addresses is not reset. This is risky if the old contracts have issues that need to be mitigated by migrating to the new contract.

## Recommendation

Reset the old allowance when setting new addresses.


# L-3. `emergencyWithdraw()` should transfer tokens to `to` instead of `msg.sender`

Link: https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/decaying-bonds/RdpxDecayingBonds.sol#L105

## Details

The function `emergencyWithdraw()` has a parameter `to` that specifies the address that should receive the funds. However, it currently transfers funds to `msg.sender`, which may not be intended.

## Recommendation

Transfer funds to `to` instead of `msg.sender`.

# L-4. Duplicate access control check in function `mint()`

Link: https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/decaying-bonds/RdpxDecayingBonds.sol#L120

## Details

The role of the sender is already checked in the modifier `onlyRole()`. However, it is checked again in the function, which is unnecessary.

```solidity
function mint(
  address to,
  uint256 expiry,
  uint256 rdpxAmount
) external onlyRole(MINTER_ROLE) {
  _whenNotPaused();
  require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter"); // @audit duplicate check with modifier
  uint256 bondId = _mintToken(to);
  bonds[bondId] = Bond(to, expiry, rdpxAmount);

  emit BondMinted(to, bondId, expiry, rdpxAmount);
}

```

## Recommendation

Remove the modifier or the require check.

# NC-1. `_beforeTokenTransfer()` is removed in newer versions of OpenZeppelin

Link: https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/dpxETH/DpxEthToken.sol#L55

## Details

In newer versions of the OpenZeppelin (OZ) library, `_beforeTokenTransfer()` has been removed. If this version of OZ is used, overriding `_beforeTokenTransfer()` will not do anything.

## Recommendation

Review the current version of OZ and consider upgrading to use the latest version.

