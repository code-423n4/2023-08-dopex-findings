## G1 
### Double-Check
In RdpxDecayingBonds.mint, onlyRole(MINTER_ROLE) modifier has been used to restrict the function, but another check was done for the same using a require statement. 
```require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");```
This check is no longer necessary as the modifier already checked for it.

## G2
### Redundant Computation 
In ```RdpxV2Core.bondWithDelegate()```, a call is made to ```_bondWithDelegate()```, which  makes call to ```calculateBondCost()```  to compute required rdpx and weth value, in the same ```calculateBondCost()```  function, premium is obtained if ```putOptionsRequired == true``` https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1195-L1198
```
 if (putOptionsRequired) {
      wethRequired += IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
        .calculatePremium(strike, rdpxRequired, timeToExpiry, 0);
    }
```
However, the same computation is done in RdpxV2Core.bondWithDelegate() again here: https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L856-L858
```
  if (putOptionsRequired) {
        premium = _purchaseOptions(returnValues.rdpxRequired);
      }
```
For a more gas-efficient approach, it is advisable to leverage the computed premium value from the initial ```calculateBondCost()``` call and return it alongside the required WETH and rdpx values.