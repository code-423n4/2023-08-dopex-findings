1. call the ```sync()``` function every time an asset is sent from one contract to the **RdpxV2Core** contract.
   Do this by adding **RdpxV2Core.sync()** in the functions that transfers or pulls assets from the RdpxV2Core contract. For example in the **swap** functions in **UniV2LiquidityAMO** and **UniV3LiquidityAMO** contracts. The sync function can be called anytime but very large amount of swaps can cause calculation issues if no one called the sync function in a very long time but this is very unlikely.


2. There are many functions where the admin could rug pull users or send tokens to another accounts:
     * ```function approveContractToSpend``` and ``` function emergencyWithdraw``` in the ```UniV2LiquidityAmo.sol``` contract.
     * ```function approveTarget``` in the ```UniV3LiquidityAmo.sol``` contract.
     * ```function emergencyWithdraw``` in the ```RdpxDecayingBonds.sol``` contract.
     * ```function emergencyWithdraw``` and ```function approveContractToSpend``` in the  ```RdpxV2Core.sol``` contract.

Users of the protocol should be made aware of these admin controlled functions. 

3. The ```RdpxV2Core.sol``` contract highly depends on the default admin role consider using multisig owner. Owner can also rug pull users by calling the pause function and then withdrawing all the funds.
OR if the private key of the defaultAdminRole is compromised or is lost the protocol will face many issues.
Admin also has the privilege to add or remove addresses like **pricingOracleAddresses**, **AMOAddresses**, etc. 

4. In the ```RdpxV2Core.sol``` contract the admin can call ``` function setRdpxBurnPercentage``` and ```function setRdpxFeePercentage``` to change the ```rdpxBurnPercentage``` and ```rdpxFeePercentage``` respectively but if they forget to add the ```DEFAULT_PRECISION``` which is **e18** it can cause many issues in the calculations.
Consider adding the ```DEFAULT_PRECISION``` inside the function which will be multiplied by the admin input value if possible. 

5. In the ```function setBondMaturity``` in ```RdpxV2Core.sol``` contract the admin can set the ```bondMaturity``` to any value he likes as long as it is greater than 0. Admin can set it to uint256 max or a very large value and those users who bond after this is set cannot redeem their bonds.
Consider adding a check in the ```setBondMaturity``` function so that the admin cannot set the ```bondMaturity``` to more than a restricted value.
Or let the users be aware of this.

6. In the ```function removeAssetFromtokenReserves``` in the ```RdpxV2Core.sol``` contract the admin has the privilege to remove any token from the reserve. If the admin removes tokens like **weth** this can cause many issues. 
Users should be made aware of this. OR restrict the admin from removing core tokens like **weth** and **rdpx** atleast.   

7. 
