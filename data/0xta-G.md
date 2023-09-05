# Gas Optimization

# Summary

| Number | Gas                                                                                                      | Context |
| :----: | :------------------------------------------------------------------------------------------------------- | :-----: |
| [G-01] | Avoid emitting storage values                                                                            |    8    |
| [G-02] | Use constants instead of type(uintx).max                                                                 |   15    |
| [G-03] | Use mappings instead of arrays                                                                           |    5    |
| [G-04] | Use assembly to write address storage values                                                             |    4    |
| [G-05] | Expressions for constant values such as a call to keccak256(), should use immutable rather than constant |   11    |
| [G-06] | Zero value check posiible Gas Save                                                                       |    3    |
| [G-07] | Using > 0 costs more gas than != 0 when used on a uint in a require() statement                          |    5    |
| [G-08] | Do not calculate constants                                                                               |    3    |
| [G-09] | Pre-increment and pre-decrement are cheaper than +1,-1                                                   |    5    |
| [G-10] | Use hardcode address instead `address(this)`                                                             |   38    |
| [G-11] | Using storage instead of memory for structs/arrays saves gas                                             |    3    |
| [G-12] | USING CALLDATA INSTEAD OF MEMORY FOR READ-ONLY ARGUMENTS IN EXTERNAL FUNCTIONS SAVES GAS                 |    4    |
| [G-13] | Shorten the array rather than copying to a new one                                                       |    5    |
| [G-14] | Duplicated if()/require checks should be refactored to a modifier or function                            |    3    |
| [G-15] | Empty blocks should be removed or emit something to save some amount of Gas                              |    1    |
| [G-16] | CAN MAKE THE VARIABLE OUTSIDE THE LOOP                                                                   |    4    |
| [G-17] | Add `unchecked` for substractions where the operands can't underflow                                     |    3    |
| [G-18] | USE ASSEMBLY TO CHECK FOR ADDRESS(0)                                                                     |    2    |

## [G-1] Avoid emitting storage values

Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. We can avoid unecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.

```solidity

349    emit LogSetAddresses(addresses);

370    emit LogSetPricingOracleAddresses(pricingOracleAddresses);

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L349

```solidity

211    emit AddressesSet(addresses);


389    emit PayFunding(
390      msg.sender,
391      totalFundingForEpoch[latestFundingPaymentPointer],
392      latestFundingPaymentPointer
393    );



451     emit CalculateFunding(
452        msg.sender,
453        amount,
454        strike,
455        premium,
456:        latestFundingPaymentPointer
457      );




