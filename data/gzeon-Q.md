# Duplicated AMO Entry can be added

https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/core/RdpxV2Core.sol#L373-L381

```solidity
  /**
   * @notice Adds a AMO contract to the AMO address array
   * @dev    Can only be called by admin
   * @param  _addr the address to add to the AMO address array
   */
  function addAMOAddress(address _addr) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_addr != address(0), 17);
    amoAddresses.push(_addr);
  }
```