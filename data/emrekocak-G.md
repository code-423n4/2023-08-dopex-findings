## Use constants instead of `type(uint256).max` and `type(uint128).max` 

### Summary
Instead of using `type(uint256).max` and `type(uint128).max` you can use constants like in recommendation. It saves 22-28 gas. 
### Instances
[`blob/main/contracts/core/RdpxV2Core.sol:343`](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L343)
[`blob/main/contracts/core/RdpxV2Core.sol:344`](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L344)
[`blob/main/contracts/core/RdpxV2Core.sol:347`](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L347)
[`blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol:209`](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L209)
[`blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol:247`](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L247)
[`blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol:106`](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L106)
[`blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol:107`](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L107)
[`blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol:155`](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L155)
[`blob/main/contracts/reLP/ReLPContract.sol:152`](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L152)
[`blob/main/contracts/reLP/ReLPContract.sol:157`](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L157)
[`blob/main/contracts/reLP/ReLPContract.sol:162`](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L162)
### Tools Used
Manual, Remix
### Recommendation
```solidity
uint private constant MAX_UINT256_NUMBER = 115792089237316195423570985008687907853269984665640564039457584007913129639935;
uint private constant MAX_UINT256_HEX = 0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff;
uint private constant MAX_UINT256_EXPONENTIATION = 2**256 - 1;

uint private constant MAX_UINT128_NUMBER = 340282366920938463463374607431768211455;
uint private constant MAX_UINT128_HEX = 0xffffffffffffffffffffffffffffffff;
uint private constant MAX_UINT128_EXPONENTIATION = 2**128 - 1;
```

