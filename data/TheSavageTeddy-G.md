## Gas optimizations

| Number | Issue Title                                     | Number of Instances |
|--------|-------------------------------------------------|---------------------|
| [G-01] | Multiplication by powers of 2 should use bit shifting | 1                   |
| [G-02] | Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead | 7                   |
| [G-03] | Use constants instead of `type(uintx).max`      | 17                  |
| [G-04] | Use `!=` instead of `>` for unsigned integar comparisons | 19                  |
| [G-05] | Do not calculate constants                     | 11                  |
| [G-06] | Hardcode address instead of using `address(this)` | 41                  |
| [G-07] | Consider using `bytes32` rather than a `string` | 4                   |
| [G-08] | `internal` functions only called once can be inlined to save gas | 1                   |


### [G-01] Multiplication by powers of 2 should use bit shifting

`<x> * 2` is the same as `<x> << 1`. While the compiler uses the SHR opcode to accomplish both, the version that uses multiplication incurs an overhead of 20 gas due to JUMPs.

*There is 1 instance of this issue:*

```solidity
File: reLP/ReLPContract.sol
232:     uint256 tokenAToRemove = ((((_amount * 4) * 1e18) /
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L232

### [G-02] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than `32` bytes, your contract's gas usage may be higher. This is because the EVM operates on `32` bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from `32` bytes to the desired size.

*There are 7 instances of this issue:*

```solidity
File: amo/UniV3LiquidityAmo.sol
  46:     uint24 fee_tier;
  55:     uint24 _fee;
  98:     uint24 _fee
  277:     uint24 _fee_tier,
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L46-L277

```solidity
File: core/RdpxV2Bond.sol
  58:     bytes4 interfaceId
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Bond.sol#L58

```solidity
File: decaying-bonds/RdpxDecayingBonds.sol
  175:     bytes4 interfaceId
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L175

```solidity
File: perp-vault/PerpetualAtlanticVault.sol
  646:     bytes4 interfaceId
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L646

### [G-03] Use constants instead of `type(uintx).max`

Using `type(uintx).max` such as `type(uint128).max` incurs more gas in distribution and for each transaction. Constants (such as `340282366920938463463374607431768211455` for `type(uint128).max`) should be used instead.

*There are 17 instances of this issue:*

```solidity
File: amo/UniV3LiquidityAmo.sol
  126:           type(uint128).max,
  127:           type(uint128).max
  190:         type(uint256).max
  223:         type(uint128).max,
  224:         type(uint128).max
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L126-L224

```solidity
File: core/RdpxV2Core.sol
  341:       type(uint256).max
  343:     IERC20WithBurn(weth).approve(addresses.dopexAMMRouter, type(uint256).max);
  344:     IERC20WithBurn(weth).approve(addresses.dpxEthCurvePool, type(uint256).max);
  347:       type(uint256).max
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L341-L347

```solidity
File: perp-vault/PerpetualAtlanticVault.sol
  209:       type(uint256).max
  247:         type(uint256).max
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L209-L247

```solidity
File: perp-vault/PerpetualAtlanticVaultLP.sol
  106:     collateral.approve(_perpetualAtlanticVault, type(uint256).max);
  107:     ERC20(rdpx).approve(_perpetualAtlanticVault, type(uint256).max);
  155:       if (allowed != type(uint256).max) {
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L106-L155

```solidity
File: reLP/ReLPContract.sol
  152:       type(uint256).max
  157:       type(uint256).max
  162:       type(uint256).max
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L152-L162


### [G-04] Use `!=` instead of `>` for unsigned integar comparisons

As a `uint` cannot be below `0`, instead of checking if it is above 0 (`x > 0`), you can check if it is not equal to zero (`x != 0`) which will perform the same check.

Using `!= 0` is cheaper than using `> 0`, therefore `!=` should be used for *unsigned* integar comparisons.


*There are 19 instances of this issue:*

