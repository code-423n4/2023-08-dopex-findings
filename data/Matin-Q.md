## Summary

### Low-Risk Issues

|  | Issue | Instances |
| --- | --- | --- |
| [L‑01] | Early users can modify the underlying assets' unit share price | 1 |
| [L‑02] | Multiple roles needed to invoke the function settle() | 1 |

Total: 2 instances over 2 issues

### Non-critical Issues

|  | Issue | Instances |
| --- | --- | --- |
| [N‑01] | Wrong natspec descriptions | 1 |

Total: 8 instances over 3 issues

Note: The above tables were created, considering the automatic findings and thus, those are not included.

---

## Low-Risk Issues

### [L‑01] **Early users can modify the underlying assets' unit share price**

Summary: 

ERC4626, an extension of ERC20, is a standard that is mostly used in yield-bearing tokens. The contract of an ERC4626 token itself is an ERC20 which serves as a "shares" token and has an underlying asset which is another ERC20. The idea behind this is in such a way that the users deposit their assets and get corresponding shares. The function `convertToShares()` is responsible for returning the corresponding share of a user with respect to his deposited assets.

```Solidity
    function convertToShares(
    uint256 assets
  ) public view returns (uint256 shares) {
    uint256 supply = totalSupply;
    uint256 rdpxPriceInAlphaToken = perpetualAtlanticVault.getUnderlyingPrice();

    uint256 totalVaultCollateral = totalCollateral() +
      ((_rdpxCollateral * rdpxPriceInAlphaToken) / 1e8);
    return
      supply == 0 ? assets : assets.mulDivDown(supply, totalVaultCollateral);
  }
```

One of the main problems with these tokens lies in the fact that the first depositor can manipulate the unit share price in a bad way. The ratio of $\large\dfrac{totalSupply}{totalAssets}$ determines the corresponding share of a deposited asset. As it is clear from the ratio if the first user deposits 1 Wei, will get 1 Wei worth of share. Now consider a scenario with the preceding steps:

1 - The same bad guy deposits $\large 10000 \times 10^{18} - 1$ amount of underlying assets.
2 - As a consequence, the abovementioned ratio skyrockets and won't be a 1:1 pair anymore.
$\large \dfrac{10000 \times 10^{18} - 1 + 1}{1} =  10^{22}$
3 - The later users who deposit large amounts of assets (e.g. $\large 19999 \times 10^{18} - 1 $) will get 1 Wei worth of shares.
4 - Thus, if they call `redeem()` after depositing, they will immediately lose $\large 9999 \times 10^{18} - 1$ (nearly half of) the assets.

*There are  instances of this issue:*

```solidity
 function convertToShares(
    uint256 assets
  ) public view returns (uint256 shares) {
    uint256 supply = totalSupply;
    uint256 rdpxPriceInAlphaToken = perpetualAtlanticVault.getUnderlyingPrice();

    uint256 totalVaultCollateral = totalCollateral() +
      ((_rdpxCollateral * rdpxPriceInAlphaToken) / 1e8);
    return
      supply == 0 ? assets : assets.mulDivDown(supply, totalVaultCollateral);
  }
```

Link(s): https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L274C1-L284C4

### [L‑02] **Multiple roles needed to invoke the function `settle()`**

Summary: 

The function `settle()` which is used for option settlement needs multiple roles to be called. First of all, the caller has to be the `RdpxV2Core` contract. Inside the function `settle()` we see that it requires another role too. The function `_isEligibleSender()` checks another role of the caller which shows this function needs multiple roles to be called. 

*There are  instances of this issue:*

```solidity
 function settle(
    uint256[] memory optionIds
  )
    external
    nonReentrant
    onlyRole(RDPXV2CORE_ROLE)
    returns (uint256 ethAmount, uint256 rdpxAmount)
  {
    _whenNotPaused();
    _isEligibleSender();

    updateFunding();
...
```

Link(s): https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L315C3-L326C21

---

## Non-critical Issues

### [N‑01] Wrong natspec descriptions

Summary: There are some descriptions irrelevant and not helpful to the end-user or developer or there might be typos and wrongs in the natspecs provided in the contracts.

*There are  instances of this issue:*

```solidity
/// @dev the pointer to the lattest funding payment timestamp
  /// @notice Explain to an end user what this does
  /// @dev Explain to a developer any extra details
  /// @return Documents the return variables of a contract’s function state variable
  /// @inheritdoc	IPerpetualAtlanticVault
  uint256 public latestFundingPaymentPointer = 0;
```

Link(s): https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L84-L89