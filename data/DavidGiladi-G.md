
### Gas Optimization Issues
|Title|Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
|[G-1] Avoid unnecessary storage updates | [Avoid unnecessary storage updates](#avoid-unnecessary-storage-updates) | 15 | 12000 |
|[G-2] Check Arguments Early | [Check Arguments Early](#check-arguments-early) | 1 | - |
|[G-3] Modulus operations that could be unchecked | [Modulus operations that could be unchecked](#modulus-operations-that-could-be-unchecked) | 1 | 85 |
|[G-4] State variables that could be declared constant | [State variables that could be declared constant](#state-variables-that-could-be-declared-constant) | 3 | 6291 |
|[G-5] State variables that could be declared immutable | [State variables that could be declared immutable](#state-variables-that-could-be-declared-immutable) | 3 | 6291 |
|[G-6] Avoid duplicate keccak256 calls | [Avoid duplicate keccak256 calls](#avoid-duplicate-keccak256-calls) | 2 | 84 |
|[G-7] Use of emit inside a loop | [Use of emit inside a loop](#use-of-emit-inside-a-loop) | 3 | 3078 |
|[G-8] Cache External Calls in Loops | [Cache External Calls in Loops](#cache-external-calls-in-loops) | 7 | 700 |
|[G-9] Optimal State Variable Order | [Optimal State Variable Order](#optimal-state-variable-order) | 1 | 5000 |
|[G-10] Redundant Bool Comparison | [Redundant Bool Comparison](#redundant-bool-comparison) | 1 | 9 |
|[G-11] Cache Frequently Called Functions in stack Variables to Reduce Gas Costs | [Cache Frequently Called Functions in stack Variables to Reduce Gas Costs](#cache-frequently-called-functions-in-stack-variables-to-reduce-gas-costs) | 3 | - |
|[G-12] Safe Subtraction Should Be Unchecked | [Safe Subtraction Should Be Unchecked](#safe-subtraction-should-be-unchecked) | 2 | 170 |
|[G-13] Unnecessary Casting of Variables | [Unnecessary Casting of Variables](#unnecessary-casting-of-variables) | 2 | - |
|[G-14] Avoid Unnecessary Computation Inside Loops | [Avoid Unnecessary Computation Inside Loops](#avoid-unnecessary-computation-inside-loops) | 1 | - |
|[G-15] Redundant Contract Existence Check in Consecutive External Calls | [Redundant Contract Existence Check in Consecutive External Calls](#redundant-contract-existence-check-in-consecutive-external-calls) | 15 | 1500 |
|[G-16] Use Assembly for Hash Calculation | [Use Assembly for Hash Calculation](#use-assembly-for-hash-calculation) | 8 | 800 |

Total: 16 issues

## Optimal State Variable Order
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 5000

### Description
Detects optimal variable order in contract storage layouts to decrease the number of storage slots used. Each storage slot used can cost at least 5,000 gas for each write operation, and potentially up to 20,000 gas if you're turning a zero value into a non-zero value. Hence, optimizing storage usage can result in significant gas savings. The real-world savings could vary depending on your contract's specific logic and the state of the Ethereum network.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- Optimization opportunity in storage variable layout of contract File: contracts/core/RdpxV2Core.sol
```
 
Line: 33          contract RdpxV2Core is
  IRdpxV2Core,
  AccessControl,
  ContractWhitelist,
  ERC721Holder,
  Pausable

```

- original variable order (count: 22 slots)
    - IRdpxV2Core.Addresses addresses
    - IRdpxV2Core.PricingOracleAddresses pricingOracleAddresses
    - IRdpxV2Core.ReserveAsset[] reserveAsset
    - address[] amoAddresses
    - address weth
    - string[] reserveTokens
    - mapping(string => uint256) reservesIndex
    - mapping(uint256 => IRdpxV2Core.Bond) bonds
    - mapping(uint256 => bool) optionsOwned
    - mapping(uint256 => bool) fundingPaidFor
    - uint256 DEFAULT_PRECISION
    - uint256 RDPX_RATIO_PERCENTAGE
    - uint256 ETH_RATIO_PERCENTAGE
    - uint256 rdpxBurnPercentage
    - uint256 rdpxFeePercentage
    - uint256 slippageTolerance
    - uint256 liquiditySlippageTolerance
    - uint256 bondMaturity
    - uint256 bondDiscountFactor
    - uint256 totalWethDelegated
    - bool isReLPActive
    - bool putOptionsRequired
    - IRdpxV2Core.Delegate[] delegates


 - optimized variable order (count: 21 slots)
    - address weth
    - bool isReLPActive
    - bool putOptionsRequired
    - IRdpxV2Core.Addresses addresses
    - IRdpxV2Core.PricingOracleAddresses pricingOracleAddresses
    - IRdpxV2Core.ReserveAsset[] reserveAsset
    - address[] amoAddresses
    - string[] reserveTokens
    - mapping(string => uint256) reservesIndex
    - mapping(uint256 => IRdpxV2Core.Bond) bonds
    - mapping(uint256 => bool) optionsOwned
    - mapping(uint256 => bool) fundingPaidFor
    - uint256 DEFAULT_PRECISION
    - uint256 RDPX_RATIO_PERCENTAGE
    - uint256 ETH_RATIO_PERCENTAGE
    - uint256 rdpxBurnPercentage
    - uint256 rdpxFeePercentage
    - uint256 slippageTolerance
    - uint256 liquiditySlippageTolerance
    - uint256 bondMaturity
    - uint256 bondDiscountFactor
    - uint256 totalWethDelegated
    - IRdpxV2Core.Delegate[] delegates


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L33-L1287](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L33-L1287)


</details>

# 


## State variables that could be declared immutable
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 6291

### Note 
I reoprted only on variable that was missed in the wining bot. 

### Description
State variables that are not updated following deployment should be declared immutable to save gas.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
- File: contracts/amo/UniV3LiquidityAmo.sol
```
 
Line: 69          address public rdpx
```
 should be immutable 

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L69](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L69)





- File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
 
Line: 46          IPerpetualAtlanticVault public perpetualAtlanticVault
```
 should be immutable 

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L46](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L46)


- File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
 
Line: 67          address public rdpx
```
 should be immutable 

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L67](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L67)



</details>

# 


## Check Arguments Early
- Severity: Gas Optimization
- Confidence: High

### Description
Checks that require() or revert() statements that check input arguments are at the top of the function. Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a gas load in a function that may ultimately revert in the unhappy case.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
 
Line: 94          require(
      _perpetualAtlanticVault != address(0) || _rdpx != address(0),
      "ZERO_ADDRESS"
    )
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L94-L97](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L94-L97)


</details>

# 


## Modulus operations that could be unchecked
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 85

### Description
Modulus operations should be unchecked to save gas since they cannot overflow or underflow. Execution of modulus operations outside `unchecked` blocks adds nothing but overhead. Saves about 30 gas.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 577          _strike % roundingPrecision
```
 should be unchecked

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L577](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L577)


</details>

# 



## Use Assembly for Hash Calculation
- Severity: Gas Optimization
- Confidence: Medium
- Total Gas Saved: 800

### Note 
I reopted only on instances that were misssing on the wining bot.

### Description
Detects places in the code where hash calculation could be done using inline assembly to optimize gas usage.

<details>

<summary>
There are 10 instances of this issue:

</summary>

###
- File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 105          (uint128 liquidity, , , , ) = get_pool.positions(
      keccak256(abi.encodePacked(address(this), _tickLower, _tickUpper))
    )
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAMO.sol#L105-L107](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAMO.sol#L105-L107)




- File: contracts/core/RdpxV2Bond.sol
```
 
Line: 22          bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE")
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L22](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L22)

 


- File: contracts/decaying-bonds/RdpxDecayingBonds.sol
```
 
Line: 31          bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE")
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L31](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L31)


- File: contracts/decaying-bonds/RdpxDecayingBonds.sol
```
 
Line: 34          bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE")
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L34](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L34)


- File: contracts/dpxETH/DpxEthToken.sol
```
 
Line: 20          bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE")
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L20](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L20)


- File: contracts/dpxETH/DpxEthToken.sol
```
 
Line: 19          bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE")
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L19](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L19)


- File: contracts/dpxETH/DpxEthToken.sol
```
 
Line: 21          bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE")
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L21](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L21)


- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 48          bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE")
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L48](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L48)


- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 45          bytes32 public constant MANAGER_ROLE = keccak256("MANAGER_ROLE")
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L45](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L45)


- File: contracts/reLP/ReLPContract.sol
```
 
Line: 70          bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE")
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L70](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L70)




</details>

# 


## Avoid unnecessary storage updates
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 12000

### Description
Avoid updating storage when the value hasn't changed. If the old value is equal to the new value, not re-storing the value will avoid a `SSTORE` operation (costing 2900 gas), potentially at the expense of a `SLOAD` operation (2100 gas) or a `WARMACCESS` operation (100 gas).

<details>

<summary>
There are 15 instances of this issue:

</summary>

###
- The function `setAddresses()` changes the state variable without first verifying if the values are different.

File: contracts/core/RdpxV2Core.sol
```
 
Line: 327          addresses = Addresses({
      dopexAMMRouter: _dopexAMMRouter,
      dpxEthCurvePool: _dpxEthCurvePool,
      rdpxDecayingBonds: _rdpxDecayingBonds,
      perpetualAtlanticVault: _perpetualAtlanticVault,
      perpetualAtlanticVaultLP: _perpetualAtlanticVaultLP,
      rdpxReserve: _rdpxReserve,
      rdpxV2ReceiptToken: _rdpxV2ReceiptToken,
      feeDistributor: _feeDistributor,
      reLPContract: _reLPContract,
      receiptTokenBonds: _receiptTokenBonds
    })
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L327-L338](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L327-L338)


- The function `setAddresses()` changes the state variable without first verifying if the values are different.

File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 198          addresses = Addresses({
      optionPricing: _optionPricing,
      assetPriceOracle: _assetPriceOracle,
      volatilityOracle: _volatilityOracle,
      feeDistributor: _feeDistributor,
      rdpx: _rdpx,
      perpetualAtlanticVaultLP: _perpetualAtlanticVaultLP,
      rdpxV2Core: _rdpxV2Core
    })
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L198-L206](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L198-L206)


- The function `setBondDiscount()` changes the state variable without first verifying if the values are different.

File: contracts/core/RdpxV2Core.sol
```
 
Line: 445          bondDiscountFactor = _bondDiscountFactor
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L445](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L445)


- The function `setBondMaturity()` changes the state variable without first verifying if the values are different.

File: contracts/core/RdpxV2Core.sol
```
 
Line: 232          bondMaturity = _bondMaturity
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L232](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L232)


- The function `setIsreLP()` changes the state variable without first verifying if the values are different.

File: contracts/core/RdpxV2Core.sol
```
 
Line: 207          isReLPActive = _isReLPActive
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L207](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L207)


- The function `setPricingOracleAddresses()` changes the state variable without first verifying if the values are different.

File: contracts/core/RdpxV2Core.sol
```
 
Line: 365          pricingOracleAddresses = PricingOracleAddresses({
      rdpxPriceOracle: _rdpxPriceOracle,
      dpxEthPriceOracle: _dpxEthPriceOracle
    })
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L365-L368](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L365-L368)


- The function `setPutOptionsRequired()` changes the state variable without first verifying if the values are different.

File: contracts/core/RdpxV2Core.sol
```
 
Line: 219          putOptionsRequired = _putOptionsRequired
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L219](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L219)


- The function `setRdpxBurnPercentage()` changes the state variable without first verifying if the values are different.

File: contracts/core/RdpxV2Core.sol
```
 
Line: 184          rdpxBurnPercentage = _rdpxBurnPercentage
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L184](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L184)


- The function `setRdpxFeePercentage()` changes the state variable without first verifying if the values are different.

File: contracts/core/RdpxV2Core.sol
```
 
Line: 197          rdpxFeePercentage = _rdpxFeePercentage
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L197](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L197)


- The function `setSlippageTolerance()` changes the state variable without first verifying if the values are different.

File: contracts/core/RdpxV2Core.sol
```
 
Line: 459          slippageTolerance = _slippageTolerance
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L459](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L459)


- The function `settle()` changes the state variable without first verifying if the values are different.

File: contracts/core/RdpxV2Core.sol
```
 
Line: 779          reserveAsset[reservesIndex["WETH"]].tokenBalance += amountOfWeth
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L779](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L779)


- The function `settle()` changes the state variable without first verifying if the values are different.

File: contracts/core/RdpxV2Core.sol
```
 
Line: 780          reserveAsset[reservesIndex["RDPX"]].tokenBalance -= rdpxAmount
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L780](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L780)


- The function `settle()` changes the state variable without first verifying if the values are different.

File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 337          optionsPerStrike[strike] -= amount
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L337](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L337)


- The function `settle()` changes the state variable without first verifying if the values are different.

File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 338          totalActiveOptions -= amount
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L338](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L338)


- The function `updateFundingDuration()` changes the state variable without first verifying if the values are different.

File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 240          fundingDuration = _fundingDuration
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L240](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L240)


</details>

# 



## State variables that could be declared constant
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 6291

### Description
State variables that are not updated following deployment should be declared constant to save gas.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
- File: contracts/core/RdpxV2Core.sol
```
 
Line: 103          uint256 public liquiditySlippageTolerance = 5e5
```
 should be constant 

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L103](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L103)


- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 104          uint256 public roundingPrecision = 1e6
```
 should be constant 

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L104](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L104)


- File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
 
Line: 52          string public underlyingSymbol
```
 should be constant 

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L52](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L52)


</details>

# 


## Avoid duplicate keccak256 calls
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 84

### Description
Detects instances where keccak256 is being called on the same string literal multiple times. It should be called once and the result should be saved to an immutable variable. If the hash is being used as a part of a function selector, the cast to bytes4 should also only be done once.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- MINTER_ROLE has a multiple call to keccak256File: contracts/core/RdpxV2Bond.sol
```
 
Line: 22          bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE")
```
File: contracts/decaying-bonds/RdpxDecayingBonds.sol
```
 
Line: 31          bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE")
```
File: contracts/dpxETH/DpxEthToken.sol
```
 
Line: 20          bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE")
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L22](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L22)


- RDPXV2CORE_ROLE has a multiple call to keccak256File: contracts/decaying-bonds/RdpxDecayingBonds.sol
```
 
Line: 34          bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE")
```
File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 48          bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE")
```
File: contracts/reLP/ReLPContract.sol
```
 
Line: 70          bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE")
```
File: contracts/relp/ReLPContract.sol
```
 
Line: 70          bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE")
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L34](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L34)


