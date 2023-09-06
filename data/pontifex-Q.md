### L-1 Wrong assets amount for initial condition
It is impossible to burn `shares` if `totalSupply == 0`. So the return value from the `PerpetualAtlanticVaultLP._convertToAssets` should be `(0,0)` for `supply == 0`. 
Then `PerpetualAtlanticVaultLP.redeem` throws "ZERO_ASSETS" instead of EVM underflow error.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L218-L229
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L159-L168

### L-2 Incorrect state handling 
The `reservesIndex[_assetSymbol]` mapping and `reserveTokens` array are incorrectly handled in the `RdpxV2Core.removeAssetFromtokenReserves` function. In case deleting `assetToDelete` at the `indexToDelete == length - 1` the `reservesIndex[assetToDelete]` will return `indexToDelete` instead of `0`. In case of deleting `assetToDelete` with `notTheLastIndex` the `reserveTokens[notTheLastIndex]` will return the `assetToDelete` symbol instead of `assetWithLastIndex`. These issues are not dangerous for the code base but should be fixed.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L270-L290

### L-3 No check for duplicates
Consider checking for duplicates of AMO addresses in the `RdpxV2Core.addAMOAddress`. The impact of this is not defined.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L378-L381

### L-4 Wrong error numbers
The `RdpxV2Core` contains `_validate` calls with wrong error numbers. There are 4 instances:
Should be 15:
```solidity
410    _validate(_amount > 0, 17);

534        ? _validate(
535          ethBalance + _amount <= (ethBalance + dpxEthBalance) / 2,
536          14
537        )
538        : _validate(
539          dpxEthBalance + _amount <= (ethBalance + dpxEthBalance) / 2,
540          14
541        );
```
Should be 13:
```solidity
1167      _validate(bondDiscount < 100e8, 14);
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L410

### L-5 The same name for different structures 
The protocol uses the same name for different structures. It can be confusing.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/IRdpxV2Core.sol#L36-L40
```solidity
36  struct Bond {
37    uint256 amount;
38    uint256 maturity;
39    uint256 timestamp;
40  }
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L40-L44
```solidity
40  struct Bond {
41    address owner;
42    uint256 expiry;
43    uint256 rdpxAmount;
44  }
```

### L-6 Inappropriate validation
The check at the line L1167 of the `RdpxV2Core.calculateBondCost` function can't prevent from EVM underflow revert at the line L1170. The `bondDiscount` should be less than `12.5 * DEFAULT_PRECISION`
```solidity
85  uint256 public constant DEFAULT_PRECISION = 1e8;
88  uint256 public constant RDPX_RATIO_PERCENTAGE = 25 * DEFAULT_PRECISION;

1167      _validate(bondDiscount < 100e8, 14);
1168
1169      rdpxRequired =
1170        ((RDPX_RATIO_PERCENTAGE - (bondDiscount / 2)) *
```
 
### L-7 View function should not revert 
`RdpxV2Core.getReserveTokenInfo` throws if the string in parameters does not match the saved string. I suppose it is necessary due to the `L-2 Incorrect state handling` issue for tests passing, but view functions as a rule should not revert.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1135-L1147 


