## Non Critical Issues

|       | Issue                                                      |
| ----- | :--------------------------------------------------------- |
| NC-1  | ADD A TIMELOCK TO CRITICAL FUNCTIONS                       |
| NC-2  | ADD TO BLACKLIST FUNCTION                                  |
| NC-3  | Be explicit declaring types                                |
| NC-4  | Boolean equality                                           |
| NC-5  | USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)       |
| NC-6  | GENERATE PERFECT CODE HEADERS EVERY TIME                   |
| NC-7  | SAME CONSTANT REDEFINED ELSEWHERE                          |
| NC-8  | NO SAME VALUE INPUT CONTROL                                |
| NC-9  | Omissions in events                                        |
| NC-10 | USE A MORE RECENT VERSION OF SOLIDITY                      |
| NC-11 | Return values of `approve()` not checked                   |
| NC-12 | The tokenAddress state variable should be renamed to token |

### [NC-1] ADD A TIMELOCK TO CRITICAL FUNCTIONS

#### Description:

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

#### **Proof Of Concept**

```solidity
File: contracts/amo/UniV2LiquidityAmo.sol

74:   function setAddresses(

109:   function setSlippageTolerance(

```

```solidity
File: contracts/core/RdpxV2Core.sol

180:   function setRdpxBurnPercentage(

193:   function setRdpxFeePercentage(

206:   function setIsreLP(bool _isReLPActive) external onlyRole(DEFAULT_ADMIN_ROLE) {

216:   function setPutOptionsRequired(

228:   function setBondMaturity(

240:   function addAssetTotokenReserves(

304:   function setAddresses(

358:   function setPricingOracleAddresses(

441:   function setBondDiscount(

455:   function setSlippageTolerance(

```

```solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol

181:   function setAddresses(

243:   function setLpAllowance(bool increase) external onlyRole(DEFAULT_ADMIN_ROLE) {

315:   function settle(

```

```solidity
File: contracts/reLP/ReLPContract.sol

90:   function setreLpFactor(

115:   function setAddresses(

171:   function setLiquiditySlippageTolerance(

186:   function setSlippageTolerance(

```

### [NC-2] ADD TO BLACKLIST FUNCTION

#### Description:

NFT thefts have increased recently, so with the addition of hacked NFTs to the platform, NFTs can be converted into liquidity. To prevent this, I recommend adding the blacklist function.

#### Recommended Mitigation Steps:

Add to Blacklist function and modifier.

### [NC-3] Be explicit declaring types

#### Description:

Instead of uint use uint256

#### **Proof Of Concept**

```solidity
File: contracts/amo/UniV3LiquidityAmo.sol

23:   function read() external view virtual returns (uint);

120:     for (uint i = 0; i < positions_array.length; i++) {

379:   event log(uint);

```

### [NC-4] Boolean equality

#### Description:

Boolean variables can be checked within conditionals directly without the use of equality operators to true/false.

https://github.com/crytic/slither/wiki/Detector-Documentation#boolean-equality

#### **Proof Of Concept**

```solidity
File: contracts/core/RdpxV2Core.sol

798:     _validate(fundingPaidFor[pointer] == false, 16);

```

### [NC-5] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)

#### Description:

Rather than using `abi.encodePacked` for appending bytes, since version 0.8.4, `bytes.concat()` is enabled.

Since version 0.8.4 for appending bytes, `bytes.concat()` can be used instead of `abi.encodePacked(,)`.

#### **Proof Of Concept**

```solidity
File: contracts/amo/UniV3LiquidityAmo.sol

106:       keccak256(abi.encodePacked(address(this), _tickLower, _tickUpper))

```

```solidity
File: contracts/core/RdpxV2Core.sol

1141:       keccak256(abi.encodePacked(asset.tokenSymbol)) ==

1142:         keccak256(abi.encodePacked(_token)),

```

### [NC-6] GENERATE PERFECT CODE HEADERS EVERY TIME

#### Description:

I recommend using header for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

### [NC-7] SAME CONSTANT REDEFINED ELSEWHERE

#### Description:

There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values):

#### **Proof Of Concept**

```solidity
File: contracts/amo/UniV2LiquidityAmo.sol

48:   uint256 public constant DEFAULT_PRECISION = 1e8;

```

```solidity
File: contracts/core/RdpxV2Bond.sol

22:   bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

```

```solidity
File: contracts/core/RdpxV2Core.sol

85:   uint256 public constant DEFAULT_PRECISION = 1e8;

```

