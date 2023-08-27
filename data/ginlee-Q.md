[L-01] should check _delegateId exists first in getDelegatePosition function
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1272
lots of devs forget that using a random key to get a value from the mapping will return a `delegatePosition` object with default values. Have seen multiple Critical severity issues related to this in combination with some flawed validation that does not check the zero values in a non-existing mapping value.

Mitigation:
validate "_delegateId" exists before getting data from struct