</details>

# 


## Use of emit inside a loop
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 3078

### Description
Emitting an event inside a loop performs a LOG op N times, where N is the loop length. This can lead to significant gas costs.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 451          emit CalculateFunding(
        msg.sender,
        amount,
        strike,
        premium,
        latestFundingPaymentPointer
      )
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L451-L457](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L451-L457)


- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 485          emit FundingPaid(
          msg.sender,
          ((currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
            1e18),
          latestFundingPaymentPointer
        )
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L485-L490](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L485-L490)


- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 494          emit FundingPaymentPointerUpdated(latestFundingPaymentPointer)
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L494](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L494)


</details>

# 


## Cache External Calls in Loops
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 700

### Description
This detector identifies instances of repeated external calls within loops. By moving the first external call outside of the loop and using low-level calls inside the loop, it is possible to save gas by avoiding repeated EXTCODESIZE opcodes, as there is no need to check again if the contract exists. 

Note: This detector only triggers if the function call does not return any value.

Prior to 0.8.10, the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent Solidity versions the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low-level calls never check for contract existence. 

<details>

<summary>
There are 7 instances of this issue:

</summary>

###
- File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 149          token.safeTransfer(msg.sender, token.balanceOf(address(this)))
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L149](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L149)


- File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 149          token.safeTransfer(msg.sender, token.balanceOf(address(this)))
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L149](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L149)


