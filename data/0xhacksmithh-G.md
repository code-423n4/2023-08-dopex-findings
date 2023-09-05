### [Gas-0] 'constant` should be a value rather than a expression
Value should directly assign to constant state variable, and a comment mentioned how that value calculated from expression
```solidity
-   bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

+   bytes32 public constant MINTER_ROLE = 0x9f2df0fed2c77648de5860a4cc508cd0818c85b8b8a1ab4ceeef8d981c8956a6 // keccak256("MINTER_ROLE")
```
```solidity
  // Create a new role identifier for the minter role
  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE"); 

  // Create a new role identifier for the Rdpx v2 core role
  bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
```
```solidity
  bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE"); // @audit
  bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
```

*Instances()*
```
File:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L22
```
```
file:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L31
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L34
```
```
file:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L19
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L20
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L21
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
```solidity
require(
      _perpetualAtlanticVault != address(0) || _rdpx != address(0), // @audit
      "ZERO_ADDRESS"
    );
```
```solidity
require(
      _tokenA != address(0) &&
        _tokenB != address(0) && // @audit assembly
        _pair != address(0) &&
        _rdpxV2Core != address(0) &&
        _tokenAReserve != address(0) &&
        _amo != address(0) &&
        _rdpxOracle != address(0) &&
        _ammFactory != address(0) &&
        _ammRouter != address(0),
      "reLPContract: address cannot be 0"
    );
```
*Instances()*
```
File:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L84-L90
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L132
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L131
```
```
file:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L95
```
```
file:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L127-L135
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
```solidity
File:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L192
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L201
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L210
```
```solidity
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L245
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L263
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L282
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L293
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L304
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

### [Gas-0] Use `bytes32` Instead of `string`
```solidity
string public underlyingSymbol;
```
```solidity
string public collateralSymbol;
```
*Instances()*
```
File:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L52
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L55
```

### [Gas-0] Instead of making multiple call to `struct` `addresses` state variable it could cached in memory.
```solidity
function reLP(uint256 _amount) external onlyRole(RDPXV2CORE_ROLE) {
+   Addresses memory _addresses = addresses;

    // get the pool reserves
    (address tokenASorted, address tokenBSorted) = UniswapV2Library.sortTokens(
-     addresses.tokenA, 
-     addresses.tokenB

+     _addresses.tokenA, 
+     _addresses.tokenB
    );
    (uint256 reserveA, uint256 reserveB) = UniswapV2Library.getReserves(
-     addresses.ammFactory,

+     _addresses.ammFactory,
      tokenASorted,
      tokenBSorted
    );

    TokenAInfo memory tokenAInfo = TokenAInfo(0, 0, 0);

    // get tokenA reserves
-   tokenAInfo.tokenAReserve = IRdpxReserve(addresses.tokenAReserve)
+   tokenAInfo.tokenAReserve = IRdpxReserve(_addresses.tokenAReserve)
      .rdpxReserve(); // rdpx reserves

    // get rdpx price
-   tokenAInfo.tokenAPrice = IRdpxEthOracle(addresses.rdpxOracle)
+   tokenAInfo.tokenAPrice = IRdpxEthOracle(_addresses.rdpxOracle)
      .getRdpxPriceInEth();

-   tokenAInfo.tokenALpReserve = addresses.tokenA == tokenASorted
+   tokenAInfo.tokenALpReserve = _addresses.tokenA == tokenASorted
      ? reserveA
      : reserveB;

    uint256 baseReLpRatio = (reLPFactor *
      Math.sqrt(tokenAInfo.tokenAReserve) *
      1e2) / (Math.sqrt(1e18)); // 1e6 precision

    uint256 tokenAToRemove = ((((_amount * 4) * 1e18) /
      tokenAInfo.tokenAReserve) *
      tokenAInfo.tokenALpReserve *
      baseReLpRatio) / (1e18 * DEFAULT_PRECISION * 1e2);

-   uint256 totalLpSupply = IUniswapV2Pair(addresses.pair).totalSupply();
+   uint256 totalLpSupply = IUniswapV2Pair(_addresses.pair).totalSupply();

    uint256 lpToRemove = (tokenAToRemove * totalLpSupply) /
      tokenAInfo.tokenALpReserve;

    // transfer LP tokens from the AMO
-   IERC20WithBurn(addresses.pair).transferFrom(
-     addresses.amo,
-     address(this),
+   IERC20WithBurn(_addresses.pair).transferFrom(
+     _addresses.amo,
+     _address(this),
      lpToRemove
    );

    // calculate min amounts to remove
    uint256 mintokenAAmount = tokenAToRemove -
      ((tokenAToRemove * liquiditySlippageTolerance) / 1e8);
    uint256 mintokenBAmount = ((tokenAToRemove * tokenAInfo.tokenAPrice) /
      1e8) -
      ((tokenAToRemove * tokenAInfo.tokenAPrice) * liquiditySlippageTolerance) /
      1e16;

-   (, uint256 amountB) = IUniswapV2Router(addresses.ammRouter).removeLiquidity(
+   (, uint256 amountB) = IUniswapV2Router(_addresses.ammRouter).removeLiquidity(

-     addresses.tokenA,
-     addresses.tokenB,
+     _addresses.tokenA,
+     _addresses.tokenB,
      lpToRemove,
      mintokenAAmount,
      mintokenBAmount,
      address(this),
      block.timestamp + 10
    );

    address[] memory path;
    path = new address[](2);
-   path[0] = addresses.tokenB;
-   path[1] = addresses.tokenA;

+   path[0] = _addresses.tokenB;
+   path[1] = _addresses.tokenA;

    // calculate min amount of tokenA to be received
    mintokenAAmount =
      (((amountB / 2) * tokenAInfo.tokenAPrice) / 1e8) -
      (((amountB / 2) * tokenAInfo.tokenAPrice * slippageTolerance) / 1e16);

-   uint256 tokenAAmountOut = IUniswapV2Router(addresses.ammRouter)
+   uint256 tokenAAmountOut = IUniswapV2Router(_addresses.ammRouter)
      .swapExactTokensForTokens(
        amountB / 2,
        mintokenAAmount,
        path,
        address(this),
        block.timestamp + 10
      )[path.length - 1];

-   (, , uint256 lp) = IUniswapV2Router(addresses.ammRouter).addLiquidity(
-     addresses.tokenA,
-     addresses.tokenB,

+   (, , uint256 lp) = IUniswapV2Router(_addresses.ammRouter).addLiquidity(
+     _addresses.tokenA,
+     _addresses.tokenB,
      tokenAAmountOut,
      amountB / 2,
      0,
      0,
      address(this),
      block.timestamp + 10
    );

    // transfer the lp to the amo
-   IERC20WithBurn(addresses.pair).safeTransfer(addresses.amo, lp);
-   IUniV2LiquidityAmo(addresses.amo).sync();

+   IERC20WithBurn(_addresses.pair).safeTransfer(_addresses.amo, lp);
+   IUniV2LiquidityAmo(_addresses.amo).sync();


    // transfer rdpx to rdpxV2Core
-   IERC20WithBurn(addresses.tokenA).safeTransfer(
+   IERC20WithBurn(_addresses.tokenA).safeTransfer(
      addresses.rdpxV2Core,
-     IERC20WithBurn(addresses.tokenA).balanceOf(address(this))
+     IERC20WithBurn(_addresses.tokenA).balanceOf(address(this))
    );
-   IRdpxV2Core(addresses.rdpxV2Core).sync();
+   IRdpxV2Core(_addresses.rdpxV2Core).sync();
  }
```
*Instances(24)*
```
File:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L202-L307
```
