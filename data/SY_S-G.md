## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] |Can make the variable outside the loop to save gas | |5| | 
| [G-02] |Use assembly to check for address(0)  | |26| | 
| [G-03] |Use bitmap to save gas  | |5| | 
| [G-04] |Require() or revert() statements that check input arguments should be at the top of the function | |1| | 
| [G-05] | Before transfer of  some functions, we should check some variables for possible gas save | |6| | 
| [G-06] |keccak256() should only need to be called on a specific string literal once | |2| | 
| [G-07] |Empty blocks should be removed or emit something | |1| | 
| [G-08] |With assembly, .call (bool success)  transfer can be done gas-optimized  | |2| | 
| [G-09] | Don't initialize variables with default value | |1| | 
| [G-10] | Add unchecked{} for subtractions where the operands cannot underflow because of a previous require() or if statement | |2| | 
| [G-11] |Using fixed bytes is cheaper than using string | |8| | 
| [G-12] |Duplicated require()/if() checks should be refactored to a modifier or function | |1| | 
| [G-13] |Using calldata instead of memory for read-only arguments in external functions, saves gas | |5| | 
| [G-14] |Multiple accesses of a mapping/array should use a local variable cache | |2| | 
| [G-15] |Should use arguments instead of state variable | |4| | 
| [G-16] |Use assembly to write address storage values  | |3| | 
| [G-17] |Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead | |1| | 
| [G-18] |Caching global variables is more expensive than using the actual variable (use msg.sender or block.timestamp instead of caching it) | |1| | 
| [G-19] |Use constants instead of type(uintx).max | |5| | 
| [G-20] |Expressions for constant values such as a call to keccak256(), should use immutable rather than constant | |5| | 






## Gas Optimizations  

## [G-1] Can make the variable outside the loop to save gas

Consider making the stack variables before the loop which gonna save gas

```solidity
file: /contracts/core/RdpxV2Core.sol

997      uint256 balance = IERC20WithBurn(reserveAsset[i].tokenAddress).balanceOf(

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L997


```solidity
file: /contracts/perp-vault/PerpetualAtlanticVault.sol

329      uint256 strike = optionPositions[optionIds[i]].strike;

330      uint256 amount = optionPositions[optionIds[i]].amount;

419      uint256 strike = strikes[i];

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L329-L330

```solidity
file: /contracts/amo/UniV3LiquidityAmo.sol

122        INonfungiblePositionManager.CollectParams
123        memory collect_params = INonfungiblePositionManager.CollectParams(

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L122-L123
## [G-2]  Use assembly to check for address(0)

  Save 6 gas per instance.

```solidity
file: /contracts/amo/UniV2LiquidityAmo.sol

83    require(
      _tokenA != address(0) &&
        _tokenB != address(0) &&
        _pair != address(0) &&
        _rdpxV2Core != address(0) &&
        _rdpxOracle != address(0) &&
        _ammFactory != address(0) &&
        _ammRouter != address(0),
      "reLPContract: address cannot be 0"
92    );


131    require(_token != address(0), "reLPContract: token cannot be 0");

132    require(_spender != address(0), "reLPContract: spender cannot be 0");

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L83-L92


```solidity
file: /contracts/core/RdpxV2Core.sol

244    require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");

316    _validate(_dopexAMMRouter != address(0), 17);
317    _validate(_dpxEthCurvePool != address(0), 17);
318    _validate(_rdpxDecayingBonds != address(0), 17);
319    _validate(_perpetualAtlanticVault != address(0), 17);
320    _validate(_perpetualAtlanticVaultLP != address(0), 17);
321    _validate(_rdpxReserve != address(0), 17);
322    _validate(_rdpxV2ReceiptToken != address(0), 17);
323    _validate(_feeDistributor != address(0), 17);
324    _validate(_reLPContract != address(0), 17);
325    _validate(_receiptTokenBonds != address(0), 17);

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L244

```solidity
file: /contracts/perp-vault/PerpetualAtlanticVault.sol

190    _validate(_optionPricing != address(0), 1);
       _validate(_assetPriceOracle != address(0), 1);
       _validate(_volatilityOracle != address(0), 1);
       _validate(_feeDistributor != address(0), 1);
       _validate(_rdpx != address(0), 1);
       _validate(_perpetualAtlanticVaultLP != address(0), 1);
196    _validate(_rdpxV2Core != address(0), 1);


```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L190-L196
## [G-3]  Use bitmap to save gas

```solidity
file: /contracts/core/RdpxV2Core.sol

79  mapping(uint256 => bool) public optionsOwned;

82  mapping(uint256 => bool) public fundingPaidFor;

135    putOptionsRequired = true;

