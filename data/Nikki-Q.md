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

2. RdpxV2Bond mints token with ID 0.
LINK:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L40

Summary:
Some functions behave when tokenId id is 0.
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L630

