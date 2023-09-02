## [L-01] RdpxV2Core.sol should prevent users from interacting before setting the correct addresse and adding reserve assets
In RdpxV2Core.sol, addresses variable and reverses assets will be set by the admin. Before that, if a user called functions like bond(), he may lose money due to the reservesIndex["asset"] will return 0. So it's better to prevent users from interacting with the contract before configuring everything correctly.

## [L-02] Unnecessary division before multiplication
In reLP() in ReLPContract.sol, when calculating tokenAToRemove there is an unnecessary division before multiplication.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L232-L235

  uint256 tokenAToRemove = ((((_amount * 4) * 1e18) /
      tokenAInfo.tokenAReserve) *
      tokenAInfo.tokenALpReserve *
      baseReLpRatio) / (1e18 * DEFAULT_PRECISION * 1e2);

Should divide the tokenAInfo.tokenAReserve at last.

## [L-03] Calculations for tokenAToRemove is different from the equations in docs
In docs, the equation to compute tokenAToRemove is: ((amount_lp * 4) / rdpx_supply) * lp_rdpx_reserves * base_relp_percent
But in code, instead of divide rdpx_supply, it uses tokenAInfo.tokenAReserve

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L232-L235

  uint256 tokenAToRemove = ((((_amount * 4) * 1e18) /
      tokenAInfo.tokenAReserve) *
      tokenAInfo.tokenALpReserve *
      baseReLpRatio) / (1e18 * DEFAULT_PRECISION * 1e2);

The docs and code should be consistent.

## [L-04] Some error messages is wrong in UniV2LiquidityAmo.sol
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L83-L92
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L112-L115
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L131-L133

These messages start with "reLPContract" while this is UniV2LiquidityAmo.sol. Change them to "UniV2LiquidityAmo"

## [L-06] constructor() in PerpetualAtlanticVaultLP.sol first set token symbol to _collateralSymbol but later set it to _collateralSymbol + "-LP"
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L81-L108
constructor() first sets the token symbol to _collateralSymbol

    ERC20(
      "PerpetualAtlanticVault LP Token",
      _collateralSymbol,
      ERC20(_collateral).decimals()
    )

But later set it to _collateralSymbol + "-LP"

    symbol = string.concat(_collateralSymbol, "-LP");

Token symbol should be consistent.

## [L-07] Incorrect check in constructor() of PerpetualAtlanticVaultLP.sol
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L81-L108
In constructor() there is a check:

    require(
      _perpetualAtlanticVault != address(0) || _rdpx != address(0),
      "ZERO_ADDRESS"
    );

should use && here instead of ||

