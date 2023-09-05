# Missing Validation on upper bound
## Code Position
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L180
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L193
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L455

## Description
The following setter functions are missing upper bounds, which may cause unexpected functionalities:
1. Both `RdpxBurnPercentage` and `RdpxFeePercentage` should be less than 100 * precision and the sum of these two values should not be larger than 100 * precision. Exceeded value may cause error in `_transfer` function.
2. `slippageTolerance` should be less than 100 * precision.

## Recommendation
Recommend adding upperbound when setting values above to avoid unexpected functionalities.

# Unused slippage variable 
## Code Position
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L109C1-L117C4

## Description
The `slippageTolerance` value is not being used in the UniV2LiquidityAMO contract, as a result, the slippage control relies on the function input, for example, `token2AmountOutMin` in swap function.

```solidity
  function setSlippageTolerance(
    uint256 _slippageTolerance
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(
      _slippageTolerance > 0,
      "reLPContract: slippage tolerance must be greater than 0"
    );
    slippageTolerance = _slippageTolerance;
  }
```
## Recommendation
Recommend removing unused contract component, or complete functionality with slippage control.

# Missing Validation on Duplicate AMO address
## Code Position
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L378

## Description
The privileged function `addAMOAddress` is being used to update the `amoAddresses` list by `DEFAULT_ADMIN_ROLE`.

However, there is no limitation on the duplicate address when adding a new AMO address.

## Recommendation
It is recommend to adding duplicate filter when adding new AMO addresses.

# Unused Contract Component
## Code Position
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L22C1-L26C2

## Description
The following abstract contract is not being used in the current `UniV3LiquidityAMO` contract:
```solidity=
  abstract contract OracleLike {
    function read() external view virtual returns (uint);
  
    function uniswapPool() external view virtual returns (address);
  }
```

## Recommendation
Recommend removing unnecessary code component.

# Unnecessary Minter Validation
## Code Position
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L120

## Description
In the contract `RdpxDecayingBonds`, the mint function is intended to be invoked by the minter, however, the minter has been checked twice through `onlyRole` and `require` statement:
```solidity
  function mint(
    address to,
    uint256 expiry,
    uint256 rdpxAmount
  ) external onlyRole(MINTER_ROLE) {
    _whenNotPaused();
    require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
    uint256 bondId = _mintToken(to);
    bonds[bondId] = Bond(to, expiry, rdpxAmount);

    emit BondMinted(to, bondId, expiry, rdpxAmount);
  }

```

## Recommendation
Recommend removing unnecessary validation for the gas/runtime optimiaztions.

# Unnecessary `_whenNotPaused` in `mint`
## Code Position
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L119

## Description
The `mint` function includes the `_whenNotPaused` function to halt its operation when the contract is paused. However, this is redundant, as the `_beforeTokenTransfer` method already provides this functionality. The following call stack helps clarify the overlap:
```text
mint
-> _mintToken
  -> _mint
    -> _beforeTokenTransfer
```

## Recommendation
Recommend removing redundant checks to optimize the codebase.