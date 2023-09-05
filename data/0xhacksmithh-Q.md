### [Low-01] Code is contradicting comments
In `decreaseAmount()` according to comment it decreases bond amount (rdpxAmount) associated with corresponding `bondId`

But in code instead of substracting `decreaseAmount` from bond's rdpxAmount it directly set that variable('rdpxAmount') to inputed decreaseAmount. 

There also absence of bond existance check before making operations on it. 
```solidity
  /// @notice Decreases the bond amount
  /// @dev Can only be called by the rdpxV2Core
  /// @param bondId id of the bond to decrease
  /// @param amount amount to decrease
  function decreaseAmount( 
    uint256 bondId,
    uint256 amount
  ) public onlyRole(RDPXV2CORE_ROLE) {
    _whenNotPaused();
-   bonds[bondId].rdpxAmount = amount; 


+  bonds[bondId].rdpxAmount = bonds[bondId].rdpxAmount - amount; // @audit underflow by default reverted
  }
```
```
file: contracts/decaying-bonds/RdpxDecayingBonds.sol
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L139-L145
```

### [Low-02] While locking Collateral via `lockCollateral()` it doesn't check that locked amount is exceeding totalCollateral or not
`lockCollateral()` simply increase `_activeCollateral` via `amount` that may lead `_activeCollateral` value to exceed `totalCollateral` in that case `totalAvailableCollateral()` call always failed.

So while increasing locked amount there should be a check that ensure increased `_activeCollateral` will never exceed `totalCollateral`

```solidity
function lockCollateral(uint256 amount) public onlyPerpVault {
    _activeCollateral += amount; 
  }
```
*Instances(1)*
```
File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L180-L182
```

### [Low-03] Functions like `mint()`, `burn()`, `burnFrom()` not respecting `pause` or `unpause` functionality
These `mint()`, `burn()`, `burnFrom()` 3 functions are `public` and only callable by `BURNER_ROLE`
Although `DpxEthToken.sol` contract have `pause()` and `unPause()` functions they are not implemented in this cases.
*Instances(3)*
```
File: contracts/dpxETH/DpxEthToken.sol
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L37-L39
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L41-L45
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L47-L53
```

### [Low-04] While setting `fee` or `slippageTolerance` there is no checks for higher limit or pre-existing value check
There should be 2 checks
   - First, inputed value should not exceeding higher limit value
   - Second, inputed value should not equal to existing state variable value
```solidity
  function setLiquiditySlippageTolerance(
    uint256 _liquiditySlippageTolerance
  ) external onlyRole(DEFAULT_ADMIN_ROLE) { 
    require(
      _liquiditySlippageTolerance > 0,
      "reLPContract: liquidity slippage tolerance must be greater than 0"
    );

+    require(
+     _liquiditySlippageTolerance > UPPER_BOUND, // A value that set in state 
+     "reLPContract: liquidity slippage tolerance must be lesser than HIGHER_BOUND"
+   );

+   uint256 val = liquiditySlippageTolerance;
+   require(val != _liquiditySlippageTolerance, 'err');

    liquiditySlippageTolerance = _liquiditySlippageTolerance;

+   emit(val, _liquiditySlippageTolerance);
  }
```
```solidity
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
```solidity
  function setRdpxBurnPercentage(
    uint256 _rdpxBurnPercentage
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_rdpxBurnPercentage > 0, 3); // @audit no upper bound
    rdpxBurnPercentage = _rdpxBurnPercentage;
    emit LogSetRdpxBurnPercentage(_rdpxBurnPercentage); // @audit should include old and new
  }
```
*Instances(3)*
```
File: contracts/core/RdpxV2Core.sol
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L180-L186
```

### [Low-05] `PerpetualAtlanticVault.sol` contract will be only compatible with other ERC20 with decimal 18, if other used as collateral then `FundingRate` calculating gives wrong result
In `_updateFundingRate()` for setting funding rate with corresponding pointers calculation as follows
```solidity
 fundingRates[latestFundingPaymentPointer] =
        (amount * 1e18) / 
        (endTime - startTime);
