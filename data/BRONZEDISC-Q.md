## QA
---

### Layout Order [1]

- The best-practices for layout within a contract is the following order: state variables, events, modifiers, constructor and functions.

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol

```solidity
// place this event before the constructor
311:  event LogSetReLpFactor(uint256 _reLPFactor);
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol

```solidity
// place this modifier before the constructor
295:  modifier onlyPerpVault() {
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol

```solidity
// place these before the constructor
368:  event RecoveredERC20(address token, uint256 amount);
369:  event RecoveredERC721(address token, uint256 id);
370:  event LogAssetsTransfered(
379:  event log(uint);
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol

```solidity
// place these before the constructor
387:  event LogAddLiquidity(
398:  event LogRemoveLiquidity(
407:  event LogSwap(
415:  event LogAssetsTransfered(
421:  event LogEmergencyWithdraw(address sender, address[] tokens);
```

---

### Function Visibility [2]

- Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private. Within a grouping, place the view and pure functions last.

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol

```solidity
// place this private function for last
129:  function _mintToken(address to) private returns (uint256 tokenId) {

// place these internal function right before private one
162:  function _beforeTokenTransfer(

// place this external variable all the other public functions
151:  function getBondsOwned(
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol

```solidity
// internal function coming before external ones
160:  function _sendTokensToRdpxV2Core() internal {

```

---

### natSpec missing [3]

Some functions are missing @params or @returns. Specification Format.” These are written with a triple slash (///) or a double asterisk block(/** ... */) directly above function declarations or statements to generate documentation in JSON format for developers and end-users. It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). These comments contain different types of tags:
- @title: A title that should describe the contract/interface @author: The name of the author (for contract, interface) 
- @notice: Explain to an end user what this does (for contract, interface, function, public state variable, event) 
- @dev: Explain to a developer any extra details (for contract, interface, function, state variable, event) 
- @param: Documents a parameter (just like in doxygen) and must be followed by parameter name (for function, event)
- @return: Documents the return variables of a contract’s function (function, public state variable)
- @inheritdoc: Copies all missing tags from the base function and must be followed by the contract name (for function, public state variable)
- @custom…: Custom tag, semantics is application-defined (for everywhere)

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol

```solidity
79:  constructor
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol

```solidity
28:  event Deposit(
35:  event Withdraw(
218:  function _convertToAssets(
231:  function _beforeTokenTransfer(

// @return missing
240:  function activeCollateral() public view returns (uint256) {
245:  function totalCollateral() public view returns (uint256) {
250:  function rdpxCollateral() public view returns (uint256) {
255:  function totalAvailableCollateral() public view returns (uint256) {

// @param and @return missing
262:  function redeemPreview(
269:  function previewDeposit(uint256 assets) public view returns (uint256) {
274:  function convertToShares(
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol

```solidity
// @param missing
237:  function updateFundingDuration(

243:  function setLpAllowance(bool increase) external onlyRole(DEFAULT_ADMIN_ROLE) {

// @return missing
588:  function _mintOptionToken() private returns (uint256 tokenId) {

594:  function _updateFundingRate(uint256 amount) private {
635:  function _beforeTokenTransfer(
645:  function supportsInterface(
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol

```solidity
23:  constructor() ERC20("Dopex Synthetic ETH", "dpxETH") {
29:  function pause() public onlyRole(PAUSER_ROLE) {
33:  function unpause() public onlyRole(PAUSER_ROLE) {
37:  function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {
41:  function burn(
47:  function burnFrom(
55:  function _beforeTokenTransfer(
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol

```solidity
18:  contract RdpxDecayingBonds is
40:  struct Bond {
46:  event BondMinted(
53:  event EmergencyWithdraw(address sender);
56:  constructor(

// @return missing
151:  function getBondsOwned(

162:  function _beforeTokenTransfer(
174:  function supportsInterface(
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol

```solidity
11:  contract RdpxV2Bond is
24:  constructor() ERC721("rDPX V2 Bond", "rDPXV2Bond") {
29:  function pause() public onlyRole(DEFAULT_ADMIN_ROLE) {
33:  function unpause() public onlyRole(DEFAULT_ADMIN_ROLE) {
37:  function mint(
45:  function _beforeTokenTransfer(
57:  function supportsInterface(
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol

```solidity
124:  constructor(address _weth) {
270:  function removeAssetFromtokenReserves(

// @return missing
1051:  function upperDepeg(
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol

```solidity
22:  abstract contract OracleLike {
23:  function read() external view virtual returns (uint);
25:  function uniswapPool() external view virtual returns (address);
28:  contract UniV3LiquidityAMO is AccessControl, ERC721Holder {
40:  struct Position {
50:  struct AddLiquidityParams {
76:  constructor(address _rdpx, address _rdpxV2Core) {
94:  function liquidityInPool(
112:  function numPositions() public view returns (uint256) {
119:  function collectFees() external onlyRole(DEFAULT_ADMIN_ROLE) {
139:  function approveTarget(
155:  function addLiquidity(
213:  function removeLiquidity(
274:  function swap(
313:  function recoverERC20(
324:  function recoverERC721(
339:  function execute(
353:  function _sendTokensToRdpxV2Core(address tokenA, address tokenB) internal {
368:  event RecoveredERC20(address token, uint256 amount);
369:  event RecoveredERC721(address token, uint256 id);
370:  event LogAssetsTransfered(
379:  event log(uint);
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol

```solidity
// @return missing
258:  function removeLiquidity(

387:  event LogAddLiquidity(
398:  event LogRemoveLiquidity(
407:  event LogSwap(
415:  event LogAssetsTransfered(
421:  event LogEmergencyWithdraw(address sender, address[] tokens);
```

---

### State variable and function names [4]

- Variables should be named according to their specifications
- private and internal `variables` should preppend with `underline`
- private and internal `functions` should preppend with `underline`
- constant state variables should be UPPER_CASE

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol

```solidity
// private and internal `functions` should preppend with `underline`
286:  function beforeWithdraw(uint256 assets, uint256 /*shares*/) internal {
```

---

### Initialized variables [5]

- Variables are default initialized with 0 for `uint / int`, 0x0 for `address` and false for `bool`

```solidity
89:  uint256 public latestFundingPaymentPointer = 0;
```
