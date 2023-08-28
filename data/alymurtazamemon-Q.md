# Dopex - Quality Assurance Report

# Low Risk Findings

## Table Of Content

| Number                                                                                                           | Issue                                                                                                 | Instances |
| :--------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------- | --------: |
| [L-01](#l-01---missing-validation-for-address0-in-the-constructors-or-in-the-functions-changing-state-variables) | Missing validation for `address(0)` in the constructors or in the functions changing state variables. |         4 |
| [L-02](#l-02---use-proper-error-message-with-contract-location-trace)                                            | Use proper `error` message with contract location trace                                               |        34 |
| [L-03](#l-03---hardcoded-address-will-do-issue-while-using-the-code-on-multiple-networks)                        | `hardcoded` address will do issue while using the code on multiple networks                           |         1 |
| [L-04](#l-04---typo-in-the-validation-of-addresses)                                                              | Typo in the validation of addresses                                                                   |         1 |

### [L-01] - Missing validation for `address(0)` in the constructors or in the functions changing state variables.

**Details**

It is recommended to check for `address(0)` when updating state variables through constructor or functions. This can save the protocol from accidental updates.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[RdpxV2Core.sol - Line 126](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L126)

```diff
+   error RdpxV2Core_InvalidAddress(address _weth);

constructor(address _weth) {
    _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
+   // * @audit zero address check missing
+   if(_weth == address(0)) revert RdpxV2Core_InvalidAddress(_weth);
    weth = _weth;
```

[RdpxV2Core.sol - Line 422](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L422)

```diff
+   // * @audit zero address check missing
+   if(_addr == address(0)) revert RdpxV2Core_InvalidAddress(_addr);
    _addToContractWhitelist(_addr);
```

[UniV3LiquidityAmo.sol - Line 76 - 78](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L76-L78)

```diff
    // define this error in the errors section
+   error UniV3LiquidityAmo_InvalidAddress();

+   // * @audit zero address validation missing
+   if(_rdpx == address(0) || _rdpxV2Core == address(0)) {
+       revert UniV3LiquidityAmo_InvalidAddress();
+   }

    rdpx = _rdpx;
    rdpxV2Core = _rdpxV2Core;
```

[PerpetualAtlanticVaultLP.sol - Line 83 - 84](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L83-L84)

```diff
-   require(
-       _perpetualAtlanticVault != address(0) || _rdpx != address(0),
-       "ZERO_ADDRESS"
-   );
+   require(
+       _perpetualAtlanticVault != address(0) ||
+           _rdpx != address(0) ||
+           _rdpxRdpxV2Core != address(0) ||
+           _collateral != address(0),
+       "ZERO_ADDRESS"
+   );
```

</details>

### [L-02] - Use proper `error` message with contract location trace.

**Details**

Throughout the code we are using the `_validate(bool _clause, uint256 _errorCode)` function and throwing the error code when something went wrong.

The actual purpose of using a revert statement is the let the user know what type of error has occurred and where it has occurred. And we also should make errors user-friendly so even non-technical people can understand.

Including the error trace (where the error occurred) is also essential for developers, throwing the same error from multiple files will create ambiguity about where the error is thrown from. Instead of looking into the error messages it is very convenient to use proper error messages with location trace (see the example below).

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[RdpxV2Core.sol - Line 183](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L183)

```solidity
    // define these in the errors section
+   error RdpxV2Core_InvalidAddress(address _weth);
+   error RdpxV2Core_InvalidParameters();
+   error RdpxV2Core_AssetNotFound();
+   error RdpxV2Core_ZeroAddress();
+   error RdpxV2Core_InvalidAmount();
+   error RdpxV2Core__AmountGreaterThatMaxPermissibleAmount();
+   error RdpxV2Core_InsufficientBondAmount();
+   error RdpxV2Core_BondHasExpired();
+   error RdpxV2Core_SenderIsNotTheOwner();
+   error RdpxV2Core_NotEnoughETHAvailableFromTheDelegate();
+   error RdpxV2Core_FundingAlreadyPaidForTheEpoch();
+   error RdpxV2Core_FeeCannotBeMoreThan100();
```

```diff
-   _validate(_rdpxBurnPercentage > 0, 3);
    // == is also a gas vise benefit than checking >
+   if(_rdpxBurnPercentage == 0) revert RdpxV2Core_InvalidParameters();
```

[RdpxV2Core.sol - Line 196](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L196)

```diff
-   // _validate(_rdpxFeePercentage > 0, 3);
+   if(_rdpxFeePercentage == 0) revert RdpxV2Core_InvalidParameters();
```

[RdpxV2Core.sol - Line 231](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L231)

```diff
-   _validate(_bondMaturity > 0, 3);
+   if(_bondMaturity == 0) revert RdpxV2Core_InvalidParameters();
```

[RdpxV2Core.sol - Line 274](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L274)

```diff
-   _validate(index != 0, 18);
+   if(index == 0) revert RdpxV2Core_AssetNotFound();
```

[RdpxV2Core.sol - Line 316 - 325](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L316-L325)

```diff
-   _validate(_dopexAMMRouter != address(0), 17);
-   _validate(_dpxEthCurvePool != address(0), 17);
-   _validate(_rdpxDecayingBonds != address(0), 17);
-   _validate(_perpetualAtlanticVault != address(0), 17);
-   _validate(_perpetualAtlanticVaultLP != address(0), 17);
-   _validate(_rdpxReserve != address(0), 17);
-   _validate(_rdpxV2ReceiptToken != address(0), 17);
-   _validate(_feeDistributor != address(0), 17);
-   _validate(_reLPContract != address(0), 17);
-   _validate(_receiptTokenBonds != address(0), 17);

+   if (_dopexAMMRouter == address(0) || _dpxEthCurvePool == address(0) || _rdpxDecayingBonds == address(0) || _perpetualAtlanticVault == address(0) || _perpetualAtlanticVaultLP == address(0) || _rdpxReserve == address(0) || _rdpxV2ReceiptToken == address(0) || _feeDistributor == address(0) || _reLPContract == address(0) || _receiptTokenBonds == address(0)) revert RdpxV2Core_ZeroAddress();
```

[RdpxV2Core.sol - Line 358](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L358)

```diff
-   _validate(_rdpxPriceOracle != address(0), 17);
-   _validate(_dpxEthPriceOracle != address(0), 17);
+   if(_rdpxPriceOracle == address(0) || _dpxEthPriceOracle == address(0)) revert RdpxV2Core_ZeroAddress();
```

[RdpxV2Core.sol - Line 379](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L379)

```diff
-   _validate(_addr != address(0), 17);
+   if(_addr == address(0)) revert RdpxV2Core_ZeroAddress();
```

[RdpxV2Core.sol - Line 391](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L391)

```diff
-   _validate(_index < amoAddresses.length, 18);
+   if(_index > amoAddresses.length) revert RdpxV2Core_AssetNotFound();
```

[RdpxV2Core.sol - Line 408 - 410](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L408-L410)

```diff
-   _validate(_token != address(0), 17);
-   _validate(_spender != address(0), 17);
-   _validate(_amount > 0, 17);

+   if(_token == address(0) || _spender == address(0)) revert RdpxV2Core_ZeroAddress();
+
+   if(_amount == 0) revert RdpxV2Core_InvalidAmount();
```

[RdpxV2Core.sol - Line 444](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L444)

```diff
-   _validate(_bondDiscountFactor > 0, 3);
+   if(_bondDiscountFactor == 0) revert RdpxV2Core_InvalidParameters();
```

[RdpxV2Core.sol - Line 458](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L458)

```diff
-   _validate(_slippageTolerance > 0, 3);
+   if(_slippageTolerance == 0) revert RdpxV2Core_InvalidParameters();
```

[RdpxV2Core.sol - Line 533 - 541](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L533-L541)

```diff
-   _ethToDpxEth
-       ? _validate(
-           ethBalance + _amount <= (ethBalance + dpxEthBalance) / 2,
-           14
-       )
-       : _validate(
-           dpxEthBalance + _amount <= (ethBalance + dpxEthBalance) / 2,
-           14
-       );

+   if (_ethToDpxEth) {
+       if (ethBalance + _amount > (ethBalance + dpxEthBalance) / 2)
+           revert RdpxV2Core__AmountGreaterThatMaxPermissibleAmount();
+   } else {
+       if (dpxEthBalance + _amount > (ethBalance + dpxEthBalance) / 2)
+           revert RdpxV2Core__AmountGreaterThatMaxPermissibleAmount();
+   }
```

[RdpxV2Core.sol - Line 635 - 641](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L635-L641)

```diff
-   _validate(amount >= _rdpxAmount, 1);
+   if(amount < _rdpxAmount) revert RdpxV2Core_InsufficientBondAmount();

-   _validate(expiry >= block.timestamp, 2);
+   if(expiry < block.timestamp) revert RdpxV2Core_BondHasExpired();

-   _validate(
-       IRdpxDecayingBonds(addresses.rdpxDecayingBonds).ownerOf(_bondId) ==
-       msg.sender,
-       9
-   );
+   if(IRdpxDecayingBonds(addresses.rdpxDecayingBonds).ownerOf(_bondId) !=
+       msg.sender) revert RdpxV2Core_SenderIsNotTheOwner();
```

[RdpxV2Core.sol - Line 716](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L716)

```diff
-   _validate(delegate.amount - delegate.activeCollateral >= wethRequired, 5);
+   if(delegate.amount - delegate.activeCollateral < wethRequired)
+       revert RdpxV2Core_NotEnoughETHAvailableFromTheDelegate();
```

[RdpxV2Core.sol - Line 798](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L798)

```diff
-   _validate(fundingPaidFor[pointer] == false, 16);
+   if(fundingPaidFor[pointer]) revert RdpxV2Core_FundingAlreadyPaidForTheEpoch();
```

[RdpxV2Core.sol - Line 827](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L827)

```diff
-   _validate(_amounts.length == _delegateIds.length, 3);
+   if(_amounts.length != _delegateIds.length) revert RdpxV2Core_InvalidParameters();
```

[RdpxV2Core.sol - Line 838](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L838)

```diff
-   _validate(_amounts[i] > 0, 4);
+   if(_amounts[i] == 0) revert RdpxV2Core_InvalidAmount();
```

[RdpxV2Core.sol - Line 901](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L901)

```diff
-   _validate(_amount > 0, 4);
+   if(_amount == 0) revert RdpxV2Core_InvalidAmount();
```

[RdpxV2Core.sol - Line 947 - 951](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L947-L951)

```diff
// fee less than 100%
-   _validate(_fee < 100e8, 8);
+   if(_fee >= 100e8) revert RdpxV2Core_FeeCannotBeMoreThan100();

// amount greater than 0.01 WETH
-   _validate(_amount > 1e16, 4);
+   if(_amount <= 1e16) revert RdpxV2Core_InvalidAmount();

// fee greater than 1%
-   _validate(_fee >= 1e8, 8);
+   if(_fee < 1e8) revert RdpxV2Core_FeeCannotBeMoreThan100();
```

// here are the remaining locations you should update similarly  
[RdpxV2Core.sol - Line 979](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L979)

[RdpxV2Core.sol - Line 981](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L981)

[RdpxV2Core.sol - Line 984](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L984)

[RdpxV2Core.sol - Line 1021 - 1028](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1021-L1028)

[RdpxV2Core.sol - Line 1057](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1057)

[RdpxV2Core.sol - Line 1087](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1087)

[RdpxV2Core.sol - Line 1140](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1140)

[RdpxV2Core.sol - Line 1167](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1167)

[PerpetualAtlanticVault.sol - Line 119](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L119)

[PerpetualAtlanticVault.sol - Line 190 - 196](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L190-L196)

[PerpetualAtlanticVault.sol - Line 265](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L265)

[PerpetualAtlanticVault.sol - Line 278 - 281](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L278-L281)

[PerpetualAtlanticVault.sol - Line 333](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L333)

[PerpetualAtlanticVault.sol - Line 376 - 380](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L376-L380)

[PerpetualAtlanticVault.sol - Line 414 - 418](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L414-L418)

</details>

### [L-03] - `hardcoded` address will do issue while using the code on multiple networks.

**Details**

I have confirmed that they will only use this on Arbitrum and they have no any future plans to use it on other networks but still it would be better to pass them through constructor.

Also comment out the address for auditors to confirm them.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[UniV3LiquidityAmo.sol - Line 82 - 88](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L82-L88)

</details>

### [L-04] - Typo in the validation of addresses.

**Details**

```solidity
require(
    _perpetualAtlanticVault != address(0) || _rdpx != address(0),
    "ZERO_ADDRESS"
);
```

Here these both addresses should not be `address(0)` but we have address the OR `||` operator here.

[PerpetualAtlanticVaultLP.sol - Line 95](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L95)

# Non Critical Findings

## Table Of Content

| Number                                                                                                                                       | Issue                                                                                                                             | Instances |
| :------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------- | --------: |
| [N-01](#n-01---use-natspec-comments-for-events-state-variables-constructor-functions-and-all-other-things-which-will-be-included-in-the-abi) | Use `Natspec` comments for events, state variables, constructor, functions and all other things which will be included in the ABI |         7 |
| [N-02](#n-02---set-the-internal-layout-of-contracts-interfaces-and-libraries)                                                                | Set the Internal Layout of Contracts, Interfaces, and Libraries                                                                   |         6 |
| [N-03](#n-03---safemath-library-is-not-needed-for-versions-greater-than-or-equal-to-080)                                                     | `SafeMath` library is not needed for versions greater than or equal to `0.8.0`                                                    |         1 |
| [N-04](#n-04---use-natspec-comments-for-smart-contracts-interfaces-and-libraries)                                                            | Use `Natspec` comments for smart contracts, interfaces and libraries                                                              |         4 |
| [N-05](#n-05---variables-should-follow-the-naming-convention)                                                                                | Variables should follow the naming convention                                                                                     |        12 |
| [N-06](#n-06---unnecessary-check-for-minter_role)                                                                                            | Unnecessary Check for `MINTER_ROLE`                                                                                               |         1 |
| [N-07](#n-07---unnecessary-imports)                                                                                                          | Unnecessary Imports                                                                                                               |         1 |
| [N-08](#n-08---developers-forgot-to-expain-these-comments)                                                                                   | Developers forgot to expain these comments                                                                                        |         1 |
| [N-09](#n-09---internal-functions-and-variables-should-follow-the-convention)                                                                | Internal functions and variables should follow the convention                                                                     |         1 |

### [N-01] - Use `Natspec` comments for events, state variables, constructor, functions and all other things which will be included in the ABI.

**Details**

It is recommended by solidity docs that solidity contracts should be fully annotated using NatSpec for all public interfaces (everything in the ABI).

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[UniV2LiquidityAmo.sol - Line 22 - 422](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L22-L422)

[UniV3LiquidityAmo.sol - Line 29 - 379](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L29-L379)

[RdpxV2Bond.sol - Line 18 - 66](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L18-L66)

[RdpxDecayingBonds.sol - Line 25 - 183](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L25-L183)

[DpxEthToken.sol - Line 19 - 62](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L19-L62)

[PerpetualAtlanticVault.sol - Line 38 - 654](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L38-L654)

[PerpetualAtlanticVaultLP.sol - Line 22 - 301](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L22-L301)

</details>

**Recommended Mitigation Steps**

Add the `Natspec` comments in the above mentioned functions.

### [N-02] - Set the Internal Layout of Contracts, Interfaces, and Libraries.

**Details**

According to the Solidity [Style Guide](https://docs.soliditylang.org/en/v0.8.21/style-guide.html#style-guide) consistency in the projects layout is very important. If all projects apply the style guides provided by solidity docs then understanding the projects and its code will be much easier for developers, and auditors.

Solidity docs;

> A style guide is about consistency. Consistency with this style guide is important. Consistency within a project is more important. Consistency within one module or function is most important.

According to the solidity styles guide;

Inside each contract, library or interface, this order should be followed:

1. Type declarations
2. State variables
3. Events
4. Errors
5. Modifiers
6. Functions

And this should be the order of functions:

-   constructor
-   receive function (if exists)
-   fallback function (if exists)
-   external
-   public
-   internal
-   private
-   View and pure functions last

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[RdpxV2Core.sol - Line 33 - 1287](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L33-L1287)

[UniV2LiquidityAmo.sol - Line 22 - 422](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L22-L422)

[UniV3LiquidityAmo.sol - Line 29 - 379](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L29-L379)

[RdpxDecayingBonds.sol - Line 25 - 183](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L25-L183)

[PerpetualAtlanticVault.sol - Line 38 - 654](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L38-L654)

[PerpetualAtlanticVaultLP.sol - Line 22 - 301](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L22-L301)

</details>

**Recommended Mitigation Steps**

Apply the recommeded layout sturcture in all mentioned contracts according to solidity styles guide.

### [N-03] - `SafeMath` library is not needed for versions greater than or equal to `0.8.0`.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[UniV2LiquidityAmo.sol - Line 24](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L24)

</details>

**Recommended Mitigation Steps**

Use just regular arithematic operations, these are now completely handled by solidity compiler which throws error on underflow and overflow.

### [N-04] - Use `Natspec` comments for smart contracts, interfaces and libraries.

**Details**

It is recommended by solidity docs that solidity contracts should be fully annotated using NatSpec for all public interfaces (everything in the ABI).

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[UniV3LiquidityAmo.sol - Line 22](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L22)

[UniV3LiquidityAmo.sol - Line 28](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L28)

[RdpxV2Bond.sol - Line 11](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L11)

[RdpxDecayingBonds.sol - Line 18](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L18)

</details>

**Recommended Mitigation Steps**

Add the `Natspec` comments in the above mentioned contracts.

### [N-05] - Variables should follow the naming convention.

**Details**

State Variables, local variables, parameters names should use mixedCase (except contant). Examples: `getBalance`, `transfer`, `verifyOwner`, `addMember`, `changeOwner`.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[UniV3LiquidityAmo.sol - Line 35 - 37](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L35-L37)

[UniV3LiquidityAmo.sol - Line 41 - 46](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L41-L46)

[UniV3LiquidityAmo.sol - Line 63 - 66](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L63-L66)

[UniV3LiquidityAmo.sol - Line 95](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L95)

[UniV3LiquidityAmo.sol - Line 100](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L100)

[UniV3LiquidityAmo.sol - Line 121](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L121)

[UniV3LiquidityAmo.sol - Line 123](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L123)

[UniV3LiquidityAmo.sol - Line 143](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L143)

[UniV3LiquidityAmo.sol - Line 220](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L220)

[UniV3LiquidityAmo.sol - Line 277](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L277)

[UniV3LiquidityAmo.sol - Line 289](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L289)

[UniV3LiquidityAmo.sol - Line 326](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L326)

</details>

**Recommended Mitigation Steps**

Update these variables' names to `mixedCase`

### [N-06] - Unnecessary Check for `MINTER_ROLE`.

**Details**

```solidity
function mint(
    address to,
    uint256 expiry,
    uint256 rdpxAmount
) external onlyRole(MINTER_ROLE) {
```

Function already has the check for `MINTER_ROLE` the inside the code we do not need to check it;

```solidity
// * @audit unnecessary statement, role is already check through modifier
require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
```

[RdpxDecayingBonds.sol - Line 120](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L120)

### [N-07] - Unnecessary Imports.

**Details**

Below mentioned imports are unnecessary.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

ERC721 is already imported in ERC721Enumerable and ERC721Burnable.

[PerpetualAtlanticVault.sol - Line 32](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L32)

</details>

**Recommended Mitigation Steps**

Remove the unnecessary imports.

### [N-08] - Developers forgot to expain these comments.

**Details**

```solidity
// * @audit explain these natspecs
/// @dev the pointer to the lattest funding payment timestamp
/// @notice Explain to an end user what this does
/// @dev Explain to a developer any extra details
/// @return Documents the return variables of a contract’s function state variable
/// @inheritdoc	IPerpetualAtlanticVault
uint256 public latestFundingPaymentPointer = 0;
```

[PerpetualAtlanticVault.sol - Line 84 - 89](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L84-L89)

### [N-09] - Internal functions and variables should follow the convention.

**Details**

Underscore Prefix for Non-external Functions and Variables

-   `_singleLeadingUnderscore`

This convention is suggested for non-external functions and state variables (`private` or `internal`).

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[PerpetualAtlanticVaultLP.sol - Line 286](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L286)

```diff
-   function beforeWithdraw(uint256 assets, uint256 /*shares*/) internal
+   function _beforeWithdraw(uint256 assets, uint256 /*shares*/) internal
```

</details>

**Recommended Mitigation Steps**

Apply the convention to these functions and variables.