```
Here for precision `1e18` is hardcodedly multiplied with colateralAmount, if less than 18 decimal token used here then result will far more than actual value. 
*Instances(2)*
```
File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L603-L605
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L610-L612
```

### [Low-06] TokenId should be burned when `rdpxAmount` associated with it changed to zero
```solidity
    uint256 bondId = _mintToken(to);
    bonds[bondId] = Bond(to, expiry, rdpxAmount);
```
Bond simply a ERC721 token, where Each tokenId associate with some parameter like receiverAdd, Expiry, rdxAmount 

rdpxAmount associated with a bond can decrease via `decreaseAmount()`, when `rdpxAmount` associated with a bond become 0, coresponding tokenId should be burnt. 
```solidity
  /// @notice Decreases the bond amount
  /// @dev Can only be called by the rdpxV2Core
  /// @param bondId id of the bond to decrease
  /// @param amount amount to decrease
  function decreaseAmount( 
    uint256 bondId,
    uint256 amount
  ) public onlyRole(RDPXV2CORE_ROLE) {
    _whenNotPaused();
    bonds[bondId].rdpxAmount = amount; 
  }
```
```
file: contracts/decaying-bonds/RdpxDecayingBonds.sol
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L121-L122
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L139-L145
```

### [Low-07] There should be a `tokenBalance` check before making token transfer
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
*Instances(2)*
```
File: contracts/amo/UniV2LiquidityAmo.sol
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L168-L171
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L172-L175
```

### [Low-08] `safeApprove()` will be incompatible with token like `USDT`
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
*Instances(2)*
```
File: contracts/amo/UniV2LiquidityAmo.sol
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L200-L207
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L328
```

### [Low-09] There Should be a `require()` which ensure that returned `pool` address from `univ3_factory.getPool()` is not zero address
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
*Instances(1)*
```
File: contracts/amo/UniV3LiquidityAmo.sol
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L100-L107
```

### [Low-10] Return value of low level call not checked only returned
As `execute()` is a external function and its making low level call and its return value not checked,
there should be a require() that revert when low level call failed.
```solidity
  function execute(
    address _to,
    uint256 _value,
    bytes calldata _data
  ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (bool, bytes memory) {
    (bool success, bytes memory result) = _to.call{ value: _value }(_data); // @audit no success checker
    return (success, result);
  }
```
*Instances(1)*
```
File: contracts/amo/UniV3LiquidityAmo.sol
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L344
```

### [Low-11] HardCoded Expiration time in `Swap()` will result in stucking Transaction in mempool for a longer period of time, till that time market sentiment may be drastically changed.
```solidity
    ISwapRouter.ExactInputSingleParams memory swap_params = ISwapRouter
      .ExactInputSingleParams(
        _tokenA,
        _tokenB,
        _fee_tier,
        address(this),
        2105300114, // Expiration: a long time from now // @audit
        _amountAtoB,
        _amountOutMinimum,
        _sqrtPriceLimitX96
      );
```
```
File: contracts/amo/UniV3LiquidityAmo.sol
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L295
```

### [Low-12] No checks for `genesis` time
`genesis` (starting time) could possibly set far Past or far future, there should some sort of time bound present while setting it in constructor.
```solidity
constructor(
......
......

    collateralToken = IERC20WithBurn(_collateralToken);
    underlyingSymbol = collateralToken.symbol();
    collateralPrecision = 10 ** collateralToken.decimals(); 
    genesis = _gensis; 
....
...
  }
```
*Instances(1)*
```
File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L124
```


### [NC-01] Confusing `struct` Name, Should change to other one.
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
```
File: contracts/amo/UniV2LiquidityAmo.sol
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L27-L45
```