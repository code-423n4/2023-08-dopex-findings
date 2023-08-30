## [G-01] Use unchecked for variable increments that are guaranteed not to overflow
If using solidity >= 0.8.0, safe arithmetic is built-in. In situations that overflow/underflow checks are redundant, code should be wrapped in an unchecked block. For instance, the for loop increment can be unchecked
```solidity
PerpetualAtlanticVault.sol:225:    for (uint256 i = 0; i < tokens.length; i++) {
PerpetualAtlanticVault.sol:328:    for (uint256 i = 0; i < optionIds.length; i++) {
PerpetualAtlanticVault.sol:413:    for (uint256 i = 0; i < strikes.length; i++) {
RdpxDecayingBonds.sol:103:    for (uint256 i = 0; i < tokens.length; i++) {
RdpxDecayingBonds.sol:156:    for (uint256 i; i < ownerTokenCount; i++) {
RdpxV2Core.sol:167:    for (uint256 i = 0; i < tokens.length; i++) {
RdpxV2Core.sol:246:    for (uint256 i = 1; i < reserveAsset.length; i++) {
RdpxV2Core.sol:775:    for (uint256 i = 0; i < optionIds.length; i++) {
RdpxV2Core.sol:836:    for (uint256 i = 0; i < _amounts.length; i++) {
RdpxV2Core.sol:996:    for (uint256 i = 1; i < reserveAsset.length; i++) {
UniV2LiquidityAmo.sol:147:    for (uint256 i = 0; i < tokens.length; i++) {
UniV3LiquidityAmo.sol:120:    for (uint i = 0; i < positions_array.length; i++) {

```

- [PerpetualAtlanticVault.sol:225](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L225)
- [PerpetualAtlanticVault.sol:328](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L328)
- [PerpetualAtlanticVault.sol:413](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L413)
- [RdpxDecayingBonds.sol:103](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L103)
- [RdpxDecayingBonds.sol:156](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L156)
- [RdpxV2Core.sol:167](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L167)
- [RdpxV2Core.sol:246](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L246)
- [RdpxV2Core.sol:775](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L775)
- [RdpxV2Core.sol:836](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L836)
- [RdpxV2Core.sol:996](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L996)
- [UniV2LiquidityAmo.sol:147](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L147)
- [UniV3LiquidityAmo.sol:120](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L120)

## [G-02] custom errors
Consider using custom errors instead of revert strings. Can save gas when the revert condition has been met during runtime.

Here are some examples of where this optimization could be used:

Note that this will only decrease runtime gas when the revert condition has been met. Regardless, it will decrease deploy time gas