```solidity
File: amo/UniV2LiquidityAmo.sol
  113:       _slippageTolerance > 0,
  133:     require(_amount > 0, "reLPContract: amount must be greater than 0");
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L113-L133

```solidity
File: core/RdpxV2Core.sol
  183:     _validate(_rdpxBurnPercentage > 0, 3);
  196:     _validate(_rdpxFeePercentage > 0, 3);
  231:     _validate(_bondMaturity > 0, 3);
  410:     _validate(_amount > 0, 17);
  444:     _validate(_bondDiscountFactor > 0, 3);
  458:     _validate(_slippageTolerance > 0, 3);
  556:       minAmount > 0 ? minAmount : minOut
  838:       _validate(_amounts[i] > 0, 4);
  901:     _validate(_amount > 0, 4);
  984:     _validate(amountWithdrawn > 0, 15);
  1021:     _validate(bonds[id].timestamp > 0, 6);
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L183-L1021

```solidity
File: perp-vault/PerpetualAtlanticVault.sol
  265:     _validate(amount > 0, 2);
  414:       _validate(optionsPerStrike[strikes[i]] > 0, 4);
  547:       _price > 0 ? _price : getUnderlyingPrice(),
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L265-L547

```solidity
File: reLP/ReLPContract.sol
  94:       _reLPFactor > 0,
  175:       _liquiditySlippageTolerance > 0,
  190:       _slippageTolerance > 0,
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L94-L190






### [G-05] Do not calculate constants

The implementation of constant variables (replacements at compile-time) leads to the recomputation of an expression assigned to a constant variable each time the variable is used, wasting gas. Therefore constants should not involve calculations, rather the final value. For clarity, use comments to show where that value is being derived from.

This includes `keccak256` hashes, and other numerical values.

*There are 11 instances of this issue:*

```solidity
File: core/RdpxV2Bond.sol
22:   bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Bond.sol#L22

```solidity
File: decaying-bonds/RdpxDecayingBonds.sol
31:   bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L31

```solidity
File: decaying-bonds/RdpxDecayingBonds.sol
34:   bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L34

```solidity
File: dpxETH/DpxEthToken.sol
19:   bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/dpxETH/DpxEthToken.sol#L19

```solidity
File: dpxETH/DpxEthToken.sol
20:   bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/dpxETH/DpxEthToken.sol#L20

```solidity
File: dpxETH/DpxEthToken.sol
21:   bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/dpxETH/DpxEthToken.sol#L21

```solidity
File: perp-vault/PerpetualAtlanticVault.sol
45:   bytes32 public constant MANAGER_ROLE = keccak256("MANAGER_ROLE");
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L45

```solidity
File: perp-vault/PerpetualAtlanticVault.sol
48:   bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L48

```solidity
File: reLP/ReLPContract.sol
70:   bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L70

```solidity
File: core/RdpxV2Core.sol
88:   uint256 public constant RDPX_RATIO_PERCENTAGE = 25 * DEFAULT_PRECISION;
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L88

```solidity
File: core/RdpxV2Core.sol
91:   uint256 public constant ETH_RATIO_PERCENTAGE = 75 * DEFAULT_PRECISION;
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L91



### [G-06] Hardcode address instead of using `address(this)`

Instead of address(this), it is more gas-efficient to pre-calculate and use the address with a hardcode. Foundry's script.sol and solmate ```LibRlp.sol``` contracts can do this.

References:
- https://book.getfoundry.sh/reference/forge-std/compute-create-address
- https://twitter.com/transmissions11/status/1518507047943245824

*There are 41 instances of this issue:*

```solidity
File: amo/UniV2LiquidityAmo.sol
  149:       token.safeTransfer(msg.sender, token.balanceOf(address(this)));
  162:       address(this)
  165:       address(this)
  212:       address(this),
  217:       address(this),
  230:         address(this),
  278:         address(this),
  323:       address(this),
  341:         address(this),
  363:     lpTokenBalance = IERC20WithBurn(addresses.pair).balanceOf(address(this));
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L149-L363

