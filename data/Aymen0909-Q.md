# QA Report

## Summary

|               | Issue         | Risk     | Instances     |
| :-------------: |:-------------|:-------------:|:-------------:|
| 1 | Potential underflow in `RdpxV2Core.calculateBondCost` will cause DOS of bond operations | Low | 1 |
| 2 | Missing `sync()` after transferring funds to `RdpxV2Core` | Low | 2 |
| 3 | Funds not transferred to correct recipient in `RdpxDecayingBonds.emergencyWithdraw`| Low | 1 |
| 4 | `UniV2LiquidityAMO.approveContractToSpend` can't approve USDT transfer | Low | 1 |
| 5 | `PerpetualAtlanticVaultLP` can never pull funds from `PerpetualAtlanticVault` | Low |  |
| 6 | `MANAGER_ROLE` not used in `PerpetualAtlanticVault` | Low |  |
| 7 | `underlyingSymbol` not set in `PerpetualAtlanticVaultLP` | NC |  |
| 8 | Unused state variable `collateralPrecision` in the `PerpetualAtlanticVault` contract | NC |  |


## Findings

### 1- Potential underflow in `RdpxV2Core.calculateBondCost` will cause DOS of bond operations :

#### Risk : Low

The function `RdpxV2Core.calculateBondCost` is used to calculate the amount of weth and rdpx tokens required to create a bond, in the case where `_rdpxBondId == 0` the function has to calculate the value of `bondDiscount` and it validate that its value is less than `100e8`.

The issue is that in the following lines of code there is this substraction operation :

```
rdpxRequired =
    ((RDPX_RATIO_PERCENTAGE - (bondDiscount / 2)) *
        _amount *
        DEFAULT_PRECISION) /
    (DEFAULT_PRECISION * rdpxPrice * 1e2);
```

where `RDPX_RATIO_PERCENTAGE = 25 * DEFAULT_PRECISION = 25e8`

