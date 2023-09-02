Use safe ERC20 Operation
ERC20 operations can be unsafe due to different implementations and vulnerabilities in the standard
in:
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L150

code:
  IERC20WithBurn(_token).approve(_target, _amount);

solution:
  IERC20WithBurn(_token).safeIncreaseAllowance(_target, _amount);
