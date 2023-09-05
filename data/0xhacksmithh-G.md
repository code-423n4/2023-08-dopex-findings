### [Gas-0] 'State Variable` could be packed to save deployment gas 
```solidity

```
*Instances()*
```
File:
```

### [Gas-0] `assembly` could use to check inputed address is Zero address or not, which will be more gas efficient
```solidity
        _tokenA != address(0) &&
        _tokenB != address(0) &&
        _pair != address(0) &&
        _rdpxV2Core != address(0) && // @audit assembly for zero address check
        _rdpxOracle != address(0) &&
        _ammFactory != address(0) &&
        _ammRouter != address(0),
```
```solidity
require(_spender != address(0), "reLPContract: spender cannot be 0")

require(_token != address(0), "reLPContract: token cannot be 0");
```
*Instances()*
```
File:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L84-L90
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L132
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L131
```

### [Gas-0] Use `!= 0` Instead `> 0` which will be more gas efficient
```solidity
require(
      _slippageTolerance > 0, 
      "reLPContract: slippage tolerance must be greater than 0"
    );
```
```solidity
require(_amount > 0, "reLPContract: amount must be greater than 0");
```
*Instances()*
```
File:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L113
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L133
```

### [Gas-0] Gas Can be save by assigning value to whole `addresses` Struct differently
In `setAddresses()` different address assigned to `addresses` struct, By doing it followingly approx *265 Gas* can saved
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
        _rdpxV2Core != address(0) && // @audit assembly for zero address check
        _rdpxOracle != address(0) &&
        _ammFactory != address(0) &&
        _ammRouter != address(0),
      "reLPContract: address cannot be 0"
    );
    addresses = Addresses({ // @audit check for addresses state variable value assigning in other way
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
### This Exicution Cost around 157576 gas 
While
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
        _rdpxV2Core != address(0) && // @audit assembly for zero address check
        _rdpxOracle != address(0) &&
        _ammFactory != address(0) &&
        _ammRouter != address(0),
      "reLPContract: address cannot be 0"
    );
    addresses.tokenA = _tokenA;
    addresses.tokenB = _tokenB;
    addresses.pair = _pair;
    addresses.rdpxV2Core = _rdpxV2Core;
    addresses.rdpxOracle = _rdpxOracle;
    addresses.ammFactory = _ammFactory;
    addresses.ammRouter = _ammRouter;
  }
```
### This Execution cost around 157312 gas, so approx 264 Gas saved in this method (Role check not included both tests, only gas used for assigning value to struct checked here via Remix IDE)
```
*Instances(1)*
```
File:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L93-L100
```

### [Gas-0] Instead calling `address(this)` everytime, address(this) could be stored in a immutable state variable.
```solidity
token.safeTransfer(msg.sender, token.balanceOf(address(this))); 

uint256 tokenABalance = IERC20WithBurn(addresses.tokenA).balanceOf( 
      address(this)
    );
uint256 tokenBBalance = IERC20WithBurn(addresses.tokenB).balanceOf(
      address(this)
    );
```
*Instances()*
```
File:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L149
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L162
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L165
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L363
```
```
file:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L106
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L160
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L165
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L189
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L285
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L294
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L331
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L354
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L355
```

### [Gas-0] `lpReceived` should be added or subtracted to `lpTokenBalance` only when `lpReceived` is > 0
```solidity
// add Liquidity
    (tokenAUsed, tokenBUsed, lpReceived) = IUniswapV2Router(addresses.ammRouter)
      .addLiquidity(
        addresses.tokenA,
        addresses.tokenB,
        tokenAAmount,
        tokenBAmount,
        tokenAAmountMin,
        tokenBAmountMin,
        address(this),
        block.timestamp + 1 // @audit will it valide
      );

    // update LP token Balance
-   lpTokenBalance += lpReceived; 

+   if(lpReceived != 0){
+       lpTokenBalance += lpReceived; 
+   }
```
```solidity
.............
............

-  lpTokenBalance -= lpAmount;

+   if(lpReceived != 0){
+       lpTokenBalance = lpTokenBalance - lpReceived; 
+   }
```
*Instances()*
```
File:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L235
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L283
```

### [Gas-0] Via re-ordering variables inside struct gas can be saved (i.e struct can be packed and consume less storage slots)
```solidity
  struct Position {
    uint256 token_id;
    address collateral_address;
    uint128 liquidity; // the liquidity of the position
+   uint24 fee_tier;
    int24 tickLower; // the tick range of the position
    int24 tickUpper;
-   uint24 fee_tier; 
  }
```
*Instances()*
```
File:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L40-L47
```

### [Gas-0] Use `constant` variable instead of `type(uint256).max`
*Instances()*
```
File:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L190
```

### [Gas-0]
*Instances()*
```
File:
```

### [Gas-0]
*Instances()*
```
File:
```
