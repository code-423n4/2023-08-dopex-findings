## [G-01] use if + revert instead of require to save gas

Consider using custom errors instead of revert strings. Can save gas when the revert condition has been met during runtime.
Here are some examples of where this optimization could be used:
```
File: contracts/amo/UniV2LiquidityAmo.sol
83: require(
      _tokenA != address(0) &&
        _tokenB != address(0) &&
        _pair != address(0) &&
        _rdpxV2Core != address(0) &&
        _rdpxOracle != address(0) &&
        _ammFactory != address(0) &&
        _ammRouter != address(0),
      "reLPContract: address cannot be 0"
    );
112: require(
      _slippageTolerance > 0,
      "reLPContract: slippage tolerance must be greater than 0"
    );
131: require(_token != address(0), "reLPContract: token cannot be 0");
132: require(_spender != address(0), "reLPContract: spender cannot be 0");
133: require(_amount > 0, "reLPContract: amount must be greater than 0");

File: contracts/core/RdpxV2Core.sol
244: require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");
247: require(
        reserveAsset[i].tokenAddress != _asset,
        "RdpxV2Core: asset already exists"
      );

File: contracts/decaying-bonds/RdpxDecayingBonds.sol
99: require(success, "RdpxReserve: transfer failed");
120: require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");

File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
94: require(
      _perpetualAtlanticVault != address(0) || _rdpx != address(0),
      "ZERO_ADDRESS"
    );
123: require((shares = previewDeposit(assets)) != 0, "ZERO_SHARES");
162: require(assets != 0, "ZERO_ASSETS");
191: require(
      collateral.balanceOf(address(this)) >= _totalCollateral + proceeds,
      "Not enough collateral token was sent"
    );
200: require(
      collateral.balanceOf(address(this)) == _totalCollateral - loss,
      "Not enough collateral was sent out"
    );
209: require(
      IERC20WithBurn(rdpx).balanceOf(address(this)) >= _rdpxCollateral + amount,
      "Not enough rdpx token was sent"
    );
287: require(
      assets <= totalAvailableCollateral(),
      "Not enough available assets to satisfy withdrawal"
    );
296: require(
      msg.sender == address(perpetualAtlanticVault),
      "Only the perp vault can call this function"
    );

```
 - UniV2LiquidityAmo.sol : [[83](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV2LiquidityAmo.sol#L83), [112](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV2LiquidityAmo.sol#L112), [131](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV2LiquidityAmo.sol#L131), [132](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV2LiquidityAmo.sol#L132), [133](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV2LiquidityAmo.sol#L133)]
 - RdpxV2Core.sol : [[244](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/core/RdpxV2Core.sol#L244), [247](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/core/RdpxV2Core.sol#L247)]
 - RdpxDecayingBonds.sol : [[99](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/decaying-bonds/RdpxDecayingBonds.sol#L99), [120](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/decaying-bonds/RdpxDecayingBonds.sol#L120)]
 - PerpetualAtlanticVaultLP.sol : [[94](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L94), [123](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L123), [162](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L162), [191](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L191), [200](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L200), [209](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L209), [287](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L287), [296](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L296)]

Note that this will only decrease runtime gas when the revert condition has been met. Regardless, it will decrease deploy time gas


## [G-02] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops
```
File: contracts/amo/UniV2LiquidityAmo.sol
147: for (uint256 i = 0; i < tokens.length; i++) {

File: contracts/amo/UniV3LiquidityAmo.sol
120: for (uint i = 0; i < positions_array.length; i++) {

File: contracts/core/RdpxV2Core.sol
167: for (uint256 i = 0; i < tokens.length; i++) {
246: for (uint256 i = 1; i < reserveAsset.length; i++) {
775: for (uint256 i = 0; i < optionIds.length; i++) {
836: for (uint256 i = 0; i < _amounts.length; i++) {
996: for (uint256 i = 1; i < reserveAsset.length; i++) {

File: contracts/decaying-bonds/RdpxDecayingBonds.sol
103: for (uint256 i = 0; i < tokens.length; i++) {
156: for (uint256 i; i < ownerTokenCount; i++) {

File: contracts/perp-vault/PerpetualAtlanticVault.sol
225: for (uint256 i = 0; i < tokens.length; i++) {
328: for (uint256 i = 0; i < optionIds.length; i++) {
413: for (uint256 i = 0; i < strikes.length; i++) {

```
 - UniV2LiquidityAmo.sol : [[147](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV2LiquidityAmo.sol#L147)]
 - UniV3LiquidityAmo.sol : [[120](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV3LiquidityAmo.sol#L120)]
 - RdpxV2Core.sol : [[167](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/core/RdpxV2Core.sol#L167), [246](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/core/RdpxV2Core.sol#L246), [775](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/core/RdpxV2Core.sol#L775), [836](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/core/RdpxV2Core.sol#L836), [996](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/core/RdpxV2Core.sol#L996)]
 - RdpxDecayingBonds.sol : [[103](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/decaying-bonds/RdpxDecayingBonds.sol#L103), [156](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/decaying-bonds/RdpxDecayingBonds.sol#L156)]
 - PerpetualAtlanticVault.sol : [[225](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/perp-vault/PerpetualAtlanticVault.sol#L225), [328](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/perp-vault/PerpetualAtlanticVault.sol#L328), [413](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/perp-vault/PerpetualAtlanticVault.sol#L413)]

## [G-03] <array>.length should not be looked up in every loop of a for-loop
The overheads outlined below are PER LOOP, excluding the first loop
- storage arrays incur a Gwarmaccess (**100 gas**)
- memory arrays use MLOAD (**3 gas**)
- calldata arrays use CALLDATALOAD (**3 gas**)

Caching the length changes each of these to a DUP<N> (**3 gas**), and gets rid of the extra DUP<N> needed to store the stack offset
Here are some examples of where this optimization could be used:
```
File: contracts/amo/UniV2LiquidityAmo.sol
147: for (uint256 i = 0; i < tokens.length; i++) {

File: contracts/amo/UniV3LiquidityAmo.sol
120: for (uint i = 0; i < positions_array.length; i++) {

File: contracts/core/RdpxV2Core.sol
167: for (uint256 i = 0; i < tokens.length; i++) {
246: for (uint256 i = 1; i < reserveAsset.length; i++) {
775: for (uint256 i = 0; i < optionIds.length; i++) {
836: for (uint256 i = 0; i < _amounts.length; i++) {
996: for (uint256 i = 1; i < reserveAsset.length; i++) {

File: contracts/decaying-bonds/RdpxDecayingBonds.sol
103: for (uint256 i = 0; i < tokens.length; i++) {

File: contracts/perp-vault/PerpetualAtlanticVault.sol
225: for (uint256 i = 0; i < tokens.length; i++) {
328: for (uint256 i = 0; i < optionIds.length; i++) {
413: for (uint256 i = 0; i < strikes.length; i++) {

```
 - UniV2LiquidityAmo.sol : [[147](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV2LiquidityAmo.sol#L147)]
 - UniV3LiquidityAmo.sol : [[120](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV3LiquidityAmo.sol#L120)]
 - RdpxV2Core.sol : [[167](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/core/RdpxV2Core.sol#L167), [246](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/core/RdpxV2Core.sol#L246), [775](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/core/RdpxV2Core.sol#L775), [836](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/core/RdpxV2Core.sol#L836), [996](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/core/RdpxV2Core.sol#L996)]
 - RdpxDecayingBonds.sol : [[103](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/decaying-bonds/RdpxDecayingBonds.sol#L103)]
 - PerpetualAtlanticVault.sol : [[225](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/perp-vault/PerpetualAtlanticVault.sol#L225), [328](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/perp-vault/PerpetualAtlanticVault.sol#L328), [413](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/perp-vault/PerpetualAtlanticVault.sol#L413)]

## [G-04] mark event arguments up to 3 arguments as indexed to save gas
Using this keyword with value types such as `uint`, `bool`, and `address` saves gas; however, this is only true for these value types, since indexing `bytes`, `strings` and `arrays` is more costly than not indexing them.

Here are some examples of where this optimization could be used:

```
File: contracts/amo/UniV2LiquidityAmo.sol
387: event LogAddLiquidity(
398: event LogRemoveLiquidity(
407: event LogSwap(
415: event LogAssetsTransfered(
421: event LogEmergencyWithdraw(address sender, address[] tokens);

File: contracts/amo/UniV3LiquidityAmo.sol
368: event RecoveredERC20(address token, uint256 amount);
369: event RecoveredERC721(address token, uint256 id);
370: event LogAssetsTransfered(
379: event log(uint);

File: contracts/decaying-bonds/RdpxDecayingBonds.sol
46: event BondMinted(
53: event EmergencyWithdraw(address sender);

File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
28: event Deposit(

```
 - UniV2LiquidityAmo.sol : [[387](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV2LiquidityAmo.sol#L387), [398](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV2LiquidityAmo.sol#L398), [407](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV2LiquidityAmo.sol#L407), [415](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV2LiquidityAmo.sol#L415), [421](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV2LiquidityAmo.sol#L421)]
 - UniV3LiquidityAmo.sol : [[368](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV3LiquidityAmo.sol#L368), [369](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV3LiquidityAmo.sol#L369), [370](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV3LiquidityAmo.sol#L370), [379](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV3LiquidityAmo.sol#L379)]
 - RdpxDecayingBonds.sol : [[46](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/decaying-bonds/RdpxDecayingBonds.sol#L46), [53](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/decaying-bonds/RdpxDecayingBonds.sol#L53)]
 - PerpetualAtlanticVaultLP.sol : [[28](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L28)] 

## [G-05] use bit shifting when applicable to save gas
in some cases when multiplying and diving by a power of 2, bit shifting can be used to save some gas

Here are some examples of where this optimization could be used:

```
File: contracts/reLP/ReLPContract.sol
 
232: uint256 tokenAToRemove = ((((_amount * 4) * 1e18) /
274: (((amountB / 2) * tokenAInfo.tokenAPrice) / 1e8) -
275: (((amountB / 2) * tokenAInfo.tokenAPrice * slippageTolerance) / 1e16);
279: amountB / 2,
290: amountB / 2,

```
- ReLPContract.sol : [[232](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L232), [274](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L274), [275](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L275), [279](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#279), [290](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#290)]

## [G-06] <x> += <y> costs more gas than <x> = <x> + <y> for state variables
Using the addition operator instead of plus-equals saves [113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)

Here are some examples of where this optimization could be used:

```
File: contracts/amo/UniV2LiquidityAmo.sol
235: lpTokenBalance += lpReceived;

File: contracts/core/RdpxV2Core.sol
964: totalWethDelegated += _amount;
1196: wethRequired += IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)

File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
132: _totalCollateral += assets;
181: _activeCollateral += amount;
195: _totalCollateral += proceeds;
213: _rdpxCollateral += amount;

File: contracts/perp-vault/PerpetualAtlanticVault.sol
493: latestFundingPaymentPointer += 1;

```
 - UniV2LiquidityAmo.sol : [[235](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV2LiquidityAmo.sol#L235)]
 - RdpxV2Core.sol : [[964](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/core/RdpxV2Core.sol#L964), [1196](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/core/RdpxV2Core.sol#L1196)]
 - PerpetualAtlanticVaultLP.sol : [[132](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L132), [181](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L181), [195](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L195), [213](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L213)]
 - PerpetualAtlanticVault.sol : [[493](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/perp-vault/PerpetualAtlanticVault.sol#L493)]