- File: contracts/core/RdpxV2Core.sol
```
 
Line: 169          token.safeTransfer(msg.sender, token.balanceOf(address(this)))
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L169](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L169)


- File: contracts/decaying-bonds/RdpxDecayingBonds.sol
```
 
Line: 105          token.safeTransfer(msg.sender, token.balanceOf(address(this)))
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L105](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L105)


- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 227          token.safeTransfer(msg.sender, token.balanceOf(address(this)))
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L227](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L227)


- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 473          collateralToken.safeTransfer(
          addresses.perpetualAtlanticVaultLP,
          (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
            1e18
        )
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L473-L477](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L473-L477)


- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 479          IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP)
          .addProceeds(
            (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
              1e18
          )
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L479-L483](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L479-L483)


</details>

# 


## Redundant Bool Comparison
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 9

### Description
This detector checks for instances where a boolean expression is compared to a boolean literal (true or false), which is redundant.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: contracts/core/RdpxV2Core.sol
```
 
Line: 798          _validate(fundingPaidFor[pointer] == false, 16)
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L798](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L798)


</details>

# 


## Cache Frequently Called Functions in stack Variables to Reduce Gas Costs
- Severity: Gas Optimization
- Confidence: High

### Description
if a function is being called multiple times within the same scope without caching the result. This can result in unnecessary gas costs due to the repeated execution of the same code.By caching the results of function calls, we can save gas by reducing the number of times a function is executed, especially in loops or repetitive code blocks. This can lead to significant gas savings, particularly in cases where the function is called many times within a loop or in a contract with a high level of usage.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
- Should cache the following function calls:
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 546          getDpxEthPrice()
```

  - File: contracts/core/RdpxV2Core.sol
