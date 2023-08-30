https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L89

Inconsistent recipient for `emergencyWithdraw` in `RdpxDecayingBonds` between native (which goes to the passed `to` address parameter) and ERC20 withdrawals (which go to the `msg.sender`).

Consider using one or the other for both transfer types.