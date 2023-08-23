_weth address passed in constructor parameter lack proper check to ensure it is not zero address.
File: ```https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L124-L126```
```
// @audit: add sanity check to _weth address to validate if the 
  // address is non zero address.
  // eg: if(_weth == address(0)) revert someCustomError();
  constructor(address _weth) {
    _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
    weth = _weth;
```