```
 
Line: 547          getDpxEthPrice()
```

File: contracts/core/RdpxV2Core.sol
```
 
Line: 1216          function getDpxEthPrice() public view returns (uint256 dpxEthPrice) 
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L546](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L546)


- Should cache the following function calls:
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 548          getEthPrice()
```

	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 549          getEthPrice()
```

File: contracts/core/RdpxV2Core.sol
```
 
Line: 1227          function getEthPrice() public view returns (uint256 ethPrice) 
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L548](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L548)


- Should cache the following function calls:
	- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 471          nextFundingPaymentTimestamp()
```

	- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 475          nextFundingPaymentTimestamp()
```

	- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 481          nextFundingPaymentTimestamp()
```

	- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 487          nextFundingPaymentTimestamp()
```

File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 563          function nextFundingPaymentTimestamp()
    public
    view
    returns (uint256 timestamp)
  
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L471](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L471)


</details>

# 


## Unnecessary Casting of Variables
- Severity: Gas Optimization
- Confidence: High

### Description
This detector scans for instances where a variable is casted to its own type. This is unnecessary and can be safely removed to improve code readability.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 100          IUniswapV3Pool get_pool = IUniswapV3Pool(
      univ3_factory.getPool(address(rdpx), _collateral_address, _fee)
    )