```solidity
File: amo/UniV3LiquidityAmo.sol
  106:       keccak256(abi.encodePacked(address(this), _tickLower, _tickUpper))
  160:       address(this),
  165:       address(this),
  189:         address(this),
  285:       address(this),
  294:         address(this),
  331:       address(this),
  354:     uint256 tokenABalance = IERC20WithBurn(tokenA).balanceOf(address(this));
  355:     uint256 tokenBBalance = IERC20WithBurn(tokenB).balanceOf(address(this));
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L106-L355

```solidity
File: core/RdpxV2Core.sol
  169:       token.safeTransfer(msg.sender, token.balanceOf(address(this)));
  483:     ).purchase(_amount, address(this));
  573:       address(this),
  654:         .safeTransferFrom(msg.sender, address(this), _rdpxAmount);
  911:       address(this),
  953:     IERC20WithBurn(weth).safeTransferFrom(msg.sender, address(this), _amount);
  998:         address(this)
  1060:       address(this),
  1102:           address(this),
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L169-L1102

```solidity
File: decaying-bonds/RdpxDecayingBonds.sol
  105:       token.safeTransfer(msg.sender, token.balanceOf(address(this)));
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L105

```solidity
File: perp-vault/PerpetualAtlanticVault.sol
  227:       token.safeTransfer(msg.sender, token.balanceOf(address(this)));
  289:     collateralToken.safeTransferFrom(msg.sender, address(this), premium);
  384:       address(this),
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L227-L384

```solidity
File: perp-vault/PerpetualAtlanticVaultLP.sol
  128:     collateral.transferFrom(msg.sender, address(this), assets);
  192:       collateral.balanceOf(address(this)) >= _totalCollateral + proceeds,
  201:       collateral.balanceOf(address(this)) == _totalCollateral - loss,
  210:       IERC20WithBurn(rdpx).balanceOf(address(this)) >= _rdpxCollateral + amount,
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L128-L210

```solidity
File: reLP/ReLPContract.sol
  245:       address(this),
  263:       address(this),
  282:         address(this),
  293:       address(this),
  304:       IERC20WithBurn(addresses.tokenA).balanceOf(address(this))
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L245-L304


### [G-07] Consider using `bytes32` rather than a `string`

Using the `bytes` types for fixed-length strings is more efficient than having the EVM have to incur the overhead of string processing. Consider whether the value _needs_ to be a `string`. A good reason to keep it as a `string` would be if the variable is defined in an interface that this project does not own.


*There are 4 instances of this issue:*

```solidity
File: core/RdpxV2Core.sol
70:   string[] public reserveTokens;
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L70

```solidity
File: perp-vault/PerpetualAtlanticVault.sol
51:   string public underlyingSymbol;
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L51

```solidity
File: perp-vault/PerpetualAtlanticVaultLP.sol
52:   string public underlyingSymbol;
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L52

```solidity
File: perp-vault/PerpetualAtlanticVaultLP.sol
55:   string public collateralSymbol;
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L55

### [G-08] `internal` functions only called once can be inlined to save gas

Not inlining costs **20 to 40 gas** because of two extra `JUMP` instructions and additional stack operations needed for function calls.

The function `beforeWithdraw()` is only called once (in the same file, line 166), so it can be inlined to save gas.

*There is 1 instance of this issue:*

```solidity
File: perp-vault/PerpetualAtlanticVaultLP.sol
286:   function beforeWithdraw(uint256 assets, uint256 /*shares*/) internal {
287:     require(
288:       assets <= totalAvailableCollateral(),
289:       "Not enough available assets to satisfy withdrawal"
290:     );
291:     _totalCollateral -= assets;
292:   }
```
https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L286-L292







