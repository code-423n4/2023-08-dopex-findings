# Gas Optimizations

| Number | Issue | Instances | Total Gas Saved |
|--------|-------|-----------|-----------------|
|[G-01]| State variables which are not modified within functions should be set as constant or immutable for values set at deployment  | 7 | 70000  |
|[G-02]| create variable  immutable and avoid external call to save gas  | 8 |   |
|[G-03]| Avoid emitting storage values  | 4 |   |
|[G-04]| bytes constants are more eficient than string constans  | 13 |   |
|[G-05]| Use assembly to hash values more efficiently  | 9 |   |
|[G-06]| Use assembly to make more efficient back-to-back calls  | 14 |   |
|[G-07]| Use assembly for loops  | 9 |   |
|[G-08]| Use assembly to grab and cast value in byte array  | 1 |   |
|[G-09]| Don’t cache value if it is only used once  | 2 |   |
|[G-10]| Use calldata instead of memory for immutable arguments | 5 | 1500  |
|[G-11]| Use != 0 instead of > 0 for unsigned integer comparison  | 19 | 57  |
|[G-12]| Don’t initialize variables with default value  | 16 | 96  |
|[G-13]| Use assembly to check for address(0)  | 44 | 264  |
|[G-14]| Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead  | 4 | 40  |
|[G-15]| Use hardcode address instead address(this)  | 40 |   |
|[G-16]| require() or revert() statements that check input arguments should be at the top of the function (Also restructured some “if’s”)  | 4 |   |
|[G-17]| Use assembly to validate msg.sender  | 2 |   |
|[G-18]| Cache external calls outside of loop to avoid re-calling function on each iteration  | 3 |   |
|[G-19]| Return values from external calls can be cached to avoid unnecessary call  | 11 | 39281  |
|[G-20]| Move storage pointer to top of function to avoid offset calculation  | 2 | 180  |
|[G-21]| Cache storage values in memory to minimize SLOADs  | 1 | 110  |
|[G-22]| Public Functions not called by they contract should be declear External  | 29 |   |
|[G-23]| Using storage instead of memory for structs/arrays saves gas  | 3 | 12600  |
|[G-24]| Multiple accesses of a mapping/array should use a storage pointer  | 7 | 280  |
|[G-25]| Using unchecked blocks for addation and subtruction to save gas  | 24 | 2040  |
|[G-26]| Save loop calls  | 2 |   |
|[G-27]| Do not calculate constants  | 2 |   |
|[G-28]| Using delete statement can save gas | 2 |  |
|[G-29]| Amounts should be checked for 0 before calling a transfer  | 1 |   |
|[G-30]| Don’t apply the same value to state variables  | 8 | 16800  |
|[G-31]| Inverting the condition of an if-else-statement wastes gas  | 8 | 24  |
|[G-32]| Assigning keccak operations to constant variables results in extra gas costs  | 9 |   |
|[G-33]| Make 3 event parameters indexed when possible  | 5 | 2647  |
|[G-34]| With assembly, .call (bool success) transfer can be done gas-optimized  | 2 |   |
|[G-35]| Use constants instead of type(uintx).max  | 17 |   |
|[G-36]| Use uint256(1)/uint256(2) instead for true and false boolean states  | 6 | 102600  |
|[G-37]| Not using the named return variable when a function returns, wastes deployment gas  | 34 |   |
|[G-38]| Can make the variable outside the loop to save gas  | 4 |   |
|[G-39]| Use functions instead of modifiers  | 1 |   |
|[G-40]| Create immutable variable to avoid an external call | 3 |   |
|[G-41]| Use a do while loop instead of a for loop  | 7 |   |
|[G-42]| Unused variables can be omitted  | 2 |   |

## [G-01] State variables which are not modified within functions should be set as constant or immutable for values set at deployment

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVault.sol

51    string public underlyingSymbol;

60    uint256 public collateralPrecision;

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L51>

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVaultLP.sol

49     ERC20 public collateral;

52     string public underlyingSymbol;

55     string public collateralSymbol;

67     address public rdpx;

70     address public rdpxRdpxV2Core;

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L49>

## [G-02] create variable  immutable and avoid external call to save gas

### create this perpetualAtlanticVault and collateral variables immutable

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVaultLP.sol

46    IPerpetualAtlanticVault public perpetualAtlanticVault;

49    ERC20 public collateral;

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L46>

### create addresses and pricingOracleAddresses variables immutable

```solidity
file:  contracts/core/RdpxV2Core.sol

47     Addresses public addresses;

50     PricingOracleAddresses public pricingOracleAddresses;

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L47>

### create collateralToken variable immutable

```solidity
file:   contracts/perp-vault/PerpetualAtlanticVault.sol

57    IERC20WithBurn public collateralToken;

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L57>

### create univ3_factory,public univ3_positions and univ3_router variable immutable

```solidity
file:   contracts/amo/UniV3LiquidityAmo.sol

35      IUniswapV3Factory public univ3_factory;

36      INonfungiblePositionManager public univ3_positions;

37      ISwapRouter public univ3_router;

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L35>

## [G-03] Avoid emitting storage values

### to avoid emitting they positions_array and positions_mapping first we get the result of this variable and after this emit this memory variable

```solidity
file:  contracts/amo/UniV3LiquidityAmo.sol

268    emit log(positions_array.length);

269    emit log(positions_mapping[pos.token_id].token_id);

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol##L268>

### first get they result of they addresses and delegates.length variable and emit the memroy variable

```solidity
file:  contracts/core/RdpxV2Core.sol

349    emit LogSetAddresses(addresses);

966     emit LogAddToDelegate(_amount, _fee, delegates.length - 1);

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L349>

## [G-04] bytes constants are more eficient than string constans

If the data can fit in 32 bytes, the bytes32 data type can be used instead of bytes or strings, as it is less robust in terms of robustness.

```solidity
file:  contracts/core/RdpxV2Core.sol

70    string[] public reserveTokens;

242   string memory _assetSymbol

271   string memory _assetSymbol

1136  string memory _token