### L-8 Redundant access check
There are several functions with both the `onlyRole` modifier and the `_isEligibleSender` check. So the protocol owner has to add the contracts with a granted role also to `whitelistedContracts` mapping. Consider removing the `_isEligibleSender` check. There are three instances:
```solidity
320    onlyRole(RDPXV2CORE_ROLE)
321    returns (uint256 ethAmount, uint256 rdpxAmount)
322  {
323    _whenNotPaused();
324    _isEligibleSender();
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L320-L324
```solidity
372  function payFunding() external onlyRole(RDPXV2CORE_ROLE) returns (uint256) {
373    _whenNotPaused();
374    _isEligibleSender();
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L372-L374
```solidity
1085  ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256 dpxEthReceived) {
1086    _isEligibleSender();
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1085-L1086

### L-9 Missed inheritance
Several contracts implement interfaces but do not inherit from them. This behavior is error-prone and might prevent the implementation from following the interface if the code
is updated. There are 2 instances:
`RdpxDecayingBonds` should inherit from `IRdpxDecayingBonds`:
```solidity
18 contract RdpxDecayingBonds is
19   ERC721,
20   ERC721Enumerable,
21   ERC721Burnable,
22   Pausable,
23   AccessControl
{
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L18-L24
`ReLPContract` should inherit from `IReLP`:
```solidity
25 contract ReLPContract is AccessControl {
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L25

### L-10 Use more specific names for variables
The `ReLPContract.setAddresses` function receives token addresses which can be easily mixed up. Consider using more specific names for variables.
```solidity
31    // token A address
32    address tokenA; // rdpx
33    // token B address
34    address tokenB; // weth

115  function setAddresses(
116    address _tokenA,
117    address _tokenB,
```

### L-11 Inaccurate comments
There are 3 instances of using inaccurate comments.
The comment `swap token A for token B` should be `swap token1 for token2`:
```solidity
335    // swap token A for token B
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L335
The comment at the lines L310-311 of the `UniV3LiquidityAMO` contract should be put before other functions which are also closed by modifier:
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L310-L311
The comment at the line L259 should be put at the line L237 before other public view functions of the `PerpetualAtlanticVaultLP` contract.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L237-L259


### L-12 Lack of `address(0)` check
There are 2 instances of `address(0)` check absence at the `PerpetualAtlanticVaultLP` contract:
```solidity
118    address receiver

147    address receiver,
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L120

### L-13 Redundant calculation
The `PerpetualAtlanticVault.calculateFunding` calculates `timeToExpiry` in a for-loop. But for any conditions the result of the calculation will be exactly `fundingDuration`. This could be a potentially wrong calculations, the [documentation](https://dopex.notion.site/rDPX-V2-RI-b45b5b402af54bcab758d62fb7c69cb4#c13cd86d4f054eec8f8d945596482b51) says `The APP contract follows a epoch system where each epoch is 1 week long. ... The funding is calculated based on the amount of options active for a given strike using BlackScholes with the duration as the start of the epoch until the end of the epoch.` So the result is correct. 
Proof of concept: `uint256 timeToExpiry = nextFundingPaymentTimestamp() - (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration)) = genesis + latestFundingPaymentPointer * fundingDuration - genesis - (latestFundingPaymentPointer - 1) * fundingDuration = latestFundingPaymentPointer * fundingDuration - (latestFundingPaymentPointer - 1) * fundingDuration = latestFundingPaymentPointer * fundingDuration - latestFundingPaymentPointer * fundingDuration + 1 * fundingDuration = fundingDuration`.
This issue is in both the QA and GAS categories.
```solidity
426      uint256 timeToExpiry = nextFundingPaymentTimestamp() -
427        (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration));
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L426-L427


### N-1 Typos
There are 12 instances:
The word `perpetualatlanticvault` should be `perpetualAtlanticVault`.
```solidity
51  /// @dev The symbol reperesenting the underlying asset of the perpetualatlanticvault lp

54  /// @dev The symbol representing the collateral token of the perpetualatlanticvault lp

77   * @param _collateral The address of the collateral asset in the perpetualatlanticvault contract
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L51
The word `LogAssetsTransfered` should be `LogAssetsTransferred`.
```solidity
177    emit LogAssetsTransfered(msg.sender, tokenABalance, tokenBBalance);

415  event LogAssetsTransfered(
```
The word `Uniswamp` should be `Uniswap`.
The word `LogAssetsTransfered` should be `LogAssetsTransferred`.
```solidity
11 // Uniswamp V3

363    emit LogAssetsTransfered(tokenABalance, tokenBBalance, tokenA, tokenB);

370  event LogAssetsTransfered(    
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L11
The word `Gensis` should be `Genesis`.
The word `_gensis` should be `_genesis`.
The word `a option` should be `an option`.
```solidity
112  /// @param _gensis Gensis time for funding calculation

117    uint256 _gensis

124    genesis = _gensis;

587  /// @dev Internal function to mint a option token
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L112


### N-2 Function names should be in mixedCase
According to the Solidity style guide function names and variables should be in mixedCase (lowerCamelCase). Events names should be in CamelCase. There are 10 instances:
The name `setreLpFactor` should be `setReLpFactor`.
The name `mintokenAAmount` should be `minTokenAAmount`.
The name `mintokenBAmount` should be `minTokenBAmount`.
```solidity
90  function setreLpFactor(

250    uint256 mintokenAAmount = tokenAToRemove -

252    uint256 mintokenBAmount = ((tokenAToRemove * tokenAInfo.tokenAPrice) /
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L90
The name `setIsreLP` should be `setIsReLP`.
The name `LogSetputOptionsRequired` should be `LogSetPutOptionsRequired`.
The name `addAssetTotokenReserves` should be `addAssetToTokenReserves`.
The name `LogAssetAddedTotokenReserves` should be `LogAssetAddedToTokenReserves`.
The name `removeAssetFromtokenReserves` should be `removeAssetFromTokenReserves`.
The name `LogAssetRemovedFromtokenReserves` should be `LogAssetRemovedFromTokenReserves`.
The name `minamountOfWeth` should be `minAmountOfWeth`.
```solidity
206  function setIsreLP(bool _isReLPActive) external onlyRole(DEFAULT_ADMIN_ROLE) {

220    emit LogSetputOptionsRequired(_putOptionsRequired);

240  function addAssetTotokenReserves(

263    emit LogAssetAddedTotokenReserves(_asset, _assetSymbol);

270  function removeAssetFromtokenReserves(

289    emit LogAssetRemovedFromtokenReserves(_assetSymbol, index);

1077   * @param minamountOfWeth min ETH token receive when swapping rdpx to ETH token

1083    uint256 minamountOfWeth,
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L206


### N-3 Redundant contract inheritance
There are four instances:
The `DpxEthToken` contract is already inherited `ERC20` through the parent contract `ERC20Burnable`.
```solidity
12 contract DpxEthToken is
13   ERC20,
14   ERC20Burnable,
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/dpxETH/DpxEthToken.sol#L12
The `RdpxV2Bond` contract is already inherited `ERC721` through the parent contracts `ERC721Enumerable` and `ERC721Burnable`.
```solidity
11 contract RdpxV2Bond is
12   ERC721,
13   ERC721Enumerable,
14   Pausable,
15   AccessControl,
16   ERC721Burnable
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L11
The `RdpxDecayingBonds` contract is already inherited `ERC721` through the parent contracts `ERC721Enumerable` and `ERC721Burnable`.
```solidity
18 contract RdpxDecayingBonds is
19   ERC721,
20   ERC721Enumerable,
21   ERC721Burnable,
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L18
The `PerpetualAtlanticVault` contract is already inherited `ERC721` through the parent contracts `ERC721Enumerable` and `ERC721Burnable`.
```solidity
28 contract PerpetualAtlanticVault is
29   IPerpetualAtlanticVault,
30   ReentrancyGuard,
31   Pausable,
32   ERC721,
33   ERC721Enumerable,
34   ERC721Burnable,
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L28

