## Impact
State variables should be declared immutable to save gas cost

SLOC
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L35C1-L37C35

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L69C1-L72C29
```
In fact those address in constructor also can be set as immutable.

## Mitigation
Change variables to immutable