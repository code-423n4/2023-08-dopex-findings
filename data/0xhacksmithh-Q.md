### [Low-0] There should be a `tokenBalance` check before making token transfer
```solidity
function _sendTokensToRdpxV2Core() internal { 
    uint256 tokenABalance = IERC20WithBurn(addresses.tokenA).balanceOf( 
      address(this)
    );
    uint256 tokenBBalance = IERC20WithBurn(addresses.tokenB).balanceOf(
      address(this)
    );

+   require(tokenABalance != 0, 'error');
+   require(tokenBBalance != 0, 'error');

    // transfer token A and B from this contract to the rdpxV2Core
    IERC20WithBurn(addresses.tokenA).safeTransfer(
      addresses.rdpxV2Core,
      tokenABalance
    );
    IERC20WithBurn(addresses.tokenB).safeTransfer(
      addresses.rdpxV2Core,
      tokenBBalance
    );
```
*Instances()*
```
File:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L168-L171
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L172-L175
```

### [Low-0] `safeApprove()` will be incompatible with token like `USDT`
Where you have to first clear pre-exiting approval and then approve further amount
```solidity
    IERC20WithBurn(addresses.tokenA).safeApprove(
      addresses.ammRouter,
      tokenAAmount
    );
    IERC20WithBurn(addresses.tokenB).safeApprove(
      addresses.ammRouter,
      tokenBAmount
    );
```
*Instances()*
```
File:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L200-L207
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L328
```

### [Low-0] There Should be a `require()` which ensure that returned `pool` address from `univ3_factory.getPool()` is not zero address
As `getPool` is a mapping in UNIV3Factory contract there is a possibility if input argument wrong or mismatch it will return default Zero address,
And in `liquidityInPool()` following function call will takes place on returned Zero address, To bypass that there should be a require() statement.
```solidity
  function liquidityInPool(
    address _collateral_address,
    int24 _tickLower,
    int24 _tickUpper,
    uint24 _fee
  ) public view returns (uint128) {
    IUniswapV3Pool get_pool = IUniswapV3Pool(
      univ3_factory.getPool(address(rdpx), _collateral_address, _fee) 
    );

+   require(address(get_pool) != address(0), 'Error');

    // goes into the pool's positions mapping, and grabs this address's liquidity
    (uint128 liquidity, , , , ) = get_pool.positions(
      keccak256(abi.encodePacked(address(this), _tickLower, _tickUpper))
    );
    return liquidity;
  }
```
*Instances()*
```
File:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L100-L107
```

### [Low-0]
*Instances()*
```
File:
```

### [Low-0]
*Instances()*
```
File:
```

### [Low-0]
*Instances()*
```
File:
```

### [Low-0]
*Instances()*
```
File:
```

### [Low-0]
*Instances()*
```
File:
```

### [Low-0]
*Instances()*
```
File:
```

### [NC-0] Confusing `struct` Name, Should change to other one.
There is a `struct` in Univ2LiquidityAmo.sol contract which used frequently whole cobe-base named as `Addresses`
Which naming is some sort of confusing with solidity `address` reserve key-word, so it creates some sort of confusion on first go so it should change to other name.

```solidity
  struct Addresses { // @audit N :: should named other as it is conusing
    // token A address
    address tokenA; // rdpx
    // token B address
    address tokenB; // weth
    // pair address
    address pair;
    // rdpxV2Core address
    address rdpxV2Core;
    // rdpx price oracle
    address rdpxOracle;
    // AMM Factory
    address ammFactory;
    // AMM Router
    address ammRouter;
  }

Addresses public addresses;
```
*Instances()*
```
File:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L27-L45
```