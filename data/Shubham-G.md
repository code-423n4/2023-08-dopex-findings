## Gas Optimization

| |Issue|Instances|
|-|:-|:-:|
| [G-01](#G-01) | Successive sorting of token addresses wastes gas | 1 |
| [G-02](#G-02) | Checking the value of `supply` early on can prevent execution of external calls & calculations | 1 |


## [G-01] Successive sorting of token addresses wastes gas

In 'reLP()', first the token addresses are sorted by calling 'UniswapV2Library.sortTokens()' & then the reserves are calculated by calling 'UniswapV2Library.getReserves()'.

```solidity
File: contracts/reLP/ReLPContract

    function reLP(uint256 _amount) external onlyRole(RDPXV2CORE_ROLE) {
    // get the pool reserves
    (address tokenASorted, address tokenBSorted) = UniswapV2Library.sortTokens(
      addresses.tokenA,
      addresses.tokenB
    );
    (uint256 reserveA, uint256 reserveB) = UniswapV2Library.getReserves(
      addresses.ammFactory,
      tokenASorted,
      tokenBSorted
    );
    .................
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L202-L212

But in the 'UniswapV2Library' we can see that the 'getReserves()' already calls the 'sortTokens' function.

```solidity
File: UniswapV2Library.sol

    function getReserves(
    address factory,
    address tokenA,
    address tokenB
    ) internal view returns (uint256 reserveA, uint256 reserveB) {
    (address token0, ) = sortTokens(tokenA, tokenB);                   --------> Sorting already happens here
    (uint256 reserve0, uint256 reserve1, ) = IUniswapV2Pair(
      pairFor(factory, tokenA, tokenB)
    ).getReserves();
    (reserveA, reserveB) = tokenA == token0
      ? (reserve0, reserve1)
      : (reserve1, reserve0);
    }
```

Thus sorting of the same token addresses occur twice inside the same function wasting gas.

## [G-02] Checking the value of `supply` early on can prevent execution of external calls & calculations

The value of 'supply' is checked at the end of the function after performing the necessary calculations. But instead if we check its value at the beginning of the function then we can prevent calling external contracts to get the underlying price & the arithmetic calculations. 

```solidity
File: PerpetualAtlanticVaultLP.sol

    function convertToShares(
    uint256 assets
   ) public view returns (uint256 shares) {
    uint256 supply = totalSupply;
    uint256 rdpxPriceInAlphaToken = perpetualAtlanticVault.getUnderlyingPrice();

    uint256 totalVaultCollateral = totalCollateral() +
      ((_rdpxCollateral * rdpxPriceInAlphaToken) / 1e8);
    return
      supply == 0 ? assets : assets.mulDivDown(supply, totalVaultCollateral);    -----> Check conditions in the beginning
   }
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L274-L284

### Recommended Step

Checking the value of **supply** early on saves gas.

```solidity
File: PerpetualAtlanticVaultLP.sol

    function convertToShares(
    uint256 assets
   ) public view returns (uint256 shares) {
    uint256 supply = totalSupply;
+   if (supply == 0) {
+       return assets;
+   }
    uint256 rdpxPriceInAlphaToken = perpetualAtlanticVault.getUnderlyingPrice();

    uint256 totalVaultCollateral = totalCollateral() +
      ((_rdpxCollateral * rdpxPriceInAlphaToken) / 1e8);
    return

-      supply == 0 ? assets : assets.mulDivDown(supply, totalVaultCollateral);
+      assets.mulDivDown(supply, totalVaultCollateral);
   }
```
