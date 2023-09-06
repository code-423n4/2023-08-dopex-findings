# L-01 :  Have a nonreentrant modifier in `emergencyWithdraw` function 
The `emergencyWithdraw` function in RpdxDecayingBonds.sol deals with eth transfers to an externel account . Exteranal calls with may reenter to the contract before state changes properly .  However , having a nonreentrant modifier is always preferred . 
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L89
# L-02: `RDPXV2CORE_ROLE` is not set in the constructor in ` PerpetualAtlanticVault.sol ` . 
`RDPXV2CORE_ROLE` has some crucial functionality in  ` PerpetualAtlanticVault.sol ` contract . Having it set in the constructor should be done while deploying the contracts . However , it is not set in the constuctor. 
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L113

# L-03 : `_convertToAssets ` should be public . 
The function `_convertToAssets ` is to check the  amount of assets for a number of shares . This is a view function . However , this should be Public function in my opinion cause users may have to check the return amount  of assets for their corresponding shares . 
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L218






# NC-01 :Lack of documentation in the code 
 The whole codebase lacks documentation . This issue makes it difficult to understand the code's purpose, functionality, and usage, which can lead to confusion, errors, and challenges when maintaining or extending the codebase. Proper documentation is crucial for code readability, collaboration, and long-term maintainability.

# NC-02 : Function naming is not accurate of `decreaseAmount` function in RpdxDecayingBonds.sol 
The function is intended to decrease the amount of `rdpxAmount ` .  However ,  the function can be used to increase the `rdpxAmount ` of a bond as it's configuration . So ,  the function should be named as `changeAmount` rather than `decreaseAmount` .
```solidity
function decreaseAmount(
    uint256 bondId,
    uint256 amount
  ) public onlyRole(RDPXV2CORE_ROLE) {
    _whenNotPaused();
    bonds[bondId].rdpxAmount = amount; 
  }
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L139

