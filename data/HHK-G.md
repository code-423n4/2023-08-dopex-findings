### GAS1: `reserveTokens` is not needed

#### Technical Details

In the core contract, the storage array [`reserveTokens`](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L70) is not needed as the token symbols are being stored in the `reserveAsset` array.

#### Recommendation

Remove `reserveTokens` and update the function [`removeAssetFromtokenReserves()`](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L270) to use the `reserveAsset` array.

### GAS2: If `minAmount` is set then no need to compute `minOut`.

#### Technical Details

In the function [`_curveSwap()`](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L515) of the core contract. `minOut` is always computed although it won't be use if `minAmount` was set in the params.

#### Recommendation

Add an if statement to not compute `minOut` if `minAmount > 0`.

### GAS3: Simplify `_calculateAmounts()` ratio

#### Technical Details

In the function [`_calculateAmounts()`](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L598) of the core contract, the `amount1` is calculated by recomputing the rdpx value and then finding the ratio of the bond amount.

But because the ratio of ETH and RDPX is a constant 75/25, we could just take 25% of the amount (rdpx part) and then apply the delegate fee.

#### Recommendation

Remove the oracle call and replace the `amount1` calculation with:
```solidity
// amount required for delegatee
amount1 = _amount * RDPX_RATIO_PERCENTAGE / (100 * DEFAULT_PRECISION)
// account for delegate fee
amount1 = (amount1 * (100e8 - _delegateFee)) / 1e10;
```

### GAS4: `_whenNotPaused()` not needed in `mint()`

#### Technical Details

In the [`mint()`](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L114) function of the rDpxDecayingBond, the check `_whenNotPaused()` is not needed as the internal mint function will call `beforeTokenTransfer()`which has this check already.

#### Recommendation

Remove `_whenNotPaused()` from the function.

### GAS5: Save array index instead of Position struct

#### Technical Details

In the UniV3 AMO contract there is [`positions_array`](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L63) and a [`positions_mapping`](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L66C39-L66C56). Because the mapping is only used to link a `tokenId` to a `Position` we could save gas by just saving the index of the `positions_array` as a simple `mapping(uint256 => uint256)`.

#### Recommendation

Use a `mapping(uint256 => uint256)` instead of `mapping(uint256 => Position)`.

### GAS6: `roundingPrecision` could be `constant`

#### Technical Details

In the PerpetualVault contract the variable [`roundingPrecision`](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L104) could be `constant` as it's never updated.

#### Recommendation

Set the variable to `constant`.

### GAS7: Pass `currentPrice` to `calculatePremium()` instead of `0`

#### Technical Details

In the function [`purchase()`](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L255C6-L255C6) of the perpetualVault, the call to `calculatePremium()` could cost less gas by passing the variable `currentPrice` instead of `0` as this will force the function to call the oracle again.

#### Recommendation

Pass `currentPrice`instead of `0`.

### GAS8: Remove `positionId` from the `OptionPosition` struct.

#### Technical Details

In the perpetualVault, the [`OptionPosition`](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L296) struct saves the `optionId`.

But because it is only used with a mapping that already store the id as key, saving it in the struct is useless.

#### Recommendation

Remove `positionId` from the `OptionPosition` struct.