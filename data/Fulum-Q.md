# [01] Set up the `reLPFactor` in the constructor to avoid DoS of the contract 

In the `ReLPContract`, the `reLPFactor` is not set in the constructor. If the admin of the contract doesn't call the `setreLpFactor` to define it, the `reLP` function of the contract is inoperable because `reLPFactor = 0` and computation with 0 make the function revert and impossible to make a bond because the bond function of `RdpxV2Core` call the `reLP` function.

It's better to define this important variable in the constructor to avoid this.

# [02] `bondMaturity` is not setup in the constructor, user can create bond immediately refundable 

In the context of `RdpxV2Core` contract.
Because the `bondMaturity` variable (which represent the time during a bond can't be withdrawn to receive receipt tokens) is not setup in the constructor, time between the contract is created and `bondMaturity` is setup, a malicious user can make bond without time to receive receipt tokens.
 
It's better to define this important variable in the constructor to avoid this.

# [03] Bad code error

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L410

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1306

In the `approveContractToSpend` function in the `RdpxV2Core` contract, the line below return the bad error if _amount is equal zero, just change the error code.

	-- _validate(_amount > 0, 17);
	++  _validate(_amount > 0, 15);
	
	