As you can see if we have `bondDiscount / 2 > RDPX_RATIO_PERCENTAGE` the operation will revert due to an underflow, and because the check present in the function only ensures that `bondDiscount < 100e8`, it means that there are many valid values for `bondDiscount` that will make the function revert. For example if we take `bondDiscount = 60e8` (which is valid as it's less than 100e8) when the subtraction op goes on we will get : 

`RDPX_RATIO_PERCENTAGE - (bondDiscount / 2) = 25e8 - (60e8 / 2) = 25e8 - 30e8 < 0` 

And thus the operation will underflow causing the call to revert, and we can easily find that any value for `bondDiscount` greater than `50e8` will make the function revert.

When the `calculateBondCost` function revert because of the mentioned issue it will cause all the bonding functions (`bond`, `bondWithDelegate`) to revert hence blocking all the bonding operations (DOS) when `_rdpxBondId == 0`.

I submit this issue as Low risk for the protocol because we can see that the admin can change the value of `bondDiscountFactor` at any time to avoid this problem, but it could be a valid medium and the devs should  take a look at it as if they are assuming in their logic that all values of `bondDiscount` less than `100e8` are possible (and hence the check : `_validate(bondDiscount < 100e8, 14)`) it will cause the protocol problems. 

Because we can see that `bondDiscount` depends on the RDPX treasury reserve which will continue to grow meaning that the admin must be monitoring at each time if the value of `bondDiscount` is getting bigger than `50e8` in order to change the `bondDiscountFactor` to avoid the DOS of bonding operation (revert of the call due to underflow), this is not really an ideal scenario.

#### Proof of Concept

File: core/RdpxV2Core.sol [Line 1156-1199](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1156C3-L1199C4)

```solidity
function calculateBondCost(
    uint256 _amount,
    uint256 _rdpxBondId
) public view returns (uint256 rdpxRequired, uint256 wethRequired) {
    uint256 rdpxPrice = getRdpxPrice();
    if (_rdpxBondId == 0) {
        uint256 bondDiscount = (bondDiscountFactor *
            Math.sqrt(IRdpxReserve(addresses.rdpxReserve).rdpxReserve()) *
            1e2) / (Math.sqrt(1e18)); // 1e8 precision

        // @audit allow values of bondDiscount < 100e8 
        _validate(bondDiscount < 100e8, 14);

        // @audit can underflow if bondDiscount > 50e8
        rdpxRequired =
            ((RDPX_RATIO_PERCENTAGE - (bondDiscount / 2)) *
                _amount *
                DEFAULT_PRECISION) /
            (DEFAULT_PRECISION * rdpxPrice * 1e2);

        wethRequired =
            ((ETH_RATIO_PERCENTAGE - (bondDiscount / 2)) * _amount) /
            (DEFAULT_PRECISION * 1e2);
    } else {
        ...
}
```

#### Mitigation

There are many options the solve this issue but the simpler is to modify the check on the amount `bondDiscount` and make it always less tha `50e8` : _validate(bondDiscount < 50e8, 14)


### 2- Missing `sync()` after transferring funds to `RdpxV2Core` :

#### Risk : Low

In the `RdpxV2Core` contract there a `sync()` function which allows the update of the state variables tracking the tokens balances (ETH, DPX,...) by setting them equal to the actual contract token balance (with the use of `token.balanceOf(address(this))`).

The issue is that there are some instances where this function must be called after transferring funds to `RdpxV2Core` but it is not which could result in a temporarily incorrect balance accounting.

#### Proof of Concept

There 2 instances of this issue :

File: amo/UniV2LiquidityAmo.sol [Line 160-178](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L160C3-L178C4)

```solidity
function _sendTokensToRdpxV2Core() internal {
    uint256 tokenABalance = IERC20WithBurn(addresses.tokenA).balanceOf(
      address(this)
    );
    uint256 tokenBBalance = IERC20WithBurn(addresses.tokenB).balanceOf(
      address(this)
    );
    // transfer token A and B from this contract to the rdpxV2Core
    IERC20WithBurn(addresses.tokenA).safeTransfer(
      addresses.rdpxV2Core,
      tokenABalance
    );
    IERC20WithBurn(addresses.tokenB).safeTransfer(
      addresses.rdpxV2Core,
      tokenBBalance
    );

    // @audit did not call IRdpxV2Core(rdpxV2Core).sync()

    emit LogAssetsTransfered(msg.sender, tokenABalance, tokenBBalance);
}
```

The `_sendTokensToRdpxV2Core()` function is used many times in the `UniV2LiquidityAmo` to transfer tokens to `RdpxV2Core` but it doesn't call the `sync()` function after doing so, we can see that a similar function is present in the other AMO contract [UniV3LiquidityAmo._sendTokensToRdpxV2Core](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L353C3-L364C4) and this one does call the `sync()` function after the transfers.

File: amo/UniV3LiquidityAmo.sol [Line 119-133](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L119C3-L133C4)

```solidity
function collectFees() external onlyRole(DEFAULT_ADMIN_ROLE) {
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

In the second instance the call to `univ3_positions.collect` will transfer tokens to the `RdpxV2Core` contract but after the loop there no call to the `sync()` function to update the tokens accounting in `RdpxV2Core`.

#### Mitigation

Add the `IRdpxV2Core(rdpxV2Core).sync()` call to the instances aforementioned to make sure that the accounting is always correct after a transfer from the AMOs contracts.


### 3- Funds not transferred to correct recipient in `RdpxDecayingBonds.emergencyWithdraw` :

#### Risk : Low

The function `RdpxDecayingBonds.emergencyWithdraw` is used by the admin to pull funds from the contract in case of emergency situation, the function allows the admin to specify a recipient `to` address as input to the function, this address should receive all the extracted funds (as explained in function Natspec comments). But due to an error in function code only the native token is transferred to `to` address and the other tokens are transferred to admin address.

This issue has a Low impact because there is no loss of funds and the tokens are transferred to admin which can later give them to the `to` address but it should be corrected.

#### Proof of Concept

File: decaying-bonds/RdpxDecayingBonds.sol [Line 89-107](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L89C3-L107C4)

```solidity
function emergencyWithdraw(
    address[] calldata tokens,
    bool transferNative,
    address payable to, // @audit should be the recipient
    uint256 amount,
    uint256 gas
) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _whenPaused();
    if (transferNative) {
        (bool success, ) = to.call{value: amount, gas: gas}("");
        require(success, "RdpxReserve: transfer failed");
    }
    IERC20WithBurn token;

    for (uint256 i = 0; i < tokens.length; i++) {
        token = IERC20WithBurn(tokens[i]);
        // @audit funds not transferred to `to` address 
        token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }
}
```

#### Mitigation

Transfer all the funds to the correct recipient address `to`.


### 4- `UniV2LiquidityAMO.approveContractToSpend` can't approve USDT transfer :

#### Risk : Low

In the `UniV2LiquidityAMO` contract, the function `approveContractToSpend` is used to approve other contracts to transfer funds outside `UniV2LiquidityAMO`, for doing so the function uses the `approve` function and there is a check ensuring that the approved amount is always greater than zero, this won't allow the approval of USDT token transfers as in that case it must first reset the approval amount to zero before setting a new one. 

#### Proof of Concept

File: amo/UniV2LiquidityAmo.sol [Line 126-135](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L126-L135)

```solidity
function approveContractToSpend(
    address _token,
    address _spender,
    uint256 _amount
) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(_token != address(0), "reLPContract: token cannot be 0");
    require(_spender != address(0), "reLPContract: spender cannot be 0");
    // @audit reverts if amount == 0
    require(_amount > 0, "reLPContract: amount must be greater than 0");
    // @audit rcan not be used for USDT
    IERC20WithBurn(_token).approve(_spender, _amount);
}
```

#### Mitigation
Use the `safeApprove` function to allow USDT tokens approval, the contract should implement a function similair to  `UniV3LiquidityAMO.approveTarget`.


### 5- `PerpetualAtlanticVaultLP` can never pull funds from `PerpetualAtlanticVault` :

#### Risk : Low

The `PerpetualAtlanticVaultLP` contract is approved to pull collateral tokens from `PerpetualAtlanticVault` and there is even a function `setLpAllowance` that can be called by the admin to increase/decrease the allowance approved for `PerpetualAtlanticVaultLP`, but by looking at the `PerpetualAtlanticVaultLP` contract code we can notice that there no call to pull funds from `PerpetualAtlanticVault` which means that `PerpetualAtlanticVaultLP` can never take the funds that are approved from `PerpetualAtlanticVault`.

This does not make sense to approve a contract to spend funds but in its code there no way to do so, this could be a critical issue and should be looked at, if the `PerpetualAtlanticVaultLP` does need to pull funds from `PerpetualAtlanticVault` then the whole contract code should be reviewed to enable it (in the other case there no need to have `setLpAllowance` function or to approve `PerpetualAtlanticVaultLP`).

#### Instance

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L207-L210

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L243-L250


### 6- `MANAGER_ROLE` not used in `PerpetualAtlanticVault` :

#### Risk : Low

The `MANAGER_ROLE` is set in the constructor of the `PerpetualAtlanticVault` contract but this role is never used inside the contract for any access control task, If this role is not intended to be used in the contract then it should be removed, but if it was supposed to have a certain power for accessing some functions inside the contract then the code should be reviewed to add include this role. 

#### Instance

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L127


### 7- `underlyingSymbol` not set in `PerpetualAtlanticVaultLP` :

#### Risk : Non critical

The `underlyingSymbol` variable is not set in the constructor of `PerpetualAtlanticVaultLP`, there is a string variable that is computed from the concatenation of the collateral token symbol and the suffix "-LP" but this is assigned to the inherited ERC20 symbol variable and `underlyingSymbol` will remain empty as there is no way to set it after deployment.

#### Instance

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L52


### 8- Unused state variable `collateralPrecision` in the PerpetualAtlanticVault contract :

#### Risk : Non critical

The value of `collateralPrecision` is set in the constructor of the PerpetualAtlanticVault contract but its value is never used afterwards in any parts of the contract, if this variable is not intended to be used in the contract then it should be removed, but if it was supposed to used in some calculation in a certain function (as far as i saw it has no use as only 18 decimals tokens are intended to be used) then the code should be reviewed to add this features. 

#### Instance

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L123
