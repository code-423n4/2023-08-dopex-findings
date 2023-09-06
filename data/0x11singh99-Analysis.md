### RdpxV2Core.sol 

It is the main contract of the system which is handling bonding and maintaining peg of `DpxEth` price with `weth` price. After analysis this contract i have some suggestions for it.

1. To maintain data of reserve assets in this Core contract. We can change it's layout to remove 1 extra mapping `reservesIndex` and 1 array `reserveTokens`. Since all three is for 1 purpose only. Avoid using dynamic arrays if possible and use mappings because search time in arrays are O(n) while in mapping it is O(1).
Use Token Symbol as key of the mapping.

```diff
   File : /contracts/core/IRdpxV2Core.sol
   //@audit this is defined inside IRdpxV2Core
        struct ReserveAsset {
           address tokenAddress;
           uint256 tokenBalance;
-          string tokenSymbol;
          }
``` 
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/IRdpxV2Core.sol#L43C2-L48C1


```diff
File: /contracts/core/RdpxV2Core.sol
-     ReserveAsset[] public reserveAsset;
       
-     string[] public reserveTokens; 

-     mapping(string => uint256) public reservesIndex;

+     mapping(string => ReserveAsset) public reserveAssets;
//  One reserve Tokens addresses array can be maintained if protocol want to know which address exists but in current need it does not seem to be required.
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L61C1-L73C51

It will simplify the accessing of reserve assets. Because contract is using symbol as the key to search asset, so it can also be done with it. By this change same goal can be achieved with many advantages. eg: Lesser state variables in contract saves lot of gas and slots by removing 2 arrays.
No need to traverse whole array when checking this token already there or not.
Adding new reserve assets and removing old is easy and bug free.(in previous one removing asset have bug because of complex logic in `removeAssetFromtokenReserves`) hard to identify bug in them.

So change code of Core contract according to this above change.

## Some other suggestions

### 1. Always check amount is greater than zero when transferring, minting and burning tokens.

In the whole protocol all contracts in most of the places amount > 0 is not checked before calling `mint` , `burn`, `transfer`and `transferFrom`. 

```solidity
169:    token.safeTransfer(msg.sender, token.balanceOf(address(this)));
```
[169](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L169)

### 2. Do not rely on underflow errors when transferring assets, or burning them. Check owner owns them by checking their balance.

Solmate's ERC20 does not check like openzeppelin that you have this much balance or not just directly subtract balance if msg.sender doesn't have it and unexpected underflow error will comes. It's better if user gets defined errors if he make some mistake. And frontend works with them more smoothly.

```solidity
File:   contracts/perp-vault/PerpetualAtlanticVaultLP.sol

 152:    if (msg.sender != owner) {
         uint256 allowed = allowance[owner][msg.sender]; // Saves gas for limited approvals.

         if (allowed != type(uint256).max) {
           allowance[owner][msg.sender] = allowed - shares; //@audit check allowed is more than or equal to shares rather than relying on underflow. If allowed is lesser than  shares
         }
         }
        (assets, rdpxAmount) = redeemPreview(shares);

        // Check for rounding error since we round down in previewRedeem.
         require(assets != 0, "ZERO_ASSETS");

        _rdpxCollateral -= rdpxAmount;

         beforeWithdraw(assets, shares);

168:    _burn(owner, shares); //@audit solmate _burn doesn't check owner has this much shares so revert with underflow error if owner does not have shares amount. better to check owner balance before calling burn.
```

[152-169](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L152C5-L169C1)
Some other instances present in code like these above.

### 3. There are lot of typos in comments even some places in code and in many places don't have proper NatSpec

Sometimes it can lead to problem and very difficult to find mistake.
Dev must use spell checker extension (if in VS code). It will highlight 100 typos. Fix them it will show next hundred.


```diff
File : /contracts/core/RdpxV2Core.sol

