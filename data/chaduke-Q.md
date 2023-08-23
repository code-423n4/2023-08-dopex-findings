QA1. Some ERC20 tokens do not support allowance approval from non-zero to non-zero. So the following RdpxV2Core.approveContractToSpend() code will not work. 

[https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/core/RdpxV2Core.sol#L403-L412](https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/core/RdpxV2Core.sol#L403-L412)

Correction: approve to zero first before approving to the designated ``amount`` for a safe approval:

```diff
function approveContractToSpend(
    address _token,
    address _spender,
    uint256 _amount
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_token != address(0), 17);
    _validate(_spender != address(0), 17);
    _validate(_amount > 0, 17);

+     uint256 allowance = IERC20WithBurn(_token).allowance(msg.sender, _spender);
+    if(allowance != 0) IERC20WithBurn(_token).approve(_spender, 0);


    IERC20WithBurn(_token).approve(_spender, _amount);
  }

```