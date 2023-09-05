### [L-01] Missing CEI pattern in `provideFunding()` could lead reentrancy risks

`RdpxV2Core#provideFunding()` calculate and transfer the funding amount to the vault. The implementation do not follows the check-effect-interaction pattern which allows the malicious owner to reenter multiple times and draining the contract funds by continuosly provideFunding to the vault. 


```solidity 
  function provideFunding()
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 fundingAmount)
  {
    _whenNotPaused();
    
    uint256 pointer = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
      .latestFundingPaymentPointer();

    _validate(fundingPaidFor[pointer] == false, 16);

    fundingAmount = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
      .payFunding();

    reserveAsset[reservesIndex["WETH"]].tokenBalance -= fundingAmount;

    fundingPaidFor[pointer] = true;

    emit LogProvideFunding(pointer, fundingAmount);
  }
```
`IPerpetualAtlanticVault(addresses.perpetualAtlanticVault).payFunding()` first transfer the collateral to vault then mark `fundingPaidFor[pointer]` to `true`. If the collateral tokens used are hookable such as ERC777s and ERC677, it could be used to reenter the contract over and over before it update the paid status.


At some places the lpVault has taken precaution to avoid reentrancy by such tokens,
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L127-L128

which also confirmed that it may use such tokens in future. Therefore I'm marking the issue as low. 

**Recommendation**
Following the CEI pattern, modify above snippet to below


```solidity 
  function provideFunding()
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 fundingAmount)
  {
    _whenNotPaused();
    
    uint256 pointer = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
      .latestFundingPaymentPointer();

    _validate(fundingPaidFor[pointer] == false, 16);

    reserveAsset[reservesIndex["WETH"]].tokenBalance -= fundingAmount;

    fundingPaidFor[pointer] = true;

    fundingAmount = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
      .payFunding();

    emit LogProvideFunding(pointer, fundingAmount);
  }
```





### [NC-01] `RdpxDecayingBonds#decreaseAmount` not necessarily decreases the bond amount

`RdpxDecayingBonds#decreaseAmount` is called by the `RdpxV2Core` to decrease the bond amount for a bondId. The code below doesn't include any logic to decrease the `rdpxAmount` in the `bonds` mapping; instead, it sets the `rdpxAmount` to a new value. 

The function naming wrongly suggest its functioning, consider changing that.

```solidity
  function decreaseAmount(
    uint256 bondId,
    uint256 amount
  ) public onlyRole(RDPXV2CORE_ROLE) {
    _whenNotPaused();
    bonds[bondId].rdpxAmount = amount;
  }
```
**Recommendation**
Change the function name to `setRdpxAmount`