```
Unnecessary cast: `address(rdpx)` it cast to the same type.<br>
[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAMO.sol#L100-L102](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAMO.sol#L100-L102)


- File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 197          Position memory pos = Position(
      tokenId,
      params._tokenA == address(rdpx) ? params._tokenB : params._tokenA,
      amountLiquidity,
      params._tickLower,
      params._tickUpper,
      params._fee
    )
```
Unnecessary cast: `address(rdpx)` it cast to the same type.<br>
[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAMO.sol#L197-L204](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAMO.sol#L197-L204)


</details>

# 


## Avoid Unnecessary Computation Inside Loops
- Severity: Gas Optimization
- Confidence: High

### Description
This detector identifies instances of computations being performed inside loops that could be performed outside the loop to save gas.

Unnecessary computation inside loops:

Any computation performed inside a loop that doesn't change with each iteration can be moved outside the loop to save gas.This detector identifies instances where the following unnecessary computations are performed inside loops:

- 1 Local variables are read inside loops but never modified within the same loop, indicating they could be cached outside the loop.

- 2 Computations involving constant expressions (literals) which result in the same output in every loop iteration.

By addressing these issues, gas usage can be optimized.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 463          block.timestamp >= nextFundingPaymentTimestamp()
```
The following computations can be cached outside of loops for gas optimization `block.timestamp >= nextFundingPaymentTimestamp()`<br>.
[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L463](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L463)


</details>

# 


## Redundant Contract Existence Check in Consecutive External Calls
- Severity: Gas Optimization
- Confidence: Medium
- Total Gas Saved: 800

### Description
This detector identifies instances where there are consecutive external calls to the same contract, where the subsequent calls could use a low-level call to save gas.

Note: This detector only triggers if the function call does not return any value. 

Prior to 0.8.10, the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent Solidity versions the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low-level calls never check for contract existence. 

<details>

<summary>
There are 8 instances of this issue:

</summary>

###


- File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 328          IERC20WithBurn(token1).safeApprove(addresses.ammRouter, token1Amount)
```
This call could be replaced with a low-level call because the contract `SafeERC20` has already been checked in <br>`Line: 321    IERC20WithBurn(token1).safeTransferFrom(
      addresses.rdpxV2Core,
      address(this),
      token1Amount
    )`<br>
[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L328](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L328)


- File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 172          IERC20WithBurn(addresses.tokenB).safeTransfer(
      addresses.rdpxV2Core,
      tokenBBalance
    )
```
This call could be replaced with a low-level call because the contract `SafeERC20` has already been checked in <br>`Line: 168    IERC20WithBurn(addresses.tokenA).safeTransfer(
      addresses.rdpxV2Core,
      tokenABalance
    )`<br>
[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L172-L175](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L172-L175)


- File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 204          IERC20WithBurn(addresses.tokenB).safeApprove(
      addresses.ammRouter,
      tokenBAmount
    )
```
This call could be replaced with a low-level call because the contract `SafeERC20` has already been checked in <br>`Line: 200    IERC20WithBurn(addresses.tokenA).safeApprove(
      addresses.ammRouter,
      tokenAAmount
    )`<br>
[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L204-L207](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L204-L207)




- File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 358          IERC20WithBurn(tokenB).safeTransfer(rdpxV2Core, tokenBBalance)
```
This call could be replaced with a low-level call because the contract `SafeERC20` has already been checked in <br>`Line: 357    IERC20WithBurn(tokenA).safeTransfer(rdpxV2Core, tokenABalance)`<br>
[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAMO.sol#L358](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAMO.sol#L358)





- File: contracts/core/RdpxV2Core.sol
```
 
Line: 662          IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress)
        .safeTransfer(
          addresses.feeDistributor,
          (_rdpxAmount * rdpxFeePercentage) / 1e10
        )
