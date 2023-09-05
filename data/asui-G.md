1. In the ```calculateBondCost``` function in [**```RdpxV2Core.sol```**](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1189C4-L1198C6) contract declare the ```strike``` and ```timeToExpiry``` only after the ```if (putOptionsRequired)``` statement to safe gas when the function is called when putOptionsRequired = false.

2. Consider adding custom reverts instead of require strings. There are many of them in the UniV2AMO, V3amo contracts and others.

3. Remove the declared return variable ```returns (uint256 shares)``` in the ```convertToShares``` function in the **PerpetualAtlanticVaultLp** contract since it is not used.

4. In the ```emergencyWithdraw``` function in the ```UniV2LiquidityAMO.sol``` contract do this ```IERC20WithBurn(tokens[i]).safeTransfer(msg.sender, token.balanceOf(address(this)));``` instead of doing this: ```token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));```
It will safe more gas and since this is called inside a for loop it will even safe more gas.

