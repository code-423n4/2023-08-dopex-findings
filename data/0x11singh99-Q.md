# Low-Level and Non-Critical issues

## Summary

### Low Level List

| Number | Issue Details                                                                                                      | Instances |
| :----: | :----------------------------------------------------------------------------------------------------------------- | :-------: |
| [L-01] | Amounts should be checked for 0 before calling a transfer                                                                 |     2     |
| [L-02] |   Check address 0 before approving.                        |     5     |
| [L-03] |   Check address 0 before transferring to it or minting to it  |     5     |
Total 4 Low Level Issues

### Non Critical List

| Number  | Issue Details                                                       | Instances |
| :-----: | :------------------------------------------------------------------ | :-------: |
| [NC-01] | Some Contracts are not following proper solidity style guide layout |     4     |
| [NC-02] | Typos                                                               |     4     |
| [NC-03] | Getter function should not revert                                   |     1     |
| [NC-04] | Misleading comments                                               |     1     |
| [NC-05] | Check amount is NON-ZERO      |     2     |

Total 5 Non Critical Issues


# LOW FINDINGS

## [L-01] Amounts should be checked for 0 before calling a transfer

In Solidity, the transfer() function is commonly used to transfer ether or tokens from one address to another. When calling the transfer() function, it is important to check the amount being transferred to ensure that it is greater than 0, as transferring 0 amounts is not necessary and can consume unnecessary gas.
It is a security best practice to validate that the amount being transferred is greater than zero to prevent unnecessary gas costs and ensure that transfers are only executed when there is a valid amount to send. Implementing this check will enhance the contract's robustness and mitigate potential issues related to zero-value transfers. It is recommended to include a conditional statement to verify that the amount to be transferred is not zero before executing any transfer functions within the contract.

**_2 Instances - 2 Files_**

**Note: These instances missed by bot report**

```solidity

File : contracts/amo/UniV2LiquidityAmo.sol

142: function emergencyWithdraw(
    address[] calldata tokens
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    IERC20WithBurn token;

    for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
149:  token.safeTransfer(msg.sender, token.balanceOf(address(this)));///@audit check
    }

     emit LogEmergencyWithdraw(msg.sender, tokens);
153:  }
```
[142-153](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L142C3-L153C4)


```solidity

File : contracts/amo/UniV3LiquidityAmo.sol
274: function swap(
      address _tokenA,
      address _tokenB,
      uint24 _fee_tier,
      uint256 _amountAtoB,
      uint256 _amountOutMinimum,
      uint160 _sqrtPriceLimitX96
      )  public onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256) {
     // transfer token from rdpx v2 core
     IERC20WithBurn(_tokenA).transferFrom(///@audit check
      rdpxV2Core,
      address(this),
      _amountAtoB
287:    );
```
[274-287](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L274C3-L287C7)

## [L-02] Check address 0 before approving tokens to it.

It is crucial to include checks for address 0 in smart contracts to prevent unexpected behavior or vulnerabilities. Without this check, the contract may not handle interactions with address 0 correctly, potentially leading to unintended consequences or security risks. It is recommended to implement a validation step to reject interactions with address 0 and provide appropriate error handling to enhance the contract's security and reliability. SafeApprove just help in reverting when fail but don't stop form approving to zero. Openzeppelin revert approving to 0 address but solmate don't so it's better to check first in your contract.

**_5 Instances - 3 Files_**

```solidity

File : contracts/amo/UniV3LiquidityAmo.sol

139: function approveTarget(
     address _target,
     address _token,
     uint256 _amount,
     bool use_safe_approve
     ) public onlyRole(DEFAULT_ADMIN_ROLE) {
     if (use_safe_approve) {
      // safeApprove needed for USDT and others for the first approval
      // You need to approve 0 every time beforehand for USDT: it resets
      TransferHelper.safeApprove(_token, _target, _amount);///@audit check _target is non zero
     } else {
       IERC20WithBurn(_token).approve(_target, _amount); ///@audit check _target is non zero
     }
152:  }
```
[139-152](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L139C3-L152C4)

```solidity

File : contracts/amo/UniV3LiquidityAmo.sol

function removeLiquidity(
    uint256 lpAmount,
    uint256 tokenAAmountMin,
    uint256 tokenBAmountMin
  )
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 tokenAReceived, uint256 tokenBReceived)
  {
    // approve the AMM Router
    IERC20WithBurn(addresses.pair).safeApprove(addresses.ammRouter, lpAmount); ///@audit check

```
[258-268](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L258C3-L268C79)

```solidity
File : contracts/amo/UniV2LiquidityAmo.sol

  Function swap
   ...
328: IERC20WithBurn(token1).safeApprove(addresses.ammRouter, token1Amount);///@audit check

```
[328](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L328)

```solidity
File : contracts/core/RdpxV2Core.sol

 function approveContractToSpend(
    address _token,
    address _spender,
    uint256 _amount
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_token != address(0), 17);
    _validate(_spender != address(0), 17);
    _validate(_amount > 0, 17);
    IERC20WithBurn(_token).approve(_spender, _amount);///@audit check
  }
```
[403-412](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L403C2-L412C4)


## [L-03] Missing check for address 0 in constructor