1137  ) public view returns (address, uint256, string memory) {

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L70>

```solidity
file: contracts/decaying-bonds/RdpxDecayingBonds.sol

57    string memory _name,

58    string memory _symbol

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L57>

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVault.sol

51     string public underlyingSymbol;

114    string memory _name,

115    string memory _symbol,

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L51>

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVaultLP.sol

52     string public underlyingSymbol;

55     string public collateralSymbol;

86     string memory _collateralSymbol

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L52>

## [G-05] Use assembly to hash values more efficiently

In the instances below, we can hash values more efficiently by using the least amount of opcodes possible via assembly.

```solidity
file:  contracts/core/RdpxV2Bond.sol

22    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L22>

```solidity
file:  contracts/decaying-bonds/RdpxDecayingBonds.sol

31     bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

34     bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L31>

```solidity
file:  contracts/dpxETH/DpxEthToken.sol

19      bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");

20      bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

21      bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L19>

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVault.sol

45     bytes32 public constant MANAGER_ROLE = keccak256("MANAGER_ROLE");

48     bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L45>

```solidity
file:  contracts/reLP/ReLPContract.sol

70     bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L70>

## [G-06] Use assembly to make more efficient back-to-back calls

In the instance below, two external calls, both of which take two function parameters, are performed. We can potentially avoid memory expansion costs by using assembly to utilize scratch space + free memory pointer memory offsets for the function calls. We will use the same memory space for both function calls.

Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls.

### use assembly for IPerpetualAtlanticVault

```solidity
file:    contracts/core/RdpxV2Core.sol

796        uint256 pointer = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
      .latestFundingPaymentPointer();
    _validate(fundingPaidFor[pointer] == false, 16);

    fundingAmount = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
      .payFunding();

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L796-L801>

```solidity
file:   contracts/core/RdpxV2Core.sol

343    IERC20WithBurn(weth).approve(addresses.dopexAMMRouter, type(uint256).max);

344    IERC20WithBurn(weth).approve(addresses.dpxEthCurvePool, type(uint256).max);

345    IERC20WithBurn(weth).approve( addresses.rdpxV2ReceiptToken,

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L343-L345>

### use assembly for mulDivDown

```solidity
file:   contracts/perp-vault/PerpetualAtlanticVaultLP.sol

226      shares.mulDivDown(totalCollateral(), supply),

227      shares.mulDivDown(_rdpxCollateral, supply)

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L226-L227>

### use assembly for safeApprove

```solidity
file:  contracts/reLP/ReLPContract.sol

150       IERC20WithBurn(addresses.pair).safeApprove(
      addresses.ammRouter,
      type(uint256).max
    );

    IERC20WithBurn(addresses.tokenA).safeApprove(
      addresses.ammRouter,
      type(uint256).max
    );

    IERC20WithBurn(addresses.tokenB).safeApprove(
      addresses.ammRouter,
      type(uint256).max
    );

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L150-L163>

### use assembly for balanceOf

```solidity
file:   contracts/amo/UniV3LiquidityAmo.sol

354      uint256 tokenABalance = IERC20WithBurn(tokenA).balanceOf(address(this));

355      uint256 tokenBBalance = IERC20WithBurn(tokenB).balanceOf(address(this));

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L354-L355>

### use assembly for safeTransfer

```solidity
file:  contracts/amo/UniV3LiquidityAmo.sol

357    IERC20WithBurn(tokenA).safeTransfer(rdpxV2Core, tokenABalance);

358    IERC20WithBurn(tokenB).safeTransfer(rdpxV2Core, tokenBBalance);

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L357-L358>

```solidity
file:   contracts/amo/UniV3LiquidityAmo.sol

253     univ3_positions.decreaseLiquidity(decreaseLiquidityParams);

254     univ3_positions.collect(collect_params);

255     univ3_positions.burn(pos.token_id);

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L253-L255>

## [G-07] Use assembly for loops

In the following instances, assembly is used for more gas efficient loops. The only memory slots that are manually used in the loops are scratch space (0x00-0x20), the free memory pointer (0x40), and the zero slot (0x60). This allows us to avoid using the free memory pointer to allocate new memory, which may result in memory expansion costs.

Note that in order to do this optimization safely we will need to cache and restore the free memory pointer after the loop. We will also set the zero slot (0x60) back to 0.

```solidity
file:  contracts/amo/UniV2LiquidityAmo.sol

147    for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L147-L150>

```solidity
file:  contracts/core/RdpxV2Core.sol

167      for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }

246  for (uint256 i = 1; i < reserveAsset.length; i++) {
      require(
        reserveAsset[i].tokenAddress != _asset,
        "RdpxV2Core: asset already exists"
      );
    }

775  for (uint256 i = 0; i < optionIds.length; i++) {
      optionsOwned[optionIds[i]] = false;
    }

996    for (uint256 i = 1; i < reserveAsset.length; i++) {
      uint256 balance = IERC20WithBurn(reserveAsset[i].tokenAddress).balanceOf(
        address(this)
      );

      if (weth == reserveAsset[i].tokenAddress) {
        balance = balance - totalWethDelegated;
      }
      reserveAsset[i].tokenBalance = balance;
    }

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L167-L170>

```solidity
file:  contracts/amo/UniV3LiquidityAmo.sol
 
120     for (uint i = 0; i < positions_array.length; i++) {
      Position memory current_position = positions_array[i];
      INonfungiblePositionManager.CollectParams
        memory collect_params = INonfungiblePositionManager.CollectParams(
          current_position.token_id,
          rdpxV2Core,
          type(uint128).max,
          type(uint128).max
        );
```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L120-L128>

```solidity
file:  contracts/decaying-bonds/RdpxDecayingBonds.sol

103   for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }

156     for (uint256 i; i < ownerTokenCount; i++) {
      tokenIds[i] = tokenOfOwnerByIndex(_address, i);
    }

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L103-L106>

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVault.sol

225     for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }
```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L225-L228>

## [G-08] Use assembly to grab and cast value in byte array

Like various similar functions, i.e. payFunding, assembly can be used to grab the value at a specified index of a byte array and cast that value to a specific uint type. In the diff below, a check is done before this operation to ensure that the offset is not out of bounds.

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVault.sol

395   return (totalFundingForEpoch[latestFundingPaymentPointer]);

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L395>

## [G-09] Don’t cache value if it is only used once

If a value is only intended to be used once then it should not be cached. Caching the value will result in unnecessary stack manipulation.

### don't cache they positions_array[i]; after they for loop cache they positions_array[i]; before they for loop to save gas or don't cache in this palace becouse after the for loop the positions_array[i]; variable is used only once chaing positions_array[i]; varible after they for loop use mor gas

```solidity
file:  contracts/amo/UniV3LiquidityAmo.sol

119     function collectFees() external onlyRole(DEFAULT_ADMIN_ROLE) {
    for (uint i = 0; i < positions_array.length; i++) {
      Position memory current_position = positions_array[i];
      INonfungiblePositionManager.CollectParams
        memory collect_params = INonfungiblePositionManager.CollectParams(
          current_position.token_id,
          rdpxV2Core,
          type(uint128).max,
          type(uint128).max
        );

      // Send to custodian address
      univ3_positions.collect(collect_params);
    }
  }

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L119-L133>

### do not cache IPerpetualAtlanticVault(addresses.perpetualAtlanticVault).latestFundingPaymentPointer(); in pointer variable because they variable is not used inside they function

```solidity
file:  contracts/core/RdpxV2Core.sol

790     function provideFunding()
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 fundingAmount)
  {
    _whenNotPaused();
    uint256 pointer = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
      .latestFundingPaymentPointer();
    _validate(fundingPaidFor[pointer] == false, 16);

    fundingAmount = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
      .payFunding();

    reserveAsset[reservesIndex["WETH"]].tokenBalance -= fundingAmount;

    fundingPaidFor[pointer] = true;

    emit LogProvideFunding(pointer, fundingAmount);
  }

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L790-808>

## [G-10] Use calldata instead of memory for immutable arguments

Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.

```solidity
file:  contracts/core/RdpxV2Core.sol

764     function settle(
    uint256[] memory optionIds
  )
  external
819     function bondWithDelegate(
    address _to,
    uint256[] memory _amounts,
    uint256[] memory _delegateIds,
    uint256 rdpxBondId
  ) public returns (uint256 receiptTokenAmount, uint256[] memory) {

1135    function getReserveTokenInfo(
    string memory _token
  ) public view returns (address, uint256, string memory) {

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L764-L767>

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVault.sol

315   function settle(
    uint256[] memory optionIds
  )
   external

405   function calculateFunding(
    uint256[] memory strikes
  ) external nonReentrant returns (uint256 fundingAmount) {

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L315-L318>

## [G-11] Use != 0 instead of > 0 for unsigned integer comparison

```solidity
file:  contracts/amo/UniV2LiquidityAmo.sol

113   _slippageTolerance > 0,

133   require(_amount > 0, "reLPContract: amount must be greater than 0");

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L113>

