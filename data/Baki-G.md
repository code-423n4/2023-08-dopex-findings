## G-01 Calculate minOut only if minAmount == 0 

## Details 
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L545

If `minAmount` is > 0 , then we should not calculate `minOut` as it is a gas wastage.

so instead what we can do is
```solidity
    if (minAmount == 0) {
     minAmount = _ethToDpxEth
      ? (((_amount * getDpxEthPrice()) / 1e8) -
        (((_amount * getDpxEthPrice()) * slippageTolerance) / 1e16))
      : (((_amount * getEthPrice()) / 1e8) -
        (((_amount * getEthPrice()) * slippageTolerance) / 1e16));
    }

    // Swap the tokens
    amountOut = dpxEthCurvePool.exchange(
      _ethToDpxEth ? int128(int256(a)) : int128(int256(b)),
      _ethToDpxEth ? int128(int256(b)) : int128(int256(a)),
      _amount,
      minAmount
    );
```

## G-02 Only compute strike & timeToExpiry if needed

### Details
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1189

Only compute `strike` & `timeToExpiry` if `putOptionsRequired` == true 

so instead what we can do is

```solidity

    if (putOptionsRequired) {
      uint256 timeToExpiry = IPerpetualAtlanticVault(
      addresses.perpetualAtlanticVault
      ).nextFundingPaymentTimestamp() - block.timestamp;

      uint256 strike = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
      .roundUp(rdpxPrice - (rdpxPrice / 4)); // 25% below the current price

      wethRequired += IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
        .calculatePremium(strike, rdpxRequired, timeToExpiry, 0);
    }
```

## G-03 token balances are tracked separately on storage costing extra gas 

there are some instances like https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L54 & https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/IRdpxV2Core.sol#L45 where token balances are tracked in a separately in storage rather than just getting the balance with `balanceOf` method and this costs more gas and also gives rise to potential balances being out of sync bugs so we recommend that separate storage for balances should not be used

## G-04 Minter role being checked twice

In the decaying bonds `mint` method https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L120 `MINTER_ROLE` is being checked twice and hence it is a gas wastage so the line mention in the link should be removed