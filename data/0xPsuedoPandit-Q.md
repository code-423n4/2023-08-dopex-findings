1) Missing precision check for slippage tolerance. As per the Natspec comments the slippage should be in 1e8 precision like 0.5% is represented as 5e5 which is also the initial amount for the slippage, make sure the new slippage is in 1e8 precision like 1% of slippage should be 1e6, since it is the protocol's invariant so the check is mandatory here to make sure the new value is in right decimals and also put a upper cap limit so that the users don't have to worry and suffer from arbitrary increase in slippage.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L455-L462

2) _stake function in RdpxV2Core does not check whether the reserve has enough tokens or not which might result in unexpected underflow error and also it is prone to solidity rounding down to zero bug since it is performing unsafe divisions on the input _amount

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L566-L583

I would recommend to add this check before subtracting the _amount at line 570.
require(reserveAsset[reservesIndex["WETH"]].tokenBalance >= _amount/2)

3) Validate optionIds before settling them.

-> check whether the optionIds are already settled or not, if they're not then consider that specific Id for settling, this would prevent from settling the settled id and it may also protect from fundloss.

Recommended check 
require(optionsOwned[optionIds[i]] = true);
then only set it to false.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L764-L783