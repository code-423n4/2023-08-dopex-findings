# [L-01] Redundant Usage of `SafeMath` Library in Solidity 0.8.19
## Lines of code
[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L29](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L29)         
[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L24](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L24)       
[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L27](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L27)        
## Impact
The `UniV3LiquidityAmo`/`UniV2LiquidityAmo`/`ReLPContract` contracts currently import and utilize the `SafeMath` library for arithmetic operations. However, starting from Solidity version 0.8.0, the language has introduced built-in overflow and underflow protection for arithmetic operations, making the `SafeMath` library redundant and unnecessary. This unnecessary use of SafeMath in a contract targeting Solidity version 0.8.19 can result in increased gas costs, added complexity, and potential confusion among developers who are already acquainted with the improved Solidity features.
## Tools Used
Manual Review
## Recommended Mitigation Steps
To address this issue and optimize the contract, it is strongly advised to:
1. Remove `SafeMath` Import: Eliminate the import statement for the `SafeMath` library from the `UniV3LiquidityAmo`/`UniV2LiquidityAmo`/`ReLPContract` contracts.
2. Remove `SafeMath` Usage: Remove any usage of `SafeMath` functions for arithmetic operations within the contract, relying on Solidity's native arithmetic checks.
# [L-02] Missing Check for Zero Address in `sortTokens` Function
## Lines of code
[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/uniswap_V2/libraries/UniswapV2Library.sol#L16-L23](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/uniswap_V2/libraries/UniswapV2Library.sol#L16-L23)
## Impact
The `sortTokens` function in the contract lacks a require statement to check that `token1` (the larger of the two token addresses) is not the zero address (`address(0)`). This omission could potentially lead to unexpected behavior or vulnerabilities if token1 happens to be the zero address.
## Proof of Concept
[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/uniswap_V2/libraries/UniswapV2Library.sol#L16-L23](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/uniswap_V2/libraries/UniswapV2Library.sol#L16-L23)
````solidity
function sortTokens(address tokenA, address tokenB) internal pure returns (address token0, address token1) {
    require(tokenA != tokenB, "IDENTICAL_ADDRESSES");
    (token0, token1) = tokenA < tokenB ? (tokenA, tokenB) : (tokenB, tokenA);
    require(token0 != address(0), "ZERO_ADDRESS");
    //@audit-issue M:should we not 'require(token1 != address(0), "ZERO_ADDRESS");'?
}
````
## Tools Used
Manual Review
## Recommended Mitigation Steps
Change to:
````solidity
function sortTokens(address tokenA, address tokenB) internal pure returns (address token0, address token1) {
    require(tokenA != tokenB, "IDENTICAL_ADDRESSES");
    (token0, token1) = tokenA < tokenB ? (tokenA, tokenB) : (tokenB, tokenA);
    require(token0 != address(0), "ZERO_ADDRESS");
    require(token1 != address(0), "ZERO_ADDRESS");
}
````