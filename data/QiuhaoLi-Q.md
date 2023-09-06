## [low] RdpxV2Core.sol _transfer should update reserveAsset[reservesIndex["RDPX"]] if (rdpxBurnPercentage + rdpxFeePercentage) < 1e10, or add constraints in setRdpxBurnPercentage/setRdpxFeePercentage

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L657
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L662
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L180
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L193

In the setters of rdpxBurnPercentage and rdpxFeePercentage, we only check if the value is bigger than zero, not check if they sum up to 1e10.And in _transfer(), we didn't check if rdpxBurnPercentage + rdpxFeePercentage) == 1e10 and didn't update reserveAsset[reservesIndex["RDPX"]].tokenBalance relatively. This can lead to two results if the admin changes rdpxBurnPercentage/rdpxFeePercentage wrongly:

1. When (rdpxBurnPercentage + rdpxFeePercentage) < 1e10, reserveAsset[reservesIndex["RDPX"]].tokenBalance will be smaller than the actual value.
2. When (rdpxBurnPercentage + rdpxFeePercentage) > 1e10, more rdpx is spent than designed. Eventually, the function of the core can be stopped since no more rdpx is usable.

To solve this, we need to make sure (rdpxBurnPercentage + rdpxFeePercentage) == 1e10 in setters or _transfer, and update the reserveAsset[reservesIndex["RDPX"]].tokenBalance if so.


## [low] PerpetualAtlanticVault should round up when calculating requiredCollateral

In purchase(), we round down the requiredCollateral. This can lead to insufficient collateral, even zero collateral when (amount * strike) < 1e8. Rounding up is safer.


## [low] UniV3LiquidityAmo should add setter for univ3_factory, univ3_positions, and univ3_router

In UniV3LiquidityAmo's constructor, we hardcoded the uniswap contract's address. However, these addresses are different on other blockchains like celo: https://docs.uniswap.org/contracts/v3/reference/deployments. To help the project migrate to more blockchains, we should have setters for these state variables.


## [low] UniV3LiquidityAmo:liquidityInPool() should check if get_pool is zero

In https://docs.uniswap.org/contracts/v3/reference/core/interfaces/IUniswapV3Factory#getpool, the univ3_factory.getPool(address(rdpx), _collateral_address, _fee) will return a zero address if the pool doesn't exist. Should add a check and revert with the info.


## [low] provideFunding should be hardcoded to decrease weth balance after payFunding

In provideFunding, there is a `reserveAsset[reservesIndex["WETH"]].tokenBalance -= fundingAmount;` after we called `PerpetualAtlanticVault(addresses.perpetualAtlanticVault).payFunding();`. And in `payFunding`, collateralToken can be set by admin, which means it can be a token other than weth. This can lead to a wrong token decrease in provideFunding if that happens. Should use a setter or get the collateralToken first and then decrease the relatively reserve asset in proivdeFuding.


## [low] UniV2LiquidityAmo/UniV3LiquidityAmo and ReLPContract is vulernable to MEV attack

There are multiple uniswap calls in UniV2LiquidityAmo/UniV3LiquidityAmo that use block.timestamp + 1 and type(uint256).max or large timestamp as the deadline. They are vulnerable to MEV attack, for example:

```solidity
    ISwapRouter.ExactInputSingleParams memory swap_params = ISwapRouter
      .ExactInputSingleParams(
        _tokenA,
        _tokenB,
        _fee_tier,
        address(this),
        2105300114, // Expiration: a long time from now @audit-issue deadline vulnerable to MEV
        _amountAtoB,
        _amountOutMinimum,
        _sqrtPriceLimitX96
      );
```

```solidity
    ISwapRouter.ExactInputSingleParams memory swap_params = ISwapRouter
      .ExactInputSingleParams(
        _tokenA,
        _tokenB,
        _fee_tier,
        address(this),
        2105300114, // Expiration: a long time from now @audit-issue deadline vulnerable to MEV
        _amountAtoB,
        _amountOutMinimum,
        _sqrtPriceLimitX96
      );
```

```solidity
    // remove liquidity
    (tokenAReceived, tokenBReceived) = IUniswapV2Router(addresses.ammRouter)
      .removeLiquidity(
        addresses.tokenA,
        addresses.tokenB,
        lpAmount,
        tokenAAmountMin,
        tokenBAmountMin,
        address(this),
        block.timestamp + 1
      );
```

```solidity
    (, uint256 amountB) = IUniswapV2Router(addresses.ammRouter).removeLiquidity(
      addresses.tokenA,
      addresses.tokenB,
      lpToRemove,
      mintokenAAmount,
      mintokenBAmount,
      address(this),
      block.timestamp + 10

    );
```

Note: They are not mentioned in https://github.com/code-423n4/2023-08-dopex/blob/main/bot-report.md#m02-using-blocktimestamp-as-the-deadlineexpiry-invites-mev.

## [low] UniV2LiquidityAmo.sol/RdpxV2Core.sol:approveContractToSpend is vulnerable to front-run attack

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L411 (also in UniV2LiquidityAmo.sol)

In approveContractToSpend, we used `IERC20WithBurn(_token).approve(_spender, _amount)` directly without setting it to zero first or using increaseAllowance or decreaseAllowance. This is vulnerable to a front-run attack where a user can front-run the approve and spend old_allowance+new_allowance totally.

```solidity
  function approveContractToSpend(
    address _token,
    address _spender,
    uint256 _amount
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_token != address(0), 17);
    _validate(_spender != address(0), 17);
    _validate(_amount > 0, 17);
    IERC20WithBurn(_token).approve(_spender, _amount); <--
  }
```

It's better to use increase or decrease APIs or set it to zero first.

## [non-critical] RdpxDecayingBonds.sol: decreaseAmount should make sure amout is less than bonds[bondId].rdpxAmount

The comment of decreaseAmount() says `Decreases the bond amount` but it directly sets bonds[bondId].rdpxAmount to the amount passed in. It should add checks to make sure it's a decrease (bonds[bondId].rdpxAmount > amount).

## [non-critical] ReLPContract.sol:reLP wrong comment of baseReLpRatio

```solidity
uint256 baseReLpRatio = (reLPFactor *
      Math.sqrt(tokenAInfo.tokenAReserve) *
      1e2) / (Math.sqrt(1e18)); // 1e6 precision
```

The comment is wrong, should be 1e8 according to the developer and context.


## [non-critical] PerpetualAtlanticVaultLP.sol approve _perpetualAtlanticVault rdpx with type(uint256).max but never used

In the constructor of PerpetualAtlanticVaultLP, we approve _perpetualAtlanticVault rdpx with type(uint256).max: `ERC20(rdpx).approve(_perpetualAtlanticVault, type(uint256).max);`. But _perpetualAtlanticVault never uses PerpetualAtlanticVaultLP's rdpx. So this approval is uselss and should be removed.




## [non-critical] UniV2LiquidityAmo.sol/RdpxV2Core.sol:approveContractToSpend should allow zero amount

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L410 (also in UniV2LiquidityAmo.sol)

In approveContractToSpend, we validated `amount > 0`, this should be allowed since setting allowance to zero is a normal operation to cancel all the approved amounts.

## [non-critical] UniV2LiquidityAmo.sol:emergencyWithdraw() should only be callable when paused

Currently, we don't have a pausable function in UniV2LiquidityAmo.sol, and the admin role can call emergencyWithdraw anytime. It's better to add a `_only_paused` check in it.


