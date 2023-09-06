## [G-01] Don't Initialize variables with default value
Description
Uninitialized variables are assigned with the types default value.

Explicitly initializing a variable with it's default value costs unnecessary gas.
```txt
2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol::147 => for (uint256 i = 0; i < tokens.length; i++) {
2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol::120 => for (uint i = 0; i < positions_array.length; i++) {
2023-08-dopex/contracts/core/RdpxV2Core.sol::167 => for (uint256 i = 0; i < tokens.length; i++) {
2023-08-dopex/contracts/core/RdpxV2Core.sol::775 => for (uint256 i = 0; i < optionIds.length; i++) {
2023-08-dopex/contracts/core/RdpxV2Core.sol::836 => for (uint256 i = 0; i < _amounts.length; i++) {
2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol::103 => for (uint256 i = 0; i < tokens.length; i++) {
2023-08-dopex/contracts/libraries/ABDKMathQuad.sol::1673 => uint256 result = 0;
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::225 => for (uint256 i = 0; i < tokens.length; i++) {
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::328 => for (uint256 i = 0; i < optionIds.length; i++) {
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::413 => for (uint256 i = 0; i < strikes.length; i++) {
2023-08-dopex/contracts/reserve/RdpxReserve.sol::74 => for (uint256 i = 0; i < tokens.length; i++) {
```
## [G-02] Cache array length outside of loops
Description
Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.
```txt
2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol::147 => for (uint256 i = 0; i < tokens.length; i++) {
2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol::343 => )[path.length - 1];
2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol::113 => return positions_array.length;
2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol::120 => for (uint i = 0; i < positions_array.length; i++) {
2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol::260 => positions_array.length - 1
2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol::268 => emit log(positions_array.length);
2023-08-dopex/contracts/core/RdpxV2Core.sol::167 => for (uint256 i = 0; i < tokens.length; i++) {
2023-08-dopex/contracts/core/RdpxV2Core.sol::246 => for (uint256 i = 1; i < reserveAsset.length; i++) {
2023-08-dopex/contracts/core/RdpxV2Core.sol::261 => reservesIndex[_assetSymbol] = reserveAsset.length - 1;
2023-08-dopex/contracts/core/RdpxV2Core.sol::280 => reservesIndex[reserveTokens[reserveTokens.length - 1]] = index;
2023-08-dopex/contracts/core/RdpxV2Core.sol::283 => reserveAsset[index] = reserveAsset[reserveAsset.length - 1];
2023-08-dopex/contracts/core/RdpxV2Core.sol::391 => _validate(_index < amoAddresses.length, 18);
2023-08-dopex/contracts/core/RdpxV2Core.sol::392 => amoAddresses[_index] = amoAddresses[amoAddresses.length - 1];
2023-08-dopex/contracts/core/RdpxV2Core.sol::775 => for (uint256 i = 0; i < optionIds.length; i++) {
2023-08-dopex/contracts/core/RdpxV2Core.sol::827 => _validate(_amounts.length == _delegateIds.length, 3);
2023-08-dopex/contracts/core/RdpxV2Core.sol::833 => _amounts.length
2023-08-dopex/contracts/core/RdpxV2Core.sol::836 => for (uint256 i = 0; i < _amounts.length; i++) {
2023-08-dopex/contracts/core/RdpxV2Core.sol::966 => emit LogAddToDelegate(_amount, _fee, delegates.length - 1);
2023-08-dopex/contracts/core/RdpxV2Core.sol::967 => return (delegates.length - 1);
2023-08-dopex/contracts/core/RdpxV2Core.sol::979 => _validate(delegateId < delegates.length, 14);
2023-08-dopex/contracts/core/RdpxV2Core.sol::996 => for (uint256 i = 1; i < reserveAsset.length; i++) {
2023-08-dopex/contracts/core/RdpxV2Core.sol::1104 => )[path.length - 1];
2023-08-dopex/contracts/core/RdpxV2Core.sol::1245 => * @notice Viewer for returning length of uint256[] delegates
2023-08-dopex/contracts/core/RdpxV2Core.sol::1246 => * @return uint256 length of delegates array
2023-08-dopex/contracts/core/RdpxV2Core.sol::1249 => return delegates.length;
2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol::103 => for (uint256 i = 0; i < tokens.length; i++) {
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::225 => for (uint256 i = 0; i < tokens.length; i++) {
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::328 => for (uint256 i = 0; i < optionIds.length; i++) {
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::413 => for (uint256 i = 0; i < strikes.length; i++) {
2023-08-dopex/contracts/reLP/ReLPContract.sol::284 => )[path.length - 1];
```
## [G-03] Use not equal to 1 instead of greater than 0 for unsigned integer comparison
When dealing with unsigned integer types, comparisons with != 0 are cheaper than with > 0.
```txt
2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol::113 => _slippageTolerance > 0,
2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol::133 => require(_amount > 0, "reLPContract: amount must be greater than 0");
2023-08-dopex/contracts/core/RdpxV2Core.sol::183 => _validate(_rdpxBurnPercentage > 0, 3);
2023-08-dopex/contracts/core/RdpxV2Core.sol::196 => _validate(_rdpxFeePercentage > 0, 3);
2023-08-dopex/contracts/core/RdpxV2Core.sol::231 => _validate(_bondMaturity > 0, 3);
2023-08-dopex/contracts/core/RdpxV2Core.sol::410 => _validate(_amount > 0, 17);
2023-08-dopex/contracts/core/RdpxV2Core.sol::444 => _validate(_bondDiscountFactor > 0, 3);
2023-08-dopex/contracts/core/RdpxV2Core.sol::458 => _validate(_slippageTolerance > 0, 3);
2023-08-dopex/contracts/core/RdpxV2Core.sol::556 => minAmount > 0 ? minAmount : minOut
2023-08-dopex/contracts/core/RdpxV2Core.sol::838 => _validate(_amounts[i] > 0, 4);
2023-08-dopex/contracts/core/RdpxV2Core.sol::901 => _validate(_amount > 0, 4);
2023-08-dopex/contracts/core/RdpxV2Core.sol::984 => _validate(amountWithdrawn > 0, 15);
2023-08-dopex/contracts/core/RdpxV2Core.sol::1021 => _validate(bonds[id].timestamp > 0, 6);
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::265 => _validate(amount > 0, 2);
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::414 => _validate(optionsPerStrike[strikes[i]] > 0, 4);
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::547 => _price > 0 ? _price : getUnderlyingPrice(),
2023-08-dopex/contracts/reLP/ReLPContract.sol::94 => _reLPFactor > 0,
2023-08-dopex/contracts/reLP/ReLPContract.sol::175 => _liquiditySlippageTolerance > 0,
2023-08-dopex/contracts/reLP/ReLPContract.sol::190 => _slippageTolerance > 0,
```
## [G-04] Use immutable for openzeppelin access controls roles declaration
⚡️ Only valid for solidity versions <0.6.12 ⚡️

