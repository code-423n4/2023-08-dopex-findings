# missing `IRdpxV2Core(rdpxV2Core).sync();` in UniV2LiquidityAmo._sendTokensToRdpxV2Core
In [UniV3LiquidityAmo._sendTokensToRdpxV2Core](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L353-L364), while `_sendTokensToRdpxV2Core` is called, the function will call `IRdpxV2Core(rdpxV2Core).sync();` to sync the balance  of tokens, so I think it's better to call the same function in [UniV2LiquidityAmo._sendTokensToRdpxV2Core](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L160C12-L178)

# Incorrect event messages in UniV2LiquidityAmo.sol which should using `"UniV2LiquidityAmo"` instead of `"reLPContract"`
In UniV2LiquidityAmo.sol, there are some incorrect event messages with string as __"reLPContract"__, which I think should be changed to __"UniV2LiquidityAmo"__
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L91
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L114C8-L114C20
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L131-L133

# function `UniV3LiquidityAmo.removeLiquidity` emits useless event
At the end of function [UniV3LiquidityAmo.removeLiquidity](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L213-L270), an log was emitted in [L269](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L269), but since `positions_mapping[pos.token_id]` has been deleted in [L263](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L263), which means `positions_mapping[pos.token_id].token_id` will be __0__ all the time, so it's useless.

# `UniV3LiquidityAmo.execute` can't send ETH
Function [UniV3LiquidityAmo.execute](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L339-L346) uses `_value` to stand for the ETH the contract will send. But by going through all the contract, there is no function which can receive ETH, which means `_value` will be __0__ all the time, so the `_value` parameter is useless

# RdpxV2Core.sol missing `approveContractToSpend` function for ERC721
In RdpxV2Core.sol, [RdpxV2Core.approveContractToSpend](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L403C12-L412) is defined to approve ERC20 tokens, but the contract missing similar function to approve ERC721.
According to [UniV3LiquidityAmo.recoverERC721](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L324-L336), in emergency case, `UniV3LiquidityAmo.recoverERC721` will transfer all ERC20 and ERC721 to `contract RdpxV2Core`, without `approveContractToSpend` for ERC721,  some ERC721 might be locked in `RdpxV2Core.sol`

# function `RdpxV2Core.approveContractToSpend` might not work for some token
Some tokens (e.g. USDT, KNC) do not allow approving an amount M > 0 when an existing amount N > 0 is already approved. This is to protect from an ERC20 attack vector described here.
But according to [RdpxV2Core.approveContractToSpend](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L403-L412), the function will [revert](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L410) if __0__ is approved, which is against USDT type token

# `RdpxV2Core.calculateBondCost`'s condition isn't good enough
Function [RdpxV2Core.calculateBondCost](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1156C12-L1199) has a condition check as [bondDiscount < 100e8](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1167), but the condition is not good enough, because [L1170](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1170C11-L1170C33) requires `bondDiscount <= 2 *RDPX_RATIO_PERCENTAGE`, otherwise the code will revert.

# outdated comment for `RdpxV2Core.upperDepeg`
As I talked to the dev, [the comment](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1047) isn't correct, and the correct comments should be `@notice Lets users mint DpxEth at a 1:1 ratio when DpxEth pegs above 1 of the ETH token` 

# `RdpxDecayingBonds.emergencyWithdraw` can't send ETH
Function [RdpxDecayingBonds.emergencyWithdraw](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L89-L107) uses `_value` to stand for the ETH the contract will send. But by going through all the contract, there is no function which can receive ETH, which means `_value` will be __0__ all the time, so the `_value` parameter is useless

# Loss of precision in `PerpetualAtlanticVault.settle`
In function [PerpetualAtlanticVault.settle](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L315-L369), the `ethAmount` is calculated as in [L335](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L335):
```solidity
    for (uint256 i = 0; i < optionIds.length; i++) {
      uint256 strike = optionPositions[optionIds[i]].strike;
      uint256 amount = optionPositions[optionIds[i]].amount;

      // check if strike is ITM
      _validate(strike >= getUnderlyingPrice(), 7);

      ethAmount += (amount * strike) / 1e8; <-------------- HERE
      rdpxAmount += amount;
      optionsPerStrike[strike] -= amount;
      totalActiveOptions -= amount;

      // Burn option tokens from user
      _burn(optionIds[i]);

      optionPositions[optionIds[i]].strike = 0;
    }
```
Because solidity doesn't support floating point numbers, [L335](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L335) will lose precision,and it costs more gas
I think we can change it to the following code
```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
index 88ac2b3..12b9693 100644
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
@@ -332,7 +333,7 @@ contract PerpetualAtlanticVault is
       // check if strike is ITM
       _validate(strike >= getUnderlyingPrice(), 7);
 
-      ethAmount += (amount * strike) / 1e8;
+      ethAmount += amount ;
       rdpxAmount += amount;
       optionsPerStrike[strike] -= amount;
       totalActiveOptions -= amount;
@@ -342,7 +343,8 @@ contract PerpetualAtlanticVault is
 
       optionPositions[optionIds[i]].strike = 0;
     }
-
+    
+    ethAmount = ethAmount * strike / 1e8;
     // Transfer collateral token from perpetual vault to rdpx rdpxV2Core
     collateralToken.safeTransferFrom(
       addresses.perpetualAtlanticVaultLP,

```

# ReLPContract.reLP missing slippage protection
In [L291-L292](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L291-L292), `0` is used as min amount of output tokens, which might be suffering sandwich attacks