485    optionsOwned[optionId] = true;

776    optionsOwned[optionIds[i]] = false;

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L79-L82
## [G-4] Require() or revert() statements that check input arguments should be at the top of the function  

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unhappy case.

```solidity
file: /contracts/perp-vault/PerpetualAtlanticVaultLP.sol

162    require(assets != 0, "ZERO_ASSETS");

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L162
## [G-5] Before transfer of  some functions, we should check some variables for possible gas save

Before transfer, we should check for amount being 0 so the function doesn't run when its not gonna do anything:

```solidity
file: /contracts/amo/UniV2LiquidityAmo.sol

149      token.safeTransfer(msg.sender, token.balanceOf(address(this)));

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L149

```solidity
file: /contracts/amo/UniV3LiquidityAmo.sol

283    IERC20WithBurn(_tokenA).transferFrom(

230    INonfungiblePositionManager(tokenAddress).safeTransferFrom(

357    IERC20WithBurn(tokenA).safeTransfer(rdpxV2Core, tokenABalance);

358    IERC20WithBurn(tokenB).safeTransfer(rdpxV2Core, tokenBBalance);
    
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L283

```solidity
file: /contracts/core/RdpxV2Bond.sol

52    super._beforeTokenTransfer(from, to, tokenId, batchSize);

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L52
## [G-6] keccak256() should only need to be called on a specific string literal once

It should be saved to an immutable variable, and the variable used instead. If the hash is being used as a part of a function selector, the cast to bytes4 should also only be done once

```solidity
file: /contracts/core/RdpxV2Bond.sol

22  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE"); //Found "MINTER_ROLE"

///@audit this keccak literal will called in another file ' /contracts/decaying-bonds/RdpxDecayingBonds.sol ' on line 31
/// @audit this literal are same which called by keccak256  

file: /contracts/decaying-bonds/RdpxDecayingBonds.sol

31    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L22

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L31

```solidity
file: /contracts/perp-vault/PerpetualAtlanticVault.sol

/// @audit "RDPXV2CORE_ROLE" on file ' /contracts/core/RdpxV2Bond.sol ' 

48    bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE"); 

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L4
## [G-7] Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.

```solidity
file: /contracts/perp-vault/PerpetualAtlanticVaultLP.sol

235  ) internal virtual {}

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L235
## [G-8] With assembly, .call (bool success)  transfer can be done gas-optimized

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.
Did you know that the normal "(bool success, bytes memory returnData) = http://target.call()" automatically copies the return data to memory even if you omit the returnData variable? If a relayer executes transactions with such calls it can lead to a gas-griefing attack.

```solidity
file: /contracts/amo/UniV3LiquidityAmo.sol

344    (bool success, bytes memory result) = _to.call{ value: _value }(_data);

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L344

```solidity
file:/contracts/decaying-bonds/RdpxDecayingBonds.sol

98      (bool success, ) = to.call{ value: amount, gas: gas }("");

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L98
## [G-9] Don't initialize variables with default value

Uninitialized variables are assigned with the types default value. Explicitly initializing a variable with it's default value costs unnecesary gas.

```solidity
file: /contracts/perp-vault/PerpetualAtlanticVault.sol

89  uint256 public latestFundingPaymentPointer = 0;

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L89
## [G-10]  Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if statement  

require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }
if(a <= b); x = b - a => if(a <= b); unchecked { x = b - a }
This will stop the check for overflow and underflow so it will save gas

```solidity
file: /contracts/core/RdpxV2Core.sol

670      uint256 discountReceivedInWeth = _bondAmount -
671        _wethAmount -
672        rdpxAmountInWeth;

1002        balance = balance - totalWethDelegated;

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L670-L672

```solidity
file: /contracts/perp-vault/PerpetualAtlanticVault.sol

467        uint256 startTime = lastUpdateTime == 0
             ? (nextFundingPaymentTimestamp() - fundingDuration)
469          : lastUpdateTime;

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L467-L469
## [G-11] Using fixed bytes is cheaper than using string

As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data.
If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.

```solidity
file: /contracts/core/RdpxV2Core.sol

70      string[] public reserveTokens;

242     string memory _assetSymbol

1136    string memory _token

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L70

```solidity
file: /contracts/decaying-bonds/RdpxDecayingBonds.sol

57    string memory _name,

58    string memory _symbol

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L57-L58

```solidity
file: /contracts/perp-vault/PerpetualAtlanticVault.sol

51  string public underlyingSymbol;

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L51


```solidity 
file: /contracts/perp-vault/PerpetualAtlanticVaultLP.sol
 
52  string public underlyingSymbol;

55  string public collateralSymbol;

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L52-L55
## [G-12] Duplicated require()/if() checks should be refactored to a modifier or function   

   to reduce code duplication and improve readability.
