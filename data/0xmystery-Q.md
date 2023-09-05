## Unusable native transfer logic in RdpxDecayingBonds.emergencyWithdraw
The `RdpxDecayingBonds` contract is not inherently designed to accept native tokens, highlighted by the absence of a `receive` or `fallback` function. This makes the native transfer logic of `emergencyWithdraw` function unusable unless in the rare and unconventional event that another contract sends native tokens to `RdpxDecayingBonds.sol` using the `selfDestruct` method. This would allow native tokens to accrue in a manner that's unexpected and not immediately apparent. The presence of such funds can be perplexing, especially given the contract's design which suggests it shouldn't typically handle native tokens. Fortunately, the embedded `emergencyWithdraw` function can be invoked by administrators to reclaim these inadvertently received tokens, mitigating the risk of irretrievable funds.

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L80-L107

```solidity
  /**
   * @notice Transfers all funds to msg.sender
   * @dev    Can only be called by the owner
   * @param  tokens The list of erc20 tokens to withdraw
   * @param  transferNative Whether should transfer the native currency
   * @param  to The address to transfer the funds to
   * @param  amount The amount to transfer
   * @param  gas The gas to use for the transfer
   **/
  function emergencyWithdraw(
    address[] calldata tokens,
    bool transferNative,
    address payable to,
    uint256 amount,
    uint256 gas
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _whenPaused();
    if (transferNative) {
      (bool success, ) = to.call{ value: amount, gas: gas }("");
      require(success, "RdpxReserve: transfer failed");
    }
    IERC20WithBurn token;

    for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }
  }
```
The protocol should decide with clarity if the contract should or should not manage native tokens. If the latter, implement further preventive measures to transparently convey this intent and remove `transferNative` logic.

## Threshold check too big
In `RdpxV2Core.calculateBondCost`, `RDPX_RATIO_PERCENTAGE` is `25 * DEFAULT_PRECISION` which is 25e8:

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L84-L88

```solidity
  /// @notice Precision used for prices, percentages and other calculations
  uint256 public constant DEFAULT_PRECISION = 1e8;

  /// @notice rDPX Ratio required when bonding
  uint256 public constant RDPX_RATIO_PERCENTAGE = 25 * DEFAULT_PRECISION;
```
According to the logic below, `bondDiscount` can be as big as `100e8 - 1`:

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1167-L1173

```solidity
      _validate(bondDiscount < 100e8, 14);

      rdpxRequired =
        ((RDPX_RATIO_PERCENTAGE - (bondDiscount / 2)) *
          _amount *
          DEFAULT_PRECISION) /
        (DEFAULT_PRECISION * rdpxPrice * 1e2);
```  
And, apparently, any value of `bondDiscount` between `50e8 + 1` and `100e8 - 1` is going to make `(RDPX_RATIO_PERCENTAGE - (bondDiscount / 2)` revert. Hence, it is recommended refactoring the validate logic as follows:

```diff
-      _validate(bondDiscount < 100e8, 14);
+      _validate(bondDiscount < 50e8 + 1, 14);
```
## Chain reorganization attack
As denoted in the [Moralis academy article](https://academy.moralis.io/blog/what-is-chain-reorganization):

"... If a node receives a new chain that’s longer than its current active chain of blocks, it will do a chain reorg to adopt the new chain, regardless of how long it is."

Depending on the outcome, if it ended up placing the transaction earlier than anticipated, quite a number of the function calls could backfire. For instance, a user making a call that is dependent on [`RdpxV2Core.calculateBondCost()`](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1162-L1177) could end up receiving lesser discount than anticipated due to a smaller value associated with the older [`bondDiscountFactor`](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L441-L448) the admin has recently increased.

(Note: On Ethereum this is unlikely but this is meant for contracts going to be deployed on any compatible EVM chain many of which like Polygon, Optimism, Arbitrum are frequently reorganized.)

## Missing check if the array index is linked to the last element
Checking if the index to be removed in `RdpxV2Core.removeAssetFromtokenReserves` is the last element can help skip some unnecessary operations. If the asset to be removed is already the last one, there's no need to overwrite its position with the last element, and you can just pop() the last item.

Here's a modified version that includes this optimization:
 
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L270-L290

```diff
/**
 * @notice Removes an asset from the reserves tokens
 * @dev Can only be called by the admin
 **/
function removeAssetFromtokenReserves(
  string memory _assetSymbol
) external onlyRole(DEFAULT_ADMIN_ROLE) {
  uint256 index = reservesIndex[_assetSymbol];
  _validate(index != 0, 18);

+  // If it's the last element, simply pop and return
+  if (index == reserveTokens.length) {
+    reservesIndex[_assetSymbol] = 0;
+    reserveAsset.pop();
+    reserveTokens.pop();
+    emit LogAssetRemovedFromtokenReserves(_assetSymbol, index);
+    return;
+  }

  // remove the asset from the mapping
  reservesIndex[_assetSymbol] = 0;

  // add new index for the last element
  reservesIndex[reserveTokens[reserveTokens.length - 1]] = index;

  // update the index of reserveAsset with the last element
  reserveAsset[index] = reserveAsset[reserveAsset.length - 1];

  // remove the last element
  reserveAsset.pop();
  reserveTokens.pop();

  emit LogAssetRemovedFromtokenReserves(_assetSymbol, index);
}
```