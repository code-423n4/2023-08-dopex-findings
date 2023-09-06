### [QN-01] Use ERC - 5143: Slippage Protection for Atlantic Vault

Project use ERC - 4626 standart

EIP - 4626 is vulnerable to the so - called inflation attacks.This attack results from the possibility to manipulate the exchange rate and front run a victim’s deposit when the vault has low liquidity volume.

Proposed mitigation

The following standard extends the EIP - 4626 Tokenized Vault standard with functions dedicated to the safe interaction between EOAs and the vault when price is subject to slippage.

https://eips.ethereum.org/EIPS/eip-5143

### [QN-02] Lack of Input Validation

For defence -in -depth purpose, it is recommended to perform additional validation against the amount that the user is attempting to deposit, withdraw and redeem to ensure that the submitted amount is valid.


```solidity
PerpetualAtlanticVaultLP.sol
118:   function deposit(
119:     uint256 assets,
120:     address receiver
121:   ) public virtual returns (uint256 shares) {
122:     // Check for rounding error since we round down in previewDeposit.
123:     require((shares = previewDeposit(assets)) != 0, "ZERO_SHARES");
@@@> require(shares <= maxMint(receiver), "mint more than max");
125:     perpetualAtlanticVault.updateFunding();
126: 
127:     // Need to transfer before minting or ERC777s could reenter.
128:     collateral.transferFrom(msg.sender, address(this), assets);
129: 
130:     _mint(receiver, shares);
131: 
132:     _totalCollateral += assets;
133: 
134:     emit Deposit(msg.sender, receiver, assets, shares);
135:   }


145:  function redeem(
146:     uint256 shares,
147:     address receiver,
148:     address owner
149:   ) public returns (uint256 assets, uint256 rdpxAmount) {
150:     perpetualAtlanticVault.updateFunding();
151: 
152:     if (msg.sender != owner) {
153:       uint256 allowed = allowance[owner][msg.sender]; // Saves gas for limited approvals.
@@@> require(shares <= maxRedeem(owner), "redeem more than max");
155:       if (allowed != type(uint256).max) {
156:         allowance[owner][msg.sender] = allowed - shares;
157:       }
158:     }
159:     (assets, rdpxAmount) = redeemPreview(shares);
160: 
161:     // Check for rounding error since we round down in previewRedeem.
162:     require(assets != 0, "ZERO_ASSETS");
163: 
164:     _rdpxCollateral -= rdpxAmount;
165: 
166:     beforeWithdraw(assets, shares);
167: 
168:     _burn(owner, shares);
169: 
170:     collateral.transfer(receiver, assets);
171: 
172:     IERC20WithBurn(rdpx).safeTransfer(receiver, rdpxAmount);
173: 
174:     emit Withdraw(msg.sender, receiver, owner, assets, shares);
175:   }
```


### [QN-03] consider delete unused code

```solidity
RdpxV2Core.sol
102:   /// @notice Liquidity slippage tolerance
103:   uint256 public liquiditySlippageTolerance = 5e5; // 0.5%
```

### [QN-04] consider duplicate variable slippageTolerance variable  

```solidity
ReLPContract.sol
76:   uint256 public slippageTolerance = 5e5; // 0.5%

UniV2LiquidityAmo.sol
51:   uint256 public slippageTolerance = 5e5; // 0.5%

RdpxV2Core.sol
100:   uint256 public slippageTolerance = 5e5; // 0.5%
```

### [QN-05] Deprecated _setupRole function used and safeApprove deprecated library functions
The _setupRole function is deprecated according to the Open Zeppelin comment
NOTE: This function is deprecated in favor of {_grantRole}
Use the recommended _grantRole function instead.

Avoid using deprecated functions. Replace _setupRole with _grantRole
```solidity
 
File: PerpetualAtlanticVault.sol

126:     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);

127:     _setupRole(MANAGER_ROLE, msg.sender);

207:     collateralToken.safeApprove(

```

```solidity
 
File: RdpxDecayingBonds.sol

61:     _setupRole(MINTER_ROLE, msg.sender);

62:     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);

```

```solidity
 
File: RdpxV2Bond.sol

25:     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);

26:     _setupRole(MINTER_ROLE, msg.sender);

```

```solidity
 
File: RdpxV2Core.sol

125:     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);

```

```solidity
 
File: ReLPContract.sol

80:     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);

81:     _setupRole(RDPXV2CORE_ROLE, msg.sender);

150:     IERC20WithBurn(addresses.pair).safeApprove(

155:     IERC20WithBurn(addresses.tokenA).safeApprove(

160:     IERC20WithBurn(addresses.tokenB).safeApprove(

```

```solidity
 
File: UniV2LiquidityAmo.sol

58:     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);

200:     IERC20WithBurn(addresses.tokenA).safeApprove(

204:     IERC20WithBurn(addresses.tokenB).safeApprove(

268:     IERC20WithBurn(addresses.pair).safeApprove(addresses.ammRouter, lpAmount);

328:     IERC20WithBurn(token1).safeApprove(addresses.ammRouter, token1Amount);

```

```solidity
 
File: UniV3LiquidityAmo.sol

80:     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);

148:       TransferHelper.safeApprove(_token, _target, _amount);

302:     TransferHelper.safeApprove(_tokenA, address(univ3_router), _amountAtoB);

```


### [QN-06] `else` reduce the level of unnecessary nesting

Omitting the else block when the if block returns a value is a practice called an "early return" or "guard clause", and in many cases can improve code clarity and relief. https://methodpoet.com/return-early-or-if-statement/ `


 ```solidity
File: PerpetualAtlanticVault.sol

578:     if (remainder == 0) {
579:       return _strike;
580:     } else {
581:       return _strike - remainder + roundingPrecision;
582:     }
```


