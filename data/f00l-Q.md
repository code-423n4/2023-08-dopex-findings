# Low: Cofusing code comments deviates from function logic

## Impact

the `emergencyWithdraw` function in the `RdpxDecayingBonds.sol` has such comments
`@param  to The address to transfer the funds to`
But actually it only use `to` parameter when `transferNative` variable is set and send tokens to `msg.sender`
Should adjust the comments