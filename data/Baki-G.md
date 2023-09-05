## G-01 Calculate minOut only if minAmount == 0 

## Details 
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L545

If `minAmount` is > 0 , then we should not calculate `minOut` as it is a gas wastage.

## G-02 Only compute strike if needed

### Details
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1189

Only compute `strike` if `putOptionsRequired` == true 