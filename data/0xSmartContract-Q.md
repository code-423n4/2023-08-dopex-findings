### QA Report Issues List

- [x] **Low 01** → Role-based `PAUSER_ROLE` is incorrectly defined
- [x] **Low 02** → `swap()` function becomes unusable after 13 years
- [x] **Low 03** → getLpPrice() value is received from Oracle with External call , but doesn't check to value for unexpected return values
- [x] **Low 04** → `maturity` may be less than expected due to arithmetic rounding
- [x] **Low 05** → There is Timelock function information in the comments, but no Timelock architecture can be seen in the project
- [x] **Low 06** → In the mint() function, the MINTER_ROLE mechanism check is used both times, which creates confusion
- [x] **Low 07** → Uses the deprecated  `_setupRole` function
- [x] **Low 08** → Although there is a role-based security approach with Access Control, the same address is defined for all roles.
- [x] **Low 09** → The `BURNER_ROLE ` constructor is also not defined
- [x] **Non-Critical  10** → MANAGER_ROLE is defined but never used
- [x] **Non-Critical  11** → Use `AccessControlDefaultAdminRule` instead of `AccessControl`
- [x] **Non-Critical  12** → Typo
- [x] **Non-Critical  13** → Project Upgrade and Stop Scenario should be
- [x] **Non-Critical  14** → Missing Event for  initialize

</br>
</br>

### [Low-01] Role-based `PAUSER_ROLE` is incorrectly defined

`RdpxReserve.pause() ` and `RdpxReserve.unpause() ` functions have `onlyRole` modifier like a other important functions in Project , but they functions has `DEFAULT_ADMIN_ROL`E check in `onlyRole` modifier instead `PAUSER_ROLE`

Since `DEFAULT_ADMIN_ROLE` and `PAUSER_ROLE` are `msg.sender`, the owner of both roles is the same in theory at first, but it should be noted that these roles can be changed in the future.

The fact that `PAUSER_ROLE` is defined in the constructor, that the function is `pause() ` and `unpase() ` and that `PAUSER_ROLE` is not used elsewhere in the `RdpxReserve` contract proves this.

```solidity

contracts\reserve\RdpxReserve.sol:
  22  
  23:   constructor(address _rdpx) {
  24:     rdpx = _rdpx;
  25:     _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
  26:     _grantRole(PAUSER_ROLE, msg.sender);
  27:   }
  28: 
  29:   /**
  30:    * @notice Pauses the vault for emergency cases
  31:    * @dev    Can only be called by the owner
  32:    **/
  33:   function pause() external onlyRole(DEFAULT_ADMIN_ROLE) { // @audit   onlyRole not PAUSER_ROLE
  34:     _pause();
  35:   }
  36: 
  37:   /**
  38:    * @notice Unpauses the vault
  39:    * @dev    Can only be called by the owner
  40:    **/
  41:   function unpause() external onlyRole(DEFAULT_ADMIN_ROLE) { // @audit   onlyRole not PAUSER_ROLE
  42:     _unpause();
  43:   }

```

### Recommendation Step

```diff

contracts\reserve\RdpxReserve.sol:
  22  
  23:   constructor(address _rdpx) {
  24:     rdpx = _rdpx;
  25:     _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
  26:     _grantRole(PAUSER_ROLE, msg.sender);
  27:   }
  28: 
  29:   /**
  30:    * @notice Pauses the vault for emergency cases
  31:    * @dev    Can only be called by the owner
  32:    **/
- 33:   function pause() external onlyRole(DEFAULT_ADMIN_ROLE) { 
+ 33:   function pause() external onlyRole(PAUSER_ROLE) {
  34:     _pause();
  35:   }
  36: 
  37:   /**
  38:    * @notice Unpauses the vault
  39:    * @dev    Can only be called by the owner
  40:    **/
- 41:   function unpause() external onlyRole(DEFAULT_ADMIN_ROLE) { 
+ 41:   function pause() external onlyRole(PAUSER_ROLE) {
  42:     _unpause();
  43:   }

```


### [Low-02] swap() function becomes unusable after 13 years

`UniV3LiquidityAmo.swap() ` function is very important role in project, in line 295 also we can see , deadline is 2105300114 unixtime value , this value means ; This function becomes unusable in 13 years later

13 years not as far and this contract not upgradable artitecture.