Access roles marked as constant results in computing the keccak256 operation each time the variable is used because assigned operations for constant variables are re-evaluated every time.

Changing the variables to immutable results in computing the hash only once on deployment, leading to gas savings.
```txt
2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol::106 => keccak256(abi.encodePacked(address(this), _tickLower, _tickUpper))
2023-08-dopex/contracts/core/RdpxV2Bond.sol::22 => bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
2023-08-dopex/contracts/core/RdpxV2Core.sol::1141 => keccak256(abi.encodePacked(asset.tokenSymbol)) ==
2023-08-dopex/contracts/core/RdpxV2Core.sol::1142 => keccak256(abi.encodePacked(_token)),
2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol::31 => bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol::34 => bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
2023-08-dopex/contracts/dpxETH/DpxEthToken.sol::19 => bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
2023-08-dopex/contracts/dpxETH/DpxEthToken.sol::20 => bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
2023-08-dopex/contracts/dpxETH/DpxEthToken.sol::21 => bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::45 => bytes32 public constant MANAGER_ROLE = keccak256("MANAGER_ROLE");
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::48 => bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
2023-08-dopex/contracts/reLP/ReLPContract.sol::70 => bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
2023-08-dopex/contracts/reserve/RdpxReserve.sol::18 => bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
2023-08-dopex/contracts/reserve/RdpxReserve.sol::19 => bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
```
## [G-05] Long revert strings
Description
Shortening revert strings to fit in 32 bytes will decrease gas costs for deployment and gas costs when the revert condition has been met.

