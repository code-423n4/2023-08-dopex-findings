# QA Report

## Low Risk Findings

### L-01 - Users can bond without providing WETH

`RdpxV2Core::calculateBondCost()` does not account for precission loss and allows users to bond an amount of `1`, without providing any WETH.

- [RdpxV2Core.sol#L904](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L904)
- [RdpxV2Core.sol#L705](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L705)

### Impact

Users can bond without 1 wei providing any WETH. This may break any assumptions regarding accountability of the Core contract, or may be combined with other issues to achieve a more critical one. This step may also be used in a `for` loop to increase the amount without providing WETH.

### Proof of Concept

Add this test to `tests/rdpxV2-core/Unit.t.sol` and run `forge test --mt "testBondNoEthCost"`:

```solidity
  function testBondNoEthCost() public {
    uint256 initialWethBalance = weth.balanceOf(address(this));

    rdpxV2Core.bond(1, 0, address(this));

    uint256 finalWethBalance = weth.balanceOf(address(this));
    assertEq(initialWethBalance, finalWethBalance);
  }
```

### Recommended Mitigation Steps

Validate that the returned `wethRequired` of the `calculateBondCost()` is greater than 0.

### L-02 - `decreaseAmount` function name is misleading

`RdpxDecayingBonds::decreaseAmount()` does not decrease the bonds amount, but replaces its value. This is misleading and can lead to integration errors if they follow the Natspec: `amount: amount to decrease`.

```solidity
  /// @param amount amount to decrease // @audit
  function decreaseAmount(
    uint256 bondId,
    uint256 amount
  ) public onlyRole(RDPXV2CORE_ROLE) {
    _whenNotPaused();
    bonds[bondId].rdpxAmount = amount; // @audit
  }
```

[RdpxDecayingBonds.sol#L144](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L144)

The only call to the function in the current scope is performed on the `RdpxV2Core` contract.

In this case the `amount` passed to the `decreaseAmount()` function is the expected decreased value, so the whole system works as expected.

```solidity
    IRdpxDecayingBonds(addresses.rdpxDecayingBonds).decreaseAmount(
        _bondId,
        amount - _rdpxAmount
    );
```

[RdpxV2Core.sol#L645](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L645)

Nevertheless, this can lead to future errors.

#### Recommended Mitigation Steps

Change the name of the function to a better name like `updateAmount()`, or refactor the function and its calls to match its description.

### L-03 - `addAssetTotokenReserves()` doesn't check that there is no token with the same symbol

The function checks that the token address is not used, but doesn't check for the symbol.

Different tokens may have the same symbol, which can be troublesome, as certain operations rely on the symbol.

- [RdpxV2Core.sol#L240-L264](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L240-L264)

#### Recommended Mitigation Steps

- Verify that the token symbol is not previously used

### L-04 - The `removeAssetFromtokenReserves()` removes tokens by symbol instead of per address

The `removeAssetFromtokenReserves()` function removes the first token it finds with a specified symbol, which may lead to an error, as multiple tokens can be added with the same symbol.

- [RdpxV2Core.sol#L270-L290](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L270-L290)

#### Recommended Mitigation Steps

Remove tokens by their address instead of by their symbol.

### L-05 - Approvals can't be completely revoked

The `approveContractToSpend()` function requires that the approved amount is > 0. This means that the function is not capable of revoke token approvals by setting them to 0.

This is also troublesome, as it is the expected behavior if a token approval wants to be removed. It can still be mitigated by sending a value of 1.

- [UniV2LiquidityAmo.sol#L133](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L133)
- [RdpxV2Core.sol#L410](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L410)

#### Recommended Mitigation Steps

Remove the unnecessary requirement of `amount > 0`.

### L-06 - `provideFunding()` may be victim to a DOS because of strict equality

`PerpetualAtlanticVault::payFunding()` has an strict equality, which will revert if `totalActiveOptions` is different than `fundingPaymentsAccountedFor[latestFundingPaymentPointer]`.

```solidity
  function payFunding() external onlyRole(RDPXV2CORE_ROLE) returns (uint256) {
    _whenNotPaused();
    _isEligibleSender();

    _validate(
@>    totalActiveOptions ==                                       // @audit
        fundingPaymentsAccountedFor[latestFundingPaymentPointer],
      6
    );
```

[PerpetualAtlanticVault.sol#L372-L380](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L372-L380)

`payFunding()` is called by `RdpxV2Core::provideFunding()`, which will make it revert if than strict equality may be broken.

#### Recommended Mitigation Steps

Change the strict equality to a comparison operator.

## Non-Critical Findings

### NC-01 - Use camelCase for function names

Functions as `addAssetTotokenReserves()` are not correctly formatted as camelCase presumably from the fact that they modify the `tokenReserves` variable.

Nevertheless they should use camelCase for consistency with the rest of the codebase

- [RdpxV2Core.sol#L240](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L240)
- [RdpxV2Core.sol#L270](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L270)
- [ReLPContract.sol#L90](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L90)

#### Recommended Mitigation Steps

Replace the function names with the camelCase version as `addAssetToTokenReserves()` for example.

### NC-02 - Unnecessary role check

Role is checked both on the modifier and the function body. Consider removing one:

```solidity
  function mint(
    address to,
    uint256 expiry,
    uint256 rdpxAmount
  ) external onlyRole(MINTER_ROLE) { // @audit
    _whenNotPaused();
    require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter"); // @audit
```

[RdpxDecayingBonds.sol#L114-L120](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L114-L120)

### NC-03 - `RDPXV2CORE_ROLE` is not assigned in `PerpetualAtlanticVault`

While the `DEFAULT_ADMIN_ROLE`, and `MANAGER_ROLE` are assigned during the `constructor()`, the `RDPXV2CORE_ROLE` is not.

Any call from the Core contract to the vault will revert until this value is set, so it would be advised to assign this role in the constructor as well.

[PerpetualAtlanticVault.sol#L113-L128](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L113-L128)

### NC-03 - Unnecessary struct attribute prefix

`TokenAInfo` attributes are all prefixed with `tokenA`. This is unnecessary, as they are known to be from a `TokenAInfo` struct. Consider removing the prefix:

```solidity
  struct TokenAInfo {
    // tokenA reserves
    uint256 tokenAReserve;
    // rdpx price
    uint256 tokenAPrice;
    // tokenA LP reserves
    uint256 tokenALpReserve;
  }
```

[ReLPContract.sol#L51-L58](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L51-L58)

### NC-04 - Repeated expression

Long expression that are repeated can lead to errors if refactored, and are harder to read.

Consider creating a variable to store the expression `(currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) / 1e18`, which is repeated 3 times:

```solidity
  collateralToken.safeTransfer(
    addresses.perpetualAtlanticVaultLP,
    (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) / // @audit
      1e18
  );

  IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP)
    .addProceeds(
      (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) / // @audit
        1e18
    );

  emit FundingPaid(
    msg.sender,
    ((currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) / // @audit
      1e18),
    latestFundingPaymentPointer
  );
```