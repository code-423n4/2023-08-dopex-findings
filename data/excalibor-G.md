


1. Use custom error messages instead of require with string reasons

For example [here](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L112C1-L115C7) using the format `if(..) revert SlippageCantBeZero()` will be cheaper then `require(..., "long reason")`

2. Admin functions can be made payable

For example [here](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L130C5-L130C5) we know that this function will only be called by an admin, in the evm by default all calls are payable so it acctually costs more gas to make it not payable, since we can assume the admin wont send funds to a function call they know wont use them, we can safely make this payable.

3.  Don't instantiate variables to their default value

For example [here](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L147C1-L150C6) we are setting `uint256 i = 0;` when its default value is already 0, so this is wasting gas. We should change it to just `uint256 i;`

4. Pre-incrementing variables is cheaper then post incrementing

For example [here](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L147C1-L150C6)   `++i` will be cheaper then `i++`

5. Use unchecked incrementing inside loops

For example [here](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L147C1-L150C6) we know the length of the array is never going to overflow so we can safely refactor the for loop as such

```solidity
for(uint256 i; i<tokens.length;) {	
	// logic...
	
	unchecked { ++i; }
}
```

6. Update state variables using Yul

The solidity compiler will use a lot of unnecessary opcodes which will increase gas costs when simply updating a state variable, to avoid this we can using in-line assembly, also known as Yul, to make sure the only opcode being called is SSTORE

For example we can refactor [this](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L283) into this:

```solidity
assembly {
	let lpTokenBalanceCache := sload(lpTokenBalance.slot)

	// overflow check
	if gt(lpAmount, lpTokenBalanceCache)  {
	revert(0,  0)
	}

	sstore(lpTokenBalance.slot, sub(lpTokenBalance.slot, sub(lpTokenBalanceCache, lpAmount)))
}
```

its important to note because in this example we are doing arithmetic for this update Yul does not inherently check for overflow or underflow so we have to also add in these checks in Yul.

7. If we are looping over a memory or storage array we can cache the length to save gas

For example [here](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L120) we are reading the length of a storage variable in each iteration to check if `i < positions_array.length` this is making an SLOAD call every iteration, instead we can refactor this for loop to cache the array and only call SLOAD once

```solidity
uint256 length = positions_array.length
for(uint i; i<length; ) {
	// logic here
	
	unchecked { ++i; }
}
```

8. Make state variables that are set at construction and cant be changed immutable

For example [here](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L35C1-L37C35) we only set these variables in the constructor and they cant be changed, so we can make them immutable so we don't call any SLOADs when reading from them in our function calls.

9. Use calldata for function params if the variable is not modified

For example [here](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L155) the struct we are sending is not modified by the function so we can change `memory` to `calldata` to save some gas

10. Do keccak256 hashing off-chain and hardcode the hash to save gas at deployment

For example [here](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L22C4-L22C4) we are saving the hash as a constant variable, we can do this hashing off-chain and hardcode the hash here to save gas at deployment, as we will not need to calculate the hash on-chain.

11. Pack struct variables when safe to do so to save gas

For example [here](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/IRdpxV2Core.sol#L36C1-L40C4) as maturity and timestamp are both timestamp variables, we can safely pack them as `uint128` to only use 1 slot to save gas.

12. Use `!= 0` instead of `> 0` for validation of uint256 variables

[Here](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L410C5-L410C5) we validate a uint256 is greater then zero, as a unsigned integer cannot be lower then zero we can safely just check that it is not equal to zero, as `!=` is a cheaper validation then `>`

13. Cache repeated view function calls results to not calculate the same thing multiple times

For example [here](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L468C35-L468C35) we are making a lot of duplicate calls to `nextFundingPaymentTimestamp()` instead we can cache the result of this call and send the cached value to all of these calls

14. Avoid emitting events in loops if possible

In [this](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L462C1-L462C1) function we are updating the `latestFundingPaymentPointer`
and emitting its event each time we update it by one, instead we can move the emit outside of the while loop, this still ensures best practices of emitting events on storage updates as the state of the contract can still be backtracked through the events.

15. Create an internal `nextFundingPaymentTimestamp()` view function that takes parameters instead of reading from storage

For example in [this](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L405) function we are making many unncessary SLOADs inside a loop due to the use of this function, and as these state variables are not being updated in this call, we can cache them outside of the loop and create an internal function that will calculate the `nextFundingPaymentTimestamp()` without reading from storage

Please note the PoC doesn't take `genesis` into account because this should be immutable as per a previous issue in this report, so it wont need changing if this change is made.

Here is a PoC refactor:

```solidity
uint256 _latestFundingPaymentPointer = latestFundingPaymentPointer;
uint256 _fundingDuration = fundingDuration;
uint256 strikesLength = strikes.length
for (uint256 i; i < strikesLength;) {

_validate(optionsPerStrike[strikes[i]] >  0, 4);

	_validate(
latestFundingPerStrike[strikes[i]] != _latestFundingPaymentPointer,
	5
	);

	uint256 strike = strikes[i];

	uint256 amount = optionsPerStrike[strike] -

	fundingPaymentsAccountedForPerStrike[_latestFundingPaymentPointer][strike];
	  
	uint256 timeToExpiry =  _nextFundingPaymentTimestamp(_latestFundingPaymentPointer, _fundingDuration) -
	
	(genesis + ((_latestFundingPaymentPointer -  1) * _fundingDuration));

	uint256 premium =  calculatePremium(
	strike,
	amount,
	timeToExpiry,
	getUnderlyingPrice()
	);
	latestFundingPerStrike[strike] = _latestFundingPaymentPointer;

	fundingAmount += premium;

	// Record number of options that funding payments were accounted for, for this epoch
	fundingPaymentsAccountedFor[_latestFundingPaymentPointer] += amount;  

	// record the number of options funding has been accounted for the epoch and strike

	fundingPaymentsAccountedForPerStrike[_latestFundingPaymentPointer][strike] += amount;

	// Record total funding for this epoch
	
	// This does not need to be done in purchase() since it's already accounted for using `addProceeds()`
	totalFundingForEpoch[_latestFundingPaymentPointer] += premium;

	emit  CalculateFunding(
	msg.sender,
	amount,
	strike,
	premium,
	_latestFundingPaymentPointer
	);
	
	unchecked { ++i; }
}
```

16. Pack storage variables into less slots if it is safe to do so

For example [here](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L73C1-L76C50) if at the current value they are only a 0.5% fee we can safely pack both these variables into one slot by defining them as `uint128`. This example is safe because a 100% fee is still well below the max number that can fit inside uint128.
	