• Modifiers can be used to perform additional checks on the function inputs or state before it is executed. By defining a modifier to perform a specific check, we can reuse it across multiple functions that require the same check.
• A function can also be used to perform a specific check and return a boolean value indicating whether the check has passed or failed. This can be useful when the check is more complex and cannot be performed easily in a modifier.

```solidity
file: /contracts/core/RdpxV2Core.sol

///@audit this if statement is come duplicated on line 920, 1195

135      if (putOptionsRequired) {

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L135
## [G-13] Using calldata instead of memory for read-only arguments in external functions, saves gas 

When a function with a memory array is called externally, the abi.decode () step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution.

```solidity
file: /contracts/core/RdpxV2Core.sol

242    string memory _assetSymbol

271    string memory _assetSymbol

765    uint256[] memory optionIds

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L242

```solidity
file: /contracts/perp-vault/PerpetualAtlanticVault.sol

316    uint256[] memory optionIds

406    uint256[] memory strikes

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L316
## [G-14] Multiple accesses of a mapping/array should use a local variable cache

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping’s value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory/calldata.

```solidity
file: /contracts/core/RdpxV2Core.sol

/// @audit the ' reserveAsset ' is array state varaible 

246    for (uint256 i = 1; i < reserveAsset.length; i++) {

/// @audit the ' amoAddresses ' is array state varaible 

391    _validate(_index < amoAddresses.length, 18);
393     amoAddresses[_index] = amoAddresses[amoAddresses.length - 1];
393    amoAddresses.pop();    

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L246
## [G-15] Should use arguments instead of state variable 

state variables should not used in emit  ,  This will save near 97 gas   

```solidity
file: /contracts/core/RdpxV2Core.sol

/// @audit ' addresses ' is state variable 

349    emit LogSetAddresses(addresses);

/// @audit ' pricingOracleAddresses ' is state variable 

370    emit LogSetPricingOracleAddresses(pricingOracleAddresses);

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L349

```solidity
file: /contracts/perp-vault/PerpetualAtlanticVault.sol

/// @audit ' addresses ' is state variable 

211      emit AddressesSet(addresses);
 
/// @audit ' latestFundingPaymentPointer ' is state variable 

494      emit FundingPaymentPointerUpdated(latestFundingPaymentPointer);

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L21
## [G-16] Use assembly to write address storage values

By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.

```solidity
file: /contracts/amo/UniV2LiquidityAmo.sol

/// @audit the ' addresses ' is storage or state variable the of value of it should be assign to assembly munner.

93    addresses = Addresses({

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L93

```solidity
file: /contracts/core/RdpxV2Core.sol

327    addresses = Addresses({

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L327

```solidity
file: /contracts/perp-vault/PerpetualAtlanticVaultLP.sol

99    rdpxRdpxV2Core = _rdpxRdpxV2Core;

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L99
## [G-17] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead         

When using elements that are smaller than 32 bytes, your contract's gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

```solidity
file: /contracts/amo/UniV3LiquidityAmo.sol

/// @audit ' _fee  ' 

101      univ3_factory.getPool(address(rdpx), _collateral_address, _fee)

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L101
## [G-18]  Caching global variables is more expensive than using the actual variable (use msg.sender or block.timestamp instead of caching it)

```solidity
file: /contracts/perp-vault/PerpetualAtlanticVault.sol

508    lastUpdateTime = block.timestamp;

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L508
## [G-19] Use constants instead of type(uintx).max

type(uint120).max or type(uint128).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity
file: /contracts/amo/UniV3LiquidityAmo.sol

126          type(uint128).max,

127          type(uint128).max

190        type(uint256).max

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L126-L127


```solidity
file: /contracts/core/RdpxV2Core.sol

343    IERC20WithBurn(weth).approve(addresses.dopexAMMRouter, type(uint256).max);

344    IERC20WithBurn(weth).approve(addresses.dpxEthCurvePool, type(uint256).max);

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L343-L344
## [G-20] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

While it doesn't save any gas because the compiler knows that developers often make this mistake, it's still best to use theright tool for the task at hand. There is a difference between constant variables and immutable variables, and they shouldeach be used in their appropriate contexts. constants should be used for literal values written into the code, and immutablevariables should be used for expressions, or values calculated in, or passed into the constructor. 

```solidity
file: /contracts/core/RdpxV2Bond.sol

22  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L22

```solidity
file: /contracts/decaying-bonds/RdpxDecayingBonds.sol

34  bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L34

```solidity
file: /contracts/dpxETH/DpxEthToken.sol

19   bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
20   bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
21   bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L19-L21