I recommend that the deadline value be in a user-definable architecture, not hardcoded.


```solidity

contracts\amo\UniV3LiquidityAmo.sol:
  272    // Swap tokenA into tokenB using univ3_router.ExactInputSingle()
  273:   // Uni V3 only
  274:   function swap(
  275:     address _tokenA,
  276:     address _tokenB,
  277:     uint24 _fee_tier,
  278:     uint256 _amountAtoB,
  279:     uint256 _amountOutMinimum,
  280:     uint160 _sqrtPriceLimitX96
  281:   ) public onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256) {
  282:     // transfer token from rdpx v2 core
  283:     IERC20WithBurn(_tokenA).transferFrom(
  284:       rdpxV2Core,
  285:       address(this),
  286:       _amountAtoB
  287:     );
  288: 
  289:     ISwapRouter.ExactInputSingleParams memory swap_params = ISwapRouter
  290:       .ExactInputSingleParams(
  291:         _tokenA,
  292:         _tokenB,
  293:         _fee_tier,
  294:         address(this),
  295:         2105300114, // Expiration: a long time from now @audit This time : 17 September 2036
  296:         _amountAtoB,
  297:         _amountOutMinimum,
  298:         _sqrtPriceLimitX96
  299:       );
  300 
```


### [Low-03] getLpPrice() value is received from Oracle with External call , but doesn't check to value for unexpected return values


There may be cases where the token prices received from Oracles have a value of 0 for any reason, I recommend checking the price from Oracle for such cases.

```solidity

contracts\amo\UniV2LiquidityAmo.sol:
  380     **/
  381:   function getLpPrice() public view returns (uint256) {
  382:     return IRdpxEthOracle(addresses.rdpxOracle).getLpPriceInEth();
  383:   }

```


### [Low-04] `maturity` may be less than expected due to arithmetic rounding


```solidity

contracts\core\RdpxV2Core.sol:
  494     **/
  495:   function _issueBond(
  496:     address _to,
  497:     uint256 _amount
  498:   ) internal returns (uint256 bondId) {
  499:     bondId = RdpxV2Bond(addresses.receiptTokenBonds).mint(_to);
  500:     bonds[bondId] = Bond({
  501:       amount: _amount,
  502:       maturity: block.timestamp + bondMaturity, // @audit this line hasn't scalar 
  503:       timestamp: block.timestamp
  504:     });
  505:   }

```

Recommended Mitigation Steps:

Add a scalar to `maturity` so that any rounding which occurs is negligible


### [Low-05] There is Timelock function information in the comments, but no Timelock architecture can be seen in the project

There is an important ROLE based authorization management mechanism in the project, so the use of timelock is highly recommended.

```solidity
Readme.md
- Check all that apply (e.g. timelock, NFT, AMM, ERC20, rollups, etc.): Timelock function, NFT, AMM, ERC-20 Token

```


### [Low-06] In the mint() function, the MINTER_ROLE mechanism check is used both times, which creates confusion

In the mint() function, the MINTER_ROLE mechanism check is used both times, one using 'onlyRole(MINTER_ROLE)' as a modifier and 'require(hasRole(MINTER_ROLE, msg.sender)' as a require), I recommend removing one of them

```solidity

contracts\decaying-bonds\RdpxDecayingBonds.sol:
  113    /// @param rdpxAmount amount of rdpx to bond
  114:   function mint(
  115:     address to,
  116:     uint256 expiry,
  117:     uint256 rdpxAmount
  118:   ) external onlyRole(MINTER_ROLE) { // @audit MINTER_ROLE check
  119:     _whenNotPaused();
  120:     require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter"); // @audit MINTER_ROLE check

  121:     uint256 bondId = _mintToken(to);
  122:     bonds[bondId] = Bond(to, expiry, rdpxAmount);
  123: 
  124:     emit BondMinted(to, bondId, expiry, rdpxAmount);
  125:   }
```

### [Low-07] Uses the deprecated  `_setupRole` function

The project uses OZ's Access Control architecture as the role-based authorization model, but Access Control’s ` _setupRole` function is deprecated and not recommended by OZ, so the `_grandRole` function should be used instead