```
This call could be replaced with a low-level call because the contract `SafeERC20` has already been checked in <br>`Line: 653    IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress)
        .safeTransferFrom(msg.sender, address(this), _rdpxAmount)`<br>
[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L662-L666](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L662-L666)


- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 353          IERC20WithBurn(addresses.rdpx).safeTransferFrom(
      addresses.rdpxV2Core,
      addresses.perpetualAtlanticVaultLP,
      rdpxAmount
    )
```
This call could be replaced with a low-level call because the contract `SafeERC20` has already been checked in <br>`Line: 347    collateralToken.safeTransferFrom(
      addresses.perpetualAtlanticVaultLP,
      addresses.rdpxV2Core,
      ethAmount
    )`<br>
[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L353-L357](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L353-L357)


- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 362          IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP)
      .unlockLiquidity(ethAmount)
```
This call could be replaced with a low-level call because the contract `IPerpetualAtlanticVaultLP` has already been checked in <br>`Line: 359    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).subtractLoss(
      ethAmount
    )`<br>
[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L362-L363](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L362-L363)





- File: contracts/relp/ReLPContract.sol
```
 
Line: 302          IERC20WithBurn(addresses.tokenA).safeTransfer(
      addresses.rdpxV2Core,
      IERC20WithBurn(addresses.tokenA).balanceOf(address(this))
    )
```
This call could be replaced with a low-level call because the contract `SafeERC20` has already been checked in <br>`Line: 298    IERC20WithBurn(addresses.pair).safeTransfer(addresses.amo, lp)`<br>
[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/relp/ReLPContract.sol#L302-L305](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/relp/ReLPContract.sol#L302-L305)


</details>

# 


## Safe Subtraction Should Be Unchecked
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 170

### Description
This detector identifies subtraction operations where the subtraction could safely be unchecked. This occurs when a prior if-statement or require() function ensures that the first operand (minuend) is greater than or equal to the second operand (subtrahend). In these cases, an unchecked subtraction would not result in an underflow error, and can save gas by skipping the redundant check.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 610          fundingRates[latestFundingPaymentPointer] =
        fundingRates[latestFundingPaymentPointer] +
        ((amount * 1e18) / (endTime - startTime))
```
 should be unchecked because there is check:<br>File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 609          endTime == startTime
```
<br><br>
[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L610-L612](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L610-L612)


- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 559          return strike > price ? ((strike - price) * amount) / 1e8 : 0
```
 should be unchecked because there is check:<br>File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 559          return strike > price ? ((strike - price) * amount) / 1e8 : 0
```
<br><br>
[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L559](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L559)


</details>

# 