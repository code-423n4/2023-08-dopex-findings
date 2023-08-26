# Unnecessary addition for deadlines

block.timestamp would work as a deadline as it will be executed in the same transaction

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/amo/UniV2LiquidityAmo.sol#L231

```solidity
        block.timestamp + 1
```

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/amo/UniV2LiquidityAmo.sol#L279

```solidity
        block.timestamp + 1
```

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/core/RdpxV2Core.sol#L1103

```solidity
        block.timestamp + 10
```

# Unnecessary caching of calldata variable

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/amo/UniV2LiquidityAmo.sol#L145-L148

```solidity
    IERC20WithBurn token;

    for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
```