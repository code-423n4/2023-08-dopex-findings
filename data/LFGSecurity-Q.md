## `setAddresses()` sets unnecessary max approval for dopexAMMRouter and dpxEthCurvePool to spend WETH
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L343
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L344
In light of the recent Curve hack, it is not a good idea to provide max approval spending to external contracts. In the case of a hack or backdoor in the approved contracts, they can potentially drain all the WETH from the RdpxV2Core contract.
Consider approving only the necessary amount of WETH, when needed. E.g When using `_curveSwap()`.

## `decreaseAmount()` sets the value of `bonds[bondId].rdpxAmount` to the provided `amount` instead of reducing it by `amount` in RdpxDecayingBonds.sol
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L144
Although the method is called by an admin, the title/implementation of the method is misleading and could lead to wrong amount updated.
Consider updating the method name to `updateAmount()`, or change to `bonds[bondId].rdpxAmount -= amount`;

## Lack of limits for setting fees
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L193-L199
There should be an upper limit to reasonable fees. Consider adding a fee limit in `setRdpxFeePercentage()`