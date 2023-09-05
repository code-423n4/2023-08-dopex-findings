| Issue Count | Issues | Instances | Gas Saves |
|----------|----------|----------|----------|
| [G-1]  |Optimizing Gas Usage through State Variable Packing  | 14  | 18000 |
| [G-2]  | Use ``assembly`` to check for ``address(0)``  | 45  | 270 |
| [G-3]  | Use ``calldata`` instead of ``memory`` for function parameters | 26  | 8112  |
| [G-4]  | Using bitmap to store bool states can save gas   | 2   | 4000   |
| [G-5]  | Expressions for constant values such as a call to keccak256(), should use immutable rather than constant   | 9  |  |
| [G-6]  | Struct can be packed  | 1  | 2000  |
| [G-7]  | Unused Function | 1   |  |
| [G-8]  | Unused EVENT can remove to save GAS  | 1  |  |
| [G-9]  | Use assembly to calculate hashes  | 12  | 960 |
| [G-10]  | ``abi.encode`` is more efficient than ``abi.encodePacked``  | 3  | 270 |
| [G-11]  | Use assembly to ``emit`` an event | 45  | 1710 |
| [G-12]  | Use of emit inside a loop | 1  | 1026 |
| [G-13]  | Unbounded Gas Consumption Risk Due to `External Call` Recipients  | 2 | |


## [G-1] Optimizing Gas Usage through State Variable Packing

### Saves ``18000``



In this contract, there is a state variable named ``DEFAULT_PRECISION`` with a constant value of ``1e8`` equivalent to ``100,000,000`` By changing the data type of this variable to uint128, the contract can accommodate values up to ``1e20``

Additionally, the ``slippageTolerance`` state variable stores the value ``5e5`` representing ``500,000`` (as per the code, denoting 0.5%). By using uint128 for this variable, it can store larger values, such as ``5e20``

Furthermore, there is a function named ``slippageTolerance`` that allows only users with a specific role to set the value. This allows users with the designated role to increase the percentage if needed.

By implementing these changes, the contract can save one storage slot, which translates to a gas savings of ``2000 gas.``




```diff
FILE: contracts/amo/UniV2LiquidityAmo.sol
     
       /// @notice Precision used for prices, percentages, and other calculations
- uint256 public constant DEFAULT_PRECISION = 1e8;
+ uint128 public constant DEFAULT_PRECISION = 1e8;

 /// @notice The slippage tolerance in swaps in 1e8 precision
- uint256 public slippageTolerance = 5e5; // 0.5%   
+ uint128 public slippageTolerance = 5e5; // 0.5%


```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L48C1-L51C50


In this contract, several state variables are being optimized to improve gas efficiency and storage usage: (8000 gas)

The state variable ``DEFAULT_PRECISION`` is currently a constant with a value of ``1e8`` equivalent to ``100,000,000`` By changing its data type to uint32, the contract can accommodate values up to ``1e20``

The ``liquiditySlippageTolerance`` state variable holds the value ``5e5`` which represents ``500,000`` (0.5%). By using a uint32 for this variable, it can store larger values, such as ``5e20``

The ``RDPX_RATIO_PERCENTAGE`` state variable is set to ``25 * DEFAULT_PRECISION`` which can be stored in a uint64 data type.

The ``ETH_RATIO_PERCENTAGE`` state variable is calculated as ``75 * DEFAULT_PRECISION`` and this percentage can efficiently be stored in a uint128.

Both ``rdpxBurnPercentage`` and``rdpxFeePercentage`` are calculated as``50 * `` and these values can be stored in uint128.

By making these adjustments, the state variables can be packed into ``3 storage slots`` instead of the current ``7 slots``, resulting in significant gas savings of ``4 slots (equivalent to 8000 gas units).``

Please note that these optimizations were carefully considered by reviewing internal functions where the state variables change to ensure compatibility and efficient usage of gas and storage.



