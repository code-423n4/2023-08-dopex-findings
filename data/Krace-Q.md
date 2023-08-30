## [Low]  PerpetualAtlanticVaultLP.constructor should use && to check the address
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L81
`require(_perpetualAtlanticVault != address(0) || _rdpx != address(0),"ZERO_ADDRESS");` means one of `_perpetualAtlanticVault` and `_rdpx` could be zero address, I think `&&` should be used here.

```
  constructor(
    address _perpetualAtlanticVault,
    address _rdpxRdpxV2Core,
    address _collateral,
    address _rdpx,
    string memory _collateralSymbol
  )
    ERC20(
      "PerpetualAtlanticVault LP Token",
      _collateralSymbol,
      ERC20(_collateral).decimals()
    )
  {
    require(
      _perpetualAtlanticVault != address(0) || _rdpx != address(0),
      "ZERO_ADDRESS"
    );
    perpetualAtlanticVault = IPerpetualAtlanticVault(_perpetualAtlanticVault);
    rdpxRdpxV2Core = _rdpxRdpxV2Core;
    collateralSymbol = _collateralSymbol;
    rdpx = _rdpx;
    collateral = ERC20(_collateral);

    symbol = string.concat(_collateralSymbol, "-LP");

    collateral.approve(_perpetualAtlanticVault, type(uint256).max);
    ERC20(rdpx).approve(_perpetualAtlanticVault, type(uint256).max);
  }
```


## [Low] Address in UniV3LiquidityAmo should be immutable
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L82-L88
The address of `univ3_factory`, `univ3_positions` and `univ3_router` are set in Constructor, and is not changeable, maybe you should set them to `immutable` or add a function to let admin change them.




## [Low] Implementation of `numPositions` may not be as expected
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L111
The comment of `UniV3LiquidityAmo.numPositions` said that it `Only counts non-withdrawn positions`, but it just returns the length of `positions_array` directly. It seems that there is no code that could judge whether a position is `non-withdrawn` or not.



## [Info] _bondWithDelegate should check MAX fee acceptable
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L699
`_bondWithDelegate` should add an argument to check the MAX fee that a user could accept, to prevent users from mistakenly selecting a delegate with a more expensive fee.

```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
index e28a65c..a3fd246 100644
--- a/contracts/core/RdpxV2Core.sol
+++ b/contracts/core/RdpxV2Core.sol
@@ -699,7 +699,8 @@ contract RdpxV2Core is
   function _bondWithDelegate(
     uint256 _amount,
     uint256 rdpxBondId,
-    uint256 delegateId
+    uint256 delegateId,
+    uint256 maxFee
   ) internal returns (BondWithDelegateReturnValue memory returnValues) {
     // Compute the bond cost
     (uint256 rdpxRequired, uint256 wethRequired) = calculateBondCost(
@@ -711,7 +712,7 @@ contract RdpxV2Core is
     reserveAsset[reservesIndex["WETH"]].tokenBalance += wethRequired;

     Delegate storage delegate = delegates[delegateId];
-
+    _validate(delegate.fee <= maxFee, 3);
     // update delegate active collateral
     _validate(delegate.amount - delegate.activeCollateral >= wethRequired, 5);
     delegate.activeCollateral += wethRequired;
```



## [Info] optionsOwned is useless
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L79
The contract uses `optionsOwned` to log whether an option id is owned or not. But it seems to be useless, when settle is invoked, `perpetualAtlanticVault` will burn the corresponding option Id, so there is no need to log the state of option Id.



## [Info] (Math.sqrt(1e18)) should be write as 1e9
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1165
There is a `(Math.sqrt(1e18))` in function `reLp`, maybe you should use 1e9 directly.



## [Info] Name of `decreaseAmount`
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L139
`RdpxDecayingBonds.decreaseAmount` actually just sets the `rdpxAmount` of the bond, I think it might be more appropriate to change the function name to `setAmount`.



## [Info] Uselsee delegate should be removed from `delegates`
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L975-L990
Once a user `withdraw` his delegate from `RdpxV2Core`, the corresponding `delegate` is useless and should be removed from `delegates` vector.