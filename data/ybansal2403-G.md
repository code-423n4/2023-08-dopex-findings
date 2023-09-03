| Number | Optimization Details                                                                                | Instances |
| :----: | :-------------------------------------------------------------------------------------------------- | :-------: |
| [G-01] | Use assembly to check for the zero address                                                          |     44     |
| [G-02] | Non efficient zero initialization                                                                   |     9     |
| [G-03] | State variables only set in the constructor should be declared immutable                            |     2     |
| [G-04] | Use constants instead of type(uintx).max or type(uintx).min or type(intx).max or type(intx).min |     14     |
| [G-05] | Using ternary operator instead of if else saves gas                                                 |     2     |
| [G-06] | Cache state variables with stack variables                                                          |    34     |
| [G-07] | Do not calculate constants variables                                                                |     9     |

Total 7 issues

## [G-01] Use assembly to check for the zero address

Using assembly for address comparisons in Solidity can save gas because it allows for more direct access to the Ethereum Virtual Machine (EVM), reducing the overhead of higher-level operations. Solidity's high-level abstraction simplifies coding but can introduce additional gas costs. Using assembly for simple operations like address comparisons can be more gas-efficient.

_Total 44 instances - 6 file:_

### Instance#1-7:

finding are marked as <=FOUND

```solidity
File: contracts/amo/UniV2LiquidityAmo.sol
83:require(
84:      _tokenA != address(0) && //<FOUND
85:        _tokenB != address(0) && //<FOUND
86:        _pair != address(0) && //<FOUND
87:        _rdpxV2Core != address(0) && //<FOUND
88:        _rdpxOracle != address(0) && //<FOUND
89:        _ammFactory != address(0) && //<FOUND
90:        _ammRouter != address(0), //<FOUND
91:      "reLPContract: address cannot be 0"
92:    );
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L83C4-L92C7

### Instance#8-9:

finding are marked as <=FOUND

```solidity
File: contracts/amo/UniV2LiquidityAmo.sol
131:    require(_token != address(0), "reLPContract: token cannot be 0"); //<FOUND
132:    require(_spender != address(0), "reLPContract: spender cannot be 0"); //<FOUND
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L131C5-L132C74

### Instance#10:

```solidity
File: contracts/core/RdpxV2Core.sol
130:tokenAddress: address(0),
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L130C7-L130C32

### Instance#11:

```solidity
File: contracts/core/RdpxV2Core.sol
244:require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L244

### Instance#12-21:

finding are marked as <=FOUND

```solidity
File: contracts/core/RdpxV2Core.sol
316:    _validate(_dopexAMMRouter != address(0), 17);//<FOUND
317:    _validate(_dpxEthCurvePool != address(0), 17);//<FOUND
318:    _validate(_rdpxDecayingBonds != address(0), 17);//<FOUND
319:    _validate(_perpetualAtlanticVault != address(0), 17);//<FOUND
320:    _validate(_perpetualAtlanticVaultLP != address(0), 17);//<FOUND
321:    _validate(_rdpxReserve != address(0), 17);//<FOUND
322:    _validate(_rdpxV2ReceiptToken != address(0), 17);//<FOUND
323:    _validate(_feeDistributor != address(0), 17);//<FOUND
324:    _validate(_reLPContract != address(0), 17);//<FOUND
325:    _validate(_receiptTokenBonds != address(0), 17);//<FOUND
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L316C5-L325C53

### Instance#22-26:

finding are marked as <=FOUND

```solidity
File: contracts/core/RdpxV2Core.sol
362:    _validate(_rdpxPriceOracle != address(0), 17);//<FOUND
363:    _validate(_dpxEthPriceOracle != address(0), 17);//<FOUND

379:  _validate(_addr != address(0), 17);//<FOUND

408:    _validate(_token != address(0), 17);//<FOUND
409:    _validate(_spender != address(0), 17);//<FOUND
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L362C5-L363C53
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L379C3-L379C40
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L408C5-L409C43

### Instance#27-34:

finding are marked as <=FOUND

```solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol
119:_validate(_collateralToken != address(0), 1);//<FOUND

190:    _validate(_optionPricing != address(0), 1);//<FOUND
191:    _validate(_assetPriceOracle != address(0), 1);//<FOUND
192:    _validate(_volatilityOracle != address(0), 1);//<FOUND
193:    _validate(_feeDistributor != address(0), 1);//<FOUND
194:    _validate(_rdpx != address(0), 1);//<FOUND
195:    _validate(_perpetualAtlanticVaultLP != address(0), 1);//<FOUND
196:    _validate(_rdpxV2Core != address(0), 1);//<FOUND
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L119C5-L119C50
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L190C1-L196C45

### Instance#35:

finding are marked as <=FOUND

```solidity
File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
95:_perpetualAtlanticVault != address(0) || _rdpx != address(0),//<FOUND
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L95C7-L95C68

### Instance#36-44:

finding are marked as <=FOUND

```solidity
File: contracts/reLP/ReLPContract.sol
127:        _tokenA != address(0) &&//<FOUND
128:        _tokenB != address(0) &&//<FOUND
129:        _pair != address(0) &&//<FOUND
130:        _rdpxV2Core != address(0) &&//<FOUND
131:        _tokenAReserve != address(0) &&//<FOUND
132:        _amo != address(0) &&//<FOUND
133:        _rdpxOracle != address(0) &&//<FOUND
134:        _ammFactory != address(0) &&//<FOUND
135:        _ammRouter != address(0),//<FOUND
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L127C7-L135C34

## [G-02] Non efficient zero initialization

_9 instances - 5 file:_

### Instance#1: Initialisation of i as zero is redundant

```solidity
File: contracts/amo/UniV2LiquidityAmo.sol
147: for (uint256 i = 0; i < tokens.length; i++) {
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L147

### Instance#2: Initialisation of i as zero is redundant

```solidity
File: contracts/amo/UniV3LiquidityAmo.sol
120: for (uint i = 0; i < positions_array.length; i++) {
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L120

### Instance#3-5: Initialisation of i as zero is redundant

```solidity
File: contracts/core/RdpxV2Core.sol
167:for (uint256 i = 0; i < tokens.length; i++) {

775: for (uint256 i = 0; i < optionIds.length; i++) {

836:for (uint256 i = 0; i < _amounts.length; i++) {
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L167
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L775
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L836

### Instance#6: Initialisation of i as zero is redundant

```solidity
File: contracts/decaying-bonds/RdpxDecayingBonds.sol
103:  for (uint256 i = 0; i < tokens.length; i++) {
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L103

### Instance#7-9: Initialisation of i as zero is redundant

```solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol
225:for (uint256 i = 0; i < tokens.length; i++) {

328: for (uint256 i = 0; i < optionIds.length; i++) {

413: for (uint256 i = 0; i < strikes.length; i++) {
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L225C5-L225C50
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L328
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L413

## [G-03] State variables only set in the constructor should be declared immutable

Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

_2 instances - 1 file:_

### Instance#1-2:

```solidity
File: contracts/amo/UniV3LiquidityAmo.sol
// @audit rdpx (constructor)
77: rdpx = _rdpx;
/// @audit rdpxV2Core (constructor)
78:    rdpxV2Core = _rdpxV2Core;
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L77C4-L78C30

## [G-04] Use constants instead of type(uintx).max or type(uintx).min or type(intx).max or type(intx).min

_Total 14 instances - 4 file:_

### Instance#1-2:

```solidity
File: contracts/amo/UniV3LiquidityAmo.sol
126:   type(uint128).max,
127:   type(uint128).max
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L126C11-L127C28

### Instance#3:

```solidity
File: contracts/amo/UniV3LiquidityAmo.sol
190:type(uint256).max
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L190C9-L190C26

### Instance#4-5:

```solidity
File: contracts/amo/UniV3LiquidityAmo.sol
223: type(uint128).max,
224:        type(uint128).max
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L223C1-L224C26

### Instance#6-8:

finding are marked as <=FOUND

```solidity
File: contracts/core/RdpxV2Core.sol
343:    IERC20WithBurn(weth).approve(addresses.dopexAMMRouter, type(uint256).max);//<FOUND
344:    IERC20WithBurn(weth).approve(addresses.dpxEthCurvePool, type(uint256).max);//<FOUND
345:    IERC20WithBurn(weth).approve(
346:      addresses.rdpxV2ReceiptToken,
347:      type(uint256).max //<FOUND
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L343C1-L347C24

### Instance#9-11:

finding are marked as <=FOUND

```solidity
File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
106:    collateral.approve(_perpetualAtlanticVault, type(uint256).max);//<FOUND
107:    ERC20(rdpx).approve(_perpetualAtlanticVault, type(uint256).max);//<FOUND

155:  if (allowed != type(uint256).max) {//<FOUND
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L106C1-L107C69
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L155

### Instance#12-14:

finding are marked as <=FOUND

```solidity
File: contracts/reLP/ReLPContract.sol
152: type(uint256).max//<=FOUND

157: type(uint256).max//<=FOUND

162: type(uint256).max//<=FOUND
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L152C6-L152C24
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L157
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L162

## [G-05] Using ternary operator instead of if-else saves gas

_Total 2 instances - 2 file:_

### Instance#1:

```solidity
File: contracts/amo/UniV3LiquidityAmo.sol
145:if (use_safe_approve) {
146:      // safeApprove needed for USDT and others for the first approval
147:      // You need to approve 0 every time beforehand for USDT: it resets
148:      TransferHelper.safeApprove(_token, _target, _amount);
149:    } else {
150:      IERC20WithBurn(_token).approve(_target, _amount);
151:    }
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L145C5-L151C6

### Instance#2:

```solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol
597:if (lastUpdateTime > nextFundingPaymentTimestamp() - fundingDuration) {
598:        startTime = lastUpdateTime;
599:      } else {
600:        startTime = nextFundingPaymentTimestamp() - fundingDuration;
601:      }
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L597C7-L601C8

## [G-06] Cache state variables with stack variables

Caching of a state variable replaces each Gwarmaccess (100 gas) with a cheaper stack read.

_Total 34 instances - 5 file:_

The following instances are missed in the automated bot report

### Instance#1-11:

```solidity
File:contracts/core/RdpxV2Core.sol
//@audit `reserveAsset.length` at line 246
261:reservesIndex[_assetSymbol] = reserveAsset.length - 1;

//@audit `reserveAsset.length` at line 280
283: reserveAsset[index] = reserveAsset[reserveAsset.length - 1];

//@audit `reserveAsset.length` at line 391
392: amoAddresses[_index] = amoAddresses[amoAddresses.length - 1];

//@audit `addresses.dpxEthCurvePool` at line 529
530:uint256 dpxEthBalance = IStableSwap(addresses.dpxEthCurvePool).balances(

//@audit `addresses.rdpxDecayingBonds` at line 638
643:IRdpxDecayingBonds(addresses.rdpxDecayingBonds).decreaseAmount(

//@audit `addresses.rdpxReserve` at line 648
677:IRdpxReserve(addresses.rdpxReserve).withdraw(

//@audit `reserveAsset[reservesIndex["RDPX"]].tokenAddress` at line 653,657
662:IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress)

//@audit `addresses.perpetualAtlanticVault` at line 796
800: fundingAmount = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)

//@audit `delegate.activeCollateral` at line 983
985:delegate.amount = delegate.activeCollateral;

//@audit `addresses.receiptTokenBonds` at line 1026
1032: RdpxV2Bond(addresses.receiptTokenBonds).burn(id);

//@audit `addresses.perpetualAtlanticVault` at line 1189,1193
1196:  wethRequired += IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L261C5-L261C59
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L283C4-L283C65
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L392
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L530
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L643
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L677
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L662
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L800
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L985
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1032
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1196

### Instance#12:

```solidity
File:contracts/decaying-bonds/RdpxDecayingBonds.sol
//@audit `_tokenIdCounter` at line 130
131:_tokenIdCounter.increment();
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L131

### Instance#13-23:

```solidity
File:contracts/perp-vault/PerpetualAtlanticVault.sol
//@audit `addresses.perpetualAtlanticVaultLP` at line 246
249: collateralToken.approve(addresses.perpetualAtlanticVaultLP, 0);

//@audit `addresses.perpetualAtlanticVaultLP` at line 348,355,359,362
364:    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addRdpx(

//@audit `addresses.rdpxV2Core` at line 349
353:IERC20WithBurn(addresses.rdpx).safeTransferFrom(
354: addresses.rdpxV2Core,

//@audit `totalFundingForEpoch[latestFundingPaymentPointer]` at line 385,387,391
395:return (totalFundingForEpoch[latestFundingPaymentPointer]);

//@audit `latestFundingPaymentPointer` at line 504
522:  latestFundingPaymentPointer

//@audit `addresses.perpetualAtlanticVaultLP` at line 511
515:IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addProceeds(

//@audit `roundingPrecision` at line 577
581:return _strike - remainder + roundingPrecision;

//@audit `_tokenIdCounter` at line 589
590:  _tokenIdCounter.increment();

//@audit `fundingDuration` at line 597
600:startTime = nextFundingPaymentTimestamp() - fundingDuration;

//@audit `lastUpdateTime` at line 597,598
607:uint256 startTime = lastUpdateTime;

//@audit `fundingRates[latestFundingPaymentPointer]` at line 595,603,610
611:fundingRates[latestFundingPaymentPointer]
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L249
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L364
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L353C4-L354C28
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L395C5-L395C64
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L522
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L515C5-L515C79
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L581C6-L581C54
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L590
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L600
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L607
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L611

### Instance#24-26:

```solidity
File:contracts/perp-vault/PerpetualAtlanticVaultLP.sol
//@audit `_totalCollateral` at line 192
195:_totalCollateral += proceeds;

//@audit `_totalCollateral` at line 201
204:_totalCollateral -= loss;

//@audit `_rdpxCollateral` at line 210
213: _rdpxCollateral += amount;


```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L189
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L204
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L213

### Instance#27-34:

```solidity
File:contracts/reLP/ReLPContract.sol
//@audit `addresses.ammRouter` at line 151,156
161:addresses.ammRouter,

//@audit `addresses.tokenA` at line 205,224,258,270,287,302
304:IERC20WithBurn(addresses.tokenA).balanceOf(address(this))

//@audit `addresses.tokenB` at line 206,259,269
288:addresses.tokenB,

//@audit `addresses.tokenB` at line 206,259,269
288:addresses.tokenB,

//@audit `addresses.rdpxV2Core` at line 303
306:IRdpxV2Core(addresses.rdpxV2Core).sync();

//@audit `addresses.amo` at line 244,298
299:IUniV2LiquidityAmo(addresses.amo).sync();

//@audit `addresses.ammRouter` at line 257,277
286:(, , uint256 lp) = IUniswapV2Router(addresses.ammRouter).addLiquidity(

//@audit `liquiditySlippageTolerance` at line 251
254: ((tokenAToRemove * tokenAInfo.tokenAPrice) * liquiditySlippageTolerance) /


```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L161
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L304
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L288
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L306
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L299
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L286
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L254

## [G-07] Do not calculate constant variables

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

_Total 9 instances - 5 file:_

### Instance#1:

```solidity
File:contracts/core/RdpxV2Bond.sol
22: bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L22

### Instance#2-3:

finding are marked as <=FOUND

```solidity
File:contracts/decaying-bonds/RdpxDecayingBonds.sol
31:  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");//<FOUND
32:
33:  // Create a new role identifier for the Rdpx v2 core role
34:  bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");//<FOUND
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L31C1-L34C74

### Instance#4-6:

finding are marked as <=FOUND

```solidity
File:contracts/dpxETH/DpxEthToken.sol
19:  bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");//<FOUND
20:  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");//<FOUND
21:  bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");//<FOUND
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L19C1-L21C66

### Instance#7-8:

finding are marked as <=FOUND

```solidity
File:contracts/perp-vault/PerpetualAtlanticVault.sol
45:bytes32 public constant MANAGER_ROLE = keccak256("MANAGER_ROLE");//<FOUND
46:
47:  /// @dev Rdpx v2 core role which can purchase and settle options
48:  bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");//<FOUND
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L45C3-L48C74

### Instance#9:

finding are marked as <=FOUND

```solidity
File:contracts/reLP/ReLPContract.sol
70: bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L70
