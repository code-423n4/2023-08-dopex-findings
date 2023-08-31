## [G-01] - Remove unnecessary super._beforeTokenTransfer() 

  - Location : https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L52

  - Description :
                   _beforeTokenTransfer() function overrides the ERC20 _beforeTokenTransfer() function, but also calls super._beforeTokenTransfer(). This call to the parent function is unnecessary because no actions are performed, so it can be removed to save gas. 

  - Recommendation : 

                 Remove Line 52 fron Rdpxv2bond to remove the super._beforeTokenTransfer() call