If the contract(s) in scope allow using Solidity >=0.8.4, consider using Custom Errors as they are more gas efficient while allowing developers to describe the error in detail using NatSpec.
```txt
2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol::5 => import { AccessControl } from "@openzeppelin/contracts/access/AccessControl.sol";
2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol::6 => import { SafeMath } from "@openzeppelin/contracts/utils/math/SafeMath.sol";
2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol::7 => import { SafeERC20 } from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol::11 => import { IUniswapV2Router } from "../uniswap_V2/IUniswapV2Router.sol";
2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol::17 => import { Math } from "@openzeppelin/contracts/utils/math/Math.sol";
2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol::91 => "reLPContract: address cannot be 0"
2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol::114 => "reLPContract: slippage tolerance must be greater than 0"
2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol::132 => require(_spender != address(0), "reLPContract: spender cannot be 0");
2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol::133 => require(_amount > 0, "reLPContract: amount must be greater than 0");
2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol::4 => import { SafeERC20 } from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol::5 => import { AccessControl } from "@openzeppelin/contracts/access/AccessControl.sol";
2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol::6 => import { ERC721Holder } from "@openzeppelin/contracts/token/ERC721/utils/ERC721Holder.sol";
2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol::7 => import { SafeMath } from "@openzeppelin/contracts/utils/math/SafeMath.sol";
2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol::12 => import "../uniswap_V3/IUniswapV3Factory.sol";
2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol::13 => import "../uniswap_V3/libraries/TickMath.sol";
2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol::14 => import "../uniswap_V3/libraries/LiquidityAmounts.sol";
2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol::15 => import "../uniswap_V3/periphery/interfaces/INonfungiblePositionManager.sol";
2023-08-dopex/contracts/core/RdpxV2Bond.sol::4 => import { ERC721 } from "@openzeppelin/contracts/token/ERC721/ERC721.sol";
2023-08-dopex/contracts/core/RdpxV2Bond.sol::5 => import { ERC721Enumerable } from "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
2023-08-dopex/contracts/core/RdpxV2Bond.sol::7 => import { AccessControl } from "@openzeppelin/contracts/access/AccessControl.sol";
2023-08-dopex/contracts/core/RdpxV2Bond.sol::8 => import { ERC721Burnable } from "@openzeppelin/contracts/token/ERC721/extensions/ERC721Burnable.sol";
2023-08-dopex/contracts/core/RdpxV2Bond.sol::9 => import { Counters } from "@openzeppelin/contracts/utils/Counters.sol";
2023-08-dopex/contracts/core/RdpxV2Core.sol::5 => import { AccessControl } from "@openzeppelin/contracts/access/AccessControl.sol";
2023-08-dopex/contracts/core/RdpxV2Core.sol::6 => import { ERC721Holder } from "@openzeppelin/contracts/token/ERC721/utils/ERC721Holder.sol";
2023-08-dopex/contracts/core/RdpxV2Core.sol::13 => import { IUniswapV2Router } from "../uniswap_V2/IUniswapV2Router.sol";
2023-08-dopex/contracts/core/RdpxV2Core.sol::15 => import { IRdpxDecayingBonds } from "../decaying-bonds/IRdpxDecayingBonds.sol";
2023-08-dopex/contracts/core/RdpxV2Core.sol::17 => import { IPerpetualAtlanticVault } from "../perp-vault/IPerpetualAtlanticVault.sol";
2023-08-dopex/contracts/core/RdpxV2Core.sol::22 => import { IRdpxV2ReceiptToken } from "../interfaces/IRdpxV2ReceiptToken.sol";
2023-08-dopex/contracts/core/RdpxV2Core.sol::26 => import { EnumerableSet } from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";
2023-08-dopex/contracts/core/RdpxV2Core.sol::27 => import { SafeERC20 } from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
2023-08-dopex/contracts/core/RdpxV2Core.sol::28 => import { Math } from "@openzeppelin/contracts/utils/math/Math.sol";
2023-08-dopex/contracts/core/RdpxV2Core.sol::244 => require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");
2023-08-dopex/contracts/core/RdpxV2Core.sol::1294 => // E5: "Not enough ETH available from the delegate",
2023-08-dopex/contracts/core/RdpxV2Core.sol::1300 => // E11: "Amount greater that max permissible amount",
2023-08-dopex/contracts/core/RdpxV2Core.sol::1301 => // E12: "Price is not lower than first lower peg",
2023-08-dopex/contracts/core/RdpxV2Core.sol::1302 => // E13: "Amount greater than max permissible amount"
2023-08-dopex/contracts/core/RdpxV2Core.sol::1305 => // E16: "Funding already paid for the epoch"
2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol::5 => import { SafeERC20 } from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol::9 => import { ERC721 } from "@openzeppelin/contracts/token/ERC721/ERC721.sol";
2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol::10 => import { ERC721Enumerable } from
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::5 => import { SafeERC20 } from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::8 => import { ReentrancyGuard } from "@openzeppelin/contracts/security/ReentrancyGuard.sol";
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::9 => import { ERC721 } from "@openzeppelin/contracts/token/ERC721/ERC721.sol";
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::10 => import { ERC721Enumerable } from "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::11 => import { ERC721Burnable } from "@openzeppelin/contracts/token/ERC721/extensions/ERC721Burnable.sol";
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::12 => import { ERC4626 } from "@openzeppelin/contracts/token/ERC20/extensions/ERC4626.sol";
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::13 => import { AccessControl } from "@openzeppelin/contracts/access/AccessControl.sol";
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::14 => import { Counters } from "@openzeppelin/contracts/utils/Counters.sol";
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::24 => import { IVolatilityOracle } from "../interfaces/IVolatilityOracle.sol";
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::660 => // E3: "Insufficient collateral for purchase",
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol::663 => // E6: "All funding payments must be accounted for",
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVaultLP.sol::6 => import { FixedPointMathLib } from "../solmate/src/utils/FixedPointMathLib.sol";
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVaultLP.sol::10 => import { Strings } from "@openzeppelin/contracts/utils/Strings.sol";
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVaultLP.sol::11 => import { SafeERC20 } from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVaultLP.sol::12 => import { SafeTransferLib } from "../solmate/src/utils/SafeTransferLib.sol";
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVaultLP.sol::193 => "Not enough collateral token was sent"
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVaultLP.sol::202 => "Not enough collateral was sent out"
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVaultLP.sol::289 => "Not enough available assets to satisfy withdrawal"
2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVaultLP.sol::298 => "Only the perp vault can call this function"
2023-08-dopex/contracts/reLP/ReLPContract.sol::5 => import { AccessControl } from "@openzeppelin/contracts/access/AccessControl.sol";
2023-08-dopex/contracts/reLP/ReLPContract.sol::6 => import { SafeMath } from "@openzeppelin/contracts/utils/math/SafeMath.sol";
2023-08-dopex/contracts/reLP/ReLPContract.sol::7 => import { SafeERC20 } from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
2023-08-dopex/contracts/reLP/ReLPContract.sol::11 => import { IUniswapV2Router } from "../uniswap_V2/IUniswapV2Router.sol";
2023-08-dopex/contracts/reLP/ReLPContract.sol::16 => import { IUniV2LiquidityAmo } from "../interfaces/IUniV2LiquidityAmo.sol";
2023-08-dopex/contracts/reLP/ReLPContract.sol::19 => import { UniswapV2Library } from "../uniswap_V2/libraries/UniswapV2Library.sol";
2023-08-dopex/contracts/reLP/ReLPContract.sol::20 => import { Math } from "@openzeppelin/contracts/utils/math/Math.sol";
2023-08-dopex/contracts/reLP/ReLPContract.sol::95 => "reLPContract: reLP factor must be greater than 0"
2023-08-dopex/contracts/reLP/ReLPContract.sol::136 => "reLPContract: address cannot be 0"
2023-08-dopex/contracts/reLP/ReLPContract.sol::176 => "reLPContract: liquidity slippage tolerance must be greater than 0"
2023-08-dopex/contracts/reLP/ReLPContract.sol::191 => "reLPContract: slippage tolerance must be greater than 0"
```
## [G-06] Use shift or rightleft instead of division or multiplication if possible
Description
A division/multiplication by any number x being a power of 2 can be calculated by shifting log2(x) to the right/left.

