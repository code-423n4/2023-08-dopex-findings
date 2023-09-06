## [L-01] Block values as a proxy for times
In the case of block.timestamp, developers often attempt to use it to trigger time-dependent events. As Ethereum is decentralized, nodes can synchronize time only to some degree. Moreover, malicious miners can alter the timestamp of their blocks, especially if they can gain advantages by doing so. However, miners can't set a timestamp smaller than the previous one (otherwise the block will be rejected), nor can they set the timestamp too far ahead in the future. Taking all of the above into consideration, developers can't rely on the preciseness of the provided timestamp.

## Proof
```sol
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L231
        block.timestamp + 1
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L279
        block.timestamp + 1
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L342
        block.timestamp + 1
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L250
block.timestamp
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L502
      maturity: block.timestamp + bondMaturity,
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L503
      timestamp: block.timestamp
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L636
      _validate(expiry >= block.timestamp, 2);
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1023
    _validate(block.timestamp > bonds[id].maturity, 7);
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1103
          block.timestamp + 10
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1194
    ).nextFundingPaymentTimestamp() - block.timestamp;
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L283
    uint256 timeToExpiry = nextFundingPaymentTimestamp() - block.timestamp;
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L463
    while (block.timestamp >= nextFundingPaymentTimestamp()) {
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L508
    lastUpdateTime = block.timestamp;
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L512
      (currentFundingRate * (block.timestamp - startTime)) / 1e18
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L516
      (currentFundingRate * (block.timestamp - startTime)) / 1e18
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L521
      ((currentFundingRate * (block.timestamp - startTime)) / 1e18),
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L264
      block.timestamp + 10
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L283
        block.timestamp + 10
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L294
      block.timestamp + 10
```
## Tools Used
Manual Review.
## Recommendation
Use Oracles.

## [L-02] Arithmetic Underflow and Overflow
```txt
I've identified an instance of integer overflow within some functions. 
This flaw enables me to manipulate the uint256 value in a way that results in its reduction to zero. Consequently, I can exploit this situation to make a transfer exceeding my actual balance.
I've discovered an integer underflow within the some functions. 
This vulnerability enables me to manipulate the uint256 value in a manner that causes it to wrap around to the maximum value. 
As a result, I can exploit this situation to withdraw an amount greater than what I actually possess.
```
## Proof
```sol
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L270
    uint256 strike = roundUp(currentPrice - (currentPrice / 4)); // 25% below the current price
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L283
    uint256 timeToExpiry = nextFundingPaymentTimestamp() - block.timestamp;
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L427
        (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration));
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L468
          ? (nextFundingPaymentTimestamp() - fundingDuration)
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L475
          (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L481-L482
            (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
              1e18
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L487-L488
          ((currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
            1e18),
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L506
      ? (nextFundingPaymentTimestamp() - fundingDuration)
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L512
      (currentFundingRate * (block.timestamp - startTime)) / 1e18
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L516
      (currentFundingRate * (block.timestamp - startTime)) / 1e18
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L521
      ((currentFundingRate * (block.timestamp - startTime)) / 1e18),
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L559
    return strike > price ? ((strike - price) * amount) / 1e8 : 0;
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L581
      return _strike - remainder + roundingPrecision;
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L597
      if (lastUpdateTime > nextFundingPaymentTimestamp() - fundingDuration) {
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L600
        startTime = nextFundingPaymentTimestamp() - fundingDuration;
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L605
        (endTime - startTime);
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L612
        ((amount * 1e18) / (endTime - startTime));
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L132
    _totalCollateral += assets;
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L181
    _activeCollateral += amount;
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L192
      collateral.balanceOf(address(this)) >= _totalCollateral + proceeds,
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L195
    _totalCollateral += proceeds;
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L213
    _rdpxCollateral += amount;
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L280-L281
    uint256 totalVaultCollateral = totalCollateral() +
      ((_rdpxCollateral * rdpxPriceInAlphaToken) / 1e8);
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L164
    _rdpxCollateral -= rdpxAmount;
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L186
    _activeCollateral -= amount;
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L204
    _totalCollateral -= loss;
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L291
    _totalCollateral -= assets;
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L156
        allowance[owner][msg.sender] = allowed - shares;
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L201-L202
      collateral.balanceOf(address(this)) == _totalCollateral - loss,
      "Not enough collateral was sent out"
// https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L256
    return _totalCollateral - _activeCollateral;
```
## Tools Used
Manual Review.
## Recommendation
Use safe math.
## [L-03] Unsafe erc20 operations
Description
ERC20 operations can be unsafe due to different implementations and vulnerabilities in the standard.