```diff
FILE: contracts/core/RdpxV2Core.sol
     
        84   /// @notice Precision used for prices, percentages and other calculations
- 85:     uint256 public constant DEFAULT_PRECISION = 1e8;
+ 85:     uint32 public constant DEFAULT_PRECISION = 1e8;
+ 103 :	  uint32 public liquiditySlippageTolerance = 5e5; // 0.5%
  86:   
  87:     /// @notice rDPX Ratio required when bonding
- 88:     uint256 public constant RDPX_RATIO_PERCENTAGE = 25 * DEFAULT_PRECISION;
+ 88:     uint64 public constant RDPX_RATIO_PERCENTAGE = 25 * DEFAULT_PRECISION;
  89:   
  90:     /// @notice ETH Ratio required when bonding
- 91:     uint256 public constant ETH_RATIO_PERCENTAGE = 75 * DEFAULT_PRECISION;
+ 91:     uint128 public constant ETH_RATIO_PERCENTAGE = 75 * DEFAULT_PRECISION;
  92:   
  93:     /// @notice The % of rdpx to burn while bonding
- 94:     uint256 public rdpxBurnPercentage = 50 * DEFAULT_PRECISION;
+ 94:     uint128 public rdpxBurnPercentage = 50 * DEFAULT_PRECISION;
  95:   
  96:     /// @notice The % of rdpx sent to fee distributor while bonding
- 97:     uint256 public rdpxFeePercentage = 50 * DEFAULT_PRECISION;
+ 97:     uint128 public rdpxFeePercentage = 50 * DEFAULT_PRECISION;
  98:   
  99:     /// @notice The slippage tolernce in swaps in 1e8 precision
+  100:    uint256 public slippageTolerance = 5e5; // 0.5%
  101:  
  102:    /// @notice Liquidity slippage tolerance
- 103:    uint256 public liquiditySlippageTolerance = 5e5; // 0.5%

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L84C1-L103C59


The ``latestFundingPaymentPointer`` initially set to ``0`` can be efficiently represented using a ``uint32`` data type.

The ``roundingPrecision`` variable can be stored using a ``uint160`` data type, accommodating its value.

For the ``fundingDuration`` which pertains to a duration of ``100 years``, a ``uint64`` data type is suitable to store this value.

Here the state variable use ``6 slot`` here we reduce to ``4 slot`` and we save ``4000 gas``


```diff
FILE: contracts/perp-vault/PerpetualAtlanticVault.sol
     
- 89   uint256 public latestFundingPaymentPointer = 0;
+ 89   uint32 public latestFundingPaymentPointer = 0;
+ 104:    uint160 public roundingPrecision = 1e6;
+ 101:    uint64 public fundingDuration = 7 days;
  90:   
  91:     /// @dev the total number of active options
  92:     uint256 public totalActiveOptions;
  93:   
  94:     /// @dev genesis timestamp
  95:     uint256 public genesis;
  96:   
  97:     /// @dev the timestamp of the last update where funding was paid for
  98:     uint256 public lastUpdateTime;
  99:   
  100:    /// @dev the duration between funding payments
- 101:    uint256 public fundingDuration = 7 days;
  102:  
  103:    /// @dev the precision to round up to
- 104:    uint256 public roundingPrecision = 1e6;


```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L89C1-L104C42.


Here, ``DEFAULT_PRECISION`` set to ``uint128``, ``liquiditySlippageTolerance`` set to ``uint64`` and
``slippageTolerance`` set to ``uint64``.Here state variable uses, ``4 slot`` we change it to ``2 slot``,here
we save ``4000 gas.``



```diff
FILE: contracts/core/RdpxV2Core.sol
     
  66   /// @notice Precision used for prices, percentages and other calculations
- 67:     uint256 public constant DEFAULT_PRECISION = 1e8;
+ 67:     uint128 public constant DEFAULT_PRECISION = 1e8;
+ 73:     uint64 public liquiditySlippageTolerance = 5e5; // 0.5%
+ 76:	  uint64 public slippageTolerance = 5e5; // 0.5%
  68:   
  69:     /// @notice rdpxV2Core role
  70:     bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
  71:   
  72:     /// @notice liquidity slippage tolerance
- 73:     uint256 public liquiditySlippageTolerance = 5e5; // 0.5%
  74:   
  75:     /// @notice The slippage tolernce in swaps in 1e8 precision
- 76:     uint256 public slippageTolerance = 5e5; // 0.5%

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L67C1-L76C50


