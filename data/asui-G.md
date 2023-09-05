1. In the ```calculateBondCost``` function in [**```RdpxV2Core.sol```**](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1189C4-L1198C6) contract declare the ```strike``` and ```timeToExpiry``` only after the ```if (putOptionsRequired)``` statement to safe gas when the function is called when putOptionsRequired = false.

2. Consider adding custom reverts instead of require strings. There are many of them in the UniV2AMO, V3amo contracts and others.

3. Remove the declared return variable ```returns (uint256 shares)``` in the [```convertToShares```](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L274C3-L284C4) function in the **PerpetualAtlanticVaultLp** contract since it is not used.

4. In the [```emergencyWithdraw```](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L148C7-L150C6) function in the ```UniV2LiquidityAMO.sol``` contract do this ```IERC20WithBurn(tokens[i]).safeTransfer(msg.sender, token.balanceOf(address(this)));``` instead of doing this: ```token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));```
It will safe more gas and since this is called inside a for loop it will even safe more gas.
This is also present in the [```emergencyWithdraw```](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L168C6-L170C6) function in the **RdpxV2Core** contract.

5. No need to declare 0 here:[```uint256 public latestFundingPaymentPointer = 0;```](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L89C3-L89C50) in the ```PerpetualAtlanticVault.sol``` since the default value is already 0.

6. Inside the [```_bondWithDelegate```](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L732C4-L743C7) function in the core contract ```(uint256 amount1, uint256 amount2) = _calculateAmounts(wethRequired, rdpxRequired, _amount, delegate.fee);``` this is called and then ```uint256 bondAmountForUser = amount1;``` this is done. Instead of declaring a new variable you can directly use the amount1 or change the amount1 to bondAmountForUser to save gas.

7. Instead of going through all the process and the for loop and finally check that the delegateId have the required amount in the [```bondWithDelegate```](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L819C3-L885C4) function in the core contract it would be more cost effective if the check is done first in the for loop. 
Add ```_validate(_delegateIds[i] >= _amounts[i], *revert code*);``` inside the for loop.  
