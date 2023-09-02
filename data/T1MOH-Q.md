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

## 3. RdpxDecayingBonds.sol can't send native value
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L97

There is functionality to send native value in `emergencyWithdraw()`. But contract doesn't have payable and receive functions, therefore there won't be ether. And contract will never be able to perform transfer of native value
```solidity
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
    
    ...
  }
```

The same with UniV3LiquidityAmo.sol
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L344

### Recommended Mitigation Steps
Remove functionality of native asset transfer or add payable function

## 4. RdpxV2Core.sol can issue bonds with immediate maturity
### Impact
Maturity is not set explicitly in constructor, it is set via setter. And this function `setBondMaturity()` can be called in last transaction of deploy, such that user frontruns it and calls `bond()`.
As a result bond is issued with maturity at current block.timestamp

### Recommended Mitigation Steps
Pause protocol in constructor, then set all the values, and unpause

## 5. Confusing comment on `upperDepeg()`
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1047

Is is stated in docs and the code that `upperDepeg()` is executed when 1 DPXETH > 1 ETH. But in comment it's stated when 1 DPXETH > 1.01 ETH
```solidity
  /**
@> * @notice Lets users mint DpxEth at a 1:1 ratio when DpxEth pegs above 1.01 of the ETH token
   * @param  _amount The amount of DpxEth to mint
   * @param minOut The minimum amount out
   **/
  function upperDepeg(
    uint256 _amount,
    uint256 minOut
  ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256 wethReceived) {
    _isEligibleSender();

@>  _validate(getDpxEthPrice() > 1e8, 10);

    ...
  }
```

## 6. UniLiquidityAmo contracts doesn't synchronize reserve balances of RdpxV2Core in some cases
### Impact
Developer from Dopex said that "we keep the balances to check the health of dpxEth". I talk about this balances:
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L61
```solidity
  /// @notice Array containg the reserve assets
  ReserveAsset[] public reserveAsset;

  /// @dev Struct containing the the rdpxV2Core reserve asset data
  struct ReserveAsset {
    address tokenAddress;
    uint256 tokenBalance;
    string tokenSymbol;
  }
```

`tokenBalance` can differ from actual balance in this cases:
1) https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L313-L322
```solidity
  function recoverERC20(
    address tokenAddress,
    uint256 tokenAmount
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    // Can only be triggered by owner or governance, not custodian
    // Tokens are sent to the custodian, as a sort of safeguard
    TransferHelper.safeTransfer(tokenAddress, rdpxV2Core, tokenAmount);

    emit RecoveredERC20(tokenAddress, tokenAmount);
  }
```
2) https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L119-L133
```solidity
  function collectFees() external onlyRole(DEFAULT_ADMIN_ROLE) {
    for (uint i = 0; i < positions_array.length; i++) {
      Position memory current_position = positions_array[i];
      INonfungiblePositionManager.CollectParams
        memory collect_params = INonfungiblePositionManager.CollectParams(
          current_position.token_id,
          rdpxV2Core,
          type(uint128).max,
          type(uint128).max
        );

      // Send to custodian address
      univ3_positions.collect(collect_params);
    }
  }
```
3) https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L160-L178
```solidity
  function _sendTokensToRdpxV2Core() internal {
    uint256 tokenABalance = IERC20WithBurn(addresses.tokenA).balanceOf(
      address(this)
    );
    uint256 tokenBBalance = IERC20WithBurn(addresses.tokenB).balanceOf(
      address(this)
    );
    // transfer token A and B from this contract to the rdpxV2Core
    IERC20WithBurn(addresses.tokenA).safeTransfer(
      addresses.rdpxV2Core,
      tokenABalance
    );
    IERC20WithBurn(addresses.tokenB).safeTransfer(
      addresses.rdpxV2Core,
      tokenBBalance
    );

    emit LogAssetsTransfered(msg.sender, tokenABalance, tokenBBalance);
  }
```

### Recommended Mitigation Steps
Call `IRdpxV2Core(rdpxV2Core).sync()` in all the referenced code blocks