## [G-2] Use ``assembly`` to check for ``address(0)``

### Saves ``270``

### Total Instance ``45``

In Solidity, smart contract developers often encounter situations where they need to check whether an Ethereum address is set to the special value ``address(0)``. This represents an empty or non-existent address, which can be crucial for handling various scenarios in a contract. While you can achieve this using standard Solidity code, ``utilizing assembly`` can significantly save gas costs and optimize your contract.
Saves 6 gas per instance.


https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L84C8-L84C8

```diff
FILE: contracts/amo/UniV2LiquidityAmo.sol

    require(
      _tokenA != address(0) &&
        _tokenB != address(0) &&
        _pair != address(0) &&
        _rdpxV2Core != address(0) &&
        _rdpxOracle != address(0) &&
        _ammFactory != address(0) &&
        _ammRouter != address(0),
      "reLPContract: address cannot be 0"
    );

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L84C8-L84C8

```diff
FILE: contracts/amo/UniV2LiquidityAmo.sol

    require(_token != address(0), "reLPContract: token cannot be 0");
    require(_spender != address(0), "reLPContract: spender cannot be 0");

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L130

```diff
FILE: contracts/core/RdpxV2Core.sol

         tokenAddress: address(0),


```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L244

```diff
FILE: contracts/core/RdpxV2Core.sol

         require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");


```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L316C1-L325C53

```diff
FILE: contracts/core/RdpxV2Core.sol

    _validate(_dopexAMMRouter != address(0), 17);
    _validate(_dpxEthCurvePool != address(0), 17);
    _validate(_rdpxDecayingBonds != address(0), 17);
    _validate(_perpetualAtlanticVault != address(0), 17);
    _validate(_perpetualAtlanticVaultLP != address(0), 17);
    _validate(_rdpxReserve != address(0), 17);
    _validate(_rdpxV2ReceiptToken != address(0), 17);
    _validate(_feeDistributor != address(0), 17);
    _validate(_reLPContract != address(0), 17);
    _validate(_receiptTokenBonds != address(0), 17);


```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L362C3-L363C53

```diff
FILE: contracts/core/RdpxV2Core.sol

    _validate(_rdpxPriceOracle != address(0), 17);
    _validate(_dpxEthPriceOracle != address(0), 17);


```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L379C4-L379C40

```diff
FILE: contracts/core/RdpxV2Core.sol

        _validate(_addr != address(0), 17);

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L408C1-L409C43

```diff
FILE: contracts/core/RdpxV2Core.sol

    _validate(_token != address(0), 17);
    _validate(_spender != address(0), 17);

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L119

```diff
FILE: contracts/perp-vault/PerpetualAtlanticVault.sol

    _validate(_collateralToken != address(0), 1);


```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L190C1-L196C45

```diff
FILE: contracts/perp-vault/PerpetualAtlanticVault.sol

    _validate(_optionPricing != address(0), 1);
    _validate(_assetPriceOracle != address(0), 1);
    _validate(_volatilityOracle != address(0), 1);
    _validate(_feeDistributor != address(0), 1);
    _validate(_rdpx != address(0), 1);
    _validate(_perpetualAtlanticVaultLP != address(0), 1);
    _validate(_rdpxV2Core != address(0), 1);


```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L95C1-L95C1

```diff
FILE: contracts/perp-vault/PerpetualAtlanticVault.sol

      _perpetualAtlanticVault != address(0) || _rdpx != address(0),



```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L127C1-L135C34

```diff
FILE: contracts/perp-vault/PerpetualAtlanticVault.sol

        _tokenA != address(0) &&
        _tokenB != address(0) &&
        _pair != address(0) &&
        _rdpxV2Core != address(0) &&
        _tokenAReserve != address(0) &&
        _amo != address(0) &&
        _rdpxOracle != address(0) &&
        _ammFactory != address(0) &&
        _ammRouter != address(0),



```


## [G-3] Use ``calldata`` instead of ``memory`` for function parameters

### Saves ``8112``

### Total Instance ``26``

###  Saves 312 GAS per iteration

One way to save gas is by using the calldata keyword instead of memory when declaring function parameters. This optimization can significantly reduce gas consumption, particularly for read-only functions and functions that don't modify the state of the blockchain.




