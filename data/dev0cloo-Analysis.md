## Codebase quality analysis
The codebase is well written and important functions have meaningful comments. However the codebase uses  a lot of "roundabout" to achieve some things. For example, there is a check in `addRdpx` function in `PerpetualAtlanticVaultLP` to make sure the Rdpx collateral is the same as the balance of the contract. This check prevents even the `PerpetualAtlanticVault` from calling the function directly and updating the collateral without first sending rdpx to the contract. This sending of rdpx could easily exist in the `addRdpx` function and allow direct updating of the collateral after transfer instead of the transfer hiding in the `settle` function of `PerpetualAtlanticVault`. 

## Architecture Improvements

### Consider allowing users to bond with both decaying bonds and rDPX balance
Decaying bonds are minted as incentives or as rebate to users who suffer losses on other Dopex products. If users try to bond with a decaying bond, they can’t bond more than the rDPX allowance of that decaying bond. Hence a situation where the user wants to use the decaying bond to save some rDPX after suffering a loss and top up the remainder of the rDPX required for a bond from their own balance is impossible. This greatly reduces the utility of the decaying bonds of the protocol. Ref: `RdpxV2Core.sol`

### Consider using ERC4626 for PerpetualAtlanticLP Vault
The ERC4626 standard introduces tokenized vaults. The current LP vault contract acts like an ERC4626(minting and burning shares among others) but it isn't. Upgrading the contract to use ERC4626 will make future integration of the contract with other services that support ERC4626 vaults a lot easier.

### Consider using a Decentralised Oracle for price feeds
Unless absolutely impossible, it is best if decentralised Oracle services like Chainlink are used for price feeds since they are more resistant to oracle manipulations. 

## Centralization Risks

### Using a custom Oracle presents a single point of failure
Dopex products use their own custom oracles to determine the prices of various products. These oracles are vulnerable to oracle manipulation exploits especially when liquidity is low in AMMs like UniSwapV3, which the protocol uses for as an AMO.

### Emergency Withdraw by Default Admin can be abused
While the protocol states that the default admin is to be trusted, it still opens up the possibility of such privileges being abused. 

## Systemic risks

### Using 1e8 for precision can eventually lead to rounding errors
The protocol uses 1e8 as the precision for calculating almost all of it's values. This becomes a risk when conversion operations include the collateral(WETH) which uses 1e18 precision. This can lead to undesirable situations involving prices of assets for users of the protocol.


## Time Spent
- Day 1: Read through docs for a high level understanding of the protocol
- Day 2: Manual review of codebase
- Day 3: Finish up analysis

### Time spent:
48 hours