```solidity
file:  contracts/core/RdpxV2Core.sol

183   _validate(_rdpxBurnPercentage > 0, 3);

196   _validate(_rdpxFeePercentage > 0, 3);

231   _validate(_bondMaturity > 0, 3);

410   _validate(_amount > 0, 17);

444   _validate(_bondDiscountFactor > 0, 3);

458   _validate(_slippageTolerance > 0, 3);

556   minAmount > 0 ? minAmount : minOut

838  _validate(_amounts[i] > 0, 4);

901  _validate(_amount > 0, 4);

984  _validate(amountWithdrawn > 0, 15);

1021  _validate(bonds[id].timestamp > 0, 6);

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L183>

```solidity
file:   contracts/perp-vault/PerpetualAtlanticVault.sol

265    _validate(amount > 0, 2);

414    _validate(optionsPerStrike[strikes[i]] > 0, 4);

547    _price > 0 ? _price : getUnderlyingPrice(),

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L265>

```solidity
file:  contracts/reLP/ReLPContract.sol

94    _reLPFactor > 0,

175   _liquiditySlippageTolerance > 0,

190   _slippageTolerance > 0,

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L94>

## [G-12] Don’t initialize variables with default value

In such cases, initializing the variables with default values would be unnecessary and can be considered a waste of gas. Additionally, initializing variables with default values can sometimes lead to unnecessary storage operations, which can increase gas costs. For example, if you have a large array of variables, initializing them all with default values can result in a lot of unnecessary storage writes, which can increase the gas costs of your contract.

```solidity
file:  contracts/core/RdpxV2Core.sol

135    putOptionsRequired = true;

485    optionsOwned[optionId] = true;

776     optionsOwned[optionIds[i]] = false;

805    fundingPaidFor[pointer] = true;

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L135>

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVault.sol

89     uint256 public latestFundingPaymentPointer = 0;

