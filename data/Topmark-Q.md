1. https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1172
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1181
```solidity
 rdpxRequired =
        ((RDPX_RATIO_PERCENTAGE - (bondDiscount / 2)) *
          _amount *
          DEFAULT_PRECISION) /
        (DEFAULT_PRECISION * rdpxPrice * 1e2);
```
DEFAULT_PRECISION is 1e8 and multiplying it with the numerator and denominator in this context has no use to prevent precision lose and even gives higher chances of DOS due to multiplication overflow in numerator and it also uses more gas. My report is simply to say 30_000/2_000 is same and better written as 30/2, both gives 15 but the latter is more readable and efficient. A second instance of this can be found in L1181 of the same RdpxV2Core.sol contract