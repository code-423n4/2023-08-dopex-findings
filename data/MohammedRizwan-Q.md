## Summary

### Low Risk Issues
|Number|Issue|Instances| |
|-|:-|:-:|:-:|
| [L&#x2011;01] | openzeppelin's `setupRole()` is deprecated | 7 |
| [L&#x2011;02] | set max limit of `slippageTolerance` in `setSlippageTolerance()` | 2 |
| [L&#x2011;03] | Calls inside loops that may address DoS | 1 |
| [L&#x2011;04] | Admin may not receive usdc/usdt if blacklisted in case of emergency | 1 |
| [L&#x2011;05] | Insufficient validations in `setAddresses()` can cause both `tokenA` and `tokenB` same | 2 |
| [L&#x2011;06] | `emergencyWithdraw()` functionality should be added in `UniV3LiquidityAmo.sol` like other contracts | 1 |
| [L&#x2011;07] | `TickMath.sol` might revert in solidity version 0.8.19 as used in contracts | 1 |
| [L&#x2011;08] | Update solc version and use unchecked in Uniswap related libraries like `LiquidityAmounts.sol` | 1 |
| [L&#x2011;09] | Solmate safeTransferLib.sol functions does not check the codesize of the token address, which may lead to fund loss | 1 |
| [L&#x2011;10] | An issue with `TransferHelper.sol` which is extensively used in `UniV3LiquidityAmo.sol` | 1 |
| [L&#x2011;11] | Do not hard code the contract address, pass as constructor argument in `UniV3LiquidityAmo.sol` | 1 |
| [L&#x2011;12] | In `RdpxDecayingBonds.sol`, `mint()` does not verify that the bond `expiry` has passed/expired | 1 |
| [L&#x2011;13] | In `DpxEthToken.sol`, `mint()`, `burn()` and `burnFrom()` should only be called when the contract is not paused | 3 |
| [L&#x2011;14] | `addToContractWhitelist()` does not check the contract code existence before whitelisting | 1 |
| [L&#x2011;15] | In `RdpxV2Bond.sol`, `mint()` should be called when the contract is not paused | 1 |
| [L&#x2011;16] | Misleading Natspec on `getLpPrice()` in `UniV2LiquidityAmo.sol` | 1 |
| [L&#x2011;17] | `deadline` should be passed as an argument instead of hardcoding it | contracts |

### [L&#x2011;01]  openzeppelin's `setupRole()` is deprecated
Contracts has used `_setupRole()` in functions for setting the roles. However `_setupRole()` is deprecated by openzeppelin in favor of `_grantRole()`. This can be checked and confirmed at openzeppelin releases [here](https://github.com/OpenZeppelin/openzeppelin-contracts/releases)

There are below instances of this issue:
[Instance 1](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L58) in `UniV2LiquidityAmo.sol` at L-58

[Instance 2](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L80) in `UniV3LiquidityAmo.sol` at L-80

[Instance 3](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L25-L26) in `RdpxV2Bond.sol` at L-25,L-26

[Instance 4](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L125) in `RdpxV2Core.sol` at L-125

[Instance 5](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L61-L62) in `RdpxDecayingBonds.sol` at L-61,L-62

[Instance 6](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L126-L127) in `PerpetualAtlanticVault.sol` at L-126

[Instance 7](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L80-L81) in `ReLPContract.sol` at L-80

### Recommended Mitigation steps
Use `_grantRole()` instead of `_setupRole()`

### [L&#x2011;02]  set max limit of `slippageTolerance` in `setSlippageTolerance()`
`setSlippageTolerance()` is used to set the `slippageTolerance` which is used in swaps 1e8 precision. The contract has bydefault a `slippageTolerance = 5e5` which is equivalent to `0.5%` and this function is used to set same in future. The issue here is it checks the min `slippageTolerance` must be greater than `zero` but it does not check the maximum `slippageTolerance` which can be set to any value by admin.

There is [instance](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L109-L117) of this issue:

[Another instance](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L186-L194) of this issue:

### Recommended Mitigation steps
Add check of max `slippageTolerance` so that it can be set within limits by admin. This must be ensured for users safety in case of malicious admin or admin may set the `slippageTolerance` to any value.

### [L&#x2011;03]  Calls inside loops that may address DoS
In `UniV2LiquidityAmo.sol`, `emergencyWithdraw()` function,
Calls to external contracts inside a loop are dangerous because it could lead to DoS if one of the calls reverts or execution runs out of gas. Such issue also introduces chance of problems with the gas limits.

**Per SWC-113:**
External calls can fail accidentally or deliberately, which can cause a DoS condition in the contract. To minimize the damage caused by such failures, it is better to isolate each external call into its own transaction that can be initiated by the recipient of the call.

Reference link- https://swcregistry.io/docs/SWC-113

Below are few instance of this issue:
[Instance 1](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L149) in `UniV2LiquidityAmo.sol` at L-149

```Solidity
File: contracts/amo/UniV2LiquidityAmo.sol

  function emergencyWithdraw(
    address[] calldata tokens
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    IERC20WithBurn token;

    for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
>>    token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }

    emit LogEmergencyWithdraw(msg.sender, tokens);
```

### Recommended Mitigation Steps
1) Avoid combining multiple calls in a single transaction, especially when calls are executed as part of a loop
2) Always assume that external calls can fail
3) Implement the contract logic to handle failed calls

### [L&#x2011;04]  Admin may not receive usdc/usdt if blacklisted in case of emergency
In `UniV2LiquidityAmo.sol`, `emergencyWithdraw()` is used to transfer the tokens in contract to msg.sender i.e admin address. However it is to be noted here that tokens can also be usdc or usdt whihc has user black listing feature. If the admin address is blacklisted then as the usdc or usdt can not be transferred to admin address. Therefore in case of emergency for which the function has been designed for wont work for usdc or usdt tokens.

There is [1 instance](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L149) of this issue:

```Solidity
File: contracts/amo/UniV2LiquidityAmo.sol

  function emergencyWithdraw(
    address[] calldata tokens
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    IERC20WithBurn token;

    for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }

    emit LogEmergencyWithdraw(msg.sender, tokens);
  }
```

### Recommended Mitigation steps
Add function to transfer the usdc,usdt or function which can blacklist the users from receiving tokens and put the admin access control to it.

For example:

```Solidity

  function emergencyWithdraw(address token, address recipient) external onlyRole(DEFAULT_ADMIN_ROLE) {
    IERC20WithBurn token;
      token.safeTransfer(recipient, token.balanceOf(address(this)));
    }

    emit LogEmergencyWithdraw(recipient, token);
  }
```

### [L&#x2011;05]  Insufficient validations in `setAddresses()` can cause both `tokenA` and `tokenB` same
In `UniV2LiquidityAmo.sol`, `setAddresses()` is used to set different addresses which is shown below,

```Solidity
File: contracts/amo/UniV2LiquidityAmo.sol

  function setAddresses(
    address _tokenA,
    address _tokenB,
    address _pair,
    address _rdpxV2Core,
    address _rdpxOracle,
    address _ammFactory,
    address _ammRouter
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(
      _tokenA != address(0) &&
        _tokenB != address(0) &&
        _pair != address(0) &&
        _rdpxV2Core != address(0) &&
        _rdpxOracle != address(0) &&
        _ammFactory != address(0) &&
        _ammRouter != address(0),
      "reLPContract: address cannot be 0"
    );
    addresses = Addresses({
      tokenA: _tokenA,
      tokenB: _tokenB,
      pair: _pair,
      rdpxV2Core: _rdpxV2Core,
      rdpxOracle: _rdpxOracle,
      ammFactory: _ammFactory,
      ammRouter: _ammRouter
    });
  }
```
The issue here is, it is possible the tokenA and tokenB can be same however this should not be the desired behavior as tokenA will be set to `rdpx` and tokenB to `weth` but there is no validation that checks both are not same.

There [ instance](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L74-L102) of this issue:

[Another instance](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L115-L148) of this issue:

### Recommended Mitigation steps

```diff
File: contracts/amo/UniV2LiquidityAmo.sol

  function setAddresses(
    address _tokenA,
    address _tokenB,
    address _pair,
    address _rdpxV2Core,
    address _rdpxOracle,
    address _ammFactory,
    address _ammRouter
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(
      _tokenA != address(0) &&
        _tokenB != address(0) &&
        _pair != address(0) &&
        _rdpxV2Core != address(0) &&
        _rdpxOracle != address(0) &&
        _ammFactory != address(0) &&
        _ammRouter != address(0),
      "reLPContract: address cannot be 0"
    );
+   require(_tokenA != _tokenB, "tokens can not be same");
    addresses = Addresses({
      tokenA: _tokenA,
      tokenB: _tokenB,
      pair: _pair,
      rdpxV2Core: _rdpxV2Core,
      rdpxOracle: _rdpxOracle,
      ammFactory: _ammFactory,
      ammRouter: _ammRouter
    });
  }
```

### [L&#x2011;06]  `emergencyWithdraw()` functionality should be added in `UniV3LiquidityAmo.sol` like other contracts
`emergencyWithdraw()` function is used in case of emergency, However this is missing in `UniV3LiquidityAmo.sol` contract. Other contracts like `RdpxDecayingBonds.sol`, `PerpetualAtlanticVault.sol`, RdpxV2Core.sol` and `UniV2LiquidityAmo.sol` have `emergencyWithdraw()`. This should be added in contract to use incase of emergency.

There is 1 instance of this issue:

### Recommedended Mitigation steps
Add `emergencyWithdraw()` function in contract. Refer the above listed contracts for similar implementation.

### [L&#x2011;07]  `TickMath.sol` might revert in solidity version 0.8.19 as used in contracts
UniswapV3’s TickMath library was changed to allow compilations for solidity version 0.8. However, adjustments to account for the implicit overflow behavior that the contract relies upon were not performed. 

This has been extensively used in most of contracts. This also includes contracts which are in scope as well as out of scope contracts.

**Lets's take an example:**
`UniV3LiquidityAmo.sol` is compiled with version 0.8.19 and has used this library in contract which can be seen as below,

```Solidity
import "../uniswap_V3/libraries/TickMath.sol";
```

In the worst case, it could be that the library always reverts (instead of overflowing as in previous versions), leading to a broken contracts(contract which have used it).

Upon finding the version of `TickMath.sol`, it has used `pragma solidity >=0.5.0` which can be seen as below,

```Solidity
pragma solidity >=0.5.0;
```

The same pragma solidity >=0.5.0; instead of pragma solidity >=0.5.0 <0.8.0; adjustments needs to made for the implicit overflow behavior in used contracts.

### Recommended Mitigation steps
Follow the implementation of the [official TickMath 0.8 branch](https://github.com/Uniswap/v3-core/blob/0.8/contracts/libraries/TickMath.sol#L27) which uses unchecked blocks
for every function.

### [L&#x2011;08]  Update solc version and use unchecked in Uniswap related libraries like `LiquidityAmounts.sol`
The `LiquidityAmounts.sol` has been used in most of contracts including in `UniV3LiquidityAmo.sol` and it is referenced from Uniswap codebase which is intended to work with Solidity compiler <0.8. These older versions have unchecked arithmetic by default and the code takes it into account.

However, dopex contracs code is intended to work with Solidity compiler 0.8.19 which doesn't have unchecked arithmetic by default.

For example, `UniV3LiquidityAmo.sol` can be checked [here](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L14)

Hence, to port the code, it has to be turned on via unchecked keyword.

For example, type(uint).max reverts for v0.8 and returns type(uint).max for older version

### Recommended Mitigation steps
1) Update the pragma of all the three files to 0.8.19 as it is used in all dopex contracts
2) Wrap all the function bodies of uniswap libraries like `LiquidityAmounts.sol` in unchecked.

### [L&#x2011;09]  Solmate safeTransferLib.sol functions does not check the codesize of the token address, which may lead to fund loss
In PerpetualAtlanticVaultLP.sol contract, The contract has used Solmate safeTransferLib.sol which doesn't check the existence of code at the token address. This is a known issue while using solmate's libraries.

Per the Solmate safeTransferLib.sol,
> Note that none of the functions in this library check that a token has code at all! .....

Hence using safeTransferLib.sol library may lead to miscalculation of funds and may lead to loss of funds , because if safetransfer() and safetransferfrom() are called on a token address that doesn't have contract in it, it will always return success, bypassing the return value check. Due to this protocol will think that funds has been transferred and successful , and records will be accordingly calculated, but in reality funds were never transferred.

## Proof of Concept
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L12

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L23

## Recommended Mitigation Steps
Use openzeppelin's safeERC20 which takes care of token code existence.

### [L&#x2011;10]  An issue with `TransferHelper.sol` which is extensively used in `UniV3LiquidityAmo.sol`
`UniV3LiquidityAmo.sol` has used the `TransferHelper.sol` which is basically copied from uniswap. This TransferHelper library has been used for `safeApprove()`, `safeTransfer()` in `UniV3LiquidityAmo.sol` contract. The issue is the library functions does not check the token address existence and it also does not check the address(0) validations for token address. There was an exploit happened due to not checking the address(0) because of the use of this library which can be referenced [here](https://rekt.news/qubit-rekt/),

According to Certik’s analysis:
> One of the root causes of the vulnerability was the fact that tokenAddress.safeTransferFrom() does not revert when the tokenAddress is the zero (null) address (0x0…000)

### Recommended Mitigation steps
Check token address is not address(0) and check the existence of token address in [UniV3LiquidityAmo.sol](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L8) OR it should be fixed in `TransferHelper.sol` which will reflect the changes wherever it is being used.

### [L&#x2011;11]  Do not hard code the contract address, pass as constructor argument in `UniV3LiquidityAmo.sol`
In `UniV3LiquidityAmo.sol`, `univ3_factory`, `univ3_positions`, `univ3_router` has been hardcoded. This should not be hardcoded in constructor though the contracts will only be deployed on Arbitrum for now but in case it deploys on other chains, such implementation is not recommended. These addresses should be passed as an constructor arguments.

There are [3 instances](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L76-L89) of this issue:

```Solidity

  constructor(address _rdpx, address _rdpxV2Core) {
    rdpx = _rdpx;
    rdpxV2Core = _rdpxV2Core;

    _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);

    univ3_factory = IUniswapV3Factory(
      0x1F98431c8aD98523631AE4a59f267346ea31F984
    );
    univ3_positions = INonfungiblePositionManager(
      0xC36442b4a4522E871399CD717aBDD847Ab11FE88
    );
    univ3_router = ISwapRouter(0xE592427A0AEce92De3Edee1F18E0157C05861564);
  }
```

### Recommended Mitigation steps
```diff

+ //  univ3_factory = 0x1F98431c8aD98523631AE4a59f267346ea31F984
+ //  univ3_positions = 0xC36442b4a4522E871399CD717aBDD847Ab11FE88
+ //  univ3_router = 0xE592427A0AEce92De3Edee1F18E0157C05861564


-  constructor(address _rdpx, address _rdpxV2Core) {
+  constructor(address _rdpx, address _rdpxV2Core, _univ3_factory, _univ3_positions, _univ3_router) {
    rdpx = _rdpx;
    rdpxV2Core = _rdpxV2Core;

    _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);

-    univ3_factory = IUniswapV3Factory(
-      0x1F98431c8aD98523631AE4a59f267346ea31F984
-    );
+    univ3_factory = IUniswapV3Factory(_univ3_factory);
-    univ3_positions = INonfungiblePositionManager(
-      0xC36442b4a4522E871399CD717aBDD847Ab11FE88
-    );
+    univ3_positions = INonfungiblePositionManager(_univ3_positions);
-    univ3_router = ISwapRouter(0xE592427A0AEce92De3Edee1F18E0157C05861564);
+    univ3_router = ISwapRouter(_univ3_router);
  }
```

### [L&#x2011;12]  In `RdpxDecayingBonds.sol`, `mint()` does not verify that the bond `expiry` has passed/expired
In `RdpxDecayingBonds.sol` contract, `expiry` is used in `mint()` function for the expiry timestamp of the bond. The issue here `mint()` is not validating if the `expiry` is still running or not, Without `expiry` verification the functions keeps allowing to be made after the stipulated time or expiry or it may end immediately without letting any one know.

```Solidity
File: contracts/decaying-bonds/RdpxDecayingBonds.sol

  function mint(
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
As it can be seen, expiry is not validated to be greater than block.timestamp. Therefore considering the impact the `expiry` must be validated to greater than block.timestamp.

### Recommended Mitigation steps
```diff

  function mint(
    address to,
    uint256 expiry,
    uint256 rdpxAmount
  ) external onlyRole(MINTER_ROLE) {
    _whenNotPaused();
    require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
+   require(expiry > block.timestamp, "bond expired");
    uint256 bondId = _mintToken(to);
    bonds[bondId] = Bond(to, expiry, rdpxAmount);

    emit BondMinted(to, bondId, expiry, rdpxAmount);
  }
```

### [L&#x2011;13]  In `DpxEthToken.sol`, `mint()`, `burn()` and `burnFrom()` should only be called when the contract is not paused
In `DpxEthToken.sol`, `mint()`, `burn()` and `burnFrom()` can be called anytime though the contract has pause and unpause functionality.

```Solidity
File: contracts/dpxETH/DpxEthToken.sol

  function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {
    _mint(to, amount);
  }

  function burn(
    uint256 _amount
  ) public override(ERC20Burnable, IDpxEthToken) onlyRole(BURNER_ROLE) {
    _burn(_msgSender(), _amount);
  }

  function burnFrom(
    address account,
    uint256 amount
  ) public override onlyRole(BURNER_ROLE) {
    _spendAllowance(account, _msgSender(), amount);
    _burn(account, amount);
  }
```
There are [3 instances](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/dpxETH/DpxEthToken.sol#L37-L53) of this issue

It is to be noted that `_beforeTokenTransfer()` can be called when the contract is not paused. I argue to call the mint and burn functions when the contract is not paused.

In other contracts like `RdpxDecayingBonds.sol` [`mint()`](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L119) can only be called when the contract is not paused. Similarly this can considered here too.

### Recommended Mitigation steps

```diff

  function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {
+   _whenNotPaused();
    _mint(to, amount);
  }

  function burn(
    uint256 _amount
  ) public override(ERC20Burnable, IDpxEthToken) onlyRole(BURNER_ROLE) {
+   _whenNotPaused();
    _burn(_msgSender(), _amount);
  }

  function burnFrom(
    address account,
    uint256 amount
  ) public override onlyRole(BURNER_ROLE) {
+   _whenNotPaused();
    _spendAllowance(account, _msgSender(), amount);
    _burn(account, amount);
  }
```

### [L&#x2011;14]  `addToContractWhitelist()` does not check the contract code existence before whitelisting
In `PerpetualAtlanticVault.sol`, `addToContractWhitelist()` function is used to whitelist contract address. However it does not check the contract code existence before whitlisting it. It is possible that any EOA address or any malicious or phoney address may get whitelisted by admin. 

There is [1 instance](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L153-L157) of this issue:

```Solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol

  function addToContractWhitelist(
    address _contract
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _addToContractWhitelist(_contract);
  }
```

### Recommended Mitigation steps
Check code existence check before whitelisting.

```diff

  function addToContractWhitelist(
    address _contract
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
+   require(_contract.code.lenght != 0, "EOA address can not be whitelisted");
    _addToContractWhitelist(_contract);
  }
```

### [L&#x2011;15]  In `RdpxV2Bond.sol`, `mint()` should be called when the contract is not paused
In `RdpxV2Bond.sol`, `mint()` can be called anytime though the contract has pause and unpause functionality.

```Solidity
File: contracts/core/RdpxV2Bond.sol

  function mint(
    address to
  ) public onlyRole(MINTER_ROLE) returns (uint256 tokenId) {
    tokenId = _tokenIdCounter.current();
    _tokenIdCounter.increment();
    _mint(to, tokenId);
  }
```

There are [1 instances](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L37-L43) of this issue

It is to be noted that `_beforeTokenTransfer()` can be called when the contract is not paused. I argue to call the mint() when the contract is not paused.

In other contracts like `RdpxDecayingBonds.sol` [`mint()`](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L119) can only be called when the contract is not paused. Similarly this can considered here too.

### Recommended Mitigation steps

```diff

  function mint(
    address to
  ) public onlyRole(MINTER_ROLE) returns (uint256 tokenId) {
+   _whenNotPaused();
    tokenId = _tokenIdCounter.current();
    _tokenIdCounter.increment();
    _mint(to, tokenId);
  }
```

### [L&#x2011;16]  Misleading Natspec on `getLpPrice()` in `UniV2LiquidityAmo.sol`
`getLpPrice()` Natspec states `Price is in 1e8 Precision`, however this is not correct, The price returned in 1e18.

There is [1 instances](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L378) of this issue:

```Solidity
File: contracts/amo/UniV2LiquidityAmo.sol

  /**
   * @notice Returns the price of a rDPX/ETH Lp token against the alpha token
   * @dev    Price is in 1e8 Precision
   * @return uint256 LP price
   **/
  function getLpPrice() public view returns (uint256) {
    return IRdpxEthOracle(addresses.rdpxOracle).getLpPriceInEth();
    }
```
I have confirmed the issue by recheching the `RdpxEthOracle.sol` which is given as below,

```Solidity
    /// @dev Returns the price of LP in ETH in 1e18 decimals
    function getLpPriceInEth() external view override returns (uint) {
```

### Recommended Mitigation steps
```diff

  /**
   * @notice Returns the price of a rDPX/ETH Lp token against the alpha token
-   * @dev    Price is in 1e8 Precision
+   * @dev    Price is in 1e18 Precision

   * @return uint256 LP price
   **/
  function getLpPrice() public view returns (uint256) {
    return IRdpxEthOracle(addresses.rdpxOracle).getLpPriceInEth();
    }
```

### [L&#x2011;17]  `deadline` should be passed as an argument instead of hardcoding it
In contracts like `UniV2LiquidityAmo.sol`, `UniV3LiquidityAmo.sol` and other contracts functions like addLiquidity(), removeLiquidity() and other functions have used deadline. The deadline is hardcoded in contract. block.timestamp can be manipulated by miners/stakers. Therefore hardcoding deadline is not recommended.

### Recommended Mitigation steps
Pass the deadline as an argument for all functions using it in contracts.
