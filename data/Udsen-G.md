## 1. NUMBER OF STORAGE READ OPERATIONS CAN BE REDUCED INSIDE THE FOR LOOP TO SAVE GAS

In the `RdpxV2Core.addAssetTotokenReserves` function a new reserve asset is added to the `reserveAsset` array. The new asset address is checked against the elements of the `reserveAsset` array inside a `for` loop as shown below:

    for (uint256 i = 1; i < reserveAsset.length; i++) {
      require(
        reserveAsset[i].tokenAddress != _asset,
        "RdpxV2Core: asset already exists"
      );
    }

Here inside the `for` loop `reserveAsset.length` is called in every iteration of the for loop which means it consumes gas for a `SLOAD` in every iteration. 

Hence it is recommended to cache the `reserveAsset.length` to local variable before the `for` loop and use that local variable as the length for the `for` loop as shown below to save gas. This will save gas on multiple `SLOAD` operations since after the first `SLOAD` operation rest will be `MLOAD` operations which consumes less gas.

    uint256 length = reserveAsset.length;
    for (uint256 i = 1; i < length; i++) {
      require(
        reserveAsset[i].tokenAddress != _asset,
        "RdpxV2Core: asset already exists"
      );
    }

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L246-L251

## 2. `delete` KEYWORD CAN BE USED TO GET A GAS REFUND INSTEAD OF SETTING THE STORAGE VARIABLE TO `0`

The `RdpxV2Core.removeAssetFromtokenReserves` function is used to remove an asset from the reserves tokens. When removing the asset the `reservesIndex` mapping index value for the given key value of `_assetSymbol` is set to zero as shown below:

    reservesIndex[_assetSymbol] = 0;

But is recommended to use the `delete` keyword to delete the storage value from the contract storage for a gas refund. The modified line of code is shown below:

    delete reservesIndex[_assetSymbol];

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L277
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L775-L777

## 3. REDUNDANT `SLOAD` OPERATION CAN BE OMITTED BY MODIFYING THE `removeAMOAddress` FUNCTION LOGIC, TO SAVE GAS

The `RdpxV2Core.removeAMOAddress` function is used to remove an AMO contract from the `amoAddresses` array. The function replaces the `AMO contract` of the index to remove, with the last item of the `amoAddresses` array. Hence this replacement performs a `SLOAD` operation as shown below:

    amoAddresses[_index] = amoAddresses[amoAddresses.length - 1];

But the problem is when the `_index == amoAddresses.length - 1` the operation becomes a redundant operation thus wasting the `SLOAD` operation. Hence the the `removeAMOAddress` function can be modified as follows to save on gas for this redundant `SLOAD` operation.

```solidity
  function removeAMOAddress(
    uint256 _index
  ) external onlyRole(DEFAULT_ADMIN_ROLE) { 
    _validate(_index < amoAddresses.length, 18);
    if(_index == amoAddresses.length - 1){
        amoAddresses.pop();
    }else{
    amoAddresses[_index] = amoAddresses[amoAddresses.length - 1];
    amoAddresses.pop();
    }
  } 
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L388-L394

## 4. `latestFundingPaymentPointer` STORAGE VARIABLE CAN BE CACHED TO A MEMORY VARIABLE IN THE `PerpetualAtlanticVault.payFunding` FUNCTION TO SAVE GAS

The `PerpetualAtlanticVault.payFunding` function uses the `latestFundingPaymentPointer` storage variable six times with in the function execution. Hence this consumes gas for six `SLOAD` operations. But `latestFundingPaymentPointer` storage variable can be cached into a memory variable. This memory variable can be used in place of the storage variable for the subsequent operations. This will save gas for five `SLOAD` operations.

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L372-L396

## 5. ARITHMETIC OPERATION CAN BE `unchecked` DUE TO PREVIOUS CONDITIONAL CHECK TO SAVE GAS

The `PerpetualAtlanticVaultLP.beforeWithdraw` is used to perform validation checks on the amount of native assets to withdraw in the `PerpetualAtlanticVaultLP.redeem` function execution. The `require` statement is as follows:

    require(
      assets <= totalAvailableCollateral(), //@audit-info - _totalCollateral - _activeCollateral
      "Not enough available assets to satisfy withdrawal"
    );

The above check ensures the `assets <= _totalCollateral - _activeCollateral` amount.

If the above conditional check passes, it is confirmed that `assets <= _totalCollateral`.

Hence the `_totalCollateral -= assets` operation can be `unchecked` to save gas as shown below:

    unchecked {
      _totalCollateral -= assets;
    }

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L287-L291

## 6. REDUNDANT `MINTER_ROLE` CHECK CAN BE OMITTED TO SAVE GAS

The `RdpxDecayingBonds.mint` function is used to mint RdpxDecayingBonds to a `to` address. This function can only be called by an address with `MINTER_ROLE`. 

The issue here is the `MINTER_ROLE` is implemented two times with in this function execution as shown below:

```solidity
  function mint(
    address to,
    uint256 expiry,
    uint256 rdpxAmount
  ) external onlyRole(MINTER_ROLE) { //@audit-info - only the MINTER can call
    _whenNotPaused();
    require(hasRole(MINTER_ROLE, msg.sender), 
    ...
  }
```

The `MINTER_ROLE` is enforced as `modifier` as well as a `require` statement. Hence the `require` statement is redundant and can be removed to save gas.

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L118-L120

## 7. `PerpetualAtlanticVaultLP.redeem` TRANSACTION CONTINUES EXECUTING WHEN THE LP TOKEN `totalSupply == 0` THUS WASTING GAS

The `PerpetualAtlanticVaultLP.redeem` function is used to redeem shares for `assets` amount and `rdpxAmount` amount. The `redeem` function calls the `redeemPreview` function which in turn calls the `_convertToAssets` function to calculate the amount of `assets` and `rdpxAmount` to return to the recipient address.

In the `PerpetualAtlanticVaultLP._convertToAssets` function, when the `totalSupply == 0` the `assets` to return will be equal to the amount of `shares` passed in as per the function logic shown below:

    return
      (supply == 0)
        ? (shares, 0) 


Once the `_convertToAssets` function returns to the `redeem` function, the `ERC20._burn` is called which decreases the shares `amount` from the `owner` balance as shown below:

        balanceOf[from] -= amount;

The issue is if the `totalSupply` is zero then the `balanceOf[from]` will also be zero hence the this will underflow the above operation thus reverting the `redeem` transaction.

Hence it is recommended to revert this function inside the `PerpetualAtlanticVaultLP._convertToAssets` function when the `totalSupply == 0` without further execution which is a waste of gas because anyway the transaction should revert if the `totalSupply == 0`.

Hence the below `require` statement should be added in the `PerpetualAtlanticVaultLP._convertToAssets` function at the beginning.

    uint256 supply = totalSupply; 
    require(supply != 0, "Total Supply Can not be Zero");

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L221-L228