```solidity
File: contracts/decaying-bonds/RdpxDecayingBonds.sol

31:   bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

34:   bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");

```

```solidity
File: contracts/dpxETH/DpxEthToken.sol

20:   bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

```

```solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol

48:   bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");

```

```solidity
File: contracts/reLP/ReLPContract.sol

70:   bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");

```

### [NC-8] NO SAME VALUE INPUT CONTROL

#### **Proof Of Concept**

```solidity
File: contracts/amo/UniV2LiquidityAmo.sol

116:     slippageTolerance = _slippageTolerance;

```

```solidity
File: contracts/amo/UniV3LiquidityAmo.sol

77:     rdpx = _rdpx;

78:     rdpxV2Core = _rdpxV2Core;

```

```solidity
File: contracts/core/RdpxV2Core.sol

126:     weth = _weth;

184:     rdpxBurnPercentage = _rdpxBurnPercentage;

197:     rdpxFeePercentage = _rdpxFeePercentage;

207:     isReLPActive = _isReLPActive;

219:     putOptionsRequired = _putOptionsRequired;

232:     bondMaturity = _bondMaturity;

445:     bondDiscountFactor = _bondDiscountFactor;

459:     slippageTolerance = _slippageTolerance;

```

```solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol

124:     genesis = _gensis;

240:     fundingDuration = _fundingDuration;

```

```solidity
File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol

99:     rdpxRdpxV2Core = _rdpxRdpxV2Core;

100:     collateralSymbol = _collateralSymbol;

101:     rdpx = _rdpx;

```

```solidity
File: contracts/reLP/ReLPContract.sol

97:     reLPFactor = _reLPFactor;

178:     liquiditySlippageTolerance = _liquiditySlippageTolerance;

193:     slippageTolerance = _slippageTolerance;

```

### [NC-9] Omissions in events

#### Description:

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters.

#### **Proof Of Concept**

```solidity
File: contracts/core/RdpxV2Core.sol

185:     emit LogSetRdpxBurnPercentage(_rdpxBurnPercentage);

198:     emit LogSetRdpxFeePercentage(_rdpxFeePercentage);

208:     emit LogSetIsReLPActive(_isReLPActive);

220:     emit LogSetputOptionsRequired(_putOptionsRequired);

233:     emit LogSetBondMaturity(_bondMaturity);

349:     emit LogSetAddresses(addresses);

370:     emit LogSetPricingOracleAddresses(pricingOracleAddresses);

447:     emit LogSetBondDiscountFactor(_bondDiscountFactor);

461:     emit LogSetSlippageTolerance(_slippageTolerance);

782:     emit LogSettle(optionIds);

1007:     emit LogSync();

```

```solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol

211:     emit AddressesSet(addresses);

494:       emit FundingPaymentPointerUpdated(latestFundingPaymentPointer);

```

```solidity
File: contracts/reLP/ReLPContract.sol

99:     emit LogSetReLpFactor(_reLPFactor);

```

### [NC-10] USE A MORE RECENT VERSION OF SOLIDITY

#### Description:

For security, it is best practice to use the latest Solidity version. For the security fix list in the versions; https://github.com/ethereum/solidity/blob/develop/Changelog.md

#### **Proof Of Concept**

```solidity
File: contracts/amo/UniV2LiquidityAmo.sol

2: pragma solidity 0.8.19;

```

```solidity
File: contracts/amo/UniV3LiquidityAmo.sol

2: pragma solidity 0.8.19;

```

```solidity
File: contracts/core/RdpxV2Bond.sol

2: pragma solidity 0.8.19;

```

```solidity
File: contracts/core/RdpxV2Core.sol

2: pragma solidity 0.8.19;

```

```solidity
File: contracts/decaying-bonds/RdpxDecayingBonds.sol

2: pragma solidity 0.8.19;

```

```solidity
File: contracts/dpxETH/DpxEthToken.sol

2: pragma solidity 0.8.19;

```

```solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol

2: pragma solidity 0.8.19;

```

```solidity
File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol

2: pragma solidity 0.8.19;

```

```solidity
File: contracts/reLP/ReLPContract.sol

2: pragma solidity 0.8.19;

```

### [NC-11] Return values of `approve()` not checked

#### Description:

Not all IERC20 implementations `revert()` when there's a failure in `approve()`. The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything

#### **Proof Of Concept**

```solidity
File: contracts/amo/UniV2LiquidityAmo.sol

134:     IERC20WithBurn(_token).approve(_spender, _amount);

```