It is therefore recommended to always either use OpenZeppelin's SafeERC20 library or at least to wrap each operation in a require statement.

To circumvent ERC20's approve functions race-condition vulnerability use OpenZeppelin's SafeERC20 library's safe{Increase|Decrease}Allowance functions.

In case the vulnerability is of no danger for your implementation, provide enough documentation explaining the reasonings.
```txt
2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol::134 => IERC20WithBurn(_token).approve(_spender, _amount);
2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol::150 => IERC20WithBurn(_token).approve(_target, _amount);
2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol::158 => IERC20WithBurn(params._tokenA).transferFrom(
2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol::163 => IERC20WithBurn(params._tokenB).transferFrom(
2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol::169 => IERC20WithBurn(params._tokenA).approve(
2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol::173 => IERC20WithBurn(params._tokenB).approve(
2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol::283 => IERC20WithBurn(_tokenA).transferFrom(
2023-08-dopex/contracts/core/RdpxV2Core.sol::339 => IERC20WithBurn(weth).approve(
2023-08-dopex/contracts/core/RdpxV2Core.sol::343 => IERC20WithBurn(weth).approve(addresses.dopexAMMRouter, type(uint256).max);
2023-08-dopex/contracts/core/RdpxV2Core.sol::344 => IERC20WithBurn(weth).approve(addresses.dpxEthCurvePool, type(uint256).max);
2023-08-dopex/contracts/core/RdpxV2Core.sol::345 => IERC20WithBurn(weth).approve(
2023-08-dopex/contracts/core/RdpxV2Core.sol::411 => IERC20WithBurn(_token).approve(_spender, _amount);
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::245 => ? collateralToken.approve(
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::249 => : collateralToken.approve(addresses.perpetualAtlanticVaultLP, 0);
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVaultLP.sol::106 => collateral.approve(_perpetualAtlanticVault, type(uint256).max);
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVaultLP.sol::107 => ERC20(rdpx).approve(_perpetualAtlanticVault, type(uint256).max);
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVaultLP.sol::128 => collateral.transferFrom(msg.sender, address(this), assets);
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVaultLP.sol::170 => collateral.transfer(receiver, assets);
2023-08-dopex/contracts/reLP/ReLPContract.sol::243 => IERC20WithBurn(addresses.pair).transferFrom(
```
## [L-04] Do not use deprecated library functions
Description
The usage of deprecated library functions should be discouraged.

This issue is mostly related to OpenZeppelin libraries.
```txt
2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol::58 => _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol::200 => IERC20WithBurn(addresses.tokenA).safeApprove(
2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol::204 => IERC20WithBurn(addresses.tokenB).safeApprove(
2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol::268 => IERC20WithBurn(addresses.pair).safeApprove(addresses.ammRouter, lpAmount);
2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol::328 => IERC20WithBurn(token1).safeApprove(addresses.ammRouter, token1Amount);
2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol::80 => _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol::148 => TransferHelper.safeApprove(_token, _target, _amount);
2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol::302 => TransferHelper.safeApprove(_tokenA, address(univ3_router), _amountAtoB);
2023-08-dopex/contracts/core/RdpxV2Bond.sol::25 => _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
2023-08-dopex/contracts/core/RdpxV2Bond.sol::26 => _setupRole(MINTER_ROLE, msg.sender);
2023-08-dopex/contracts/core/RdpxV2Core.sol::125 => _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol::61 => _setupRole(MINTER_ROLE, msg.sender);
2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol::62 => _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::126 => _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::127 => _setupRole(MANAGER_ROLE, msg.sender);
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::207 => collateralToken.safeApprove(
2023-08-dopex/contracts/reLP/ReLPContract.sol::80 => _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
2023-08-dopex/contracts/reLP/ReLPContract.sol::81 => _setupRole(RDPXV2CORE_ROLE, msg.sender);
2023-08-dopex/contracts/reLP/ReLPContract.sol::150 => IERC20WithBurn(addresses.pair).safeApprove(
2023-08-dopex/contracts/reLP/ReLPContract.sol::155 => IERC20WithBurn(addresses.tokenA).safeApprove(
2023-08-dopex/contracts/reLP/ReLPContract.sol::160 => IERC20WithBurn(addresses.tokenB).safeApprove(
```