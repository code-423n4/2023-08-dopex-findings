## 1. Memory will be corrupted if add reserve with the same symbol
### Impact
There is check to prevent adding asset with duplicate address, but it doesn't check whether tokenSymbol was previously used.
If yes, old assetReserve will be overriden with the new one

### Proof of Concept
Suppose there already exists "DPXETH" reserve with totalSupply = 1000.
Now reserve "DPXETH" is added but with different address, old is overriden
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L240-L264
```solidity
  function addAssetTotokenReserves(
    address _asset,
    string memory _assetSymbol
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");

    for (uint256 i = 1; i < reserveAsset.length; i++) {
      require(
        reserveAsset[i].tokenAddress != _asset,
        "RdpxV2Core: asset already exists"
      );
    }

    ReserveAsset memory asset = ReserveAsset({
      tokenAddress: _asset,
      tokenBalance: 0,
      tokenSymbol: _assetSymbol
    });
    reserveAsset.push(asset);
    reserveTokens.push(_assetSymbol);

    //@audit HERE OLD RESERVE WILL BE OVERRIDEN WITH THE NEW ONE
@>  reservesIndex[_assetSymbol] = reserveAsset.length - 1;

    emit LogAssetAddedTotokenReserves(_asset, _assetSymbol);
  }
```

### Tools Used
Manual Review

### Recommended Mitigation Steps

```solidity
    for (uint256 i = 1; i < reserveAsset.length; i++) {
      require(
        reserveAsset[i].tokenAddress != _asset,
        "RdpxV2Core: asset already exists"
      );

+     require(reserveAsset[i].tokenSymbol != _assetSymbol);
    }
```
