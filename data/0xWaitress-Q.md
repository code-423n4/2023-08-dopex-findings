[L-1] there is no validation that rdpxBurnPercentage and rdpxFeePercentage adds up to 100%.

rdpxBurnPercentage and rdpxFeePercentage represents the two destinations of RDPX during transfer, 1 is burnt and 1 is transferred to the fee dsitributor. however they can be set separately and arbitrarily. 

```solidity
  /// @notice The % of rdpx to burn while bonding
  uint256 public rdpxBurnPercentage = 50 * DEFAULT_PRECISION;

  /// @notice The % of rdpx sent to fee distributor while bonding
  uint256 public rdpxFeePercentage = 50 * DEFAULT_PRECISION;

...

  function setRdpxBurnPercentage(
    uint256 _rdpxBurnPercentage
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_rdpxBurnPercentage > 0, 3);
    rdpxBurnPercentage = _rdpxBurnPercentage;
    emit LogSetRdpxBurnPercentage(_rdpxBurnPercentage);
  }

  /**
   * @notice Sets the rdpx fee percentage
   * @dev    Can only be called by admin
   * @param  _rdpxFeePercentage the fee percentage to set in 1e8 precision
   **/
  function setRdpxFeePercentage(
    uint256 _rdpxFeePercentage
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_rdpxFeePercentage > 0, 3);
    rdpxFeePercentage = _rdpxFeePercentage;
    emit LogSetRdpxFeePercentage(_rdpxFeePercentage);
  }

```

Recommendation:
combine them into a setter and the other one gets updated automically.


[N-1] _ammFactory is not used in ReLPContract or UniV2LiquidityAmo.

```solidity
  function setAddresses(
    address _tokenA,
    address _tokenB,
    address _pair,
    address _rdpxV2Core,
    address _rdpxOracle,
    address _ammFactory,
    address _ammRouter
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(
      _tokenA != address(0) &&
        _tokenB != address(0) &&
        _pair != address(0) &&
        _rdpxV2Core != address(0) &&
        _rdpxOracle != address(0) &&
        _ammFactory != address(0) &&
        _ammRouter != address(0),
      "reLPContract: address cannot be 0"
    );
    addresses = Addresses({
      tokenA: _tokenA,
      tokenB: _tokenB,
      pair: _pair,
      rdpxV2Core: _rdpxV2Core,
      rdpxOracle: _rdpxOracle,
      ammFactory: _ammFactory,
      ammRouter: _ammRouter
    });
  }
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L123

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L80


[L-2] _sendTokensToRdpxV2Core in UniV2LiquidityAmo does not call V2Core::sync while it does on UniV3LiquidityAmo

UniV2LiquidityAmo
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

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L160


UniV3LiquidityAmo
```solidity
  function _sendTokensToRdpxV2Core(address tokenA, address tokenB) internal {
    uint256 tokenABalance = IERC20WithBurn(tokenA).balanceOf(address(this));
    uint256 tokenBBalance = IERC20WithBurn(tokenB).balanceOf(address(this));
    // transfer token A and B from this contract to the rdpxV2Core
    IERC20WithBurn(tokenA).safeTransfer(rdpxV2Core, tokenABalance);
    IERC20WithBurn(tokenB).safeTransfer(rdpxV2Core, tokenBBalance);

    // sync token balances
    IRdpxV2Core(rdpxV2Core).sync();

    emit LogAssetsTransfered(tokenABalance, tokenBBalance, tokenA, tokenB);
  }
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L353

Recommendation: make it consistent for both function to call sync or not.
