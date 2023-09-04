## [G-01] use revert instead of require blah
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
```
