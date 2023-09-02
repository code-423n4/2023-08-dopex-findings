## Gas Optimizations

|        | Issue                                                                       |
| ------ | :-------------------------------------------------------------------------- |
| GAS-1  | BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE |
| GAS-2  | Using bools for storage incurs overhead                                     |
| GAS-3  | Don’t compare boolean expressions to boolean literals                       |
| GAS-4  | Setting the constructor to payable                                          |
| GAS-5  | USE FUNCTION INSTEAD OF MODIFIERS                                           |
| GAS-6  | KECCAK256() SHOULD ONLY NEED TO BE CALLED ON A SPECIFIC STRING LITERAL ONCE |
| GAS-7  | MAKING CONSTANT VARIABLES PRIVATE WILL SAVE GAS DURING DEPLOYMENT           |
| GAS-8  | The increment in for loop postcondition can be made unchecked               |
| GAS-9  | TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT                         |
| GAS-10 | Usage of `uint`/`int` smaller than 32 bytes (256 bits) incurs overhead      |
| GAS-11 | Use bitmaps to save gas                                                     |
| GAS-12 | USE BYTES32 INSTEAD OF STRING                                               |
| GAS-13 | Can make the variable outside the loop to save gas                          |

### [GAS-1] BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE

#### Description:

Before transfer, we should check for amount being 0 so the function doesnt run when its not gonna do anything:

#### **Proof Of Concept**

```solidity
File: contracts/amo/UniV3LiquidityAmo.sol

158:     IERC20WithBurn(params._tokenA).transferFrom(

163:     IERC20WithBurn(params._tokenB).transferFrom(

283:     IERC20WithBurn(_tokenA).transferFrom(

```

```solidity
File: contracts/core/RdpxV2Core.sol

624:   function _transfer(

861:       _transfer(

924:     _transfer(rdpxRequired, wethRequired - premium, _amount, rdpxBondId);

```

```solidity
File: contracts/decaying-bonds/RdpxDecayingBonds.sol

91:     bool transferNative,

97:     if (transferNative) {

99:       require(success, "RdpxReserve: transfer failed");

```

```solidity
File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol

128:     collateral.transferFrom(msg.sender, address(this), assets);

170:     collateral.transfer(receiver, assets);

```

```solidity
File: contracts/reLP/ReLPContract.sol

243:     IERC20WithBurn(addresses.pair).transferFrom(

```

### [GAS-2] Using bools for storage incurs overhead

#### Description:

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

#### **Proof Of Concept**

```solidity
File: contracts/core/RdpxV2Core.sol

79:   mapping(uint256 => bool) public optionsOwned;

82:   mapping(uint256 => bool) public fundingPaidFor;

115:   bool public isReLPActive;

118:   bool public putOptionsRequired;

```

### [GAS-3] Don’t compare boolean expressions to boolean literals

#### Description:

`if (<x> == true) => if (<x>)`, `if (<x> == false) => if (!<x>)`

#### **Proof Of Concept**

```solidity
File: contracts/core/RdpxV2Core.sol

798:     _validate(fundingPaidFor[pointer] == false, 16);

```

### [GAS-4] Setting the constructor to payable

#### Description:

Saves ~13 gas per instance

#### **Proof Of Concept**

```solidity
File: contracts/amo/UniV2LiquidityAmo.sol

57:   constructor() {

```

```solidity
File: contracts/amo/UniV3LiquidityAmo.sol

76:   constructor(address _rdpx, address _rdpxV2Core) {

```

```solidity
File: contracts/core/RdpxV2Bond.sol

24:   constructor() ERC721("rDPX V2 Bond", "rDPXV2Bond") {

```

```solidity
File: contracts/core/RdpxV2Core.sol

124:   constructor(address _weth) {

```

```solidity
File: contracts/decaying-bonds/RdpxDecayingBonds.sol

56:   constructor(

```

```solidity
File: contracts/dpxETH/DpxEthToken.sol

23:   constructor() ERC20("Dopex Synthetic ETH", "dpxETH") {

```

```solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol

113:   constructor(

```

```solidity
File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol

81:   constructor(

```

```solidity
File: contracts/reLP/ReLPContract.sol

79:   constructor() {

```

### [GAS-5] USE FUNCTION INSTEAD OF MODIFIERS

#### **Proof Of Concept**

```solidity
File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol

295:   modifier onlyPerpVault() {

```

### [GAS-6] KECCAK256() SHOULD ONLY NEED TO BE CALLED ON A SPECIFIC STRING LITERAL ONCE

#### **Proof Of Concept**

```solidity
File: contracts/core/RdpxV2Bond.sol

22:   bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

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

### [GAS-7] MAKING CONSTANT VARIABLES PRIVATE WILL SAVE GAS DURING DEPLOYMENT

#### Description:

When constants are marked public, extra getter functions are created, increasing the deployment cost. Marking these functions private will decrease gas cost. One can still read these variables through the source code. If they need to be accessed by an external contract, a separate single getter function can be used to return all constants as a tuple. There are four instances of public constants.

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

88:   uint256 public constant RDPX_RATIO_PERCENTAGE = 25 * DEFAULT_PRECISION;

91:   uint256 public constant ETH_RATIO_PERCENTAGE = 75 * DEFAULT_PRECISION;

```

```solidity
File: contracts/decaying-bonds/RdpxDecayingBonds.sol

31:   bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

34:   bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");

```

