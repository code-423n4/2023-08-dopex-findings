1-Cache Array Length Outside of Loop to save a lot of gas
2-use Postfix increment rather than Prefix increment to reduce gas cost
3-dont initialize value of i as it is 0 by default to save gas

in:
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L120

code:
for (uint i = 0; i < positions_array.length; i++) {

solution:
uint positions_arrayLength = positions_array.length;
for (uint i; i < positions_arrayLength; ++i) {

