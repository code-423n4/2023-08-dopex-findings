## Summary
[I-01] Typos.
[I-02] Unused contract.
[I-03] Comment states that function can be called only by `owner` but can be called only by `admin`.
[I-04] Comment does not specify that only `admin` can call function which has `onlyRole` modifier.
[I-05] Use naming in event.
[I-06] Different Natspec convention in files.
[I-07] Different `import` order in files.
[I-08] Different error handling in files.
[I-09] Error codes missmatch.
[I-10] Wrong separation of functions.
[I-11] Function placed in wrong function blocks.
[I-12] Roles not set in constructor.
[I-13] Different `deadline` in files.
[I-14] View functions has named return, but does not use it.

### [I-01] Typos.

```solidity
@audit Uniswap V3
// Uniswamp V3
```

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/amo/UniV3LiquidityAmo.sol#L11

```solidity
@audit _genesis, Genesis
/// @param _gensis Gensis time for funding calculation
```

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/perp-vault/PerpetualAtlanticVault.sol#L112

```solidity
@audit _genesis
uint256 _gensis)
```

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/perp-vault/PerpetualAtlanticVault.sol#L117

```solidity
@audit _genesis
genesis = _gensis;
```

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/perp-vault/PerpetualAtlanticVault.sol#L124

### [I-02] Unused contract.
In `UniV3LiquidityAmo.sol` there is `abstract contract OracleLike` that is not used anywhere. Consider removing it if not used.

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/amo/UniV3LiquidityAmo.sol#L22-L26

### [I-03] Comment states that function can be called only by `owner` but can be called only by `admin`.
In the Natspec of some function, it is stated that this function can be called only by the `owner`. However the only modifier of this function is `onlyRole(DEFAULT_ADMIN_ROLE)`. It is better to change the `owner` to `admin` in the comments to avoid confusion.

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/amo/UniV2LiquidityAmo.sol#L139

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/core/RdpxV2Core.sol#L142

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/core/RdpxV2Core.sol#L150

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/core/RdpxV2Core.sol#L158

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/decaying-bonds/RdpxDecayingBonds.sol#L82

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/perp-vault/PerpetualAtlanticVault.sol#L134

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/perp-vault/PerpetualAtlanticVault.sol#L142

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/perp-vault/PerpetualAtlanticVault.sol#L150

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/perp-vault/PerpetualAtlanticVault.sol#L161

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/perp-vault/PerpetualAtlanticVault.sol#L172

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/perp-vault/PerpetualAtlanticVault.sol#L216

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/perp-vault/PerpetualAtlanticVault.sol#L235

### [I-04] Comment does not specify that only `admin` can call function which has `onlyRole` modifier.
In the Natspec of some functions that have `onlyRole(DEFAULT_ADMIN_ROLE)` modifier, it is not said that only the `admin` can execute them.

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/amo/UniV2LiquidityAmo.sol#L183

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/amo/UniV2LiquidityAmo.sol#L253

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/amo/UniV2LiquidityAmo.sol#L299

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/core/RdpxV2Core.sol#L1047

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/core/RdpxV2Core.sol#L1074

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/perp-vault/PerpetualAtlanticVault.sol#L243

### [I-05] Use naming in event.
The event `log` in `UniV3LiquidityAmo.sol` does not specify the name of the parameter emmited.

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/amo/UniV3LiquidityAmo.sol#L379

### [I-06] Different Natspec convention in files.
Some files follow one Natspec convention and other use different convention. Consider using the same Natspec convention in all files.

### [I-07] Different `import` order in files.
In some files the order of imports is the following `Contracts, Interfaces, Libraries` and in some is the following `Libraries, Contracts, Interfaces`. Consider using the same import order in all files

### [I-08] Different error handling in files.
The files `RdpxV2Core.sol` and `PerpetualAtlanticVault.sol` use the function `_validate` to revert if a certain condition is not met. All the other files use `require/revert` without this function. Consider using the same error handling method in all files.

### [I-09] Error codes missmatch.
The files `RdpxV2Core.sol` and `PerpetualAtlanticVault.sol` have error codes used for reverting. However they do not match. For example in one of the files the `Zero address` code is `E17` and in the other it is `E1`. Also the error messages for similar validations are not the same. Consider using the same codes and error messages in both files for better code readability.

### [I-10] Wrong separation of functions.
In the file `PerpetualAtlanticVaultLP.sol` there are two function block. One is called `VIEWS` and the other is `PUBLIC VIEW FUNCTIONS`. However all the functions in both blocks are doing the same, they are all `public view` functions. Consider combining them into one block to remove confusion.

### [I-11] Function placed in wrong function blocks.
In `PerpetualAtlanticVaultLP.sol` the function `beforeWithdraw` is placed into `PUBLIC VIEW FUNCTIONS` block. However this function is `internal` and is not a `view` function. Consider moving the function elsewhere.

### [I-12] Roles not set in constructor.
The roles `RDPXV2CORE_ROLE` and `BURNER_ROLE` are not set in constructors. The `RDPXV2CORE_ROLE` is not set in `RdpxDecayingBonds.sol` and `PerpetualAtlanticVault.sol`. The role `BURNER_ROLE` is not set in `DpxEthToken.sol`. I know that the `admin` can set them later, but I decided to include this as an `informational`, bacause all the other roles are set in the constructor. Even in `ReLPContract.sol` the `RDPXV2CORE_ROLE` is set in the constructor.

### [I-13] Different `deadline` in files.
In some files for the `deadline`, `block.timestamp` is used. However in other files we see different values used for the `deadline` like `type(uint256).max`, `block.timestamp + 1`, `2105300114` and `block.timestamp + 10`. Consider using only one of the following to avoid confusion.

### [I-14] View functions has named return, but does not use it.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L566

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L576