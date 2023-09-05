# [NC-01] Missing Validation Checks for tokenA and tokenB in **approveContractToSpend** function 

Since the contract is only dealing with weth and rdpx tokens , so all the transactions that happen must involve those tokens only. Please consider adding a check for checking if the token address passed is either tokenA or tokenB

```  
function approveContractToSpend(
    address _token,
    address _spender,
    uint256 _amount
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(_token == addresses.tokenA || _token ==addresses.tokenB )
    require(_token != address(0), "reLPContract: token cannot be 0");
    require(_spender != address(0), "reLPContract: spender cannot be 0");
    require(_amount > 0, "reLPContract: amount must be greater than 0");
    IERC20WithBurn(_token).approve(_spender, _amount); 

  }

```
Link : https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L126-L135

# [NC-02] Consider using addLiquidityETH instead of addLiquidity 

Since the contract is dealing with weth , its suggested to use addLiquidityETH instead of using addLiquidity . Its also given in the uniswap docs ( https://docs.uniswap.org/contracts/v2/guides/smart-contract-integration/providing-liquidity#:~:text=use%20addLiquidity.%20If%20WETH%20is%20involved%2C%20use%20addLiquidityETH )

Link : https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L223

# [NC-03] Adding Checks for avoiding slippage 

While adding liquidity , consider adding a check for amount0Min and amount1Min to check that they are greater than 0. This will help avoid slippage for while adding liquidity . Consider adding this check 
```    
require(_amount0Min!=0 && _amount1Min!=0);
```

Link : 

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L223

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L178-L191