485   emit FundingPaid(
486          msg.sender,
487          ((currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
488            1e18),
489          latestFundingPaymentPointer
480        );



494      emit FundingPaymentPointerUpdated(latestFundingPaymentPointer);    storage values




519    emit FundingPaid(
520      msg.sender,
521      ((currentFundingRate * (block.timestamp - startTime)) / 1e18),
522      latestFundingPaymentPointer
523    );
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol

## [G-2] Use constants instead of type(uintx).max

It's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

```solidity
File: RdpxV2Core.sol

341:      type(uint256).max
342    );
343:    IERC20WithBurn(weth).approve(addresses.dopexAMMRouter, type(uint256).max);
344:    IERC20WithBurn(weth).approve(addresses.dpxEthCurvePool, type(uint256).max);
345    IERC20WithBurn(weth).approve(
346      addresses.rdpxV2ReceiptToken,
347:      type(uint256).max
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol

```solidity
File: UniV3LiquidityAmo.sol

126          type(uint128).max,
127          type(uint128).max

190        type(uint256).max

223        type(uint128).max,
224        type(uint128).max

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol

```solidity

209      type(uint256).max

247        type(uint256).max
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol

```solidity

106    collateral.approve(_perpetualAtlanticVault, type(uint256).max);

107    ERC20(rdpx).approve(_perpetualAtlanticVault, type(uint256).max);

155      if (allowed != type(uint256).max) {
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol

```solidity
File: ReLPContract.sol

152      type(uint256).max

157      type(uint256).max

162      type(uint256).max

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L152

## [G-3] Use mappings instead of arrays

Arrays are useful when you need to maintain an ordered list of data that can be iterated over, but they have a higher gas cost for read and write operations, especially when the size of the array is large. This is because Solidity needs to iterate over the entire array to perform certain operations, such as finding a specific element or deleting an element.

Mappings, on the other hand, are useful when you need to store and access data based on a key, rather than an index. Mappings have a lower gas cost for read and write operations, especially when the size of the mapping is large, since Solidity can perform these operations based on the key directly, without needing to iterate over the entire data structure.

```solidity
File: UniV3LiquidityAmo.sol

63  Position[] public positions_array;
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L63

```solidity
File: RdpxV2Core.sol

61  ReserveAsset[] public reserveAsset;

64  address[] public amoAddresses;

70  string[] public reserveTokens;

121  Delegate[] public delegates;
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L61

## [G-4] Use assembly to write address storage values

```solidity
File: UniV3LiquidityAmo.sol

    rdpx = _rdpx;
    rdpxV2Core = _rdpxV2Core;
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L76-L80

```solidity
File: RdpxV2Core

126    weth = _weth;
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L126

```solidity
File: PerpetualAtlanticVaultLP.sol


    perpetualAtlanticVault = IPerpetualAtlanticVault(_perpetualAtlanticVault);
    rdpxRdpxV2Core = _rdpxRdpxV2Core;
    collateralSymbol = _collateralSymbol;
    rdpx = _rdpx;
    collateral = ERC20(_collateral);
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L81-L108

## [G-5] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

```solidity
File: DpxEthToken.sol

19  bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");

20  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

21  bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L19-L21

```solidity
File: PerpetualAtlanticVault.sol

45  bytes32 public constant MANAGER_ROLE = keccak256("MANAGER_ROLE");6

48  bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L44C1-L48C74

```solidity
File: ReLPContract.sol

70    bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L70

```solidity
File: RdpxV2Bond.sol


22  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L22

```solidity
File: RdpxDecayingBonds.sol

31    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");


34    bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L31

## [G-6] Zero value check posiible Gas Save

```solidity
File: UniV3LiquidityAmo.sol

283    IERC20WithBurn(_tokenA).transferFrom(
284      rdpxV2Core,
285      address(this),
286      _amountAtoB
287    );

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L283-L287

```solidity
File: PerpetualAtlanticVaultLP.sol

    collateral.transferFrom(msg.sender, address(this), assets);
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L128

## [G-7] Using > 0 costs more gas than != 0 when used on a uint in a require() statement

```solidity
File: UniV2LiquidityAmo.sol

112    require(
113      _slippageTolerance > 0,
114      "reLPContract: slippage tolerance must be greater than 0"
115    );



133    require(_amount > 0, "reLPContract: amount must be greater than 0");

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L113

```solidity
File: ReLPContract.sol

93    require(
94      _reLPFactor > 0,
95      "reLPContract: reLP factor must be greater than 0"
96    );



174    require(
175      _liquiditySlippageTolerance > 0,
176      "reLPContract: liquidity slippage tolerance must be greater than 0"
177    );



189    require(
190      _slippageTolerance > 0,
191      "reLPContract: slippage tolerance must be greater than 0"
192    );
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L174-L177

## [G-8] Do not calculate constants

```solidity
File: RdpxV2Core.sol


88  uint256 public constant RDPX_RATIO_PERCENTAGE = 25 * DEFAULT_PRECISION;

91  uint256 public constant ETH_RATIO_PERCENTAGE = 75 * DEFAULT_PRECISION;

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L91

## [G-9] Pre-increment and pre-decrement are cheaper than +1,-1

```solidity
File: RdpxV2Core.sol

261    reservesIndex[_assetSymbol] = reserveAsset.length - 1;

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L261

```solidity
File: UniV2LiquidityAmo.sol

231        block.timestamp + 1

279        block.timestamp + 1

342        block.timestamp + 1

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol

## [G-10] Use hardcode address instead address(this)

```solidity
File: RdpxV2Core.sol

169      token.safeTransfer(msg.sender, token.balanceOf(address(this)));

483    ).purchase(_amount, address(this));

573      address(this),

654        .safeTransferFrom(msg.sender, address(this), _rdpxAmount);

911      address(this),

953    IERC20WithBurn(weth).safeTransferFrom(msg.sender, address(this), _amount);

998        address(this)

1060      address(this),

1102          address(this),

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L169

```solidity
File: PerpetualAtlanticVaultLP.sol


128    collateral.transferFrom(msg.sender, address(this), assets);

192      collateral.balanceOf(address(this)) >= _totalCollateral + proceeds,

201      collateral.balanceOf(address(this)) == _totalCollateral - loss,

210      IERC20WithBurn(rdpx).balanceOf(address(this)) >= _rdpxCollateral + amount,

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L280C1-L281C57

```solidity
File:  UniV2LiquidityAmo.sol


149      token.safeTransfer(msg.sender, token.balanceOf(address(this)));

162      address(this)

165      address(this)

212      address(this),

217      address(this),

230        address(this),

278        address(this),

323      address(this),

341        address(this),

363    lpTokenBalance = IERC20WithBurn(addresses.pair).balanceOf(address(this));

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol

```solidity
File: ReLPContract.sol

245      address(this),

263      address(this),

282        address(this),

293      address(this),

304      IERC20WithBurn(addresses.tokenA).balanceOf(address(this))
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L228C1-L230C33

```solidity
File: UniV3LiquidityAmo.sol


106      keccak256(abi.encodePacked(address(this), _tickLower, _tickUpper))

160      address(this),

165      address(this),

189        address(this),

285      address(this),

294        address(this),

331      address(this),

354 uint256 tokenABalance = IERC20WithBurn(tokenA).balanceOf(address(this));

355    uint256 tokenBBalance = IERC20WithBurn(tokenB).balanceOf(address(this));
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol

## [G-11] Using storage instead of memory for structs/arrays saves gas

Using a memory pointer for a storage struct/array will effectively load all the fields of that data type from storage (SLOAD) into memory (MSTORE). Using a storage pointer will allow you to read specific fields from storage as you need them. If you are not going to use all of the fields of your data type then you should use a storage pointer so that you don’t incur extra Gcoldsload (2100 gas) for fields that you will never use.

```solidity
File: RdpxV2Core.sol


function getReserveTokenInfo(
    string memory _token
  ) public view returns (address, uint256, string memory) {
    ReserveAsset memory asset = reserveAsset[reservesIndex[_token]];

    _validate(
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1138

```solidity
File: UniV3LiquidityAmo.sol


    for (uint i = 0; i < positions_array.length; i++) {
      Position memory current_position = positions_array[i];
      INonfungiblePositionManager.CollectParams
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L121

```solidity
File: UniV3LiquidityAmo.sol


  function removeLiquidity(
    uint256 positionIndex,
    uint256 minAmount0,
    uint256 minAmount1
  ) public onlyRole(DEFAULT_ADMIN_ROLE) {
    Position memory pos = positions_array[positionIndex];
    INonfungiblePositionManager.CollectParams

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L213-L219

## [G-12] USING CALLDATA INSTEAD OF MEMORY FOR READ-ONLY ARGUMENTS IN EXTERNAL FUNCTIONS SAVES GAS

You can use calldata instead of memory for read-only arguments in external functions to optimize gas usage. When you use calldata, the function parameters are not copied to memory, reducing gas consumption for large input data. This is especially useful when dealing with large arrays or complex data structures as function arguments.

```solidity
File: RdpxV2Core.sol


  function bondWithDelegate(
    address _to,
:    uint256[] memory _amounts,
:   uint256[] memory _delegateIds,
    uint256 rdpxBondId
  ) public returns (uint256 receiptTokenAmount, uint256[] memory) {
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L821-L822

```solidity
File: PerpetualAtlanticVault.sol

  function settle(
:    uint256[] memory optionIds
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L316

```solidity
File:RdpxV2Core.sol

  function settle(
:    uint256[] memory optionIds
  )

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L765

## [G-13] Shorten the array rather than copying to a new one

```solidity
File: RdpxV2Core.sol


   uint256[] memory delegateReceiptTokenAmounts = new uint256[](
      _amounts.length


1093      path = new address[](2);
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L832

```solidity
File: RdpxDecayingBonds.sol

155    uint256[] memory tokenIds = new uint256[](ownerTokenCount);
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol

```solidity
File: ReLPContract.sol

268    path = new address[](2);
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L268

```solidity
File: UniV2LiquidityAmo


331    path = new address[](2);
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L331

## [G-14] Duplicated if() checks should be refactored to a modifier or function

```solidity
File: RdpxV2Core.sol

856          if (putOptionsRequired) {
920          if (putOptionsRequired) {
1195         if (putOptionsRequired) {

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L856

## [G-15] Empty blocks should be removed or emit something to save some amount of Gas

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.
If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation.

If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be
nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when
the code is later modified (if( x ) {}else if(y){ . . . }else{ . . . } => if ( !x) {if(y) { . . . }else{ . . .}}) .
Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas.

    	  } else {
    			  // todo - check else case for any ETH lost
    		   }
    	    }

```solidity
File: PerpetualAtlanticVaultLP.sol

  function _beforeTokenTransfer(
    address from,
    address,
    uint256
  ) internal virtual {}
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L231-L235

## [G-16] CAN MAKE THE VARIABLE OUTSIDE THE LOOP

```solidity
File: UniV3LiquidityAmo.sol


121      Position memory current_position = positions_array[i];
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L121

```solidity
File: PerpetualAtlanticVault.sol


329      uint256 strike = optionPositions[optionIds[i]].strike;
330      uint256 amount = optionPositions[optionIds[i]].amount;
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol

## [G-17] Add `unchecked` for substractions where the operands can't underflow

```solidity
File: RdpxV2Core.sol

261    reservesIndex[_assetSymbol] = reserveAsset.length - 1;
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L261

```solidity
File: PerpetualAtlanticVault.sol

576      return _strike - remainder + roundingPrecision;
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L576

## [G-18] USE ASSEMBLY TO CHECK FOR ADDRESS(0)

```solidity
File: ReLPContract.sol

   require(
      _tokenA != address(0) &&
        _tokenB != address(0) &&
        _pair != address(0) &&
        _rdpxV2Core != address(0) &&
        _tokenAReserve != address(0) &&
        _amo != address(0) &&
        _rdpxOracle != address(0) &&
        _ammFactory != address(0) &&
        _ammRouter != address(0),
      "reLPContract: address cannot be 0"
    );
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L115-L164

```solidity
File:UniV2LiquidityAmo.sol

  _tokenA != address(0) &&
        _tokenB != address(0) &&
        _pair != address(0) &&
        _rdpxV2Core != address(0) &&
        _rdpxOracle != address(0) &&
        _ammFactory != address(0) &&
        _ammRouter != address(0),
      "reLPContract: address cannot be 0"
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol