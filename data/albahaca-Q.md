# Low severity

# summary 

|      | issue | instance |
|------|-------|----------|
|[L-01]| Contracts are not using their OZ upgradeable counterparts|35|
|[L-02]| Deposit emits wrong caller address|2|
|[L-03]| Inadequate Logic in 'require' Statement for Non-Zero Address Check|1|
|[L-04]| Gas griefing/theft is possible on unsafe external call|1|
|[L-05]| Security Vulnerability - Potential Arbitrary Transfer of ERC20 Tokens|6|
|[L-06]| Ignoring Return Value in approveContractToSpend Function|6|
|[L-07]| External Calls Inside Loop in emergencyWithdraw Function|1|
|[L-08]| Reentrancy in addLiquidity Function|2|
|[L-09]| Missing Event Emission in setAddresses Function |2|
|[L-10]| Missing virtual and super in _beforeTokenTransfer Function|1|

## [L-01] Contracts are not using their OZ upgradeable counterparts
Description
The non-upgradeable standard version of OpenZeppelin's library, such as `Ownable`, `Pausable`, `Address`, `Context`, `SafeERC20`, `ERC1967Upgrade` etc, are inherited / used by both the proxy and the implementation contracts.

As a result, when attempting to use the upgrades plugin mentioned, the following errors are raised:

```
Error: Contract `FiatTokenV1` is not upgrade safe

contracts/v1/FiatTokenV1.sol:58: Variable `totalSupply_` is assigned an initial value
  Move the assignment to the initializer
  https://zpl.in/upgrades/error-004

contracts/v1/Pausable.sol:49: Variable `paused` is assigned an initial value
  Move the assignment to the initializer
  https://zpl.in/upgrades/error-004

contracts/v1/Ownable.sol:28: Contract `Ownable` has a constructor
  Define an initializer instead
  https://zpl.in/upgrades/error-001

contracts/util/Address.sol:186: Use of delegatecall is not allowed
  https://zpl.in/upgrades/error-002
       
```
Having reviewed these errors, none had any adversarial impact:

`totalSupply_` and `paused` are explictly assigned the default values `0` and `false`
the implementation contracts utilises the internal `_transferOwnership()` in the initializer, thus transferring ownership to `newOwner` regardless of who the current owner is
`Address's` `delegatecall` is only used by the `ERC1967Upgrade` contract. Comparing both the `Address` and `ERC1967Upgrade` contracts against their upgradeable counterparts show similar behaviour (differences are some refactoring done to shift the delegatecall into the `ERC1967Upgrade` contract).
Nevertheless, it would be safer to use the upgradeable versions of the library contracts to avoid unexpected behaviour.

### Recommended Mitigation Steps
Where applicable, use the contracts from `@openzeppelin/contracts-upgradeable` instead of `@openzeppelin/contracts`.


```solidity
File: amo/UniV2LiquidityAmo.sol
5    import { AccessControl } from "@openzeppelin/contracts/access/AccessControl.sol";

7    import { SafeERC20 } from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L5-L7

```solidity
File: amo/UniV3LiquidityAmo.sol
4  import { SafeERC20 } from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
5  import { AccessControl } from "@openzeppelin/contracts/access/AccessControl.sol";
6  import { ERC721Holder } from "@openzeppelin/contracts/token/ERC721/utils/ERC721Holder.sol";
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L4-L6

```solidity
File: core/RdpxV2Bond.sol
import { ERC721 } from "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import { ERC721Enumerable } from "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
import { Pausable } from "../helper/Pausable.sol";
import { AccessControl } from "@openzeppelin/contracts/access/AccessControl.sol";
import { ERC721Burnable } from "@openzeppelin/contracts/token/ERC721/extensions/ERC721Burnable.sol";
import { Counters } from "@openzeppelin/contracts/utils/Counters.sol";
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L4-L9

```solidity
File: core/RdpxV2Core.sol
5    import { AccessControl } from "@openzeppelin/contracts/access/AccessControl.sol";
6    import { ERC721Holder } from "@openzeppelin/contracts/token/ERC721/utils/ERC721Holder.sol";

26  import { EnumerableSet } from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";
27  import { SafeERC20 } from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
28  import { Math } from "@openzeppelin/contracts/utils/math/Math.sol";
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L5-L6

```solidity
File: decaying-bonds/RdpxDecayingBonds.sol
import { ERC721 } from "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import { ERC721Enumerable } from "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
import { ERC721Burnable } from "@openzeppelin/contracts/token/ERC721/extensions/ERC721Burnable.sol";
import { AccessControl } from "@openzeppelin/contracts/access/AccessControl.sol";
import { Counters } from "@openzeppelin/contracts/utils/Counters.sol";
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L9-L13

```solidity
File: dpxETH/DpxEthToken.sol
import { ERC20 } from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import { ERC20Burnable } from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
import { Pausable } from "../helper/Pausable.sol";
import { AccessControl } from "@openzeppelin/contracts/access/AccessControl.sol";
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L4-L7

```solidity
File: perp-vault/PerpetualAtlanticVault.sol
import { SafeERC20 } from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

// Contracts
import { ReentrancyGuard } from "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import { ERC721 } from "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import { ERC721Enumerable } from "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
import { ERC721Burnable } from "@openzeppelin/contracts/token/ERC721/extensions/ERC721Burnable.sol";
import { ERC4626 } from "@openzeppelin/contracts/token/ERC20/extensions/ERC4626.sol";
import { AccessControl } from "@openzeppelin/contracts/access/AccessControl.sol";
import { Counters } from "@openzeppelin/contracts/utils/Counters.sol";
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L5-L14

```solidity
File: perp-vault/PerpetualAtlanticVaultLP.sol
import { Strings } from "@openzeppelin/contracts/utils/Strings.sol";
import { SafeERC20 } from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L10-L11

```solidity
File: reLP/ReLPContract.sol
import { AccessControl } from "@openzeppelin/contracts/access/AccessControl.sol";
import { SafeMath } from "@openzeppelin/contracts/utils/math/SafeMath.sol";
import { SafeERC20 } from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L5-L7

## [L-02] Deposit emits wrong caller address
In the PerpetualAtlanticVaultLP.deposit function the Deposit is emitted ([Link](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L134)).

It is defined as:
[Link](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L28-L33)
```solidity
event Deposit(
    address indexed caller,
    address indexed owner,
    uint256 assets,
    uint256 shares
  );
```
Fix:
```solidity
-     emit Deposit(msg.sender, receiver, assets, shares);
+     emit Deposit(caller, receiver, assets, shares);
```

same issue here
```solidity
File: perp-vault/PerpetualAtlanticVaultLP.sol
174  emit Withdraw(msg.sender, receiver, owner, assets, shares);
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L174

## [L-03] Inadequate Logic in 'require' Statement for Non-Zero Address Check
If your intention is to require that both _perpetualAtlanticVault and _rdpx are not equal to the zero address, you should change the logical OR (||) operator to a logical AND (&&) operator. Here's the modified code:

```solidity
File: perp-vault/PerpetualAtlanticVaultLP.sol
94   require(
      _perpetualAtlanticVault != address(0) || _rdpx != address(0),
      "ZERO_ADDRESS"
    );
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L94-L97

## [L-04] Gas griefing/theft is possible on unsafe external call
return data (bool success,) has to be stored due to EVM architecture, if in a usage like below, 'out' and 'outsize' values are given (0,0) . Thus, this storage disappears and may come from external contracts a possible Gas griefing/theft problem is avoided

```solidity
File: decaying-bonds/RdpxDecayingBonds.sol
98  (bool success, ) = to.call{ value: amount, gas: gas }("");
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L98

## [L-05] Security Vulnerability - Potential Arbitrary Transfer of ERC20 Tokens
The current implementation introduces a potential security vulnerability. By not utilizing msg.sender as the 'from' address, the contract allows tokens to be transferred from an arbitrary address (addresses.rdpxV2Core) without proper authorization. This can lead to unauthorized token transfers, manipulation of balances, and potential financial losses for the affected users.

```solidity
File: amo/UniV2LiquidityAmo.sol
215  IERC20WithBurn(addresses.tokenB).safeTransferFrom(
      addresses.rdpxV2Core,
      address(this),
      tokenBAmount
    );
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L215-L219

Recommendation:
Use msg.sender as from in transferFrom.

Fix code:
```solidity
IERC20WithBurn(addresses.tokenB).safeTransferFrom(
  msg.sender,
  address(this),
  tokenBAmount
);
```


```solidity
File: amo/UniV2LiquidityAmo.sol
210 IERC20WithBurn(addresses.tokenA).safeTransferFrom(
      addresses.rdpxV2Core,
      address(this),
      tokenAAmount
    );
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L210

```solidity
File: amo/UniV2LiquidityAmo.sol
321 IERC20WithBurn(token1).safeTransferFrom(
      addresses.rdpxV2Core,
      address(this),
      token1Amount
    );
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L321-L325


```solidity
File: perp-vault/PerpetualAtlanticVault.sol
347 collateralToken.safeTransferFrom(
      addresses.perpetualAtlanticVaultLP,
      addresses.rdpxV2Core,
      ethAmount
    );

353 IERC20WithBurn(addresses.rdpx).safeTransferFrom(
      addresses.rdpxV2Core,
      addresses.perpetualAtlanticVaultLP,
      rdpxAmount
    );    

382 collateralToken.safeTransferFrom(
      addresses.rdpxV2Core,
      address(this),
      totalFundingForEpoch[latestFundingPaymentPointer]
    );    
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L347

## [L-06] Ignoring Return Value in approveContractToSpend Function
The return value of an external call is not stored in a local or state variable.
By ignoring the return value of the approve function, the contract loses the ability to verify the success or failure of the approval transaction. This can result in unexpected behaviors or security risks. Ignoring return values can lead to situations where a transaction appears successful, but the approval was not actually granted, potentially causing issues with subsequent contract interactions.

```solidity
File: amo/UniV2LiquidityAmo.sol
134  IERC20WithBurn(_token).approve(_spender, _amount);
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L134

Recommendation:
Ensure that all the return values of the function calls are used.

* same issue in RdpxV2Bond contract

```solidity
File: core/RdpxV2Bond.sol
339  IERC20WithBurn(weth).approve(
      addresses.perpetualAtlanticVault,
      type(uint256).max
    );
    IERC20WithBurn(weth).approve(addresses.dopexAMMRouter, type(uint256).max);
    IERC20WithBurn(weth).approve(addresses.dpxEthCurvePool, type(uint256).max);
    IERC20WithBurn(weth).approve(
      addresses.rdpxV2ReceiptToken,
      type(uint256).max
    );


411   IERC20WithBurn(_token).approve(_spender, _amount);
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L339-L348



## [L-07] External Calls Inside Loop in emergencyWithdraw Function
Performing external calls within a loop can result in a high gas cost and may lead to a denial-of-service vulnerability. If the tokens array contains a large number of elements, the gas cost for executing the loop can exceed the block gas limit. This scenario could prevent or delay the successful execution of the emergencyWithdraw function, leading to a denial-of-service attack.

```solidity
File: amo/UniV2LiquidityAmo.sol
147 for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L147-L150

Recommendation
To mitigate this vulnerability, it is recommended to adopt a pull strategy instead of a push strategy for external calls. Rather than transferring tokens directly within the loop, consider updating the function to allow users to withdraw their tokens individually on-demand.


## [L-08] Reentrancy in addLiquidity Function
Summary
The addLiquidity function in the UniV2LiquidityAMO contract has been identified as potentially vulnerable to reentrancy attacks. The function makes external calls before updating state variables, which could allow an attacker to exploit the contract's state during these calls and perform unauthorized actions.

Description:
Detection of the reentrancy bug. Only report reentrancy that acts as a double call (see reentrancy-eth, reentrancy-no-eth).

```solidity
File: amo/UniV2LiquidityAmo.sol
189   function addLiquidity(
    uint256 tokenAAmount,
    uint256 tokenBAmount,
    uint256 tokenAAmountMin,
    uint256 tokenBAmountMin
  )
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 tokenAUsed, uint256 tokenBUsed, uint256 lpReceived)
  {
    // approve the AMM Router
    IERC20WithBurn(addresses.tokenA).safeApprove(
      addresses.ammRouter,
      tokenAAmount
    );
    IERC20WithBurn(addresses.tokenB).safeApprove(
      addresses.ammRouter,
      tokenBAmount
    );

    // transfer token A and B from the rdpxV2Core to this contract
    IERC20WithBurn(addresses.tokenA).safeTransferFrom(
      addresses.rdpxV2Core,
      address(this),
      tokenAAmount
    );
    IERC20WithBurn(addresses.tokenB).safeTransferFrom(
      addresses.rdpxV2Core,
      address(this),
      tokenBAmount
    );

    // add Liquidity
    (tokenAUsed, tokenBUsed, lpReceived) = IUniswapV2Router(addresses.ammRouter)
      .addLiquidity(
        addresses.tokenA,
        addresses.tokenB,
        tokenAAmount,
        tokenBAmount,
        tokenAAmountMin,
        tokenBAmountMin,
        address(this),
        block.timestamp + 1
      );

    // update LP token Balance
    lpTokenBalance += lpReceived;

    // send unused token A and token B back to rdpxV2Core
    _sendTokensToRdpxV2Core();

    emit LogAddLiquidity(
      msg.sender,
      tokenAAmount,
      tokenBAmount,
      tokenAAmountMin,
      tokenBAmountMin,
      tokenAUsed,
      tokenBUsed,
      lpReceived
    );
  }
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L189-L250

Recommendation:
Apply the check-effects-interactions pattern.

*  same Reentrancy issue in this function
```solidity
File: amo/UniV2LiquidityAmo.sol
258   function removeLiquidity(
    uint256 lpAmount,
    uint256 tokenAAmountMin,
    uint256 tokenBAmountMin
  )
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 tokenAReceived, uint256 tokenBReceived)
  {
    // approve the AMM Router
    IERC20WithBurn(addresses.pair).safeApprove(addresses.ammRouter, lpAmount);

    // remove liquidity
    (tokenAReceived, tokenBReceived) = IUniswapV2Router(addresses.ammRouter)
      .removeLiquidity(
        addresses.tokenA,
        addresses.tokenB,
        lpAmount,
        tokenAAmountMin,
        tokenBAmountMin,
        address(this),
        block.timestamp + 1
      );

    // update LP token Balance
    lpTokenBalance -= lpAmount;

    // send unused token A and token B back to rdpxV2Core
    _sendTokensToRdpxV2Core();

    emit LogRemoveLiquidity(
      msg.sender,
      lpAmount,
      tokenAAmountMin,
      tokenBAmountMin,
      tokenAReceived,
      tokenBReceived
    );
  }
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L258-L296


## [L-09] Missing Event Emission in setAddresses Function 
- The setAddresses function in the UniV2LiquidityAMO contract does not emit an event upon successful execution. This omission makes it difficult for external entities to track and verify updates to the contract's addresses.

- The setAddresses function allows the contract admin to update several important addresses used within the contract. However, there is no event emitted at the end of the function to provide a log of the address update. Emitting an event is crucial for transparency and facilitating external systems' monitoring and verification of changes to the contract's state.

`ther are 2 instace of this issue`
`1:`
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L74-L102
`2:`
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L109-L117

## [L-10] Missing virtual and super in _beforeTokenTransfer Function
 The _beforeTokenTransfer function in the RdpxV2Bond contract is missing the virtual modifier and the super call. This omission prevents the function from properly overriding the parent contract's implementation and may lead to unexpected behavior.

```solidity
File: core/RdpxV2Bond.sol
45 function _beforeTokenTransfer(
    address from,
    address to,
    uint256 tokenId,
    uint256 batchSize
  ) internal override(ERC721, ERC721Enumerable) {
    _whenNotPaused();
    super._beforeTokenTransfer(from, to, tokenId, batchSize);
  }
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L45-L53

Recommendation
To address this issue, it is recommended to add the virtual modifier to the _beforeTokenTransfer function and include a super call to invoke the parent contract's implementation. 