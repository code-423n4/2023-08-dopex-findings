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

## 2. Allow to set 0 approval
### Impact
For example USDT reverts when set non-zero approval from non-zero approval. Current implementation doesn't allow to change approval for such token, because requires `amount > 0`

### Proof of Concept
1. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L133
```solidity
  function approveContractToSpend(
    address _token,
    address _spender,
    uint256 _amount
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(_token != address(0), "reLPContract: token cannot be 0");
    require(_spender != address(0), "reLPContract: spender cannot be 0");
    require(_amount > 0, "reLPContract: amount must be greater than 0");
    IERC20WithBurn(_token).approve(_spender, _amount);
  }
```
2. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L410
```solidity
  function approveContractToSpend(
    address _token,
    address _spender,
    uint256 _amount
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_token != address(0), 17);
    _validate(_spender != address(0), 17);
    _validate(_amount > 0, 17);
    IERC20WithBurn(_token).approve(_spender, _amount);
  }
```

### Recommended Mitigation Steps
Remove this requirement, allow to pass 0
