|ID|Title|Category|Severity|Instances|
|-|:-|:-:|:-:|:-:|
| [[G-1](#g-1--using-bools-for-storage-incurs-overhead)] | Using bools for storage incurs overhead | Gas Optimization | Informational | 2 |
| [[G-2](#g-2--use-assembly-to-check-for-address0)] | Use assembly to check for `address(0)` | Gas Optimization | Informational | 43 |
| [[G-3](#g-3--cache-array-length-outside-of-loop)] | Cache array length outside of loop | Gas Optimization | Informational | 9 |
| [[G-4](#g-4--functions-guaranteed-to-revert-when-called-by-normal-users-can-be-marked-payable)] | Functions guaranteed to revert when called by normal users can be marked payable | Gas Optimization | Informational | 22 |
| [[G-5](#g-5--keccak256-expressions-which-are-fixed,-can-be-precalculated-and-assigned-to-the-constant-variables)] | `keccak256()` EXPRESSIONS WHICH ARE FIXED, CAN BE PRECALCULATED AND ASSIGNED TO THE CONSTANT VARIABLES. | Gas Optimization | Informational | 9 |
| [[G-6](#g-6--long-revert-strings)] | Long revert strings | Gas Optimization | Informational | 3 |
| [[G-7](#g-7-->=-costs-less-gas-than->)] | `>=` costs less gas than `>` | Gas Optimization | Informational | 24 |
| [[G-8](#g-8--++i-costs-less-gas-than-i++,-especially-when-it's-used-in-for-loops---ii---too)] | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) | Gas Optimization | Informational | 12 |
| [[G-9](#g-9--x-+=-yx--=-y-costs-more-gas-than-x-=-x-+-yx-=-x---y-for-state-variables)] | `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables | Gas Optimization | Informational | 24 |
| [[G-10](#g-10--usage-of-uintsints-smaller-than-32-bytes-256-bits-incurs-overhead)] | Usage of `uints/ints` smaller than 32 bytes (256 bits) incurs overhead | Gas Optimization | Informational | 9 |
| [[G-11](#g-11--using-private-rather-than-public-for-constants,-saves-gas)] | Using `private` rather than `public` for constants, saves gas | Gas Optimization | Informational | 14 |
| [[G-12](#g-12--don't-initialize-variables-with-default-value)] | Don't initialize variables with default value | Gas Optimization | Informational | 10 |
| [[G-13](#g-13--inverting-the-condition-of-an-if-else-statement-wastes-gas)] | Inverting the condition of an if-else-statement wastes gas | Gas Optimization | Informational | 3 |
| [[G-14](#g-14--++ii++-should-be-unchecked{++i}unchecked{i++}-when-it-is-not-possible-for-them-to-overflow,-as-is-the-case-when-used-in-for--and-while-loops)] | `++i`/`i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as is the case when used in `for`- and `while`-loops | Gas Optimization | Informational | 12 |
| [[G-15](#g-15--gas-saving-is-achieved-by-removing-the-delete-keyword-~60k)] | Gas saving is achieved by removing the `delete` keyword (~60k) | Gas Optimization | Informational | 1 |
| [[G-16](#g-16--with-assembly,-call-bool-success-transfer-can-be-done-gas-optimized)] | With assembly, `.call (bool success)` transfer can be done gas-optimized | Gas Optimization | Informational | 2 |
| [[G-17](#g-17--use-constants-instead-of-typeuintxmax)] | Use constants instead of `type(uintx).max` | Gas Optimization | Informational | 16 |
| [[G-18](#g-18--use-!=-0-instead-of->-0-for-unsigned-integer-comparison)] | Use `!= 0` instead of `> 0` for unsigned integer comparison | Gas Optimization | Informational | 19 |
| [[G-19](#g-19--use-shift-rightleft-instead-of-divisionmultiplication-if-possible)] | Use shift Right/Left instead of division/multiplication if possible | Gas Optimization | Informational | 20 |
| [[G-20](#g-20--don't-compare-boolean-expressions-to-boolean-literals)] | Don't compare boolean expressions to boolean literals | Gas Optimization | Informational | 1 |
| [[G-21](#g-21--constructors-can-be-marked-payable)] | Constructors can be marked `payable` | Gas Optimization | Informational | 10 |
| [[G-22](#g-22--don't-use-safemath-once-the-solidity-version-is-080-or-greater)] | Don't use `SafeMath` once the solidity version is 0.8.0 or greater | Gas Optimization | Informational | 3 |
| [[G-23](#g-23--use-assembly-to-hash-instead-of-solidity)] | Use assembly to hash instead of Solidity | Gas Optimization | Informational | 3 |
| [[G-24](#g-24--move-owner-checks-to-a-modifier-for-gas-efficant)] | Move owner checks to a modifier for gas efficant | Gas Optimization | Informational | 1 |
| [[G-25](#g-25--use-bytes32-instead-of-string)] | USE BYTES32 INSTEAD OF STRING | Gas Optimization | Informational | 1 |
| [[G-26](#g-26--part-of-the-code-can-be-pre-calculated)] | Part of the code can be pre calculated | Gas Optimization | Informational | 9 |
| [[G-27](#g-27---multiple-addressid-mappings-can-be-combined-into-a-single-mapping-of-an-addressid-to-a-struct,-where-appropriate)] |  Multiple `address/ID` mappings can be combined into a single mapping of an address/ID to a struct, where appropriate | Gas Optimization | Informational | 13 |
| [[G-28](#g-28--ternary-operation-is-cheaper-than-if-else-statement)] | Ternary operation is cheaper than if-else statement | Gas Optimization | Informational | 7 |
| [[G-29](#g-29--unnecessary-use-of-_msgsender)] | Unnecessary use of `_msgSender()` | Gas Optimization | Informational | 2 |
| [[G-30](#g-30--calling-length-in-a-for-loop-wastes-gas)] | Calling `.length` in a for loop wastes gas | Gas Optimization | Informational | 9 |
| [[G-31](#g-31--unchecked-{}-can-be-used-on-the-division-of-two-uints-in-order-to-save-gas)] | `unchecked {}` can be used on the division of two `uints` in order to save gas | Gas Optimization | Informational | 14 |

## **G-1** | Using bools for storage incurs overhead

### **Description** :-

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
115:bool public
```
#### **Context**:-
[RdpxV2Core.sol#L115](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L115)
```solidity
118:bool public
```
#### **Context**:-
[RdpxV2Core.sol#L118](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L118)

## **G-2** | Use assembly to check for `address(0)`

### **Description** :-

Saves 6 gas per instance if using assembly to check for address(0)

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol
```
```solidity
84:      _tokenA != address(0)
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L84](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L84)
```solidity
File:2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol
```
```solidity
85:        _tokenB != address(0)
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L85](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L85)
```solidity
File:2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol
```
```solidity
86:        _pair != address(0)
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L86](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L86)
```solidity
File:2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol
```
```solidity
87:        _rdpxV2Core != address(0)
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L87](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L87)
```solidity
File:2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol
```
```solidity
88:        _rdpxOracle != address(0)
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L88](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L88)
```solidity
File:2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol
```
```solidity
89:        _ammFactory != address(0)
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L89](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L89)
```solidity
File:2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol
```
```solidity
90:        _ammRouter != address(0)
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L90](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L90)
```solidity
File:2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol
```
```solidity
131:    require(_token != address(0)
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L131](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L131)
```solidity
File:2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol
```
```solidity
132:    require(_spender != address(0)
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L132](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L132)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
244:    require(_asset != address(0)
```
#### **Context**:-
[RdpxV2Core.sol#L244](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L244)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
316:    _validate(_dopexAMMRouter != address(0)
```
#### **Context**:-
[RdpxV2Core.sol#L316](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L316)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
317:    _validate(_dpxEthCurvePool != address(0)
```
#### **Context**:-
[RdpxV2Core.sol#L317](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L317)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
318:    _validate(_rdpxDecayingBonds != address(0)
```
#### **Context**:-
[RdpxV2Core.sol#L318](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L318)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
319:    _validate(_perpetualAtlanticVault != address(0)
```
#### **Context**:-
[RdpxV2Core.sol#L319](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L319)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
320:    _validate(_perpetualAtlanticVaultLP != address(0)
```
#### **Context**:-
[RdpxV2Core.sol#L320](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L320)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
321:    _validate(_rdpxReserve != address(0)
```
#### **Context**:-
[RdpxV2Core.sol#L321](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L321)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
322:    _validate(_rdpxV2ReceiptToken != address(0)
```
#### **Context**:-
[RdpxV2Core.sol#L322](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L322)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
323:    _validate(_feeDistributor != address(0)
```
#### **Context**:-
[RdpxV2Core.sol#L323](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L323)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
324:    _validate(_reLPContract != address(0)
```
#### **Context**:-
[RdpxV2Core.sol#L324](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L324)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
325:    _validate(_receiptTokenBonds != address(0)
```
#### **Context**:-
[RdpxV2Core.sol#L325](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L325)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
362:    _validate(_rdpxPriceOracle != address(0)
```
#### **Context**:-
[RdpxV2Core.sol#L362](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L362)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
363:    _validate(_dpxEthPriceOracle != address(0)
```
#### **Context**:-
[RdpxV2Core.sol#L363](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L363)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
379:    _validate(_addr != address(0)
```
#### **Context**:-
[RdpxV2Core.sol#L379](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L379)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
408:    _validate(_token != address(0)
```
#### **Context**:-
[RdpxV2Core.sol#L408](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L408)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
409:    _validate(_spender != address(0)
```
#### **Context**:-
[RdpxV2Core.sol#L409](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L409)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
119:    _validate(_collateralToken != address(0)
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L119](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L119)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
190:    _validate(_optionPricing != address(0)
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L190](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L190)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
191:    _validate(_assetPriceOracle != address(0)
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L191](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L191)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
192:    _validate(_volatilityOracle != address(0)
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L192](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L192)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
193:    _validate(_feeDistributor != address(0)
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L193](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L193)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
194:    _validate(_rdpx != address(0)
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L194](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L194)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
195:    _validate(_perpetualAtlanticVaultLP != address(0)
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L195](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L195)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
196:    _validate(_rdpxV2Core != address(0)
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L196](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L196)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
```solidity
95:      _perpetualAtlanticVault != address(0) || _rdpx != address(0)
```
#### **Context**:-
[PerpetualAtlanticVaultLP.sol#L95](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L95)
```solidity
File:2023-08-dopex/contracts/reLP/ReLPContract.sol
```
```solidity
127:      _tokenA != address(0)
```
#### **Context**:-
[ReLPContract.sol#L127](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L127)
```solidity
File:2023-08-dopex/contracts/reLP/ReLPContract.sol
```
```solidity
128:        _tokenB != address(0)
```
#### **Context**:-
[ReLPContract.sol#L128](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L128)
```solidity
File:2023-08-dopex/contracts/reLP/ReLPContract.sol
```
```solidity
129:        _pair != address(0)
```
#### **Context**:-
[ReLPContract.sol#L129](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L129)
```solidity
File:2023-08-dopex/contracts/reLP/ReLPContract.sol
```
```solidity
130:        _rdpxV2Core != address(0)
```
#### **Context**:-
[ReLPContract.sol#L130](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L130)
```solidity
File:2023-08-dopex/contracts/reLP/ReLPContract.sol
```
```solidity
131:        _tokenAReserve != address(0)
```
#### **Context**:-
[ReLPContract.sol#L131](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L131)
```solidity
File:2023-08-dopex/contracts/reLP/ReLPContract.sol
```
```solidity
132:        _amo != address(0)
```
#### **Context**:-
[ReLPContract.sol#L132](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L132)
```solidity
File:2023-08-dopex/contracts/reLP/ReLPContract.sol
```
```solidity
133:        _rdpxOracle != address(0)
```
#### **Context**:-
[ReLPContract.sol#L133](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L133)
```solidity
File:2023-08-dopex/contracts/reLP/ReLPContract.sol
```
```solidity
134:        _ammFactory != address(0)
```
#### **Context**:-
[ReLPContract.sol#L134](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L134)
```solidity
File:2023-08-dopex/contracts/reLP/ReLPContract.sol
```
```solidity
135:        _ammRouter != address(0)
```
#### **Context**:-
[ReLPContract.sol#L135](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L135)

## **G-3** | Cache array length outside of loop

### **Description** :-

If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol
```
```solidity
147:for (uint256 i = 0; i < tokens.length; i++) {
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L147](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L147)
```solidity
File:2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol
```
```solidity
120:for (uint i = 0; i < positions_array.length; i++) {
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L120](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L120)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
167:for (uint256 i = 0; i < tokens.length; i++) {
```
#### **Context**:-
[RdpxV2Core.sol#L167](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L167)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
246:for (uint256 i = 1; i < reserveAsset.length; i++) {
```
#### **Context**:-
[RdpxV2Core.sol#L246](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L246)
```solidity
996:for (uint256 i = 1; i < reserveAsset.length; i++) {
```
#### **Context**:-
[RdpxV2Core.sol#L996](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L996)
```solidity
File:2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol
```
```solidity
103:for (uint256 i = 0; i < tokens.length; i++) {
```
#### **Context**:-
[RdpxDecayingBonds.sol#L103](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L103)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
225:for (uint256 i = 0; i < tokens.length; i++) {
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L225](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L225)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
328:for (uint256 i = 0; i < optionIds.length; i++) {
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L328](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L328)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
413:for (uint256 i = 0; i < strikes.length; i++) {
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L413](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L413)

## **G-4** | Functions guaranteed to revert when called by normal users can be marked payable

### **Description** :-

If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol
```
```solidity
119:function collectFees() external onlyRole(DEFAULT_ADMIN_ROLE) {
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L119](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L119)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
144:function pause() external onlyRole(DEFAULT_ADMIN_ROLE) {
```
#### **Context**:-
[RdpxV2Core.sol#L144](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L144)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
152:function unpause() external onlyRole(DEFAULT_ADMIN_ROLE) {
```
#### **Context**:-
[RdpxV2Core.sol#L152](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L152)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
206:function setIsreLP(bool _isReLPActive) external onlyRole(DEFAULT_ADMIN_ROLE) {
```
#### **Context**:-
[RdpxV2Core.sol#L206](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L206)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
378:function addAMOAddress(address _addr) external onlyRole(DEFAULT_ADMIN_ROLE) {
```
#### **Context**:-
[RdpxV2Core.sol#L378](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L378)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Bond.sol
```
```solidity
29:function pause() public onlyRole(DEFAULT_ADMIN_ROLE) {
```
#### **Context**:-
[RdpxV2Bond.sol#L29](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Bond.sol#L29)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Bond.sol
```
```solidity
33:function unpause() public onlyRole(DEFAULT_ADMIN_ROLE) {
```
#### **Context**:-
[RdpxV2Bond.sol#L33](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Bond.sol#L33)
```solidity
File:2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol
```
```solidity
70:function pause() external onlyRole(DEFAULT_ADMIN_ROLE) {
```
#### **Context**:-
[RdpxDecayingBonds.sol#L70](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L70)
```solidity
File:2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol
```
```solidity
76:function unpause() external onlyRole(DEFAULT_ADMIN_ROLE) {
```
#### **Context**:-
[RdpxDecayingBonds.sol#L76](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L76)
```solidity
File:2023-08-dopex/contracts/dpxETH/DpxEthToken.sol
```
```solidity
29:function pause() public onlyRole(PAUSER_ROLE) {
```
#### **Context**:-
[DpxEthToken.sol#L29](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/dpxETH/DpxEthToken.sol#L29)
```solidity
File:2023-08-dopex/contracts/dpxETH/DpxEthToken.sol
```
```solidity
33:function unpause() public onlyRole(PAUSER_ROLE) {
```
#### **Context**:-
[DpxEthToken.sol#L33](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/dpxETH/DpxEthToken.sol#L33)
```solidity
File:2023-08-dopex/contracts/dpxETH/DpxEthToken.sol
```
```solidity
37:function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {
```
#### **Context**:-
[DpxEthToken.sol#L37](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/dpxETH/DpxEthToken.sol#L37)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
136:function pause() external onlyRole(DEFAULT_ADMIN_ROLE) {
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L136](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L136)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
144:function unpause() external onlyRole(DEFAULT_ADMIN_ROLE) {
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L144](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L144)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
243:function setLpAllowance(bool increase) external onlyRole(DEFAULT_ADMIN_ROLE) {
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L243](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L243)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
372:function payFunding() external onlyRole(RDPXV2CORE_ROLE) returns (uint256) {
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L372](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L372)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
```solidity
180:function lockCollateral(uint256 amount) public onlyPerpVault {
```
#### **Context**:-
[PerpetualAtlanticVaultLP.sol#L180](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L180)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
```solidity
185:function unlockLiquidity(uint256 amount) public onlyPerpVault {
```
#### **Context**:-
[PerpetualAtlanticVaultLP.sol#L185](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L185)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
```solidity
190:function addProceeds(uint256 proceeds) public onlyPerpVault {
```
#### **Context**:-
[PerpetualAtlanticVaultLP.sol#L190](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L190)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
```solidity
199:function subtractLoss(uint256 loss) public onlyPerpVault {
```
#### **Context**:-
[PerpetualAtlanticVaultLP.sol#L199](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L199)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
```solidity
208:function addRdpx(uint256 amount) public onlyPerpVault {
```
#### **Context**:-
[PerpetualAtlanticVaultLP.sol#L208](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L208)
```solidity
File:2023-08-dopex/contracts/reLP/ReLPContract.sol
```
```solidity
202:function reLP(uint256 _amount) external onlyRole(RDPXV2CORE_ROLE) {
```
#### **Context**:-
[ReLPContract.sol#L202](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L202)

## **G-5** | `keccak256()` EXPRESSIONS WHICH ARE FIXED, CAN BE PRECALCULATED AND ASSIGNED TO THE CONSTANT VARIABLES.

### **Description** :-

Instead of calculating the keccak256() when the contract is made, pre calculate them before and only give the result to a constant

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/core/RdpxV2Bond.sol
```
```solidity
22:constant MINTER_ROLE = keccak256("MINTER_ROLE");
```
#### **Context**:-
[RdpxV2Bond.sol#L22](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Bond.sol#L22)
```solidity
File:2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol
```
```solidity
31:constant MINTER_ROLE = keccak256("MINTER_ROLE");
```
#### **Context**:-
[RdpxDecayingBonds.sol#L31](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L31)
```solidity
File:2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol
```
```solidity
34:constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
```
#### **Context**:-
[RdpxDecayingBonds.sol#L34](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L34)
```solidity
File:2023-08-dopex/contracts/dpxETH/DpxEthToken.sol
```
```solidity
19:constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
```
#### **Context**:-
[DpxEthToken.sol#L19](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/dpxETH/DpxEthToken.sol#L19)
```solidity
File:2023-08-dopex/contracts/dpxETH/DpxEthToken.sol
```
```solidity
20:constant MINTER_ROLE = keccak256("MINTER_ROLE");
```
#### **Context**:-
[DpxEthToken.sol#L20](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/dpxETH/DpxEthToken.sol#L20)
```solidity
File:2023-08-dopex/contracts/dpxETH/DpxEthToken.sol
```
```solidity
21:constant BURNER_ROLE = keccak256("BURNER_ROLE");
```
#### **Context**:-
[DpxEthToken.sol#L21](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/dpxETH/DpxEthToken.sol#L21)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
45:constant MANAGER_ROLE = keccak256("MANAGER_ROLE");
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L45](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L45)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
48:constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L48](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L48)
```solidity
File:2023-08-dopex/contracts/reLP/ReLPContract.sol
```
```solidity
70:constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
```
#### **Context**:-
[ReLPContract.sol#L70](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L70)

## **G-6** | Long revert strings

### **Description** :-

Revert strings that are longer than 32 bytes require at least one additional mstore, along with additional overhead for computing memory offset, etc.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol
```
```solidity
132:require(_spender != address(0), "reLPContract: spender cannot be 0")
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L132](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L132)
```solidity
File:2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol
```
```solidity
133:require(_amount > 0, "reLPContract: amount must be greater than 0")
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L133](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L133)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
244:require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address")
```
#### **Context**:-
[RdpxV2Core.sol#L244](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L244)

## **G-7** | `>=` costs less gas than `>`

### **Description** :-

The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol
```
```solidity
113: > 0
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L113](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L113)
```solidity
133: > 0
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L133](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L133)
```solidity
File:2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol
```
```solidity
120: < p
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L120](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L120)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
167: < t
```
#### **Context**:-
[RdpxV2Core.sol#L167](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L167)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
183: > 0
```
#### **Context**:-
[RdpxV2Core.sol#L183](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L183)
```solidity
196: > 0
```
#### **Context**:-
[RdpxV2Core.sol#L196](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L196)
```solidity
231: > 0
```
#### **Context**:-
[RdpxV2Core.sol#L231](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L231)
```solidity
410: > 0
```
#### **Context**:-
[RdpxV2Core.sol#L410](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L410)
```solidity
444: > 0
```
#### **Context**:-
[RdpxV2Core.sol#L444](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L444)
```solidity
458: > 0
```
#### **Context**:-
[RdpxV2Core.sol#L458](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L458)
```solidity
556: > 0
```
#### **Context**:-
[RdpxV2Core.sol#L556](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L556)
```solidity
838: > 0
```
#### **Context**:-
[RdpxV2Core.sol#L838](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L838)
```solidity
901: > 0
```
#### **Context**:-
[RdpxV2Core.sol#L901](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L901)
```solidity
984: > 0
```
#### **Context**:-
[RdpxV2Core.sol#L984](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L984)
```solidity
1021: > 0
```
#### **Context**:-
[RdpxV2Core.sol#L1021](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L1021)
```solidity
File:2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol
```
```solidity
103: < t
```
#### **Context**:-
[RdpxDecayingBonds.sol#L103](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L103)
```solidity
File:2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol
```
```solidity
156: < o
```
#### **Context**:-
[RdpxDecayingBonds.sol#L156](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L156)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
225: < t
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L225](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L225)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
265: > 0
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L265](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L265)
```solidity
414: > 0
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L414](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L414)
```solidity
547: > 0
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L547](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L547)
```solidity
File:2023-08-dopex/contracts/reLP/ReLPContract.sol
```
```solidity
94: > 0
```
#### **Context**:-
[ReLPContract.sol#L94](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L94)
```solidity
175: > 0
```
#### **Context**:-
[ReLPContract.sol#L175](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L175)
```solidity
190: > 0
```
#### **Context**:-
[ReLPContract.sol#L190](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L190)

## **G-8** | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too)

### **Description** :-

*Saves 5 gas per loop*

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol
```
```solidity
147:i++
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L147](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L147)
```solidity
File:2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol
```
```solidity
120:i++
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L120](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L120)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
167:i++
```
#### **Context**:-
[RdpxV2Core.sol#L167](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L167)
```solidity
246:i++
```
#### **Context**:-
[RdpxV2Core.sol#L246](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L246)
```solidity
775:i++
```
#### **Context**:-
[RdpxV2Core.sol#L775](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L775)
```solidity
836:i++
```
#### **Context**:-
[RdpxV2Core.sol#L836](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L836)
```solidity
996:i++
```
#### **Context**:-
[RdpxV2Core.sol#L996](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L996)
```solidity
File:2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol
```
```solidity
103:i++
```
#### **Context**:-
[RdpxDecayingBonds.sol#L103](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L103)
```solidity
156:i++
```
#### **Context**:-
[RdpxDecayingBonds.sol#L156](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L156)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
225:i++
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L225](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L225)
```solidity
328:i++
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L328](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L328)
```solidity
413:i++
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L413](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L413)

## **G-9** | `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables

### **Description** :-

Using the addition operator instead of plus-equals saves 113 gas

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol
```
```solidity
235:+=
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L235](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L235)
```solidity
File:2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol
```
```solidity
283:-=
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L283](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L283)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
486:-=
```
#### **Context**:-
[RdpxV2Core.sol#L486](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L486)
```solidity
570:-=
```
#### **Context**:-
[RdpxV2Core.sol#L570](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L570)
```solidity
720:-=
```
#### **Context**:-
[RdpxV2Core.sol#L720](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L720)
```solidity
780:-=
```
#### **Context**:-
[RdpxV2Core.sol#L780](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L780)
```solidity
803:-=
```
#### **Context**:-
[RdpxV2Core.sol#L803](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L803)
```solidity
1106:-=
```
#### **Context**:-
[RdpxV2Core.sol#L1106](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L1106)
```solidity
1110:-=
```
#### **Context**:-
[RdpxV2Core.sol#L1110](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L1110)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
302:+=
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L302](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L302)
```solidity
303:+=
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L303](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L303)
```solidity
304:+=
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L304](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L304)
```solidity
309:+=
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L309](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L309)
```solidity
335:+=
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L335](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L335)
```solidity
336:+=
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L336](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L336)
```solidity
437:+=
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L437](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L437)
```solidity
440:+=
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L440](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L440)
```solidity
445:+=
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L445](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L445)
```solidity
449:+=
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L449](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L449)
```solidity
493:+=
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L493](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L493)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
```solidity
132:+=
```
#### **Context**:-
[PerpetualAtlanticVaultLP.sol#L132](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L132)
```solidity
181:+=
```
#### **Context**:-
[PerpetualAtlanticVaultLP.sol#L181](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L181)
```solidity
195:+=
```
#### **Context**:-
[PerpetualAtlanticVaultLP.sol#L195](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L195)
```solidity
213:+=
```
#### **Context**:-
[PerpetualAtlanticVaultLP.sol#L213](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L213)

## **G-10** | Usage of `uints/ints` smaller than 32 bytes (256 bits) incurs overhead

### **Description** :-

When using elements that are smaller than 32 bytes, your contracts gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol
```
```solidity
43:uint128
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L43](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L43)
```solidity
99:uint128
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L99](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L99)
```solidity
105:uint128
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L105](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L105)
```solidity
126:uint128
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L126](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L126)
```solidity
127:uint128
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L127](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L127)
```solidity
193:uint128
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L193](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L193)
```solidity
223:uint128
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L223](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L223)
```solidity
224:uint128
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L224](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L224)
```solidity
235:uint128
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L235](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L235)

## **G-11** | Using `private` rather than `public` for constants, saves gas

### **Description** :-

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol
```
```solidity
48:public constant DEFAULT_PRECISION =
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L48](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L48)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
85:public constant DEFAULT_PRECISION =
```
#### **Context**:-
[RdpxV2Core.sol#L85](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L85)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
88:public constant RDPX_RATIO_PERCENTAGE =
```
#### **Context**:-
[RdpxV2Core.sol#L88](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L88)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
91:public constant ETH_RATIO_PERCENTAGE =
```
#### **Context**:-
[RdpxV2Core.sol#L91](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L91)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Bond.sol
```
```solidity
22:public constant MINTER_ROLE =
```
#### **Context**:-
[RdpxV2Bond.sol#L22](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Bond.sol#L22)
```solidity
File:2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol
```
```solidity
31:public constant MINTER_ROLE =
```
#### **Context**:-
[RdpxDecayingBonds.sol#L31](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L31)
```solidity
File:2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol
```
```solidity
34:public constant RDPXV2CORE_ROLE =
```
#### **Context**:-
[RdpxDecayingBonds.sol#L34](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L34)
```solidity
File:2023-08-dopex/contracts/dpxETH/DpxEthToken.sol
```
```solidity
19:public constant PAUSER_ROLE =
```
#### **Context**:-
[DpxEthToken.sol#L19](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/dpxETH/DpxEthToken.sol#L19)
```solidity
File:2023-08-dopex/contracts/dpxETH/DpxEthToken.sol
```
```solidity
20:public constant MINTER_ROLE =
```
#### **Context**:-
[DpxEthToken.sol#L20](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/dpxETH/DpxEthToken.sol#L20)
```solidity
File:2023-08-dopex/contracts/dpxETH/DpxEthToken.sol
```
```solidity
21:public constant BURNER_ROLE =
```
#### **Context**:-
[DpxEthToken.sol#L21](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/dpxETH/DpxEthToken.sol#L21)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
45:public constant MANAGER_ROLE =
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L45](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L45)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
48:public constant RDPXV2CORE_ROLE =
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L48](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L48)
```solidity
File:2023-08-dopex/contracts/reLP/ReLPContract.sol
```
```solidity
67:public constant DEFAULT_PRECISION =
```
#### **Context**:-
[ReLPContract.sol#L67](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L67)
```solidity
File:2023-08-dopex/contracts/reLP/ReLPContract.sol
```
```solidity
70:public constant RDPXV2CORE_ROLE =
```
#### **Context**:-
[ReLPContract.sol#L70](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L70)

## **G-12** | Don't initialize variables with default value

### **Description** :-

If a variable is not initialized, it is assumed to have the default value. Explicitly initializing a variable with its default value costs unnecessary gas. For more info, see [Mudit Gupta's Blog](https://mudit.blog/solidity-tips-and-tricks-to-save-gas-and-reduce-bytecode-size/) at point `No need to initialize variables with default values` .

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol
```
```solidity
147:uint256 i = 0;
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L147](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L147)
```solidity
File:2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol
```
```solidity
120:uint i = 0;
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L120](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L120)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
167:uint256 i = 0;
```
#### **Context**:-
[RdpxV2Core.sol#L167](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L167)
```solidity
775:uint256 i = 0;
```
#### **Context**:-
[RdpxV2Core.sol#L775](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L775)
```solidity
836:uint256 i = 0;
```
#### **Context**:-
[RdpxV2Core.sol#L836](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L836)
```solidity
File:2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol
```
```solidity
103:uint256 i = 0;
```
#### **Context**:-
[RdpxDecayingBonds.sol#L103](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L103)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
89:uint256 public latestFundingPaymentPointer = 0;
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L89](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L89)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
225:uint256 i = 0;
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L225](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L225)
```solidity
328:uint256 i = 0;
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L328](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L328)
```solidity
413:uint256 i = 0;
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L413](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L413)

## **G-13** | Inverting the condition of an if-else-statement wastes gas

### **Description** :-

Flipping the `true` and `false` blocks instead saves 3 gas

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol
```
```solidity
199:== address(rdpx) ?
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L199](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L199)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
525:= coin0 == weth ?
```
#### **Context**:-
[RdpxV2Core.sol#L525](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L525)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
```solidity
283:== 0 ?
```
#### **Context**:-
[PerpetualAtlanticVaultLP.sol#L283](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L283)

## **G-14** | `++i`/`i++` should be `unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as is the case when used in `for`- and `while`-loops

### **Description** :-

The `unchecked` keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves **30-40 gas [per loop](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked)**

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol
```
```solidity
147:i++
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L147](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L147)
```solidity
File:2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol
```
```solidity
120:i++
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L120](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L120)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
167:i++
```
#### **Context**:-
[RdpxV2Core.sol#L167](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L167)
```solidity
246:i++
```
#### **Context**:-
[RdpxV2Core.sol#L246](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L246)
```solidity
775:i++
```
#### **Context**:-
[RdpxV2Core.sol#L775](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L775)
```solidity
836:i++
```
#### **Context**:-
[RdpxV2Core.sol#L836](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L836)
```solidity
996:i++
```
#### **Context**:-
[RdpxV2Core.sol#L996](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L996)
```solidity
File:2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol
```
```solidity
103:i++
```
#### **Context**:-
[RdpxDecayingBonds.sol#L103](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L103)
```solidity
156:i++
```
#### **Context**:-
[RdpxDecayingBonds.sol#L156](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L156)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
225:i++
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L225](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L225)
```solidity
328:i++
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L328](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L328)
```solidity
413:i++
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L413](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L413)

## **G-15** | Gas saving is achieved by removing the `delete` keyword (~60k)

### **Description** :-

30k gas savings were made by removing the `delete` keyword. The reason for using the `delete` keyword here is to reset the struct values (set to default value) in every operation. However, the struct values do not need to be zero each time the function is run. Therefore, the `delete` key word is unnecessary. If it is removed, around 30k gas savings will be achieved.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol
```
```solidity
263: delete positions_mapping[pos.token_id];
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L263](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L263)

## **G-16** | With assembly, `.call (bool success)` transfer can be done gas-optimized

### **Description** :-

`return` data `(bool success,)` has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol
```
```solidity
344:= _to.call{ value: _value }(_data);
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L344](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L344)
```solidity
File:2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol
```
```solidity
98:= to.call{ value: amount, gas: gas }("");
```
#### **Context**:-
[RdpxDecayingBonds.sol#L98](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L98)

## **G-17** | Use constants instead of `type(uintx).max`

### **Description** :-

it uses more gas in the distribution process and also for each transaction than constant usage.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol
```
```solidity
126:type(uint128).max
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L126](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L126)
```solidity
127:type(uint128).max
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L127](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L127)
```solidity
223:type(uint128).max
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L223](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L223)
```solidity
224:type(uint128).max
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L224](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L224)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
341:type(uint256).max
```
#### **Context**:-
[RdpxV2Core.sol#L341](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L341)
```solidity
343:type(uint256).max
```
#### **Context**:-
[RdpxV2Core.sol#L343](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L343)
```solidity
344:type(uint256).max
```
#### **Context**:-
[RdpxV2Core.sol#L344](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L344)
```solidity
347:type(uint256).max
```
#### **Context**:-
[RdpxV2Core.sol#L347](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L347)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
209:type(uint256).max
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L209](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L209)
```solidity
247:type(uint256).max
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L247](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L247)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
```solidity
106:type(uint256).max
```
#### **Context**:-
[PerpetualAtlanticVaultLP.sol#L106](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L106)
```solidity
107:type(uint256).max
```
#### **Context**:-
[PerpetualAtlanticVaultLP.sol#L107](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L107)
```solidity
155:type(uint256).max
```
#### **Context**:-
[PerpetualAtlanticVaultLP.sol#L155](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L155)
```solidity
File:2023-08-dopex/contracts/reLP/ReLPContract.sol
```
```solidity
152:type(uint256).max
```
#### **Context**:-
[ReLPContract.sol#L152](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L152)
```solidity
157:type(uint256).max
```
#### **Context**:-
[ReLPContract.sol#L157](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L157)
```solidity
162:type(uint256).max
```
#### **Context**:-
[ReLPContract.sol#L162](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L162)

## **G-18** | Use `!= 0` instead of `> 0` for unsigned integer comparison

### **Description** :-

It is cheaper to use `!= 0` than > `0` for uint256.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol
```
```solidity
113:> 0
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L113](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L113)
```solidity
133:> 0
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L133](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L133)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
183:> 0
```
#### **Context**:-
[RdpxV2Core.sol#L183](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L183)
```solidity
196:> 0
```
#### **Context**:-
[RdpxV2Core.sol#L196](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L196)
```solidity
231:> 0
```
#### **Context**:-
[RdpxV2Core.sol#L231](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L231)
```solidity
410:> 0
```
#### **Context**:-
[RdpxV2Core.sol#L410](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L410)
```solidity
444:> 0
```
#### **Context**:-
[RdpxV2Core.sol#L444](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L444)
```solidity
458:> 0
```
#### **Context**:-
[RdpxV2Core.sol#L458](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L458)
```solidity
556:> 0
```
#### **Context**:-
[RdpxV2Core.sol#L556](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L556)
```solidity
838:> 0
```
#### **Context**:-
[RdpxV2Core.sol#L838](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L838)
```solidity
901:> 0
```
#### **Context**:-
[RdpxV2Core.sol#L901](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L901)
```solidity
984:> 0
```
#### **Context**:-
[RdpxV2Core.sol#L984](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L984)
```solidity
1021:> 0
```
#### **Context**:-
[RdpxV2Core.sol#L1021](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L1021)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
265:> 0
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L265](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L265)
```solidity
414:> 0
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L414](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L414)
```solidity
547:> 0
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L547](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L547)
```solidity
File:2023-08-dopex/contracts/reLP/ReLPContract.sol
```
```solidity
94:> 0
```
#### **Context**:-
[ReLPContract.sol#L94](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L94)
```solidity
175:> 0
```
#### **Context**:-
[ReLPContract.sol#L175](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L175)
```solidity
190:> 0
```
#### **Context**:-
[ReLPContract.sol#L190](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L190)

## **G-19** | Use shift Right/Left instead of division/multiplication if possible

### **Description** :-

A division/multiplication by any number x being a power of 2 can be calculated by shifting log2(x) to the right/left. While the DIV opcode uses 5 gas, the SHR opcode only uses 3 gas. Furthermore, Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol
```
```solidity
373: / 1
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L373](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L373)
```solidity
File:2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol
```
```solidity
310: / G
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L310](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L310)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
535: / 2
```
#### **Context**:-
[RdpxV2Core.sol#L535](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L535)
```solidity
539: / 2
```
#### **Context**:-
[RdpxV2Core.sol#L539](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L539)
```solidity
570: / 2
```
#### **Context**:-
[RdpxV2Core.sol#L570](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L570)
```solidity
574: / 2
```
#### **Context**:-
[RdpxV2Core.sol#L574](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L574)
```solidity
579: / 2
```
#### **Context**:-
[RdpxV2Core.sol#L579](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L579)
```solidity
1170: / 2
```
#### **Context**:-
[RdpxV2Core.sol#L1170](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L1170)
```solidity
1176: / 2
```
#### **Context**:-
[RdpxV2Core.sol#L1176](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L1176)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
270: / 4
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L270](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L270)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
276: / 1
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L276](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L276)
```solidity
335: / 1
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L335](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L335)
```solidity
512: / 1
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L512](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L512)
```solidity
516: / 1
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L516](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L516)
```solidity
521: / 1
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L521](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L521)
```solidity
550: / 1
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L550](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L550)
```solidity
559: / 1
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L559](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L559)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
```solidity
281: / 1
```
#### **Context**:-
[PerpetualAtlanticVaultLP.sol#L281](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L281)
```solidity
File:2023-08-dopex/contracts/reLP/ReLPContract.sol
```
```solidity
230: / (
```
#### **Context**:-
[ReLPContract.sol#L230](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L230)
```solidity
235: / (
```
#### **Context**:-
[ReLPContract.sol#L235](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L235)

## **G-20** | Don't compare boolean expressions to boolean literals

### **Description** :-

if (<x> == true) => if (<x>), if (<x> == false) => if (!<x>).

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
798:== false
```
#### **Context**:-
[RdpxV2Core.sol#L798](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L798)

## **G-21** | Constructors can be marked `payable`

### **Description** :-

Payable functions cost less gas to execute, since the compiler does not have to add extra checks to ensure that a payment wasn't provided. A constructor can safely be marked as payable, since only the deployer would be able to pass funds, and the project itself would not pass any funds.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol
```
```solidity
57:constructor
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L57](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L57)
```solidity
File:2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol
```
```solidity
76:constructor
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L76](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L76)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
124:constructor
```
#### **Context**:-
[RdpxV2Core.sol#L124](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L124)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Bond.sol
```
```solidity
24:constructor
```
#### **Context**:-
[RdpxV2Bond.sol#L24](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Bond.sol#L24)
```solidity
File:2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol
```
```solidity
56:constructor
```
#### **Context**:-
[RdpxDecayingBonds.sol#L56](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L56)
```solidity
File:2023-08-dopex/contracts/dpxETH/DpxEthToken.sol
```
```solidity
23:constructor
```
#### **Context**:-
[DpxEthToken.sol#L23](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/dpxETH/DpxEthToken.sol#L23)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
108:constructor
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L108](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L108)
```solidity
113:constructor
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L113](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L113)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
```solidity
81:constructor
```
#### **Context**:-
[PerpetualAtlanticVaultLP.sol#L81](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L81)
```solidity
File:2023-08-dopex/contracts/reLP/ReLPContract.sol
```
```solidity
79:constructor
```
#### **Context**:-
[ReLPContract.sol#L79](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L79)

## **G-22** | Don't use `SafeMath` once the solidity version is 0.8.0 or greater

### **Description** :-

Version 0.8.0 introduces internal overflow checks, so using SafeMath is redundant and adds overhead

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol
```
```solidity
6:SafeMath.sol
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L6](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L6)
```solidity
File:2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol
```
```solidity
7:SafeMath.sol
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L7](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L7)
```solidity
File:2023-08-dopex/contracts/reLP/ReLPContract.sol
```
```solidity
6:SafeMath.sol
```
#### **Context**:-
[ReLPContract.sol#L6](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L6)

## **G-23** | Use assembly to hash instead of Solidity

### **Description** :-

hash can be done by use of assembly instead of keccak256(abi.encodePacked()) [see](https://github.com/code-423n4/2023-01-astaria-findings/blob/8059d5e7c4b3b7155259c7990226c91dc2f375fb/data/kaden-G.md#use-assembly-to-hash-instead-of-solidity)

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol
```
```solidity
106:keccak256(abi.encodePacked(a
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L106](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L106)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
1141:keccak256(abi.encodePacked(a
```
#### **Context**:-
[RdpxV2Core.sol#L1141](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L1141)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
1142:keccak256(abi.encodePacked(_
```
#### **Context**:-
[RdpxV2Core.sol#L1142](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L1142)

## **G-24** | Move owner checks to a modifier for gas efficant

### **Description** :-

It's better to use a modifier for simple owner checks for an easier inspection of functions. This is also more gas efficient as it does not control with external call.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
```solidity
152:if (msg.sender != owner)
```
#### **Context**:-
[PerpetualAtlanticVaultLP.sol#L152](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L152)

## **G-25** | USE BYTES32 INSTEAD OF STRING

### **Description** :-

Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
73:(string => uint256)
```
#### **Context**:-
[RdpxV2Core.sol#L73](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L73)

## **G-26** | Part of the code can be pre calculated

### **Description** :-

If You know what data to hash, there is no need to consume more computational power to hash it using keccak256 , you’ll end up consuming 2x amount of gas.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/core/RdpxV2Bond.sol
```
```solidity
22:keccak256("MINTER_ROLE");
```
#### **Context**:-
[RdpxV2Bond.sol#L22](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Bond.sol#L22)
```solidity
File:2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol
```
```solidity
31:keccak256("MINTER_ROLE");
```
#### **Context**:-
[RdpxDecayingBonds.sol#L31](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L31)
```solidity
File:2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol
```
```solidity
34:keccak256("RDPXV2CORE_ROLE");
```
#### **Context**:-
[RdpxDecayingBonds.sol#L34](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L34)
```solidity
File:2023-08-dopex/contracts/dpxETH/DpxEthToken.sol
```
```solidity
19:keccak256("PAUSER_ROLE");
```
#### **Context**:-
[DpxEthToken.sol#L19](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/dpxETH/DpxEthToken.sol#L19)
```solidity
File:2023-08-dopex/contracts/dpxETH/DpxEthToken.sol
```
```solidity
20:keccak256("MINTER_ROLE");
```
#### **Context**:-
[DpxEthToken.sol#L20](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/dpxETH/DpxEthToken.sol#L20)
```solidity
File:2023-08-dopex/contracts/dpxETH/DpxEthToken.sol
```
```solidity
21:keccak256("BURNER_ROLE");
```
#### **Context**:-
[DpxEthToken.sol#L21](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/dpxETH/DpxEthToken.sol#L21)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
45:keccak256("MANAGER_ROLE");
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L45](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L45)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
48:keccak256("RDPXV2CORE_ROLE");
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L48](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L48)
```solidity
File:2023-08-dopex/contracts/reLP/ReLPContract.sol
```
```solidity
70:keccak256("RDPXV2CORE_ROLE");
```
#### **Context**:-
[ReLPContract.sol#L70](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L70)

## **G-27** |  Multiple `address/ID` mappings can be combined into a single mapping of an address/ID to a struct, where appropriate

### **Description** :-

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol
```
```solidity
66:mapping(
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L66](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L66)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
73:mapping(
```
#### **Context**:-
[RdpxV2Core.sol#L73](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L73)
```solidity
76:mapping(
```
#### **Context**:-
[RdpxV2Core.sol#L76](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L76)
```solidity
79:mapping(
```
#### **Context**:-
[RdpxV2Core.sol#L79](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L79)
```solidity
82:mapping(
```
#### **Context**:-
[RdpxV2Core.sol#L82](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L82)
```solidity
File:2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol
```
```solidity
37:mapping(
```
#### **Context**:-
[RdpxDecayingBonds.sol#L37](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L37)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
63:mapping(
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L63](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L63)
```solidity
66:mapping(
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L66](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L66)
```solidity
69:mapping(
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L69](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L69)
```solidity
73:mapping(
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L73](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L73)
```solidity
76:mapping(
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L76](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L76)
```solidity
79:mapping(
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L79](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L79)
```solidity
82:mapping(
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L82](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L82)

## **G-28** | Ternary operation is cheaper than if-else statement

### **Description** :-

There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol
```
```solidity
316: else {
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L316](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L316)
```solidity
File:2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol
```
```solidity
149: else {
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L149](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L149)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
651: else {
```
#### **Context**:-
[RdpxV2Core.sol#L651](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L651)
```solidity
1178: else {
```
#### **Context**:-
[RdpxV2Core.sol#L1178](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L1178)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
580: else {
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L580](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L580)
```solidity
599: else {
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L599](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L599)
```solidity
606: else {
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L606](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L606)

## **G-29** | Unnecessary use of `_msgSender()`

### **Description** :-

The use of `_msgSender()` when there is no implementation of a meta transaction mechanism that uses it, such as EIP-2771, very slightly increases gas consumption.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/dpxETH/DpxEthToken.sol
```
```solidity
44:_msgSender(), _amount);
```
#### **Context**:-
[DpxEthToken.sol#L44](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/dpxETH/DpxEthToken.sol#L44)
```solidity
File:2023-08-dopex/contracts/dpxETH/DpxEthToken.sol
```
```solidity
51:_msgSender(), amount);
```
#### **Context**:-
[DpxEthToken.sol#L51](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/dpxETH/DpxEthToken.sol#L51)

## **G-30** | Calling `.length` in a for loop wastes gas

### **Description** :-

Rather than calling .length for an array in a for loop declaration, it is far more gas efficient to cache this length before and use that instead. This will prevent the array length from being called every loop iteration

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol
```
```solidity
147:for (uint256 i = 0; i < tokens.length; i++)
```
#### **Context**:-
[UniV2LiquidityAmo.sol#L147](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L147)
```solidity
File:2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol
```
```solidity
120:for (uint i = 0; i < positions_array.length; i++)
```
#### **Context**:-
[UniV3LiquidityAmo.sol#L120](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L120)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
167:for (uint256 i = 0; i < tokens.length; i++)
```
#### **Context**:-
[RdpxV2Core.sol#L167](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L167)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
246:for (uint256 i = 1; i < reserveAsset.length; i++)
```
#### **Context**:-
[RdpxV2Core.sol#L246](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L246)
```solidity
996:for (uint256 i = 1; i < reserveAsset.length; i++)
```
#### **Context**:-
[RdpxV2Core.sol#L996](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L996)
```solidity
File:2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol
```
```solidity
103:for (uint256 i = 0; i < tokens.length; i++)
```
#### **Context**:-
[RdpxDecayingBonds.sol#L103](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L103)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
225:for (uint256 i = 0; i < tokens.length; i++)
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L225](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L225)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
328:for (uint256 i = 0; i < optionIds.length; i++)
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L328](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L328)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
413:for (uint256 i = 0; i < strikes.length; i++)
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L413](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L413)

## **G-31** | `unchecked {}` can be used on the division of two `uints` in order to save gas

### **Description** :-



#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
535:          ethBalance + _amount <= (ethBalance + dpxEthBalance) /
```
#### **Context**:-
[RdpxV2Core.sol#L535](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L535)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
539:          dpxEthBalance + _amount <= (ethBalance + dpxEthBalance) /
```
#### **Context**:-
[RdpxV2Core.sol#L539](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L539)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
570:    reserveAsset[reservesIndex["WETH"]].tokenBalance -= _amount /
```
#### **Context**:-
[RdpxV2Core.sol#L570](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L570)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
574:      _amount /
```
#### **Context**:-
[RdpxV2Core.sol#L574](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L574)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
579:      .deposit(_amount /
```
#### **Context**:-
[RdpxV2Core.sol#L579](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L579)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
1165:        1e2) /
```
#### **Context**:-
[RdpxV2Core.sol#L1165](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L1165)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
1170:        ((RDPX_RATIO_PERCENTAGE - (bondDiscount /
```
#### **Context**:-
[RdpxV2Core.sol#L1170](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L1170)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
1176:        ((ETH_RATIO_PERCENTAGE - (bondDiscount /
```
#### **Context**:-
[RdpxV2Core.sol#L1176](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L1176)
```solidity
File:2023-08-dopex/contracts/core/RdpxV2Core.sol
```
```solidity
1190:      .roundUp(rdpxPrice - (rdpxPrice /
```
#### **Context**:-
[RdpxV2Core.sol#L1190](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L1190)
```solidity
File:2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol
```
```solidity
270:    uint256 strike = roundUp(currentPrice - (currentPrice /
```
#### **Context**:-
[PerpetualAtlanticVault.sol#L270](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L270)
```solidity
File:2023-08-dopex/contracts/reLP/ReLPContract.sol
```
```solidity
230:      1e2) /
```
#### **Context**:-
[ReLPContract.sol#L230](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L230)
```solidity
File:2023-08-dopex/contracts/reLP/ReLPContract.sol
```
```solidity
235:      baseReLpRatio) /
```
#### **Context**:-
[ReLPContract.sol#L235](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L235)
```solidity
File:2023-08-dopex/contracts/reLP/ReLPContract.sol
```
```solidity
274:      (((amountB /
```
#### **Context**:-
[ReLPContract.sol#L274](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L274)
```solidity
275:      (((amountB /
```
#### **Context**:-
[ReLPContract.sol#L275](https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L275)