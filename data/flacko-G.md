## Cache the value of `nextFundingPaymentTimestamp()` in `updateFundingPaymentPointer()`

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L462

In `updateFundingPaymentPointer()` (PerpetualAtlanticVault.sol), the `nextFundingPaymentTimestamp()` method is called 7 times in total every time the loop iterates (and the if statement in it evaluates to `true`).

Instead, the calls can be reduced to 2 by declaring a local variable and caching the value

```solidity
  function updateFundingPaymentPointer() public {
    uint256 _nextFundingPaymentTimestamp = nextFundingPaymentTimestamp();
    ...
  }
```

at the top level of the function (before the while loop), and in the end of the the loop this variable can be updated with the result of a new call to the function.

```solidity
   while (block.timestamp >= _nextFundingPaymentTimestamp) {
      ...
      latestFundingPaymentPointer += 1;

      _nextFundingPaymentTimestamp = nextFundingPaymentTimestamp();
      ...
   }
   ...
}
```


## Cache the value of `nextFundingPaymentTimestamp()` in `_updateFundingRate()`

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L594-L614

In `_updateFundingRate()` (PerpetualAtlanticVault.sol), the `nextFundingPaymentTimestamp()` method is called 3 times in the first branch of the if-else statement. 

Instead of doing that, the value can be cached in a local variable before the if-else statement and be used afterwards instead of doing additional calls.

```solidity
    function _updateFundingRate(uint256 amount) private {
        uint256 _nextFundingPaymentTimestamp = nextFundingPaymentTimestamp();
        ...
    }
```

## Unused storage variable `liquiditySlippageTolerance`

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L103

```solidity
  uint256 public liquiditySlippageTolerance = 5e5; // 0.5%
```

The variable is seemingly unused neither in the `RdpxV2Core` contract nor in any of the other contracts.

## Unnecessary casts to the `IStableSwap` interface in `_curveSwap`

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L515-L558

At the top level of the function `_curveSwap`, there's a local variable holding the interface to the DPXETH curve pool contract.

```solidity
    IStableSwap dpxEthCurvePool = IStableSwap(addresses.dpxEthCurvePool);
```

In the `if` statement of that function, the ETH and DPXETH balances are fetched using:

```solidity
      uint256 ethBalance = IStableSwap(addresses.dpxEthCurvePool).balances(a);
      uint256 dpxEthBalance = IStableSwap(addresses.dpxEthCurvePool).balances(
        b
      );
```

Instead, the local `dpxEthCurvePool` variable can be used directly:

```solidity
      uint256 ethBalance = dpxEthCurvePool.balances(a);
      uint256 dpxEthBalance = dpxEthCurvePool.balances(b);
```

## Cache DPXETH and ETH prices in `_curveSwap`

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L515-L558
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L545-L549

When calculating the minimum out amount in `_curveSwap`, 2 calls are made in each branch of the ternary operation to fetch either the price of DPXETH or the price of ETH.

```solidity
 uint256 minOut = _ethToDpxEth
      ? (((_amount * getDpxEthPrice()) / 1e8) -
        (((_amount * getDpxEthPrice()) * slippageTolerance) / 1e16))
      : (((_amount * getEthPrice()) / 1e8) -
        (((_amount * getEthPrice()) * slippageTolerance) / 1e16));
```

The prices, however, can be cached in 2 local variables:

```solidity
    uint256 dpxEthPrice = getDpxEthPrice();
    uint256 ethPrice = getEthPrice();
```

and be used instead of doing 1 extra call each time.

## Cache the value of `IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress)` in `_transfer`

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L651-L666

In the `else` branch of the if-else statement in the `_transfer` function of `RdpxV2Core`, the cast to the `IERC20WithBurn` interface of the RDPX token address can be cached in a local variable:

```solidity
      IERC20WithBurn rdpx = IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress);
```

and used in the following `safeTransferFrom`, `burn` and `safeTransfer` calls:

```solidity
      rdpx.safeTransferFrom(msg.sender, address(this), _rdpxAmount);
      rdpx.burn(_rdpxAmount * rdpxBurnPercentage / 1e10);
      rdpx.safeTransfer(addresses.feeDistributor, _rdpxAmount * rdpxFeePercentage / 1e10);
``` 

## Calculate `strike` and `timeToExpiry` only in the `if (putOptionsRequired)` statement in `calculateBondCost`

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1189-L1194

The `strike` and `timeToExpiry` local variables are only used if `putOptionsRequired` is true and the code inside the if statement executes, but if that if `putOptionsRequired` is `false`, these two variables are going to go unused very time the function is called.

So they can safely be moved inside the if statement since they are only used by the code inside it.

```soldity
    if (putOptionsRequired) {
      uint256 strike = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
        .roundUp(rdpxPrice - (rdpxPrice / 4)); // 25% below the current price

      uint256 timeToExpiry = IPerpetualAtlanticVault(
        addresses.perpetualAtlanticVault
      ).nextFundingPaymentTimestamp() - block.timestamp;

      wethRequired += IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
        .calculatePremium(strike, rdpxRequired, timeToExpiry, 0);
    }
```