```solidity

lib\openzeppelin-contracts\contracts\access\AccessControl.sol:
  203       *
  204:      * NOTE: This function is deprecated in favor of {_grantRole}.
  205:      */
  206:     function _setupRole(bytes32 role, address account) internal virtual {
  207:         _grantRole(role, account);
  208:     }
  209 
```


```solidity

12 results - 8 files

contracts\amo\UniV2LiquidityAmo.sol:
  57    constructor() {
  58:     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
  59    }

contracts\amo\UniV3LiquidityAmo.sol:
  79  
  80:     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
  81  

contracts\core\RdpxV2Bond.sol:
  24    constructor() ERC721("rDPX V2 Bond", "rDPXV2Bond") {
  25:     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
  26:     _setupRole(MINTER_ROLE, msg.sender);
  27    }

contracts\core\RdpxV2Core.sol:
  124    constructor(address _weth) {
  125:     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);

contracts\decaying-bonds\RdpxDecayingBonds.sol:
  61:     _setupRole(MINTER_ROLE, msg.sender);
  62:     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
  63      _tokenIdCounter.increment();



contracts\perp-vault\PerpetualAtlanticVault.sol:
  125  
  126:     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
  127:     _setupRole(MANAGER_ROLE, msg.sender);
  128    }

contracts\reLP\ReLPContract.sol:
  79    constructor() {
  80:     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
  81:     _setupRole(RDPXV2CORE_ROLE, msg.sender);
  82    }
```


### [Low-08] Although there is a role-based security approach with Access Control, the same address is defined for all roles.

Although there is role-based identification with Access Control, all roles are defined as msg.sender instead of different addresses and are not behind any timelock - multisign, this keeps the centrality risk high.
More importantly, it is no different from the single onlyOwner pattern, even if this address will be changed later, it will not be specified in these documents.

```solidity

contracts\dpxETH\DpxEthToken.sol:
  22  
  23:   constructor() ERC20("Dopex Synthetic ETH", "dpxETH") {
  24:     _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
  25:     _grantRole(PAUSER_ROLE, msg.sender);
  26:     _grantRole(MINTER_ROLE, msg.sender);
  27:   }
```

Recommendation:

Role definitions can be made for addresses managed with Multisign, especially in a multisig wallet structure in ADMIN_ROLE, and the changes to be made can be managed during the timelock process, this will reduce the centrality risk to a minimum.
Different addresses should be defined for all roles, if the first definition will be msg.sender, information should be added to the documents about it will be changed in the future.



### [Low-09] The `BURNER_ROLE ` constructor is also not defined

The methodology of using the Access Control architecture of the project; Roles are defined in the constructor. In the `DpxEthToken` contract, the authoritative `burn()` and `burnFrom()` functions are not defined, `BURNER_ROLE` is not defined, it is not suitable for the flow of this project,

Although this role can be defined later by ADMIN_ROLE, it is not suitable for the overall architecture of the project.


```solidity
contracts\dpxETH\DpxEthToken.sol:
  22  
  23:   constructor() ERC20("Dopex Synthetic ETH", "dpxETH") {
  24:     _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
  25:     _grantRole(PAUSER_ROLE, msg.sender);
  26:     _grantRole(MINTER_ROLE, msg.sender);
  27:   }

  
  41:   function burn(
  42:     uint256 _amount
  43:   ) public override(ERC20Burnable, IDpxEthToken) onlyRole(BURNER_ROLE) {
  44:     _burn(_msgSender(), _amount);
  45:   }

 47:   function burnFrom(
  48:     address account,
  49:     uint256 amount
  50:   ) public override onlyRole(BURNER_ROLE) {
  51:     _spendAllowance(account, _msgSender(), amount);
  52:     _burn(account, amount);
  53:   }

```

Recommendation Step;

```diff

contracts\dpxETH\DpxEthToken.sol:
  22  
  23:   constructor() ERC20("Dopex Synthetic ETH", "dpxETH") {
  24:     _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
  25:     _grantRole(PAUSER_ROLE, msg.sender);
  26:     _grantRole(MINTER_ROLE, msg.sender);
+ _grantRole(BURNER_ROLE, msg.sender);

  27:   }
```

### [Non Critical-10] MANAGER_ROLE is defined but never used

MANAGER_ROLE is defined from `PerpetualAtlanticVault` contracts constructor, if such a role exists, it is expected to be used anywhere

but it is not used anywhere, if there is an incorrect ROL definition, it should be corrected by the project, otherwise the reason for this should be stated in the documents / NatSpec comments.

