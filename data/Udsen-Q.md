## 1. IF THE OWNER IS BLACKLISTED IT COULD REVERT THE EMERGENCY FUNDS WITHDRAWAL TRANSACTION

The `RdpxV2Core.emergencyWithdraw` function is used to withdraw the all the funds to the `msg.sender` (owner) in an emergency (when the contract is paused). This done by passing in the array of tokens to withdraw as an input parameter to the `emergencyWithdraw` function.

There is a probability that `USDT` or `USDC` being used as tokens in this protocol. These tokens can blacklist a user from using the token.

The problem here is that if the `msg.sender(owner)` address is blacklisted in any of the tokens which are goint to be withdrawn, the entire transaction will revert thus not allowing the owner to transfer all funds to the `msg.sender`.

This can have serious implications since this is an emergency withdrawal, the transaction revert can delay the withdrawals thus allowing an attacker to exploit the contract and steal the funds during this delayed period.

This issue is further aggravated since there is no functionality to chnage the `owner` or the account with the `DEFAULT_ADMIN_ROLE` role of the contract. Becuase this `DEFAULT_ADMIN_ROLE` role is set in the constructor of the contract.

Hence it is recommended to add a functionality to change the `DEFAULT_ADMIN_ROLE` account in this contract. If the current account with the `DEFAULT_ADMIN_ROLE` is blacklisted for a certain token then the account with the `DEFAULT_ADMIN_ROLE` can be chnaged to a new address.

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L167-L170

## 2. LACK OF INPUT VALIDATION FOR DATA ASSIGNMENT IN NEW `ReserveAsset` STRUCTS

The `RdpxV2Core.addAssetTotokenReserves` function is used to add a new reserve token to the `reserveAsset` array. The new `ReserveAsset` struct is constructed as shown below:

    ReserveAsset memory asset = ReserveAsset({
      tokenAddress: _asset,
      tokenBalance: 0,
      tokenSymbol: _assetSymbol
    });

But there is no check to verify that the passed in `_assetSymbol` string is `!= ""` (not equal to an empty string). Hence it is recommended to add the `require` statement to verify that the passed in `_assetSymbol` string is a valid value. The `require` statement can be added as follows:

    require(_assetSymbol != "", "Asset Symbol Can Not Be Empty");
    ReserveAsset memory asset = ReserveAsset({
      tokenAddress: _asset,
      tokenBalance: 0,
      tokenSymbol: _assetSymbol
    });

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L253-L257

## 3. `ZERO` IS USED AS THE `_assetSymbol` OF THE `zeroAsset` IN THE `reserveAsset` ARRAY AND CAN BE USED AGAIN AS THE `_assetSymbol` FOR ANOTHER NEW ASSET ERRORNEOUSLY

The `RdpxV2Core.addAssetTotokenReserves` is used to add a new reserve token to the `reserveAsset` array. The element of the `reserveAsset` is added inside the `RdpxV2Core.constructor` as shown below:

    ReserveAsset memory zeroAsset = ReserveAsset({
      tokenAddress: address(0),
      tokenBalance: 0,
      tokenSymbol: "ZERO"
    });
    reserveAsset.push(zeroAsset); 

Here the tokenSymbol is given as `ZERO` string. But the problem here is that same `ZERO` string can be added in the `addAssetTotokenReserves` function when adding a new asset to the `reserveAsset`. If the `ZERO` is added as an `_assetSymbol` it could be confusing since this `_assetSymbol` is already configured for the `zeroAsset` in the `reserveAsset` array.

Hence it is recommended to add the following check in the `addAssetTotokenReserves` to verify `ZERO != _assetSymbol` in the `addAssetTotokenReserves` function as shown below:

```solidity
  function addAssetTotokenReserves(
    address _asset,
    string memory _assetSymbol
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");
    require(_assetSymbol != "ZERO", "RdpxV2Core: asset symbol cannot be `ZERO`");
    ...
  }
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L240-L244
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L132

## 4. `RdpxV2Core` CONTRACT CAN BE ATOMICALLY WHITELISTED IN THE `perpetualAtlanticVault.constructor` TO MITIGATE POSSIBLE HUMAN ERROR WHICH COULD BREAK THE CRITICAL FUNCTION EXEUCTION

The `RdpxV2Core.settle` function is used to settle the `options`. This function calls the `perpetualAtlanticVault.settle` function where the actula option settling logic is executed. The `perpetualAtlanticVault.settle` function checks whether `RdpxV2Core` contract address is whitelisted by calling the `_isEligibleSender()` function.

The problem here is that `RdpxV2Core` contract address has to be seperately whitelisted by the `DEFAULT_ADMIN_ROLE` by calling the `perpetualAtlanticVault.addToContractWhitelist` function. Hence if the `admin` forgets to add the `RdpxV2Core` address to the whitelist the `RdpxV2Core.settle` transaction will revert and execution will fail.

Hence it is recommended to add the `RdpxV2Core` contract address to the whitelist in the `perpetualAtlanticVault.constructor` when contract is deployed so that `RdpxV2Core` contract will not need to be whitelisted manually by the `admin` in a seperate transaction which is susceptible to human error.

Hence it is recommended to update the `perpetualAtlanticVault.constructor` as shown below:

```solidity
  constructor(
    string memory _name,
    string memory _symbol,
    address _collateralToken,
    address _rdpxV2Core;
    uint256 _gensis
  ) ERC721(_name, _symbol) {
    _validate(_collateralToken != address(0), 1);

    collateralToken = IERC20WithBurn(_collateralToken);
    underlyingSymbol = collateralToken.symbol();
    collateralPrecision = 10 ** collateralToken.decimals();
    genesis = _gensis;

    addToContractWhitelist(_rdpxV2Core);

    _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
    _setupRole(MANAGER_ROLE, msg.sender);
  }
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L324
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L153-L157
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L113-L128

## 5. WHEN AN OPTION IS SETTLED THE `strike` PRICE IS RESET TO `0` BUT `amount` IS NOT RESET TO `0`

In the `perpetualAtlanticVault.settle` function once a particular `option` has been settled the respective  `strike` price for that `optionId` has be reset to `0` as shown below:

      optionPositions[optionIds[i]].strike = 0; 

But the `amount` of the option, for that particular `optionId` is not reset to `0`. Hence to keep a consistent state during the function execution it is recommended to reset the `amount` of the option to `0` once the option has been settled, as shown below:

      optionPositions[optionIds[i]].amount = 0; 

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L343

## 6. MANUAL ERROR IN TOKEN APPROVAL BY THE ADMIN COULD REVERT THE CRITICAL FUND TRANSFER TRANSACTION

The `perpetualAtlanticVault.settle` function is used to settle the options. During this function execution the total calculate the `rdpx` token amount is transferred from the `RdpxV2Core` contract to the `PerpetualAtlanticVaultLP` contract as follows:

    IERC20WithBurn(addresses.rdpx).safeTransferFrom(
      addresses.rdpxV2Core,
      addresses.perpetualAtlanticVaultLP,
      rdpxAmount
    ); 

But the `perpetualAtlanticVault` is not explicitly approved to transfer `rdpx tokens` by the `RdpsV2Core` contract. It seems the `admin` will use the `RdpxV2Core.approveContractToSpend` function to approve the `perpetualAtlanticVault` contract. But this is a manual operation which is susceptible to human error.

If there is an error (especially since return value of approve function is not checked inside the `approveContractToSpend` function) in token approval the `perpetualAtlanticVault.settle` transaction will revert thus breaking the functionality.

Hence it is recommended to approve the `perpetualAtlanticVault` contract for `rdpx` tokens for type(uint256).max amount inside the `RdpxV2Core.constructor` during contract deployment. This will ensure no manual approval is required by the admin for the `perpetualAtlanticVault` contract to spend `rdpx` tokens.

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L403-L412
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L353-L357
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L124-L136

