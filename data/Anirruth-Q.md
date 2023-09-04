## Spelling error:
change from ot -> to
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1073
change from lattest to latest
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/IPerpetualAtlanticVault.sol#L79

## Comment deviate from the logic:
`/// @notice Decreases the bond amount
  /// @dev Can only be called by the rdpxV2Core
  /// @param bondId id of the bond to decrease
  /// @param amount amount to decrease
  function decreaseAmount(
    uint256 bondId,
    uint256 amount
  ) public onlyRole(RDPXV2CORE_ROLE) {
    _whenNotPaused();
    bonds[bondId].rdpxAmount = amount;
  }`
From the amount param it is written amount to decrease but that is not the case , it should be written as "amount to decrease to" as the amount is not being decreased from the bond,but decreased to the inputted amount
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L135-L145