```solidity

contracts\perp-vault\PerpetualAtlanticVault.sol:
  112    /// @param _gensis Gensis time for funding calculation
  113:   constructor(
  114:     string memory _name,
  115:     string memory _symbol,
  116:     address _collateralToken,
  117:     uint256 _gensis
  118:   ) ERC721(_name, _symbol) {
  119:     _validate(_collateralToken != address(0), 1);
  120: 
  121:     collateralToken = IERC20WithBurn(_collateralToken);
  122:     underlyingSymbol = collateralToken.symbol();
  123:     collateralPrecision = 10 ** collateralToken.decimals();
  124:     genesis = _gensis;
  125: 
  126:     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
  127:     _setupRole(MANAGER_ROLE, msg.sender);
  128:   }
```


### [Non Critical-11] Use `AccessControlDefaultAdminRule` instead of `AccessControl`


The main difference between ` AccessControl ` and ` AccessControlDefaultAdminRole ` is how the default administrator role is assigned.

AccessControl: OpenZeppelin's ` AccessControl ` library supports different roles and assigning those roles to specific accounts. However, no roles were initially assigned. This means that the contract creator (deployer) has to manually assign these roles by calling a custom function.

AccessControlDefaultAdminRole: This version automatically assigns the address that created the agreement (usually the person deploying the agreement) with ` DEFAULT_ADMIN_ROLE ` by default when the agreement is created (deployed). This is useful when you initially want a particular address to have administrative rights.

Therefore, if you want to see the administrator role automatically assigned when you deploy a contract, you may prefer to use ` AccessControlDefaultAdminRole `

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/extensions/AccessControlDefaultAdminRules.sol


```solidity

contracts\amo\UniV2LiquidityAmo.sol:
   5: import { AccessControl } from "@openzeppelin/contracts/access/AccessControl.sol";
  22: contract UniV2LiquidityAMO is AccessControl {

contracts\amo\UniV3LiquidityAmo.sol:
   5: import { AccessControl } from "@openzeppelin/contracts/access/AccessControl.sol";
  28: contract UniV3LiquidityAMO is AccessControl, ERC721Holder {


contracts\core\RdpxV2Core.sol:
   5: import { AccessControl } from "@openzeppelin/contracts/access/AccessControl.sol";
  35:   AccessControl,

```

### [Non Critical-12] Typo


```diff

contracts\amo\UniV3LiquidityAmo.sol:
  10  
-  11: // Uniswamp V3
+ 11: // Uniswap V3
  12: import "../uniswap_V3/IUniswapV3Factory.sol";
  13: import "../uniswap_V3/libraries/TickMath.sol";
  14  import "../uniswap_V3/libraries/LiquidityAmounts.sol";

```

### [Non Critical-13] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation.
This can also be called an " EMERGENCY STOP (CIRCUIT BREAKER) PATTERN ".


https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol


### [Non Critical-14] Missing Event for  initialize

Use event-emit when setting the startup items in the project's constructor.

This is recommended so that Web3 Dapps and Analysis tools can monitor the set data and the project.

```solidity

contracts\amo\UniV2LiquidityAmo.sol:
  57:   constructor() {
  58      _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);

contracts\amo\UniV3LiquidityAmo.sol:
  75  
  76:   constructor(address _rdpx, address _rdpxV2Core) {
  77      rdpx = _rdpx;

contracts\core\RdpxV2Bond.sol:
  23  
  24:   constructor() ERC721("rDPX V2 Bond", "rDPXV2Bond") {
  25      _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);

contracts\core\RdpxV2Core.sol:
  124:   constructor(address _weth) {
  125      _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);

contracts\decaying-bonds\RdpxDecayingBonds.sol:
  56:   constructor(
  57      string memory _name,

contracts\dpxETH\DpxEthToken.sol:
  22  
  23:   constructor() ERC20("Dopex Synthetic ETH", "dpxETH") {
  24      _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);

contracts\perp-vault\PerpetualAtlanticVault.sol:
  112    /// @param _gensis Gensis time for funding calculation
  113:   constructor(
  114      string memory _name,

contracts\perp-vault\PerpetualAtlanticVaultLP.sol:
  81:   constructor(
  82      address _perpetualAtlanticVault,

```