Allowing address 0 to be used as a constructor argument can lead to unintended behavior or security vulnerabilities, as address 0 typically represents an uninitialized or non-existent address. To mitigate this issue, it's crucial to implement a proper check in the constructor to prevent the contract from being deployed with address 0 as a parameter, thereby enhancing the contract's security and reliability.

**Note: These instances missed by bot report**

**_1 Instance - 1 File_**

```solidity
File : contracts/perp-vault/PerpetualAtlanticVaultLP.so

102: collateral = ERC20(_collateral);

```
[102](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L102)

# Non-Critical FINDINGS

## [NC-01] Some Contracts are not following proper solidity style guide layout
/ Layout of Contract: According to solidity Docs
// version
// imports
// errors
// interfaces, libraries, contracts
// Type declarations
// State variables
// Events
// Modifiers
// Functions

// Layout of Functions:
// constructor
// receive function (if exists)
// fallback function (if exists)
// external
// public
// internal
// private
// internal & private view & pure functions
// external & public view & pure functions
https://docs.soliditylang.org/en/latest/style-guide.html#order-of-layout
**Note: These instances missed by bot report**

**_4 Instances - 3 Files_**

```solidity
File : contracts/amo/UniV2LiquidityAmo.sol

///@audit internal function come earlier
160:  function _sendTokensToRdpxV2Core() internal {
```
[160](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L160)

```solidity
File : contracts/amo/UniV3LiquidityAmo.sol

///@audit events should be declare above 
event RecoveredERC20(address token, uint256 amount);
  event RecoveredERC721(address token, uint256 id);
  event LogAssetsTransfered(
    uint256 tokenAAmount,
    uint256 tokenBAmount,
    address tokenAAddress,
    address tokenBAddress
  );
  /*
   **  burn tokenAmount from the recipient and send tokens to the receipient
   */
  event log(uint);
}
```
[368-380](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L368C3-L380C2)

```solidity
File : contracts/core/RdpxV2Core.sol

///@audit _purchaseOptions came earlier
function _purchaseOptions(
    uint256 _amount
  ) internal returns (uint256 premium) {

```
[471-473](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L471C3-L473C41)


```solidity
File : contracts/core/RdpxV2Core.sol

///@audit declare above 
1286:  error RdpxV2CoreError(uint256);
```
[1286](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1286)


## [NC-02] Typos

**Note: These instances missed by bot report**

**_4 Instances - 4 Files_**

```solidity
File : contracts/decaying-bonds/RdpxDecayingBonds.sol

 // Array of bonds ///@audit bonds is mapping not array
  mapping(uint256 => Bond) public bonds;

```
[36-37](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L36C2-L37C41)

```solidity
File : contracts/core/RdpxV2Core.sol

///@audit make token t capital
240: function addAssetTotokenReserves(

```
[240](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L240)

```solidity
File : contracts/perp-vault/PerpetualAtlanticVault.sol

117: uint256 _gensis ///@audit this is _genesis not _gensis
```
[117](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L117)

```solidity
File : contracts/perp-vault/PerpetualAtlanticVaultLP.sol

///@audit internal function and modifier should be declare above

function beforeWithdraw(uint256 assets, uint256 /*shares*/) internal {
    require(
      assets <= totalAvailableCollateral(),
      "Not enough available assets to satisfy withdrawal"
    );
    _totalCollateral -= assets;
  }

  // ================================ MODIFIERS ================================ //
  modifier onlyPerpVault() {
    require(
      msg.sender == address(perpetualAtlanticVault),
      "Only the perp vault can call this function"
    );
    _;
  }
}

```
[286-302](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L286C3-L302C2)

## [NC-03] Getter function should not revert 

**_1 Instance - 1 File_**

```solidity
File : contracts/core/RdpxV2Core.sol

  function getReserveTokenInfo(
    string memory _token
  ) public view returns (address, uint256, string memory) {
    ReserveAsset memory asset = reserveAsset[reservesIndex[_token]];

    _validate(
      keccak256(abi.encodePacked(asset.tokenSymbol)) ==
        keccak256(abi.encodePacked(_token)),
      19
    );

    return (asset.tokenAddress, asset.tokenBalance, asset.tokenSymbol);
  }
```
[1135-1147](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1135C1-L1147C4)

## [NC-04] Misleading comments 

**_1 Instance - 1 File_**

```solidity
File : contracts/core/RdpxV2Core.sol

///@audit Array misleading, in comment below it is  Delegate[] public delegates
/**
   * @notice Viewer for returning length of uint256[] delegates
   * @return uint256 length of delegates array
   **/
  function getDelegatesLength() external view returns (uint256) {
    return delegates.length;
  }
```
[1244-1250](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1244C3-L1250C4)

## [NC-05] Check amount is NON-ZERO 

**_2 Instance - 2 File_**

```solidity
File : contracts/decaying-bonds/RdpxDecayingBonds.sol

if (transferNative) {
      (bool success, ) = to.call{ value: amount, gas: gas }("");
      require(success, "RdpxReserve: transfer failed");
```
[97-99](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L97C5-L99C56)

```solidity
File : contracts/dpxETH/DpxEthToken.sol

   _spendAllowance(account, _msgSender(), amount);
    _burn(account, amount);
  }

```
[51-53](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L51C2-L53C4)