```solidity
File: contracts/amo/UniV3LiquidityAmo.sol

150:       IERC20WithBurn(_token).approve(_target, _amount);

169:     IERC20WithBurn(params._tokenA).approve(

173:     IERC20WithBurn(params._tokenB).approve(

```

```solidity
File: contracts/core/RdpxV2Core.sol

339:     IERC20WithBurn(weth).approve(

343:     IERC20WithBurn(weth).approve(addresses.dopexAMMRouter, type(uint256).max);

344:     IERC20WithBurn(weth).approve(addresses.dpxEthCurvePool, type(uint256).max);

345:     IERC20WithBurn(weth).approve(

411:     IERC20WithBurn(_token).approve(_spender, _amount);

```

```solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol

245:       ? collateralToken.approve(

249:       : collateralToken.approve(addresses.perpetualAtlanticVaultLP, 0);

```

```solidity
File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol

106:     collateral.approve(_perpetualAtlanticVault, type(uint256).max);

107:     ERC20(rdpx).approve(_perpetualAtlanticVault, type(uint256).max);

```

### [NC-12] The tokenAddress state variable should be renamed to token

#### Description:

The tokenAddress state variable of VTVLVesting should be renamed to token as this variable represents an IERC20 interface rather that just an address. Renaming it to token aligns better with its usage.

#### **Proof Of Concept**

```solidity
File: contracts/amo/UniV3LiquidityAmo.sol

314:     address tokenAddress,

319:     TransferHelper.safeTransfer(tokenAddress, rdpxV2Core, tokenAmount);

321:     emit RecoveredERC20(tokenAddress, tokenAmount);

325:     address tokenAddress,

330:     INonfungiblePositionManager(tokenAddress).safeTransferFrom(

335:     emit RecoveredERC721(tokenAddress, token_id);

```

```solidity
File: contracts/core/RdpxV2Core.sol

130:       tokenAddress: address(0),

248:         reserveAsset[i].tokenAddress != _asset,

254:       tokenAddress: _asset,

572:     IDpxEthToken(reserveAsset[reservesIndex["DPXETH"]].tokenAddress).mint(

653:       IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress)

657:       IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress).burn(

662:       IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress)

997:       uint256 balance = IERC20WithBurn(reserveAsset[i].tokenAddress).balanceOf(

1001:       if (weth == reserveAsset[i].tokenAddress) {

1059:     IDpxEthToken(reserveAsset[reservesIndex["DPXETH"]].tokenAddress).mint(

1094:       path[0] = reserveAsset[reservesIndex["RDPX"]].tokenAddress;

1119:     IDpxEthToken(reserveAsset[reservesIndex["DPXETH"]].tokenAddress).burn(

1146:     return (asset.tokenAddress, asset.tokenBalance, asset.tokenSymbol);

```

## Low Issues

|     | Issue                                                 |
| --- | :---------------------------------------------------- |
| L-1 | Avoid `transfer()`/`send()` as reentrancy mitigations |
| L-2 | CURRENT DECAY PERCENTAGE COULD BE TOO HIGH            |
| L-3 | Do not use deprecated library functions               |
| L-4 | EMPTY BLOCKS SHOULD BE REMOVED OR EMIT SOMETHING      |
| L-5 | UNIFY RETURN CRITERIA                                 |
| L-6 | Incorrect return values for ERC721 `ownerOf()`        |
| L-7 | Timestamp Dependence                                  |
| L-8 | Unsafe ERC20 operation(s)                             |

### [L-1] Avoid `transfer()`/`send()` as reentrancy mitigations

#### Description:

Although `transfer()` and `send()` have been recommended as a security best-practice to prevent reentrancy attacks because they only forward 2300 gas, the gas repricing of opcodes may break deployed contracts. Use `call()` instead, without hardcoded gas limits along with checks-effects-interactions pattern or reentrancy guards for reentrancy protection.

#### **Proof Of Concept**

```solidity
File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol

170:     collateral.transfer(receiver, assets);

```

### [L-2] CURRENT DECAY PERCENTAGE COULD BE TOO HIGH

#### Description:

Code contains empty block

Currently, we have a decay per period of 70% and a period of 1 day. This means that every day, price drop will be 70%. While I understand that the protocol strives for an exponential decay dutch auction format, with the current numbers, price of NFT will quickly be negligible.

An example of how quickly the price can drop.

Day 0: 1000 Day 1: 300 Day 2: 90 Day 3: 27