225    for (uint256 i = 0; i < tokens.length; i++) {

328    for (uint256 i = 0; i < optionIds.length; i++) {

343    optionPositions[optionIds[i]].strike = 0;

413    for (uint256 i = 0; i < strikes.length; i++) {

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L89>

```solidity
file:  contracts/amo/UniV2LiquidityAmo.sol

147    for (uint256 i = 0; i < tokens.length; i++) {

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L147>

```solidity
file:  contracts/amo/UniV3LiquidityAmo.sol

120   for (uint i = 0; i < positions_array.length; i++) {

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L120>

```solidity
file:  contracts/core/RdpxV2Core.sol

167    for (uint256 i = 0; i < tokens.length; i++) {

277    reservesIndex[_assetSymbol] = 0;

775    for (uint256 i = 0; i < optionIds.length; i++) {

836    for (uint256 i = 0; i < _amounts.length; i++) {

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L167>

```solidity
file:   contracts/decaying-bonds/RdpxDecayingBonds.sol

103    for (uint256 i = 0; i < tokens.length; i++) {

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L103>

## [G-13] Use assembly to check for address(0)

Using assembly to check for the zero address can result in significant gas savings compared to using a Solidity expression; especially if the check is performed frequently or in a loop. However, it’s important to note that using assembly can make the code less readable and harder to maintain, so it should be used judiciously and with caution.

```solidity
file:   contracts/amo/UniV2LiquidityAmo.sol

84      _tokenA != address(0) &&

85      _tokenB != address(0) &&

86      _pair != address(0) &&

87      _rdpxV2Core != address(0) &&

88      _rdpxOracle != address(0) &&

89      _ammFactory != address(0) &&

90      _ammRouter != address(0),

131     require(_token != address(0), "reLPContract: token cannot be 0");

132     require(_spender != address(0), "reLPContract: spender cannot be 0");

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L84>

```solidity
file:   contracts/core/RdpxV2Core.sol

130     tokenAddress: address(0),

244     require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");

316     _validate(_dopexAMMRouter != address(0), 17);

317     _validate(_dpxEthCurvePool != address(0), 17);

318     _validate(_rdpxDecayingBonds != address(0), 17);

319     _validate(_perpetualAtlanticVault != address(0), 17);

320     _validate(_perpetualAtlanticVaultLP != address(0), 17);

321     _validate(_rdpxReserve != address(0), 17);

322     _validate(_rdpxV2ReceiptToken != address(0), 17);

323     _validate(_feeDistributor != address(0), 17);

324     _validate(_reLPContract != address(0), 17);

325     _validate(_receiptTokenBonds != address(0), 17);

362     _validate(_rdpxPriceOracle != address(0), 17);

363     _validate(_dpxEthPriceOracle != address(0), 17);

379    _validate(_addr != address(0), 17);

408    _validate(_token != address(0), 17);

409    _validate(_spender != address(0), 17);
```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L130>

```solidity
file:   contracts/perp-vault/PerpetualAtlanticVault.sol

119     _validate(_collateralToken != address(0), 1);

190     _validate(_optionPricing != address(0), 1);

120     _validate(_assetPriceOracle != address(0), 1);

121     _validate(_volatilityOracle != address(0), 1);

122     _validate(_feeDistributor != address(0), 1);

123     _validate(_rdpx != address(0), 1);

124     _validate(_perpetualAtlanticVaultLP != address(0), 1);

125     _validate(_rdpxV2Core != address(0), 1);

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L119>

```solidity
file:   contracts/perp-vault/PerpetualAtlanticVaultLP.sol

95     _perpetualAtlanticVault != address(0) || _rdpx != address(0),

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L95>

```solidity
file:   contracts/reLP/ReLPContract.sol

127        _tokenA != address(0) &&

128        _tokenB != address(0) &&

129        _pair != address(0) &&

130        _rdpxV2Core != address(0) &&

131        _tokenAReserve != address(0) &&

132        _amo != address(0) &&

133        _rdpxOracle != address(0) &&

134        _ammFactory != address(0) &&

135        _ammRouter != address(0),

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L127>

## [G-14] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

```solidity
file: contracts/amo/UniV3LiquidityAmo.sol

46    uint24 fee_tier;

55    uint24 _fee;

98    uint24 _fee

277   uint24 _fee_tier,

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L46>

## [G-15] Use hardcode address instead address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.

```solidity
file:   contracts/amo/UniV2LiquidityAmo.sol

149     token.safeTransfer(msg.sender, token.balanceOf(address(this)));

162     address(this)

165     address(this)

212     address(this)

217     address(this)

230     address(this)

278     address(this)

323     address(this)

341     address(this)

363     lpTokenBalance = IERC20WithBurn(addresses.pair).balanceOf(address(this));

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#149>

```solidity
file:  contracts/amo/UniV3LiquidityAmo.sol

106    keccak256(abi.encodePacked(address(this), _tickLower, _tickUpper))

160    address(this),

165    address(this),

189    address(this),

294    address(this),

331    address(this),

354    uint256 tokenABalance = IERC20WithBurn(tokenA).balanceOf(address(this));

355    uint256 tokenBBalance = IERC20WithBurn(tokenB).balanceOf(address(this));

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L106>

```solidity
file:  contracts/core/RdpxV2Core.sol

169    token.safeTransfer(msg.sender, token.balanceOf(address(this)));

483    ).purchase(_amount, address(this));

573    address(this),

654   .safeTransferFrom(msg.sender, address(this), _rdpxAmount);

911    address(this),

953    IERC20WithBurn(weth).safeTransferFrom(msg.sender, address(this), _amount);

998    address(this)

1060   address(this)

1102   address(this)

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L169>

```solidity
file:  contracts/decaying-bonds/RdpxDecayingBonds.sol

105    token.safeTransfer(msg.sender, token.balanceOf(address(this)));

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L105>

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVault.sol

227    token.safeTransfer(msg.sender, token.balanceOf(address(this)));

289    collateralToken.safeTransferFrom(msg.sender, address(this), premium);

384    address(this),

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L227>

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVaultLP.sol

128    collateral.transferFrom(msg.sender, address(this), assets);

192    collateral.balanceOf(address(this)) >= _totalCollateral + proceeds,

201    collateral.balanceOf(address(this)) == _totalCollateral - loss,

210    IERC20WithBurn(rdpx).balanceOf(address(this)) >= _rdpxCollateral + amount,

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L128>

```solidity
file:  contracts/reLP/ReLPContract.sol

245   address(this),

263   address(this),

282   address(this),

293   address(this),

304   IERC20WithBurn(addresses.tokenA).balanceOf(address(this))

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L245>

## [G-16] require() or revert() statements that check input arguments should be at the top of the function (Also restructured some “if’s”)

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting alot of gas in a function that may ultimately revert in the unhappy case.

### use if statment before _whenPaused()

```solidity
file:  contracts/decaying-bonds/RdpxDecayingBonds.sol

89      function emergencyWithdraw(
    address[] calldata tokens,
    bool transferNative,
    address payable to,
    uint256 amount,
    uint256 gas
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _whenPaused();
    if (transferNative) {
      (bool success, ) = to.call{ value: amount, gas: gas }("");
      require(success, "RdpxReserve: transfer failed");
    }

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L89-L100>

### use if statment before perpetualAtlanticVault.updateFunding()

```slidity
file:  contracts/perp-vault/PerpetualAtlanticVaultLP.sol

145      function redeem(
    uint256 shares,
    address receiver,
    address owner
  ) public returns (uint256 assets, uint256 rdpxAmount) {
    perpetualAtlanticVault.updateFunding();

    if (msg.sender != owner) {
      uint256 allowed = allowance[owner][msg.sender]; // Saves gas for limited approvals.

      if (allowed != type(uint256).max) {
        allowance[owner][msg.sender] = allowed - shares;
      }
    }

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L145-L158>

### use require statment before _whenNotPaused()

```solidity
file:   contracts/decaying-bonds/RdpxDecayingBonds.sol

114      function mint(
    address to,
    uint256 expiry,
    uint256 rdpxAmount
  ) external onlyRole(MINTER_ROLE) {
    _whenNotPaused();
    require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
    uint256 bondId = _mintToken(to);
    bonds[bondId] = Bond(to, expiry, rdpxAmount);

    emit BondMinted(to, bondId, expiry, rdpxAmount);
  }

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L114-L125>

### use if statment in they top of they function

```solidity
file:  contracts/core/RdpxV2Core.sol

1080      function lowerDepeg(
    uint256 _rdpxAmount,
    uint256 _wethAmount,
    uint256 minamountOfWeth,
    uint256 minOut
  ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256 dpxEthReceived) {
    _isEligibleSender();
    _validate(getDpxEthPrice() < 1e8, 13);

    uint256 amountOfWethOut;

    if (_rdpxAmount != 0) {
      address[] memory path;
      path = new address[](2);
      path[0] = reserveAsset[reservesIndex["RDPX"]].tokenAddress;
      path[1] = weth;

      amountOfWethOut = IUniswapV2Router(addresses.dopexAMMRouter)
        .swapExactTokensForTokens(
          _rdpxAmount,
          minamountOfWeth,
          path,
          address(this),
          block.timestamp + 10
        )[path.length - 1];

      reserveAsset[reservesIndex["RDPX"]].tokenBalance -= _rdpxAmount;
    }

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1080-L1107>

## [G-17] Use assembly to validate msg.sender

We can use assembly to efficiently validate msg.sender for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVaultLP.sol

152   if (msg.sender != owner) {

296       require(
      msg.sender == address(perpetualAtlanticVault),
      "Only the perp vault can call this function"
    );

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L152>

## [G-18] Cache external calls outside of loop to avoid re-calling function on each iteration

Performing STATICCALLs that do not depend on variables incremented in loops should always try to be avoided within the loop. In the following instances, we are able to cache the external calls outside of the loop to save a STATICCALL (100 gas) per loop iteration.

### Cache IERC20WithBurn() ann token.safeTransfer outside of loop to save 1 STATICCALL per loop iteration

```solidity
file:  contracts/core/RdpxV2Core.sol

996       for (uint256 i = 1; i < reserveAsset.length; i++) {
      uint256 balance = IERC20WithBurn(reserveAsset[i].tokenAddress).balanceOf(
        address(this)
      );

      if (weth == reserveAsset[i].tokenAddress) {
        balance = balance - totalWethDelegated;
      }
      reserveAsset[i].tokenBalance = balance;
    }

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L996-L1005>

### Cache token.balanceOf() ann token.safeTransfer outside of loop to save 1 STATICCALL per loop iteration

```solidity
file:  contracts/amo/UniV2LiquidityAmo.sol

147     for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L147-L150>

### Cache token.balanceOf() ann token.safeTransfer outside of loop to save 1 STATICCALL per loop iteration

```solidity
file:  contracts/core/RdpxV2Core.sol

167   for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L167-L170>

## [G-19] Return values from external calls can be cached to avoid unnecessary call

External calls are expensive as they use the STATICCALL/CALL opcode (~100 gas). If you are calling the same external function more than once you should cache the return value to avoid an unnecessary STATICCALL/CALL.

```solidity
file:  contracts/core/RdpxV2Core.sol

411   IERC20WithBurn(_token).approve(_spender, _amount);

953   IERC20WithBurn(weth).safeTransferFrom(msg.sender, address(this), _amount);

987   IERC20WithBurn(weth).safeTransfer(msg.sender, amountWithdrawn);

1097  amountOfWethOut = IUniswapV2Router(addresses.dopexAMMRouter)

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L411>

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVault.sol

227   token.safeTransfer(msg.sender, token.balanceOf(address(this)));

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L227>

```solidity
file:  contracts/reLP/ReLPContract.sol

298    IERC20WithBurn(addresses.pair).safeTransfer(addresses.amo, lp);

302   IERC20WithBurn(addresses.tokenA).safeTransfer(
      addresses.rdpxV2Core,
      IERC20WithBurn(addresses.tokenA).balanceOf(address(this))
    );

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L298>

```solidity
file:  contracts/amo/UniV2LiquidityAmo.sol

149    token.safeTransfer(msg.sender, token.balanceOf(address(this)));

321     IERC20WithBurn(token1).safeTransferFrom(
      addresses.rdpxV2Core,
      address(this),
      token1Amount
    );
  
328  IERC20WithBurn(token1).safeApprove(addresses.ammRouter, token1Amount);

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L149>

```solidity
file:  contracts/amo/UniV2LiquidityAmo.sol

372   function getLpTokenBalanceInWeth() external view returns (uint256) {
    return (lpTokenBalance * getLpPrice()) / 1e8;
  }

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L372-L374>

## [G-20] Move storage pointer to top of function to avoid offset calculation

We can avoid unnecessary offset calculations by moving the storage pointer to the top of the function.

### Move Delegate storage delegate = delegates[delegateId]; storage pointer to top to save gas 90

```solidity
file:  contracts/core/RdpxV2Core.sol

699     function _bondWithDelegate(
    uint256 _amount,
    uint256 rdpxBondId,
    uint256 delegateId
  ) internal returns (BondWithDelegateReturnValue memory returnValues) {
    // Compute the bond cost
    (uint256 rdpxRequired, uint256 wethRequired) = calculateBondCost(
      _amount,
      rdpxBondId
    );

    // update ETH token reserve
    reserveAsset[reservesIndex["WETH"]].tokenBalance += wethRequired;

    Delegate storage delegate = delegates[delegateId];

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L699-L713>

### Move Delegate storage delegate = delegates[delegateId]; storage pointer to top to save gas 90

```solidity
file:   contracts/core/RdpxV2Core.sol

975      function withdraw(
    uint256 delegateId
  ) external returns (uint256 amountWithdrawn) {
    _whenNotPaused();
    _validate(delegateId < delegates.length, 14);
    Delegate storage delegate = delegates[delegateId];
    _validate(delegate.owner == msg.sender, 9);

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L975-L981>

## [G-21] Cache storage values in memory to minimize SLOADs

The code can be optimized by minimizing the number of SLOADs.

### RdpxV2Core.sol._curveSwap: dpxEthCurvePool should be cached(saves 110 gas on average)

```solidity
file:  contracts/core/RdpxV2Core.sol

528   if (validate) {
      uint256 ethBalance = IStableSwap(addresses.dpxEthCurvePool).balances(a);
      uint256 dpxEthBalance = IStableSwap(addresses.dpxEthCurvePool).balances(
        b
      );
    
```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L528-L532>

## [G-22] Public Functions not called by they contract should be declear External

The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

```solidity
file:  contracts/amo/UniV3LiquidityAmo.sol

94     function liquidityInPool(
    address _collateral_address,
    int24 _tickLower,
    int24 _tickUpper,
    uint24 _fee
  ) public view returns (uint128) {

112  function numPositions() public view returns (uint256) {

139    function approveTarget(
    address _target,
    address _token,
    uint256 _amount,
    bool use_safe_approve
  ) public onlyRole(DEFAULT_ADMIN_ROLE) {

155    function addLiquidity(
    AddLiquidityParams memory params
  ) public onlyRole(DEFAULT_ADMIN_ROLE) {

213  function removeLiquidity(
    uint256 positionIndex,
    uint256 minAmount0,
    uint256 minAmount1
  ) public onlyRole(DEFAULT_ADMIN_ROLE) {


274    function swap(
    address _tokenA,
    address _tokenB,
    uint24 _fee_tier,
    uint256 _amountAtoB,
    uint256 _amountOutMinimum,
    uint160 _sqrtPriceLimitX96
  ) public onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256) {    

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L94-L99>

```solidity
file:  contracts/core/RdpxV2Bond.sol

29   function pause() public onlyRole(DEFAULT_ADMIN_ROLE) {

33   function unpause() public onlyRole(DEFAULT_ADMIN_ROLE) {

37   function mint(
    address to
  ) public onlyRole(MINTER_ROLE) returns (uint256 tokenId) {

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L29>

```solidity
file:  contracts/core/RdpxV2Core.sol

819      function bondWithDelegate(
         address _to,
         uint256[] memory _amounts,
         uint256[] memory _delegateIds,
         uint256 rdpxBondId
       ) public returns (uint256 receiptTokenAmount, uint256[] memory) {

1135    function getReserveTokenInfo(
          string memory _token
        ) public view returns (address, uint256, string memory) {

1206   function getLpPrice() public view returns (uint256) {
      
```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L819-L824>

```solidity
file:  contracts/decaying-bonds/RdpxDecayingBonds.sol

139     function decreaseAmount(
    uint256 bondId,
    uint256 amount
  ) public onlyRole(RDPXV2CORE_ROLE) {

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L139-L142>

```solidity
file:   contracts/dpxETH/DpxEthToken.sol

29     function pause() public onlyRole(PAUSER_ROLE) {

33     function unpause() public onlyRole(PAUSER_ROLE) {

37     function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {

41     function burn(
       uint256 _amount
       ) public override(ERC20Burnable, IDpxEthToken) onlyRole(BURNER_ROLE) {

47     function burnFrom(
       address account,
       uint256 amount
     ) public override onlyRole(BURNER_ROLE) {

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L29>

```solidity
file:   contracts/perp-vault/PerpetualAtlanticVault.sol

554     function calculatePnl(
        uint256 price,
        uint256 strike,
        uint256 amount
      ) public pure returns (uint256) {
      

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L554-L558>

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVaultLP.sol

118     function deposit(
        uint256 assets,
        address receiver
      ) public virtual returns (uint256 shares) {

145     function redeem(
        uint256 shares,
        address receiver,
        address owner
      ) public returns (uint256 assets, uint256 rdpxAmount) {     

180    function lockCollateral(uint256 amount) public onlyPerpVault {

185    function unlockLiquidity(uint256 amount) public onlyPerpVault {

190    function addProceeds(uint256 proceeds) public onlyPerpVault {

199    function subtractLoss(uint256 loss) public onlyPerpVault {

208    function addRdpx(uint256 amount) public onlyPerpVault {

240    function activeCollateral() public view returns (uint256) {

250    function rdpxCollateral() public view returns (uint256) {

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L118-L121>

## [G-23] Using storage instead of memory for structs/arrays saves gas

Using a memory pointer for a storage struct/array will effectively load all the fields of that data type from storage (SLOAD) into memory (MSTORE). Using a storage pointer will allow you to read specific fields from storage as you need them. If you are not going to use all of the fields of your data type then you should use a storage pointer so that you don’t incur extra Gcoldsload (2100 gas) for fields that you will never use.

Note: These are instances that the automated report missed.

```solidity
file:  contracts/amo/UniV3LiquidityAmo.sol

218    Position memory pos = positions_array[positionIndex];

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L218>

```solidity
file:  contracts/core/RdpxV2Core.sol

1138    ReserveAsset memory asset = reserveAsset[reservesIndex[_token]];

1272    Delegate memory delegatePosition = delegates[_delegateId];

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1138>

## [G-24] Multiple accesses of a mapping/array should use a storage pointer

Caching a mapping’s value in a storage pointer when the value is accessed multiple times saves ~40 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable’s reference instead of repeatedly fetching it.

To achieve this, declare a storage pointer for the variable and use it instead of repeatedly fetching the reference in a map or an array. As an example, instead of repeatedly calling stakes[tokenId_], save its reference via a storage pointer: StakeInfo storage stakeInfo = stakes[tokenId_] and use the pointer instead.

### use storage pointer for reservesIndex[_assetSymbol]

```solidity
file:  contracts/core/RdpxV2Core.sol

270     function removeAssetFromtokenReserves(
    string memory _assetSymbol
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    uint256 index = reservesIndex[_assetSymbol];
    _validate(index != 0, 18);

    // remove the asset from the mapping
    reservesIndex[_assetSymbol] = 0;

    // add new index for the last element
    reservesIndex[reserveTokens[reserveTokens.length - 1]] = index;

    // update the index of reserveAsset with the last element
    reserveAsset[index] = reserveAsset[reserveAsset.length - 1];

    // remove the last element
    reserveAsset.pop();
    reserveTokens.pop();

    emit LogAssetRemovedFromtokenReserves(_assetSymbol, index);
  }

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L270-L290>

### use storage pointer for fundingRates[latestFundingPaymentPointer]

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVault.sol

594     function _updateFundingRate(uint256 amount) private {
    if (fundingRates[latestFundingPaymentPointer] == 0) {
      uint256 startTime;
      if (lastUpdateTime > nextFundingPaymentTimestamp() - fundingDuration) {
        startTime = lastUpdateTime;
      } else {
        startTime = nextFundingPaymentTimestamp() - fundingDuration;
      }
      uint256 endTime = nextFundingPaymentTimestamp();
      fundingRates[latestFundingPaymentPointer] =
        (amount * 1e18) /
        (endTime - startTime);
    } else {
      uint256 startTime = lastUpdateTime;
      uint256 endTime = nextFundingPaymentTimestamp();
      if (endTime == startTime) return;
      fundingRates[latestFundingPaymentPointer] =
        fundingRates[latestFundingPaymentPointer] +
        ((amount * 1e18) / (endTime - startTime));
    }
  }

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L594-L614>

### use storage pointer for totalFundingForEpoch[latestFundingPaymentPointer]

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVault.sol
  
358         totalFundingForEpoch[latestFundingPaymentPointer]
    );
    _updateFundingRate(totalFundingForEpoch[latestFundingPaymentPointer]);

    emit PayFunding(
      msg.sender,
      totalFundingForEpoch[latestFundingPaymentPointer],
      latestFundingPaymentPointer
    );

    return (totalFundingForEpoch[latestFundingPaymentPointer]);
  }

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L358-L396>

### use storage Pointer for optionPositions[optionIds[i]]

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVault.sol

328       for (uint256 i = 0; i < optionIds.length; i++) {
      uint256 strike = optionPositions[optionIds[i]].strike;
      uint256 amount = optionPositions[optionIds[i]].amount;

      // check if strike is ITM
      _validate(strike >= getUnderlyingPrice(), 7);

      ethAmount += (amount * strike) / 1e8;
      rdpxAmount += amount;
      optionsPerStrike[strike] -= amount;
      totalActiveOptions -= amount;

      // Burn option tokens from user
      _burn(optionIds[i]);

      optionPositions[optionIds[i]].strike = 0;
    }

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L328-L344>

### use storage pionter for positions_mapping

```solidity
file:  contracts/amo/UniV3LiquidityAmo.sol

262      positions_array.pop();
    delete positions_mapping[pos.token_id];

    // send tokens to rdpxV2Core
    _sendTokensToRdpxV2Core(tokenA, tokenB);

    emit log(positions_array.length);
    emit log(positions_mapping[pos.token_id].token_id);
  }

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L262-L270>

### use storage pointer for  fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][strike]

```solidity
file:   contracts/perp-vault/PerpetualAtlanticVault.sol
 
421    
      uint256 amount = optionsPerStrike[strike] -
        fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
          strike
        ];

      uint256 timeToExpiry = nextFundingPaymentTimestamp() -
        (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration));

      uint256 premium = calculatePremium(
        strike,
        amount,
        timeToExpiry,
        getUnderlyingPrice()
      );

      latestFundingPerStrike[strike] = latestFundingPaymentPointer;
      fundingAmount += premium;

      // Record number of options that funding payments were accounted for, for this epoch
      fundingPaymentsAccountedFor[latestFundingPaymentPointer] += amount;

      // record the number of options funding has been accounted for the epoch and strike
      fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
        strike
      ] += amount;


```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L421-L445>

### use storage pointer for fundingPaidFor[pointer]

```solidity
file:  contracts/core/RdpxV2Core.sol

790     function provideFunding()
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 fundingAmount)
  {
    _whenNotPaused();
    uint256 pointer = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
      .latestFundingPaymentPointer();
    _validate(fundingPaidFor[pointer] == false, 16);

    fundingAmount = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
      .payFunding();

    reserveAsset[reservesIndex["WETH"]].tokenBalance -= fundingAmount;

    fundingPaidFor[pointer] = true;

    emit LogProvideFunding(pointer, fundingAmount);
  }

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L790-L807>

## [G-25] Using unchecked blocks for addation and subtruction to save gas

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isnâ€™t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.

```solidity
file:  contracts/core/RdpxV2Core.sol

502    maturity: block.timestamp + bondMaturity,

535    ethBalance + _amount <= (ethBalance + dpxEthBalance) / 2,

539    dpxEthBalance + _amount <= (ethBalance + dpxEthBalance) / 2,

614    amount2 = _amount - amount1;

645    amount - _rdpxAmount

678    _rdpxAmount + extraRdpxToWithdraw

863    returnValues.wethRequired - premium,

983    amountWithdrawn = delegate.amount - delegate.activeCollateral;

1002   balance = balance - totalWethDelegated;

1103   block.timestamp + 10

1113   _wethAmount + amountOfWethOut,

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L502>

```solidity
file:  contracts/reLP/ReLPContract.sol

283    block.timestamp + 10

294    block.timestamp + 10

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L283>

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVaultLP.sol

156    allowance[owner][msg.sender] = allowed - shares;

192    collateral.balanceOf(address(this)) >= _totalCollateral + proceeds,

201    collateral.balanceOf(address(this)) == _totalCollateral - loss,

210    IERC20WithBurn(rdpx).balanceOf(address(this)) >= _rdpxCollateral + amount,

256    return _totalCollateral - _activeCollateral;

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L156>

```solidity
file:   contracts/perp-vault/PerpetualAtlanticVault.sol

581     return _strike - remainder + roundingPrecision;

600     startTime = nextFundingPaymentTimestamp() - fundingDuration;

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L581>

```solidity
file:  contracts/amo/UniV2LiquidityAmo.sol

231     block.timestamp + 1

279     block.timestamp + 1

342     block.timestamp + 1

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L231>

```solidity
file:  contracts/amo/UniV3LiquidityAmo.sol

260    positions_array.length - 1

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L260>

## [G-26] Save loop calls

### Instead of calling reserveAsset[i] 3 times in each loop for fetching data, it can be saved as a variable

```solidity
file:

996       for (uint256 i = 1; i < reserveAsset.length; i++) {
      uint256 balance = IERC20WithBurn(reserveAsset[i].tokenAddress).balanceOf(
        address(this)
      );

      if (weth == reserveAsset[i].tokenAddress) {
        balance = balance - totalWethDelegated;
      }
      reserveAsset[i].tokenBalance = balance;
    }

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L996-L1005>

### Instead of calling optionPositions[optionIds[i]] 3 times in each loop for fetching data, it can be saved as a variable

```solidity
file:   contracts/perp-vault/PerpetualAtlanticVault.sol

328        for (uint256 i = 0; i < optionIds.length; i++) {
      uint256 strike = optionPositions[optionIds[i]].strike;
      uint256 amount = optionPositions[optionIds[i]].amount;

      // check if strike is ITM
      _validate(strike >= getUnderlyingPrice(), 7);

      ethAmount += (amount * strike) / 1e8;
      rdpxAmount += amount;
      optionsPerStrike[strike] -= amount;
      totalActiveOptions -= amount;

      // Burn option tokens from user
      _burn(optionIds[i]);

      optionPositions[optionIds[i]].strike = 0;
    }

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L328-L344>

## [G-27] Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

```solidity
file:  contracts/core/RdpxV2Core.sol

88     uint256 public constant RDPX_RATIO_PERCENTAGE = 25 * DEFAULT_PRECISION;

91     uint256 public constant ETH_RATIO_PERCENTAGE = 75 * DEFAULT_PRECISION;

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L88>

## [G-28] Using delete statement can save gas

```solidity
file:  contracts/core/RdpxV2Core.sol

277   reservesIndex[_assetSymbol] = 0;

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L277>

```solidity
file:   contracts/perp-vault/PerpetualAtlanticVault.sol

343     optionPositions[optionIds[i]].strike = 0;

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L343>

## [G-29] Amounts should be checked for 0 before calling a transfer

Checking non-zero transfer values can avoid an expensive external call and save gas.
While this is done at some places, it’s not consistently done in the solution.
I suggest adding a non-zero-value check here:

```solidity
file:  contracts/dpxETH/DpxEthToken.sol

55     function _beforeTokenTransfer(
    address from,
    address to,
    uint256 amount
  ) internal override {
    _whenNotPaused();
    super._beforeTokenTransfer(from, to, amount);
  }

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L55-L62>

## [G‑30] Don’t apply the same value to state variables

If _whenDefault is already NEVER, it’ll save 2100 gas to not set it to that value again

There is 8 instance of this issue:

```solidity
file:  contracts/core/RdpxV2Core.sol

94     uint256 public rdpxBurnPercentage = 50 * DEFAULT_PRECISION;

        /// @notice The % of rdpx sent to fee distributor while bonding
97     uint256 public rdpxFeePercentage = 50 * DEFAULT_PRECISION;

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L94-L97>

```solidity
file:  contracts/core/RdpxV2Core.sol

100   uint256 public slippageTolerance = 5e5; // 0.5%

      /// @notice Liquidity slippage tolerance
103   uint256 public liquiditySlippageTolerance = 5e5; // 0.5%

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L100-L103>

```solidity
file:  contracts/reLP/ReLPContract.sol

73     uint256 public liquiditySlippageTolerance = 5e5; // 0.5%

       /// @notice The slippage tolernce in swaps in 1e8 precision
76     uint256 public slippageTolerance = 5e5; // 0.5%

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L73-L76>

```solidity
file:    contracts/core/RdpxV2Core.sol

286       reserveAsset.pop();

287       reserveTokens.pop();

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L286-L287>

## [G‑31] Inverting the condition of an if-else-statement wastes gas

Flipping the true and false blocks instead saves 3 gas

```solidity
file: contracts/amo/UniV3LiquidityAmo.sol

199   params._tokenA == address(rdpx) ? params._tokenB : params._tokenA,

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L199>

```solidity
file:  contracts/core/RdpxV2Core.sol

525    (uint256 a, uint256 b) = coin0 == weth ? (0, 1) : (1, 0);

553    _ethToDpxEth ? int128(int256(a)) : int128(int256(b)),

554    _ethToDpxEth ? int128(int256(b)) : int128(int256(a)),

556    minAmount > 0 ? minAmount : minOut

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L525>

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVault.sol

547    _price > 0 ? _price : getUnderlyingPrice(),

559    return strike > price ? ((strike - price) * amount) / 1e8 : 0;

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L547>

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVaultLP.sol

283     supply == 0 ? assets : assets.mulDivDown(supply, totalVaultCollateral);

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L283>

## [G-32] Assigning keccak operations to constant variables results in extra gas costs

"constants" expressions are expressions. As such, keccak assigned to a constant variable are re-computed at each use of the variable, which will consume gas unnecessarily. To save gas, Changing the variable from constant to immutable will make the computation run only once and therefore save gas.

```solidity
file:  contracts/core/RdpxV2Bond.sol

22     bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L22>

```solidity
file:   contracts/decaying-bonds/RdpxDecayingBonds.sol

31     bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

34     bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L31>

```solidity
file:  contracts/dpxETH/DpxEthToken.sol

19    bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");

20    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

21    bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L19>

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVault.sol

45      bytes32 public constant MANAGER_ROLE = keccak256("MANAGER_ROLE");

48      bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L45>

```solidity
file:  contracts/reLP/ReLPContract.sol

70     bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L70>

### Recommended Mitigation Steps

Change the variable from constant to immutable

## [G-33] Make 3 event parameters indexed when possible

It’s the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```solidity
file:  contracts/amo/UniV3LiquidityAmo.sol

379    event log(uint);

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L379>

```solidity
file:  contracts/decaying-bonds/RdpxDecayingBonds.sol

53   event EmergencyWithdraw(address sender);

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L53>

```solidity
file: contracts/reLP/ReLPContract.sol

311   event LogSetReLpFactor(uint256 _reLPFactor);

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L311>

```solidity
file:  contracts/amo/UniV2LiquidityAmo.sol

414    event LogAssetsTransfered(
    address indexed sender,
    uint256 tokenAAmount,
    uint256 tokenBAmount
  );

421    event LogEmergencyWithdraw(address sender, address[] tokens);

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L414-L418>

## [G-34] With assembly, .call (bool success) transfer can be done gas-optimized

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.

```solidity
file:   contracts/amo/UniV3LiquidityAmo.sol

344    (bool success, bytes memory result) = _to.call{ value: _value }(_data);

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L344>

```solidity
file:   contracts/decaying-bonds/RdpxDecayingBonds.sol

98     (bool success, ) = to.call{ value: amount, gas: gas }("");

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L98>

## [G-35] Use constants instead of type(uintx).max

type(uint120).max or type(uint112).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity
file:  contracts/core/RdpxV2Core.sol

341    type(uint256).max

343    IERC20WithBurn(weth).approve(addresses.dopexAMMRouter, type(uint256).max);

344    IERC20WithBurn(weth).approve(addresses.dpxEthCurvePool, type(uint256).max);

347    type(uint256).max

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L341>

```solidity
file:  contracts/reLP/ReLPContract.sol

152    type(uint256).max

157    type(uint256).max

162    type(uint256).max

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L152>

```solidity
file:   contracts/perp-vault/PerpetualAtlanticVaultLP.sol

106     collateral.approve(_perpetualAtlanticVault, type(uint256).max);

107     ERC20(rdpx).approve(_perpetualAtlanticVault, type(uint256).max);

155     if (allowed != type(uint256).max) {

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L106>

```solidity
file:   contracts/perp-vault/PerpetualAtlanticVault.sol

209     type(uint256).max

247     type(uint256).max

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L209>

```solidity
file:   contracts/amo/UniV3LiquidityAmo.sol

126     type(uint128).max,

127     type(uint128).max

190     type(uint256).max

223     type(uint128).max,

224     type(uint128).max,


```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L126>

## [G-36] Use uint256(1)/uint256(2) instead for true and false boolean states

```solidity
file:  contracts/core/RdpxV2Core.sol

135    putOptionsRequired = true;

485    optionsOwned[optionId] = true;

805    fundingPaidFor[pointer] = true;

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L135>

```solidity
file:   contracts/core/RdpxV2Core.sol

776     optionsOwned[optionIds[i]] = false;

798    _validate(fundingPaidFor[pointer] == false, 16);

1065   wethReceived = _curveSwap(_amount, false, true, minOut);


```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L776>

## [G-37] Not using the named return variable when a function returns, wastes deployment gas

```solidity
file:   contracts/amo/UniV2LiquidityAmo.sol

308     ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256 token2Amount) {

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L308>

```solidity
file:   contracts/amo/UniV3LiquidityAmo.sol

343     ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (bool, bytes memory) {

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L343>

```solidity
file:  contracts/core/RdpxV2Bond.sol

39     ) public onlyRole(MINTER_ROLE) returns (uint256 tokenId) {

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L39>

```solidity
file:  contracts/core/RdpxV2Core.sol

473    ) internal returns (uint256 premium) {

498    ) internal returns (uint256 bondId) {

520    ) internal returns (uint256 amountOut) {

569    ) internal returns (uint256 receiptTokenAmount) {

603    ) internal view returns (uint256 amount1, uint256 amount2) {

793    returns (uint256 fundingAmount)

824    ) public returns (uint256 receiptTokenAmount, uint256[] memory) {

898    ) public returns (uint256 receiptTokenAmount) {

977    ) external returns (uint256 amountWithdrawn) {

1019   ) external returns (uint256 receiptTokenAmount) {

1054   ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256 wethReceived) {

1085   ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256 dpxEthReceived) {

1137   ) public view returns (address, uint256, string memory) {

1159   ) public view returns (uint256 rdpxRequired, uint256 wethRequired) {

1216   function getDpxEthPrice() public view returns (uint256 dpxEthPrice) {

1227   function getEthPrice() public view returns (uint256 ethPrice) {

1262     )
     public
    view
    returns (
      address delegate,
      uint256 amount,
      uint256 fee,
      uint256 activeCollateral
    )
```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L473>

```solidity
file:  contracts/decaying-bonds/RdpxDecayingBonds.sol

129    function _mintToken(address to) private returns (uint256 tokenId) {
  
153    ) external view returns (uint256[] memory) {

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L129>

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVault.sol

262    returns (uint256 premium, uint256 tokenId)

321    returns (uint256 ethAmount, uint256 rdpxAmount)

407    ) external nonReentrant returns (uint256 fundingAmount) {

544    ) public view returns (uint256 premium) {

566    returns (uint256 timestamp)

576    function roundUp(uint256 _strike) public view returns (uint256 strike) {

588    function _mintOptionToken() private returns (uint256 tokenId) {

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L262>

```solidity
file:   contracts/perp-vault/PerpetualAtlanticVaultLP.sol

121    ) public virtual returns (uint256 shares) {

149    ) public returns (uint256 assets, uint256 rdpxAmount) {

220    ) internal view virtual returns (uint256 assets, uint256 rdpxAmount) {

276    ) public view returns (uint256 shares) {

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L121>

## [G-38] Can make the variable outside the loop to save gas

Consider making the stack variables before the loop which gonna save gas

```solidity
file:   contracts/core/RdpxV2Core.sol

997     uint256 balance = IERC20WithBurn(reserveAsset[i].tokenAddress).balanceOf(
        address(this)
      );

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L997-L999>

```solidity
file:   contracts/perp-vault/PerpetualAtlanticVault.sol

329     uint256 strike = optionPositions[optionIds[i]].strike;

330     uint256 amount = optionPositions[optionIds[i]].amount;

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L329>

```solidity
file:  contracts/amo/UniV3LiquidityAmo.sol

121    Position memory current_position = positions_array[i];

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L121>

## [G-39] Use functions instead of modifiers

Modifiers:
Modifiers are used to add pre or post-conditions to functions. They are typically applied using the modifier keyword and can be used to enforce access control, check conditions, or perform other checks before or after a function's execution. Modifiers are often used to reduce code duplication and enforce certain rules consistently across multiple functions.

However, using modifiers can lead to increased gas costs, especially when multiple function are applied to a single modifier. Each modifier adds overhead in terms of gas consumption because its code is effectively inserted into the function at the point of the modifier keyword. This results in additional gas usage for each modifier, which can accumulate if multiple modifiers are involved.

Functions:
Functions, on the other hand, are more efficient in terms of gas consumption. When you define a function, its code is written only once and can be called from multiple places within the contract. This reduces gas costs compared to using modifiers since the function's code is not duplicated every time it's called.

By using functions instead of modifiers, you can avoid the gas overhead associated with multiple modifiers and create more gas-efficient smart contracts. If you find that certain logic needs to be applied to multiple functions, you can write a separate function to encapsulate that logic and call it from the functions that require it. This way, you achieve code reusability without incurring unnecessary gas costs.

Let's illustrate the difference between using modifiers and functions with a simple example.

Using Modifiers:

```solidity
pragma solidity ^0.8.0;

contract BankAccountUsingModifiers {
    address public owner;
    uint public balance;

    modifier onlyOwner() {
        require(msg.sender == owner, "Only the owner can perform this action.");
        _;
    }

    constructor() {
        owner = msg.sender;
    }

    function withdraw(uint amount) public onlyOwner {
        require(amount <= balance, "Insufficient balance.");
        balance -= amount;
        // Perform withdrawal logic
    }

    function transfer(address recipient, uint amount) public onlyOwner {
        require(amount <= balance, "Insufficient balance.");
        balance -= amount;
        // Perform transfer logic
    }
}

```

Using Functions:

```solidity
  
pragma solidity ^0.8.0;

contract BankAccountUsingFunctions {
    address public owner;
    uint public balance;

    constructor() {
        owner = msg.sender;
    }

    function onlyOwner() private view returns (bool) {
        return msg.sender == owner;
    }

    function withdraw(uint amount) public {
        require(onlyOwner(), "Only the owner can perform this action.");
        require(amount <= balance, "Insufficient balance.");
        balance -= amount;
        // Perform withdrawal logic
    }

    function transfer(address recipient, uint amount) public {
        require(onlyOwner(), "Only the owner can perform this action.");
        require(amount <= balance, "Insufficient balance.");
        balance -= amount;
        // Perform transfer logic
    }
}

```

In the first contract (BankAccountUsingModifiers), we used a modifier called onlyOwner to check if the caller is the owner of the bank account. The onlyOwner modifier is applied to both the withdraw and transfer functions to ensure that only the owner can perform these actions. However, using the modifier will result in additional gas costs since the modifier's code is duplicated in both functions.

In the second contract (BankAccountUsingFunctions), we replaced the modifier with a private function called onlyOwner, which checks if the caller is the owner. We then call this function at the beginning of both the withdraw and transfer functions. By doing this, we avoid the gas overhead associated with using modifiers, making the contract more gas-efficient.

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVaultLP.sol

295     modifier onlyPerpVault() {
        require(
          msg.sender == address(perpetualAtlanticVault),
          "Only the perp vault can call this function"
        );
        _;
      }

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L295-L301>

## [G-40] Create immutable variable to avoid an external call

Instead of performing an external call to get the perpetualAtlanticVaultLp address each time purchase() is invoked, we can perform this external call once in the constructor and store the perpetualAtlanticVaultLp as an immutable variable. Doing this will save 1 external call each time purchase() is invoked.

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVault.sol

271     IPerpetualAtlanticVaultLP perpetualAtlanticVaultLp = IPerpetualAtlanticVaultLP(
        addresses.perpetualAtlanticVaultLP
      );

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L271>

Instead of performing an external call to get the get_pool address each time liquidityInPool() is invoked, we can perform this external call once in the constructor and store the get_pool as an immutable variable. Doing this will save 1 external call each time liquidityInPool() is invoked.

```solidity
file:  contracts/amo/UniV3LiquidityAmo.sol
 
100  IUniswapV3Pool get_pool = IUniswapV3Pool(
      univ3_factory.getPool(address(rdpx), _collateral_address, _fee)
    );

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L100>

Instead of performing an external call to get the token address each time emergencyWithdraw() is invoked, we can perform this external call once in the constructor and store the token as an immutable variable. Doing this will save 1 external call each time emergencyWithdraw() is invoked.

```solidity
file:  contracts/amo/UniV2LiquidityAmo.sol

145      IERC20WithBurn token;

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L145>

## [G-41] Use a do while loop instead of a for loop

A do while loop will cost less gas since the condition is not being checked for the first iteration.

Note: Other optimizations, such as caching length, precrementing, and using unchecked blocks are not included in the Diff since they are included in the automated findings report.

```solidity
file: contracts/amo/UniV2LiquidityAmo.sol

147   for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L147-L150>

```solidity
file:  contracts/core/RdpxV2Core.sol

167     for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }

246   for (uint256 i = 1; i < reserveAsset.length; i++) {
      require(
        reserveAsset[i].tokenAddress != _asset,
        "RdpxV2Core: asset already exists"
      );
    }

775  for (uint256 i = 0; i < optionIds.length; i++) {
      optionsOwned[optionIds[i]] = false;
    }


```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L167-L170>

```solidity
file:  contracts/decaying-bonds/RdpxDecayingBonds.sol

103    for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }

156  for (uint256 i; i < ownerTokenCount; i++) {
      tokenIds[i] = tokenOfOwnerByIndex(_address, i);
    }

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L103-L106>

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVault.sol

225    for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L225-L228>

## [G-42] Unused variables can be omitted

```solidity
file: contracts/core/RdpxV2Core.sol

103   uint256 public liquiditySlippageTolerance = 5e5; // 0.5%

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L103>

```solidity
file:  contracts/perp-vault/PerpetualAtlanticVaultLP.sol

52    string public underlyingSymbol;

```

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L52>
