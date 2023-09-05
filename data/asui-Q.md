1. call the [```sync()```](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L995C3-L1008C4) function every time an asset is sent from one contract to the **RdpxV2Core** contract.
   Do this by adding **RdpxV2Core.sync()** in the functions that transfers or pulls assets from the RdpxV2Core contract. For example in the **swap** functions in **UniV2LiquidityAMO** and **UniV3LiquidityAMO** contracts. The sync function can be called anytime but very large amount of swaps can cause calculation issues if no one called the sync function in a very long time but this is very unlikely to happen.


2. There are many functions where the admin could rug pull users or send tokens to another accounts:
     * [```function approveContractToSpend```](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L126C3-L135C4) and [``` function emergencyWithdraw```](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L142C3-L153C4) in the ```UniV2LiquidityAmo.sol``` contract.
     * [```function approveTarget```](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L139C1-L153C1) in the ```UniV3LiquidityAmo.sol``` contract.
     * [```function emergencyWithdraw```](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L89C3-L107C4) in the ```RdpxDecayingBonds.sol``` contract.
     * [```function emergencyWithdraw```](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L161C3-L174C1) and [```function approveContractToSpend```](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L403C3-L412C4) in the  ```RdpxV2Core.sol``` contract.

Users of the protocol should be made aware of these admin controlled functions.

3. In the [```swap```](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L274C3-L308C4) function in the ```UniV3LiquidityAmo.sol``` contract the admin can swap any **tokenA** to **tokenB** while the **tokenA** has to be taken from the core contract the **tokenB** can be any other type of token so the admin can swap any token and this token will be sent back to the core contract. So add proper functions to deal with these tokens or restrict the tokens to be swap to only **weth** and **rdpx** like we did in the ```UniV2LiquidityAMO.sol``` contract.
There is also no restriction for the admin to swap so he can swap how much time he likes. 

4. The [```RdpxV2Core.sol```](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol) contract highly depends on the default admin role consider using multisig owner. Owner can also rug pull users by calling the pause function and then withdrawing all the funds.
OR if the private key of the defaultAdminRole is compromised or is lost the protocol will face many issues.
Admin also has the privilege to add or remove addresses like **pricingOracleAddresses**, **AMOAddresses**, etc. 

5. In the [```RdpxV2Core.sol```](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol) contract the admin can call [```function setRdpxBurnPercentage```](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L180C3-L186C4) and [```function setRdpxFeePercentage```](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L193C3-L200C1) to change the ```rdpxBurnPercentage``` and ```rdpxFeePercentage``` respectively but if they forget to add the [```DEFAULT_PRECISION```](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L85C3-L85C51) which is **e18** or miscalculate it, it can cause many issues in the calculations.
Consider adding the ```DEFAULT_PRECISION``` inside the function which will be multiplied by the admin input value if possible. 

6. In the [```function setBondMaturity```](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L228C3-L234C4) in ```RdpxV2Core.sol``` contract the admin can set the ```bondMaturity``` to any value he likes as long as it is greater than 0. Admin can set it to uint256 max or a very large value and those users who bond after this is set cannot redeem their bonds.
Consider adding a check in the ```setBondMaturity``` function so that the admin cannot set the ```bondMaturity``` to more than a restricted value.
Or let the users be aware of this.

7. In the [```function removeAssetFromtokenReserves```](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L270C3-L290C4) in the ```RdpxV2Core.sol``` contract the admin has the privilege to remove any token from the reserve. If the admin removes tokens like **weth** this can cause many issues. 
Users should be made aware of this. OR restrict the admin from removing core tokens like **weth** and **rdpx** atleast.   

8. In the ```RdpxV2Core.sol``` contract the admin can call the [```upperDepeg```](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1051C3-L1071C1) or the [```lowerDepeg```](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1080C3-L1124C4) function to achieve the pegged of rdpx and eth 1:1 ratio but both this functions doesnt check that after they are called and the swap is done it doesn't check that it is actually pegged in the desired ratio. Admin could pass a wrong value and instead of pegging it to 1:1 he could mistakenly or intentionally further the 1:1 desired pegged.
like In the ```upperDepeg``` function there is this check ```_validate(getDpxEthPrice() > 1e8, 10)``` so as long as this check pass admin could pass a very large amount and further make the ratio worse.

9. Wrong error code passed in the [```calculateBondCost```](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1167C6-L1167C43) function in the ```RdpxV2Core.sol``` contract.

10. The [**constructor**](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L82C4-L89C4) in the ```UniV3LiquidityAMO.sol``` contract has hard coded address values. Avoid these if possible.

11. In the ```UniV3LiquidityAmo.sol``` contract in the function [```execute```](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L339C3-L346C4) the function will pass even if the call fails. Try adding require(success) so that the transaction will revert if the call fails.

12. In the [```convertToShares```](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L274C3-L284C4) in ```PerpetualAtlanticVaultLp.sol``` contract return value [```returns (uint256 shares)```](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L276C3-L276C41) is declared but not used. Either use it or remove it.

13. [```import { ERC721 } from "@openzeppelin/contracts/token/ERC721/ERC721.sol";```](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L4C1-L4C74) this import in the **RdpxV2Bond** contract is not needed because the [**ERC721Enumerable**](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L5C1-L5C105) already imports it.
