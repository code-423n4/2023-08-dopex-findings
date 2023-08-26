# LOW REPORT

## [L-01] The slippageTolerance is no used in UniV2LiquidityAmo.sol contract 

The slippageTolerance variable is declared in the contract, even have a function to modify it but is never used in the uniswap releted function, the related uniswap functions has his own input value where the admin set the slippage :

```
 uint256 public slippageTolerance = 5e5; // 0.5%
 
...
file: contracts/amo/UniV2LiquidityAmo.sol
function setSlippageTolerance(
    uint256 _slippageTolerance
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(
      _slippageTolerance > 0,
      "reLPContract: slippage tolerance must be greater than 0"
    );
    slippageTolerance = _slippageTolerance;
  }
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L109C3-L117C4

### Recomendation

Use the slippageTolerance in case that admin set 0 slippage input in the uniswap releted function.

## [L-02] Set a deadline not too far from the present in addLiquidity function in UniV3LiquidityAmo.sol contract

Check the addLiquidity function:
```
 function addLiquidity(
    AddLiquidityParams memory params
  ) public onlyRole(DEFAULT_ADMIN_ROLE) {
      ...
      INonfungiblePositionManager.MintParams
      memory mintParams = INonfungiblePositionManager.MintParams(
        params._tokenA,
        params._tokenB,
        params._fee,
        params._tickLower,
        params._tickUpper,
        params._amount0Desired,
        params._amount1Desired,
        params._amount0Min,
        params._amount1Min,
        address(this),
        type(uint256).max
      );
...
}
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L155C1-L211C4

The contract is setting a deadline so far from the future so the transaction can be included in a worst moment

### Recomendation

Use a deadline no too far from the future like the other uniswapv3 releted function do.

## [L-03] The genesis time should be no available to set in the past in PerpetualAtlanticVault.sol contract

The genesis contract can not be set in the pass :

```
constructor(
    string memory _name,
    string memory _symbol,
    address _collateralToken,
    uint256 _gensis
  ) ERC721(_name, _symbol) {
    ...
    genesis = _gensis;

    ...
  }
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L113C3-L128C4

### Recomendation

Add validation to see if the genesis is not set in the pass.

## [L-04] The deposit function in PerpetualAtlanticVaultLP.sol contract is no following the CEI pattern

```
file: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
function deposit(
    uint256 assets,
    address receiver
  ) public virtual returns (uint256 shares) {
    ...

    // Need to transfer before minting or ERC777s could reenter.
    collateral.transferFrom(msg.sender, address(this), assets);

    _mint(receiver, shares);

    _totalCollateral += assets;

    emit Deposit(msg.sender, receiver, assets, shares);
  }
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L118C3-L135C4

### Recomendation

Follow the CEI pattern






