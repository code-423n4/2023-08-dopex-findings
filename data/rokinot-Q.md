# Low

### UniV2LiquidityAMO: Balance function may return wrong value

[`getLpTokenBalanceInWeth()`](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L372-L374) does not call `sync()` while retrieving the amount of LP token balance in the contract, which leads to the contract possibly returning misleading values. Note that `sync()` is only called by EOA's or the core contract while a user is bonding, and when reLP'ing is active.

### UniV2LiquidityAMO: Slippage tolerance is never enforced

`slippageTolerance` is a variable in the contract which sets the slippage for swaps. There's a setting function [here](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L109-L117) but this variable is not used elsewhere, meaning it's useless. Swaps will depend on the input `token2AmountOutMin` the admin sends in order to have proper slippage.

### UniV3LiquidityAMO: Contract will stop working in 2036

In the `swap()` function, the `ExactInputSingleParams()` is sending the value `2105300114` as deadline. The contract will cease it's most important function in September 17th, 2036 as the swap function will revert past this date since the deadline will be lower than the current time.

```
    ISwapRouter.ExactInputSingleParams memory swap_params = ISwapRouter
      .ExactInputSingleParams(
        _tokenA,
        _tokenB,
        _fee_tier,
        address(this),
        2105300114, // Expiration: a long time from now
        _amountAtoB,
        _amountOutMinimum,
        _sqrtPriceLimitX96
      );
```

Given smart contract swaps are atomic, and frontrunning cannot be executed in Arbitrum due to a centralized sequencer, the protocol could consider simply adding the current timestamp as parameter (or maybe an even higher value).

### UniV3LiquidityAMO: `recoverERC721()` does not, in fact, recovers them

The function to recover ERC721's (found [here](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L324-L336)) sends them to the rDPX V2 core contract, however said contract has no function to retrieve them, rendering the function useless and losing the token forever. If we take a closer look at `emergencyWithdraw()` shown below, we'll see it cannot properly retrieve ERC721's as they only function with ERC20's.

```
  function emergencyWithdraw(
    address[] calldata tokens
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _whenPaused();
    IERC20WithBurn token;

    for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }

    emit LogEmergencyWithdraw(msg.sender, tokens);
  }
```

The comment below can be found in the recovery function:

```
    // Only the owner address can ever receive the recovery withdrawal
```

Which leaves me to assume this was an accident. The V2 core contract inherits `ERC721Holder` which only includes the `onERC721Received()` function. The transferring to the core contract will succeed but that's as far as it goes. In order to fix this, consider sending this recovered asset to an owner's EOA address instead.

### RdpxV2Core: Admin can break bonding by setting fee and/or burn percentage to > 100%

According to the documentations and the code settings, 50% of rDPX received is burned while 50% is sent to veDPX holders. These are enforced though the `rdpxFeePercentage` and `rdpxBurnPercentage`. Their respective set functions are lacking the validation checks that guarantees the sum of both won't exceed to 100%

```
  function setRdpxBurnPercentage(
    uint256 _rdpxBurnPercentage
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_rdpxBurnPercentage > 0, 3);
    rdpxBurnPercentage = _rdpxBurnPercentage;
    emit LogSetRdpxBurnPercentage(_rdpxBurnPercentage);
  }

  function setRdpxFeePercentage(
    uint256 _rdpxFeePercentage
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_rdpxFeePercentage > 0, 3);
    rdpxFeePercentage = _rdpxFeePercentage;
    emit LogSetRdpxFeePercentage(_rdpxFeePercentage);
  }
```

Right now the only guarantee is neither will be zero. If their percentages exceed 100%, the core contract will either exceedingly burn it's own tokens necessary to later purchase put options or send these tokens to the fee distributor. If they're below 100% then the contract leaves some dust behind.

### PerpetualAtlanticVaultLP: Redemptions are temporarily locked if all collateral is consumed

In the event all of the collateral available in the vault LP is used to purchase options, the vault will be left with 0 WETH. This means users won't be able to withdraw due to the line below

```
    // Check for rounding error since we round down in previewRedeem.
    require(assets != 0, "ZERO_ASSETS");
```

This will last until the next funding payment, which can take up to a week. Consider allowing users to withdraw only rDPX tokens if this is the case.

### RdpxV2Core: Bond maturity is initially set to zero

[The `bondMaturity` variable is not set in it's declaration](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L106), so it begins with a zero value. It's not set in the constructor neither, which means bonds which instantly mature can be created until the admin calls `setBondMaturity()`, a function that checks this maturity is higher than zero. This also goes against the documentation:

`- rDPX V2 Bond Maturity: The bond maturity of the rDPX V2 Bond. Initially to be set to 5 days.`

# Non-critical

### Misleading comment in RdpxV2Core

It can be found [here](
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1047)

```Lets users mint DpxEth at a 1:1 ratio when DpxEth pegs above 1.01 of the ETH token```. Only administrators can call upperDepeg.

### Two functions are not using proper camelCase

[addAssetTotokenReserves](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L240) and [removeAssetFromtokenReserves](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L270)

### Redundant check in RdpxV2Core

[This validation check](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L984) is redundant, as the line below, executed before the validation check, reverts due to underflow:

```
    amountWithdrawn = delegate.amount - delegate.activeCollateral;
```

### Maximum bondDiscount is 50% rather than 100% in RdpxV2Core

In the `calculateBondCost()` function, the bond discount is verified to ensure it's below 100%. According to a DM I had with one of the sponsors, the discount factor should be set around 2%-4%, which means this is unlikely to be reached. The function logic however reverts in case discount is higher than 50% in [this line](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1170).

This happens because `RDPX_RATIO_PERCENTAGE` is 25%, and it's subtracted from the bond discount divided by 2. This makes the validation check in line 1167 redundant.