-  52:  /* Inital tokens in the reserve //@audit typo in comments
+  52:  /* Initial tokens in the reserve

-  99:   /// @notice The slippage tolernce in swaps in 1e8 precision
+  99:   /// @notice The slippage tolerance in swaps in 1e8 precision

-  117:  /// @notice Whether put options are requred
+  117:  /// @notice Whether put options are required

```
[52](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/coreRdpxV2Core.sol#L52)
[99](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/coreRdpxV2Core.sol#L99)
[117](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/coreRdpxV2Core.sol#L117)

```diff
File: contracts/perp-vault/PerpetualAtlanticVault.sol

- 117:       uint256 _gensis  //@audit typo in code
+ 117:       uint256 _genesis 

- 124:      genesis = _gensis;
+ 124:      genesis = _genesis;
```
[117](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L117)
[124](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L124)

Many other instances present in code like these above.

### 4. No fuzz testing is done. Many important files and functions also don't have unit tests ie. they are not thoroughly tested.

It increases the missed edge cases and increase the potential number of vulnerabilities. There is no fuzz testing also in test suite. This can easily be done by foundry.

From  9 files in scope. Only 3 Files have proper Unit tests for them.
`RdpxV2Core.sol`,`PerpetualAtlanticVault.sol` and `RdpxDecayingBonds.sol` 
Other 6 files doesn't even have proper unit tests. They are just used in integration tests of rdpxv2-Core and perp-vault it is not enough. 
And even more surprising is these 3 files having unit tests doesn't have tests for all the functions. It is not good. `RdpxV2Core.sol` has many functions which are not tested in unit tests one of them is `removeAssetFromtokenReserves`. This function has faulty logic not updated correctly the `reserveTokens`
`reservesIndex`. I found that bug by testing it. If it was tested and have test case the bug would not be there. I am talking about one function like this many more bugs can be intrduced if code is not properly tested.


*Note : Lack of test cases and typos shows protocol devs was in hurry to make it completed. Here is my sincere suggestion that please test thoroughly your code with unit, fuzzing, integration and even symbolic execution. To make sure code is doing what is expected before coming to audit. Auditors can't test thoroughly because of time limitations of contest. But devs can take time when developing and testing, tests help in auditing. Contracts are immutable not like web2 where we can deploy again. So never make them in hurry specially testing.* 

### 5. Distribute the power within the system by creating separate roles for separate functionality and use Openzeppelin `AccessControl.sol`'s `_setRoleAdmin` function to set admin for each particular role.
Centralization and rug pull risk.

Distribute the power within the system by creating separate roles for separate functionality and use Openzeppelin `AccessControl.sol`'s `_setRoleAdmin` function to set admin for each particular role then call `_setUpRole` to set the role. Let teh admin of that role handle the role members of a role. For new roles assign trusted different addresses inside `constructor` rather than all roles to deployer only. It will mitigate the risk. And use multisig Or DAO system to deploy as these roles.

You can see here all roles are assigned only to deployer of the contracts. And he can do pretty much everything with protocol withdraw assets by emergencyWithdraw or assign any role to anyone after that because he has DEFAULT_ADMIN_ROLE

```solidity
File : /contracts/core/RdpxV2Core.sol

124:     constructor(address _weth) {
125:     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
        
         ...

```
[124-125](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L124C2-L125C48)

```solidity
File : contracts/dpxETH/DpxEthToken.sol

23:   constructor() ERC20("Dopex Synthetic ETH", "dpxETH") {
24:    _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
25:    _grantRole(PAUSER_ROLE, msg.sender);
26:    _grantRole(MINTER_ROLE, msg.sender);
  }
```
[23-26](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L23C2-L27C4)

Like these above in other contracts also (wherever Access Control used for roles) all the roles is assigned only to their deployer including DEFAULT_ADMIN_ROLE by that he can give any role to anyone.

### 6. Code seems little scattered and complex for many simple functionalities. Contains lot of calculations where bug can be introduced.

Making small, simple, modular code it is easy to secure simple code rather complex scattered. In that type of code bug can be introduced in many places. Sme calculations makes no sense what is the the basis of that and proper docs was not available also to understand that.

### Time spent:
60 hours