Create an error when the first value is bigger than the second one.

Add this piece of code.
```
_validate(delegate.amount - delegate.activeCollateral, 15);
```

Otherwise tx could just revert without any error message

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L983