#### **Proof Of Concept**

```solidity
File: contracts/core/RdpxV2Core.sol

15: import { IRdpxDecayingBonds } from "../decaying-bonds/IRdpxDecayingBonds.sol";

15: import { IRdpxDecayingBonds } from "../decaying-bonds/IRdpxDecayingBonds.sol";

15: import { IRdpxDecayingBonds } from "../decaying-bonds/IRdpxDecayingBonds.sol";

307:     address _rdpxDecayingBonds,

318:     _validate(_rdpxDecayingBonds != address(0), 17);

330:       rdpxDecayingBonds: _rdpxDecayingBonds,

330:       rdpxDecayingBonds: _rdpxDecayingBonds,

631:       (, uint256 expiry, uint256 amount) = IRdpxDecayingBonds(

632:         addresses.rdpxDecayingBonds

638:         IRdpxDecayingBonds(addresses.rdpxDecayingBonds).ownerOf(_bondId) ==

638:         IRdpxDecayingBonds(addresses.rdpxDecayingBonds).ownerOf(_bondId) ==

643:       IRdpxDecayingBonds(addresses.rdpxDecayingBonds).decreaseAmount(

643:       IRdpxDecayingBonds(addresses.rdpxDecayingBonds).decreaseAmount(

```

```solidity
File: contracts/decaying-bonds/RdpxDecayingBonds.sol

18: contract RdpxDecayingBonds is

```

#### Recommended Mitigation Steps:

One recommendation is to reduce this amount to a more reasonable 30-50%. It still maintains the property of exponential decrease, but at a slower pace.

### [L-3] Do not use deprecated library functions

#### Description:

You use safeApprove of openZeppelin although it’s deprecated. (see [here](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/566a774222707e424896c0c390a84dc3c13bdcb2/contracts/token/ERC20/utils/SafeERC20.sol#L38%3E)).

You should change it to increase/decrease Allowance as OpenZeppilin says.

Use of deprecated functions/operators such as block.blockhash() for blockhash(), msg.gas for gasleft(), throw for revert(), sha3() for keccak256(), callcode() for delegatecall(), suicide() for selfdestruct(), constant for view or var for actual type name should be avoided to prevent unintended errors with newer compiler versions.

https://swcregistry.io/docs/SWC-111

#### **Proof Of Concept**

```solidity
File: contracts/amo/UniV2LiquidityAmo.sol

58:     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);

200:     IERC20WithBurn(addresses.tokenA).safeApprove(

204:     IERC20WithBurn(addresses.tokenB).safeApprove(

268:     IERC20WithBurn(addresses.pair).safeApprove(addresses.ammRouter, lpAmount);

328:     IERC20WithBurn(token1).safeApprove(addresses.ammRouter, token1Amount);

```

```solidity
File: contracts/amo/UniV3LiquidityAmo.sol

80:     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);

148:       TransferHelper.safeApprove(_token, _target, _amount);

302:     TransferHelper.safeApprove(_tokenA, address(univ3_router), _amountAtoB);

```

```solidity
File: contracts/core/RdpxV2Bond.sol

25:     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);

26:     _setupRole(MINTER_ROLE, msg.sender);

```

```solidity
File: contracts/core/RdpxV2Core.sol

125:     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);

```

```solidity
File: contracts/decaying-bonds/RdpxDecayingBonds.sol

61:     _setupRole(MINTER_ROLE, msg.sender);

62:     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);

```

```solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol

126:     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);

127:     _setupRole(MANAGER_ROLE, msg.sender);

207:     collateralToken.safeApprove(

```

```solidity
File: contracts/reLP/ReLPContract.sol

80:     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);

81:     _setupRole(RDPXV2CORE_ROLE, msg.sender);

150:     IERC20WithBurn(addresses.pair).safeApprove(

155:     IERC20WithBurn(addresses.tokenA).safeApprove(

160:     IERC20WithBurn(addresses.tokenB).safeApprove(

```

### [L-4] EMPTY BLOCKS SHOULD BE REMOVED OR EMIT SOMETHING

#### Description:

Code contains empty block

Empty blocks should be removed or emit something - The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be `abstract` and the function signatures be added without any default implementation. If the block is an empty `if`-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (`if(x){}else if(y){...}else{...}` => `if(!x){if(y){...}else{...}}`). Empty `receive()`/`fallback()` payable functions that are not used, can be removed to save deployment gas.

#### **Proof Of Concept**

```solidity
File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol

235:   ) internal virtual {}

```

#### Recommended Mitigation Steps:

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.

### [L-5] UNIFY RETURN CRITERIA

In contracts, sometimes the name of the return variable is not defined and sometimes is, unifying the way of writing the code makes the code more uniform and readable.

#### **Proof Of Concept**

#### Recommended Mitigation Steps:

add `{return x}` if you want to return the updated value or else remove `returns(uint)` from the `function(){}` if no value you wanted to return

```solidity
File: contracts/amo/UniV2LiquidityAmo.sol

197:     returns (uint256 tokenAUsed, uint256 tokenBUsed, uint256 lpReceived)

265:     returns (uint256 tokenAReceived, uint256 tokenBReceived)

308:   ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256 token2Amount) {

372:   function getLpTokenBalanceInWeth() external view returns (uint256) {

381:   function getLpPrice() public view returns (uint256) {

```

```solidity
File: contracts/core/RdpxV2Bond.sol

39:   ) public onlyRole(MINTER_ROLE) returns (uint256 tokenId) {

63:     returns (bool)

```

```solidity
File: contracts/core/RdpxV2Core.sol

473:   ) internal returns (uint256 premium) {

498:   ) internal returns (uint256 bondId) {

520:   ) internal returns (uint256 amountOut) {

569:   ) internal returns (uint256 receiptTokenAmount) {

603:   ) internal view returns (uint256 amount1, uint256 amount2) {

703:   ) internal returns (BondWithDelegateReturnValue memory returnValues) {

769:     returns (uint256 amountOfWeth, uint256 rdpxAmount)

793:     returns (uint256 fundingAmount)

824:   ) public returns (uint256 receiptTokenAmount, uint256[] memory) {

898:   ) public returns (uint256 receiptTokenAmount) {

944:   ) external returns (uint256) {

977:   ) external returns (uint256 amountWithdrawn) {

1019:   ) external returns (uint256 receiptTokenAmount) {

1054:   ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256 wethReceived) {

1085:   ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256 dpxEthReceived) {

1137:   ) public view returns (address, uint256, string memory) {

1159:   ) public view returns (uint256 rdpxRequired, uint256 wethRequired) {

1206:   function getLpPrice() public view returns (uint256) {

1216:   function getDpxEthPrice() public view returns (uint256 dpxEthPrice) {

1227:   function getEthPrice() public view returns (uint256 ethPrice) {

1238:   function getRdpxPrice() public view returns (uint256) {

1248:   function getDelegatesLength() external view returns (uint256) {

1265:     returns (

```

```solidity
File: contracts/decaying-bonds/RdpxDecayingBonds.sol

129:   function _mintToken(address to) private returns (uint256 tokenId) {

153:   ) external view returns (uint256[] memory) {

180:     returns (bool)

```

```solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol

262:     returns (uint256 premium, uint256 tokenId)

321:     returns (uint256 ethAmount, uint256 rdpxAmount)

372:   function payFunding() external onlyRole(RDPXV2CORE_ROLE) returns (uint256) {

407:   ) external nonReentrant returns (uint256 fundingAmount) {

529:   function getUnderlyingPrice() public view returns (uint256) {

534:   function getVolatility(uint256 _strike) public view returns (uint256) {

544:   ) public view returns (uint256 premium) {

558:   ) public pure returns (uint256) {

566:     returns (uint256 timestamp)

576:   function roundUp(uint256 _strike) public view returns (uint256 strike) {

588:   function _mintOptionToken() private returns (uint256 tokenId) {

651:     returns (bool)

```

```solidity
File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol

121:   ) public virtual returns (uint256 shares) {

149:   ) public returns (uint256 assets, uint256 rdpxAmount) {

220:   ) internal view virtual returns (uint256 assets, uint256 rdpxAmount) {

240:   function activeCollateral() public view returns (uint256) {

245:   function totalCollateral() public view returns (uint256) {

250:   function rdpxCollateral() public view returns (uint256) {

255:   function totalAvailableCollateral() public view returns (uint256) {

264:   ) public view returns (uint256, uint256) {

269:   function previewDeposit(uint256 assets) public view returns (uint256) {

276:   ) public view returns (uint256 shares) {

```

### [L-6] Incorrect return values for ERC721 `ownerOf()`

#### Description:

Incorrect return values for ERC721 ownerOf(): Contracts compiled with solc >= 0.4.22 interacting with ERC721 ownerOf() that returns a bool instead of address type will revert. Use OpenZeppelin’s ERC721 contracts.

https://github.com/crytic/slither/wiki/Detector-Documentation#incorrect-erc721-interface

#### **Proof Of Concept**

```solidity
File: contracts/core/RdpxV2Core.sol

638:         IRdpxDecayingBonds(addresses.rdpxDecayingBonds).ownerOf(_bondId) ==

1026:       msg.sender == RdpxV2Bond(addresses.receiptTokenBonds).ownerOf(id),

```

### [L-7] Timestamp Dependence

#### Description:

Contracts often need access to time values to perform certain types of functionality. Values such as block.timestamp, and block.number can give you a sense of the current time or a time delta, however, they are not safe to use for most purposes.

In the case of block.timestamp, developers often attempt to use it to trigger time-dependent events. As Ethereum is decentralized, nodes can synchronize time only to some degree. Moreover, malicious miners can alter the timestamp of their blocks, especially if they can gain advantages by doing so. However, miners cant set a timestamp smaller than the previous one (otherwise the block will be rejected), nor can they set the timestamp too far ahead in the future. Taking all of the above into consideration, developers cant rely on the preciseness of the provided timestamp.

Reference: (https://github.com/kadenzipfel/smart-contract-vulnerabilities/blob/master/vulnerabilities/timestamp-dependence.md)

#### **Proof Of Concept**

```solidity
File: contracts/amo/UniV2LiquidityAmo.sol

231:         block.timestamp + 1

279:         block.timestamp + 1

342:         block.timestamp + 1

```

```solidity
File: contracts/amo/UniV3LiquidityAmo.sol

250:           block.timestamp

```

```solidity
File: contracts/core/RdpxV2Core.sol

502:       maturity: block.timestamp + bondMaturity,

503:       timestamp: block.timestamp

636:       _validate(expiry >= block.timestamp, 2);

1023:     _validate(block.timestamp > bonds[id].maturity, 7);

1103:           block.timestamp + 10

1194:     ).nextFundingPaymentTimestamp() - block.timestamp;

```

```solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol

283:     uint256 timeToExpiry = nextFundingPaymentTimestamp() - block.timestamp;

463:     while (block.timestamp >= nextFundingPaymentTimestamp()) {

508:     lastUpdateTime = block.timestamp;

512:       (currentFundingRate * (block.timestamp - startTime)) / 1e18

516:       (currentFundingRate * (block.timestamp - startTime)) / 1e18

521:       ((currentFundingRate * (block.timestamp - startTime)) / 1e18),

```

```solidity
File: contracts/reLP/ReLPContract.sol

264:       block.timestamp + 10

283:         block.timestamp + 10

294:       block.timestamp + 10

```

### [L-8] Unsafe ERC20 operation(s)

#### Description:

There are ERC20 tokens that charge fee for every transfer() / transferFrom().

#### **Proof Of Concept**

```solidity
File: contracts/amo/UniV2LiquidityAmo.sol

134:     IERC20WithBurn(_token).approve(_spender, _amount);

```

```solidity
File: contracts/amo/UniV3LiquidityAmo.sol

150:       IERC20WithBurn(_token).approve(_target, _amount);

158:     IERC20WithBurn(params._tokenA).transferFrom(

163:     IERC20WithBurn(params._tokenB).transferFrom(

169:     IERC20WithBurn(params._tokenA).approve(

173:     IERC20WithBurn(params._tokenB).approve(

283:     IERC20WithBurn(_tokenA).transferFrom(

```

```solidity
File: contracts/core/RdpxV2Core.sol

339:     IERC20WithBurn(weth).approve(

343:     IERC20WithBurn(weth).approve(addresses.dopexAMMRouter, type(uint256).max);

344:     IERC20WithBurn(weth).approve(addresses.dpxEthCurvePool, type(uint256).max);

345:     IERC20WithBurn(weth).approve(

411:     IERC20WithBurn(_token).approve(_spender, _amount);

```

```solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol

245:       ? collateralToken.approve(

249:       : collateralToken.approve(addresses.perpetualAtlanticVaultLP, 0);

```

```solidity
File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol

106:     collateral.approve(_perpetualAtlanticVault, type(uint256).max);

107:     ERC20(rdpx).approve(_perpetualAtlanticVault, type(uint256).max);

128:     collateral.transferFrom(msg.sender, address(this), assets);

170:     collateral.transfer(receiver, assets);

```

```solidity
File: contracts/reLP/ReLPContract.sol

243:     IERC20WithBurn(addresses.pair).transferFrom(

```