```solidity
File: contracts/dpxETH/DpxEthToken.sol

19:   bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");

20:   bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

21:   bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");

```

```solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol

45:   bytes32 public constant MANAGER_ROLE = keccak256("MANAGER_ROLE");

48:   bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");

```

```solidity
File: contracts/reLP/ReLPContract.sol

67:   uint256 public constant DEFAULT_PRECISION = 1e8;

70:   bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");

```

### [GAS-8] The increment in for loop postcondition can be made unchecked

#### Description:

This is only relevant if you are using the default solidity checked arithmetic.
the for loop postcondition, i.e., `i++` involves checked arithmetic, which is not required. This is because the value of i is always strictly less than `length <= 2**256 - 1`. Therefore, the theoretical maximum value of i to enter the for-loop body is `2**256 - 2`. This means that the `i++` in the for loop can never overflow. Regardless, the overflow checks are performed by the compiler.

#### **Proof Of Concept**

```solidity
File: contracts/amo/UniV2LiquidityAmo.sol

147:     for (uint256 i = 0; i < tokens.length; i++) {

```

```solidity
File: contracts/amo/UniV3LiquidityAmo.sol

120:     for (uint i = 0; i < positions_array.length; i++) {

```

```solidity
File: contracts/core/RdpxV2Core.sol

167:     for (uint256 i = 0; i < tokens.length; i++) {

246:     for (uint256 i = 1; i < reserveAsset.length; i++) {

775:     for (uint256 i = 0; i < optionIds.length; i++) {

836:     for (uint256 i = 0; i < _amounts.length; i++) {

996:     for (uint256 i = 1; i < reserveAsset.length; i++) {

```

```solidity
File: contracts/decaying-bonds/RdpxDecayingBonds.sol

103:     for (uint256 i = 0; i < tokens.length; i++) {

156:     for (uint256 i; i < ownerTokenCount; i++) {

```

```solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol

225:     for (uint256 i = 0; i < tokens.length; i++) {

328:     for (uint256 i = 0; i < optionIds.length; i++) {

413:     for (uint256 i = 0; i < strikes.length; i++) {

```

### [GAS-9] TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT

#### Description:

There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.
Note that this optimization seems to be dependent on usage of a more recent Solidity version. The following gas savings are on version 0.8.17.

#### **Proof Of Concept**

```solidity
File: contracts/amo/UniV3LiquidityAmo.sol

145:  if (use_safe_approve) {
      // safeApprove needed for USDT and others for the first approval
      // You need to approve 0 every time beforehand for USDT: it resets
      TransferHelper.safeApprove(_token, _target, _amount);
    } else {
      IERC20WithBurn(_token).approve(_target, _amount);
    }

```

```solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol

578:  if (remainder == 0) {
      return _strike;
    } else {
      return _strike - remainder + roundingPrecision;
    }

597: if (lastUpdateTime > nextFundingPaymentTimestamp() - fundingDuration) {
        startTime = lastUpdateTime;
      } else {
        startTime = nextFundingPaymentTimestamp() - fundingDuration;
      }

```

### [GAS-10] Usage of `uint`/`int` smaller than 32 bytes (256 bits) incurs overhead

#### Description:

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.
https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Use a larger size then downcast where needed.

#### **Proof Of Concept**

```solidity
File: contracts/amo/UniV3LiquidityAmo.sol

44:     int24 tickLower; // the tick range of the position

45:     int24 tickUpper;

46:     uint24 fee_tier;

53:     int24 _tickLower;

54:     int24 _tickUpper;

55:     uint24 _fee;

96:     int24 _tickLower,

97:     int24 _tickUpper,

98:     uint24 _fee

277:     uint24 _fee_tier,

```

### [GAS-11] Use bitmaps to save gas

#### Description:

Use mapping(uint256 => uint256) instead of mapping(uint256 => bool)

#### **Proof Of Concept**

#### Recommended Mitigation Steps:

Use mapping(uint256 => uint256) instead of mapping(uint256 => bool)

```solidity
File: contracts/core/RdpxV2Core.sol

79:   mapping(uint256 => bool) public optionsOwned;

82:   mapping(uint256 => bool) public fundingPaidFor;

```

### [GAS-12] USE BYTES32 INSTEAD OF STRING

#### Description:

Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.

#### **Proof Of Concept**

```solidity
File: contracts/core/RdpxV2Core.sol

70:   string[] public reserveTokens;

73:   mapping(string => uint256) public reservesIndex;

242:     string memory _assetSymbol

271:     string memory _assetSymbol

1136:     string memory _token

1137:   ) public view returns (address, uint256, string memory) {

```

```solidity
File: contracts/decaying-bonds/RdpxDecayingBonds.sol

57:     string memory _name,

58:     string memory _symbol

```

```solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol

51:   string public underlyingSymbol;

114:     string memory _name,

115:     string memory _symbol,

```

```solidity
File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol

52:   string public underlyingSymbol;

55:   string public collateralSymbol;

86:     string memory _collateralSymbol

104:     symbol = string.concat(_collateralSymbol, "-LP");

```

### [GAS-13] Can make the variable outside the loop to save gas

#### Description:

Make it outside and only use it inside.

#### **Proof Of Concept**

```solidity
File: contracts/amo/UniV3LiquidityAmo.sol

120:     for (uint i = 0; i < positions_array.length; i++) {

```
