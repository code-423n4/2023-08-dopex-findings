1. Incorrect POP operation of the value `reserveTokens` in the function `removeAssetFromtokenReserves`.
Link:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L287

Summary:
```solidity
    // remove the asset from the mapping
    reservesIndex[_assetSymbol] = 0;

    // add new index for the last element
    reservesIndex[reserveTokens[reserveTokens.length - 1]] = index;

    // update the index of reserveAsset with the last element
    reserveAsset[index] = reserveAsset[reserveAsset.length - 1];

    // remove the last element
    reserveAsset.pop();
    reserveTokens.pop();
```
This would always delete the last symbol of `reserveTokens` in the array, irrespective of the `_assetSymbol` provided to delete.

POC:
```solidity
  function testPopBugPoc() public {
    // rdpxV2Core.addAssetTotokenReserves(address(rdpx), "RDPX");     - 1
    // rdpxV2Core.addAssetTotokenReserves(address(weth), "WETH");     - 2
    // rdpxV2Core.addAssetTotokenReserves(address(dpxETH), "DPXETH"); - 3
    assertEq(rdpxV2Core.getReserveAssetLength(), 4);
    assertEq(rdpxV2Core.getReserveTokensLength(), 3);

    rdpxV2Core.removeAssetFromtokenReserves("WETH");
    uint len = rdpxV2Core.getReserveTokensLength();
    string memory tokens_ = rdpxV2Core.reserveTokens(len - 1);
    assertNotEq(tokens_, "DPXETH");
    assertEq(tokens_, "WETH");
  }
```

Recommondations:
```
  function testPopBugPoc() public {
    rdpxV2Core.addAssetTotokenReserves(address(rdpx), "RDPX");     - 1
    rdpxV2Core.addAssetTotokenReserves(address(weth), "WETH");     - 2
    rdpxV2Core.addAssetTotokenReserves(address(dpxETH), "DPXETH"); - 3
    assertEq(rdpxV2Core.getReserveAssetLength(), 4);
    assertEq(rdpxV2Core.getReserveTokensLength(), 3);

    rdpxV2Core.removeAssetFromtokenReserves("WETH");
    uint len = rdpxV2Core.getReserveTokensLength();
    string memory tokens_ = rdpxV2Core.reserveTokens(len - 1);

    assertNotEq(tokens_, "DPXETH");
    assertEq(tokens_, "WETH");
  }
```

2. `RdpxDecayingBonds` Bond Owner Not Modified on Transfer.
Link:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L41

Summary:
If the bond token is transferred to another user, the Bond Owner in the struct will not get updated.

3. Unnecessary Bond Mints

Link:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L827
The function doesn't check if the arrays are empty, this could allow any user to mint a nil bond.

Recommendation:
```
  function bondWithDelegate(
    address _to,
    uint256[] memory _amounts,
    uint256[] memory _delegateIds,
    uint256 rdpxBondId
  ) public returns (uint256 receiptTokenAmount, uint256[] memory) {
    _whenNotPaused();
    // Validate amount
+   _validate(_amounts.length != 0, 3);   
    _validate(_amounts.length == _delegateIds.length, 3);
    ...
  }
```

4. RdpxV2Bond mints token with ID 0.
LINK:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L40

Summary:
Some functions behave when tokenId id is 0. This might return unexpected results.
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L630


5. Missing Allowances
Link: 
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L578
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L354

Summary:
There are no instances of allowances when using `safeTransformFrom`.

6. Inconsistent Slippage Precision
Link:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L100

Summary:
In code, it is stated that the value of slippage tolerance is of 1e8 precision. 
To represent `0.5%`, `5e5` is used instead of `5e7`.
```
  /// @notice The slippage tolernce in swaps in 1e8 precision
  uint256 public slippageTolerance = 5e5; // 0.5%
```