```diff
FILE: contracts/amo/UniV3LiquidityAmo.sol
     121.     Position memory current_position = positions_array[i];
     156.     AddLiquidityParams memory params
     179.     memory mintParams = INonfungiblePositionManager.MintParams(
     197.     Position memory pos = Position(
     218.     Position memory pos = positions_array[positionIndex];
     220.     memory collect_params = INonfungiblePositionManager.CollectParams(
     244.     memory decreaseLiquidityParams = INonfungiblePositionManager
     289.     ISwapRouter.ExactInputSingleParams memory swap_params = ISwapRouter
     343.    ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (bool, bytes memory) {
     344.    (bool success, bytes memory result) = _to.call{ value: _value }(_data);

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L121

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L156C5-L156C37

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L179C1-L180C1

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L197C1-L198C1

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L218C4-L220C73

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L244C1-L245C1

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L343


```diff
FILE: contracts/core/RdpxV2Core.sol
     
    242.    string memory _assetSymbol
    253.    ReserveAsset memory asset = ReserveAsset({
    271.     string memory _assetSymbol
    765.      uint256[] memory optionIds
    821.     uint256[] memory _amounts,
    822.    uint256[] memory _delegateIds,
    832.        uint256[] memory delegateReceiptTokenAmounts = new uint256[](
    841.            memory returnValues = BondWithDelegateReturnValue(0, 0, 0, 0);
    1136.       string memory _token
    1138.     ReserveAsset memory asset = reserveAsset[reservesIndex[_token]];


```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L242C1-L243C1

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L253

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L271

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L765

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L821C1-L822C35

 https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L832C1-L833C1
 
 https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L841
 
 
 https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1136C1-L1137C1
 
 https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1138
 
 
 
 ```diff
FILE: contracts/decaying-bonds/RdpxDecayingBonds.sol
        
    
    153.  external view returns (uint256[] memory) {
    155.   uint256[] memory tokenIds = new uint256[](ownerTokenCount);
     

```
 
 
 https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L153
 
 
  ```diff
FILE: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
        
    
    316.      uint256[] memory optionIds

    406.       uint256[] memory strikes

     

```
 
 https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L316
 
 https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L406
 
 
   ```diff
FILE: contracts/reLP/ReLPContract.sol
        
    
    316.          address[] memory path;


```
 
 https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L267C1-L268C1


## [G-4] Using bitmap to store bool states can save gas

Using a bitmap instead of a bool array or a bool mapping to store boolean states can save gas fees. This is because the bitmap can store 256 boolean values in a single slot instead of 256 slots, which can save gas when writing bool values or when reading multiple bool values from the same slot.

There are 2 instances:



```diff
FILE: contracts/core/RdpxV2Core.sol
     
79. mapping(uint256 => bool) public optionsOwned;

82. mapping(uint256 => bool) public fundingPaidFor;


```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L79C1-L82C50


## [G-5] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

See this issue for a detail description of the issue Refer:https://github.com/ethereum/solidity/issues/9232

There are 9 instances:



```diff
FILE: contracts/core/RdpxV2Bond.sol

22. bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");


```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L22C3-L22C66

```diff
contracts/decaying-bonds/RdpxDecayingBonds.sol

31.  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

  // Create a new role identifier for the Rdpx v2 core role
34.  bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");


```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L31C1-L35C1

```diff
contracts/dpxETH/DpxEthToken.sol

19.  bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
20.  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
21.  bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");


```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L19C2-L22C1

```diff

contracts/perp-vault/PerpetualAtlanticVault.sol

45.  bytes32 public constant MANAGER_ROLE = keccak256("MANAGER_ROLE");

  /// @dev Rdpx v2 core role which can purchase and settle options
48.  bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");


```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L45C1-L48C74


```diff
contracts/reLP/ReLPContract.sol

70. bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L70C3-L70C74


```

## [G-6] Struct can be packed ``(save 2000)``

Alternatively, we have the option to employ ``uint96``, capable of accommodating expiration times spanning over ``99+ years.``

By doing so, we optimize our storage usage, conserving one slot and reducing gas consumption by ``2000 units``.




```diff
FILE: decaying-bonds/RdpxDecayingBonds.sol


  40. struct Bond {
  41.    address owner;
- 42.    uint256 expiry;
+ 42.    uint96 expiry;
  43.    uint256 rdpxAmount;
  44.   }



```


https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L40C2-L44C4


## [G-7] Unused Function

Within the contract ``PerpetualAtlanticVaultLP.sol``, there exists a function labeled ``_beforeTokenTransfer`` that lacks any operational logic. It is advisable to consider its removal to reduce gas consumption.



```diff
FILE: contracts/decaying-bonds/RdpxDecayingBonds.sol


  function _beforeTokenTransfer(
    address from,
    address,
    uint256
  ) internal virtual {}


```


https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L231C3-L235C24


## [G-8] Unused EVENT can remove to save GAS

Within the contract, there exists an event declaration ``event EmergencyWithdraw(address sender);`` that remains unused throughout the entire contract. By eliminating this function, we can conserve gas resources.




```diff
FILE: contracts/decaying-bonds/RdpxDecayingBonds.sol


  53. event EmergencyWithdraw(address sender);


```


https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L53C1-L54C1


## [G-9] Use assembly to calculate hashes

Using assembly to calculate hashes can save 80 gas per instance.


Total instance: 12

Gas Saved: 960

```diff
FILE: contracts/core/RdpxV2Bond.sol

22.  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L22C1-L23C1


```diff
FILE: contracts/decaying-bonds/RdpxDecayingBonds.sol

31. bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

  // Create a new role identifier for the Rdpx v2 core role
34.  bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L31C3-L34C74

```diff
FILE: contracts/dpxETH/DpxEthToken.sol

19. bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
20. bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
21. bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L19C2-L21C66

```diff
FILE: contracts/perp-vault/PerpetualAtlanticVault.sol

45. bytes32 public constant MANAGER_ROLE = keccak256("MANAGER_ROLE");

  /// @dev Rdpx v2 core role which can purchase and settle options
48.  bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L45C3-L48C74

```diff
FILE: contracts/reLP/ReLPContract.sol

70.  bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");


```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L70C1-L71C1


```diff
FILE: contracts/amo/UniV3LiquidityAmo.sol

 106.     keccak256(abi.encodePacked(address(this), _tickLower, _tickUpper))

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L105C5-L106C73

```diff
FILE: contracts/core/RdpxV2Core.sol

1141.  keccak256(abi.encodePacked(asset.tokenSymbol)) ==
1142.        keccak256(abi.encodePacked(_token)),

```
     
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1141C1-L1142C45

## [G-10] ``abi.encode`` is more efficient than ``abi.encodePacked``

``abi.encode`` uses less gas than ``abi.encodePacked``: the gas saved depends on the number of arguments, with an average of ~90 per argument. Test available here.

Refer: https://gist.github.com/DadeKuma/6f87abe82039476d3b0820aa694649f5


Total instance: 3

Gas Saved: 270

```diff
FILE: contracts/amo/UniV3LiquidityAmo.sol

106      keccak256(abi.encodePacked(address(this), _tickLower, _tickUpper))

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L106C1-L107C1

```diff
FILE: contracts/core/RdpxV2Core.sol

1141.        keccak256(abi.encodePacked(asset.tokenSymbol)) ==
1142.       keccak256(abi.encodePacked(_token)),

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1141C5-L1142C45

## [G-11] Use assembly to ``emit`` an event

To efficiently emit events, it's possible to utilize assembly by making use of scratch space and the free memory pointer. This approach has the advantage of potentially avoiding the costs associated with memory expansion.

However, it's important to note that in order to safely optimize this process, it is preferable to cache and restore the free memory pointer.

A good example of such practice can be seen in Solady's codebase.

Refer:https://github.com/Vectorized/solady/blob/main/src/tokens/ERC1155.sol#L167



Total instance: 45

Gas Saved: 1710

```diff
FILE: contracts/amo/UniV2LiquidityAmo.sol

 152.   emit LogEmergencyWithdraw(msg.sender, tokens);
 117.   emit LogAssetsTransfered(msg.sender, tokenABalance, tokenBBalance);
 240.   emit LogAddLiquidity(
 288.   emit LogRemoveLiquidity(
 348.   emit LogSwap(

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol

```diff
FILE: contracts/amo/UniV3LiquidityAmo.sol

268.    emit log(positions_array.length);
269.    emit log(positions_mapping[pos.token_id].token_id);
321.    emit RecoveredERC20(tokenAddress, tokenAmount);
335.    emit RecoveredERC721(tokenAddress, token_id);
363.    emit LogAssetsTransfered(tokenABalance, tokenBBalance, tokenA, tokenB);


```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol

```diff
FILE: contracts/core/RdpxV2Core.sol

172.     emit LogEmergencyWithdraw(msg.sender, tokens);
185.    emit LogSetRdpxBurnPercentage(_rdpxBurnPercentage);
198.    emit LogSetRdpxFeePercentage(_rdpxFeePercentage);
208.    emit LogSetIsReLPActive(_isReLPActive);
220.    emit LogSetputOptionsRequired(_putOptionsRequired);
223.    emit LogSetBondMaturity(_bondMaturity);
263.    emit LogAssetAddedTotokenReserves(_asset, _assetSymbol);
289.    emit LogAssetRemovedFromtokenReserves(_assetSymbol, index);
349.    emit LogSetAddresses(addresses);
370.    emit LogSetPricingOracleAddresses(pricingOracleAddresses);
447.    emit LogSetBondDiscountFactor(_bondDiscountFactor);
461.    emit LogSetSlippageTolerance(_slippageTolerance);
782.    emit LogSettle(optionIds);
807.    emit LogProvideFunding(pointer, fundingAmount);
875.    emit LogBondWithDelegate(
932.    emit LogBond(rdpxRequired, wethRequired, receiptTokenAmount);
966.    emit LogAddToDelegate(_amount, _fee, delegates.length - 1);
989.    emit LogDelegateWithdraw(delegateId, amountWithdrawn);
1007.    emit LogSync();
1041.    emit LogRedeem(to, receiptTokenAmount);
1069.    emit LogUpperDepeg(_amount, wethReceived);
1123.    emit LogLowerDepeg(_rdpxAmount, _wethAmount, dpxEthReceived);


```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol


```diff
FILE: contracts/decaying-bonds/RdpxDecayingBonds.sol

124.    emit BondMinted(to, bondId, expiry, rdpxAmount);

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol


```diff
FILE: contracts/decaying-bonds/RdpxDecayingBonds.sol

   99.     emit LogSetReLpFactor(_reLPFactor);

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L70C1-L71C1


```diff
FILE: contracts/perp-vault/PerpetualAtlanticVaultLP.sol

  134.     emit Deposit(msg.sender, receiver, assets, shares);
  174.    emit Withdraw(msg.sender, receiver, owner, assets, shares);

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol

```diff
FILE: contracts/perp-vault/PerpetualAtlanticVault.sol

211.   emit AddressesSet(addresses);
230.   emit EmergencyWithdraw(msg.sender, tokens);
311.   emit Purchase(strike, amount, premium, to, msg.sender);
368.   emit Settle(ethAmount, rdpxAmount, optionIds);
389.   emit PayFunding(
451.   emit CalculateFunding(
485.   emit FundingPaid(
494.   emit FundingPaymentPointerUpdated(latestFundingPaymentPointer);
519.   emit FundingPaid(

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol

## [G-12] Use of emit inside a loop ``Saves 1026``


Emitting an event inside a loop performs a LOG op N times, where N is the loop length. Consider refactoring the code to emit the event only once at the end of loop. Gas savings should be multiplied by the average loop length.


```diff
FILE: contracts/perp-vault/PerpetualAtlanticVault.sol

451.      emit CalculateFunding(

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L451C1-L452C1


## [G-13] Unbounded Gas Consumption Risk Due to `External Call` Recipients

## Number of instances found 2


Add a two-step process for any critical address changes.


```diff
contracts/amo/UniV3LiquidityAmo.sol

344.     (bool success, bytes memory result) = _to.call{ value: _value }(_data);

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L344C1-L345C1

```diff
contracts/decaying-bonds/RdpxDecayingBonds.sol

98.   (bool success, ) = to.call{ value: amount, gas: gas }("");

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L98C1-L99C1