While the DIV opcode uses 5 gas, the SHR opcode only uses 3 gas. Furthermore, Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting.
```txt
2023-08-dopex/contracts/core/RdpxV2Core.sol::535 => ethBalance + _amount <= (ethBalance + dpxEthBalance) / 2,
2023-08-dopex/contracts/core/RdpxV2Core.sol::539 => dpxEthBalance + _amount <= (ethBalance + dpxEthBalance) / 2,
2023-08-dopex/contracts/core/RdpxV2Core.sol::570 => reserveAsset[reservesIndex["WETH"]].tokenBalance -= _amount / 2;
2023-08-dopex/contracts/core/RdpxV2Core.sol::574 => _amount / 2
2023-08-dopex/contracts/core/RdpxV2Core.sol::579 => .deposit(_amount / 2);
2023-08-dopex/contracts/core/RdpxV2Core.sol::1170 => ((RDPX_RATIO_PERCENTAGE - (bondDiscount / 2)) *
2023-08-dopex/contracts/core/RdpxV2Core.sol::1176 => ((ETH_RATIO_PERCENTAGE - (bondDiscount / 2)) * _amount) /
2023-08-dopex/contracts/core/RdpxV2Core.sol::1190 => .roundUp(rdpxPrice - (rdpxPrice / 4)); // 25% below the current price
2023-08-dopex/contracts/reLP/ReLPContract.sol::232 => uint256 tokenAToRemove = ((((_amount * 4) * 1e18) /
2023-08-dopex/contracts/reLP/ReLPContract.sol::274 => (((amountB / 2) * tokenAInfo.tokenAPrice) / 1e8) -
2023-08-dopex/contracts/reLP/ReLPContract.sol::275 => (((amountB / 2) * tokenAInfo.tokenAPrice * slippageTolerance) / 1e16);
2023-08-dopex/contracts/reLP/ReLPContract.sol::279 => amountB / 2,
2023-08-dopex/contracts/reLP/ReLPContract.sol::290 => amountB / 2,
```