## 7. NATSPEC COMMENTS GIVEN FOR THE RETURN VALUES OF `amount 1` AND `amount 2` ARE WRONG

The `RdpxV2Core._calculateAmounts` funciton is used to calculate the eth amount received by `delegate` and `delegatee` during a `bond with delegate` transaction.

Here the `Natspec` comments for the `amount 1 and amount 2` return variables are given as follows:

```solidity
   * @return amount1 The amount received by the `delegate`
   * @return amount2 The amount received by the `delegatee`
```

But this is wrong. The amount1 is the amount received by the delegatee and amount2 is the amount received by the delegate.

Hence above Natspec comments should be corrected as follows:

```solidity
   * @return amount1 The amount received by the `delegatee`
   * @return amount2 The amount received by the `delegate`
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L595-L596

## 8. `RdpxV2Core.redeem` FUNCTION DELETES THE BOND NFT BUT DOES NOT DELETE THE `Bond` STRUCT FROM THE `bonds` MAPPING 

`RdpxV2Core.redeem` function is used to `withdraw DSC from a matured bond`. The `redeem` function burns the bond NFT for the given `bond Id` as shwon below:

    RdpxV2Bond(addresses.receiptTokenBonds).burn(id);

But the issue is it does not delete the `Bond` struct from the `bonds` mapping for the respective `bond ID`. As a result even though the bond NFT is deleted the respective `Bond` struct entry is still prevalent in the `bonds` mapping.

Hence it is recommended to delete the `Bond` struct from the `bonds` mapping for the `bond Id` which is being redeemed. The function can be modified as follows to accomodate the above change.

```solidity
  delete bonds[id];
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1016-L1042

## 9. LACK INPUT VALIDATION FOR THE ADDRESSES PASSED INTO THE `PerpetualAtlanticVaultLP.constructor`

The `PerpetualAtlanticVaultLP.constructor` performs multiple input validation for addresses to ensure `address(0)` is not passed in as an input to the contract address state initialization. It is shown below:

    require(
      _perpetualAtlanticVault != address(0) || _rdpx != address(0),
      "ZERO_ADDRESS"
    );

But there is no input validation for the passed in `rdpxRdpxV2Core` contract address.

Hence it is recommended to perform the `address(0)` check for the `rdpxRdpxV2Core` address as well inside the `PerpetualAtlanticVaultLP.constructor`.

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L94-L97

## 10. LACK OF INPUT VALIDATION DURING LOW LEVEL `call` FUNCTION EXECUTION

The `RdpxDecayingBonds.emergencyWithdraw` function is used to withdraw funds from the contract when the protocol is paused. It enables to withdraw native tokens as well using low level `call` function as shown below:

    if (transferNative) {
      (bool success, ) = to.call{ value: amount, gas: gas }("");
      require(success, "RdpxReserve: transfer failed");
    }

But there is no input validation for the `to` address used. Hence if `address(0)` is passed in as the `to` address the native token assets will be lost.

Hence it is recommended to perform the following `require` statement before the low level call function.

```solidity
  require(to != address(0), "Can not transfer funds to address(0)");
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L97-L100

## 11. TYPO'S IN THE NATSPEC COMMENTS SHOULD BE CORRECTED

```solidity
   * @param  _slippageTolerance the `slipage` tolerance
```

Above comment should be corrected as follows:

```solidity
   * @param  _slippageTolerance the `slippage` tolerance
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L453

```solidity
  /// @dev the pointer to the `lattest` funding payment timestamp
```

Above comment should be corrected as follows:

```solidity
  /// @dev the pointer to the `latest` funding payment timestamp
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L84

```solidity
    // amount required for `delegatee`
```

Above comment should be corrected as follows:

```solidity
    // amount required for `delegate`
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L607

```solidity
   * @notice Viewer for returning length of uint256[] delegates
```

Above comment should be corrected as follows:

```solidity
   * @notice Viewer for returning length of Delegate[] delegates
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1245