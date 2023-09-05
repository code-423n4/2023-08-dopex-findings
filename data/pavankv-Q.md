## 1. Add check to shares in below function to avoid zero :-

**Summary**
In below function try to add zeroness check to shares variable because the function _convertToAssets() called by other function also don't have any check on shares . If shares value is supplied zero by naive user or by mistake even though having larger number of supply(totalSupply) this function _convertToAssets() retruns always ZERO.

**Before**
```solidity
  function _convertToAssets(
    uint256 shares
  ) internal view virtual returns (uint256 assets, uint256 rdpxAmount) {
    uint256 supply = totalSupply;
    return
      (supply == 0)
        ? (shares, 0)
        : (
          shares.mulDivDown(totalCollateral(), supply),
          shares.mulDivDown(_rdpxCollateral, supply)
        );
  }  
```

**After**
```solidity
  function _convertToAssets(
    uint256 shares
  ) internal view virtual returns (uint256 assets, uint256 rdpxAmount) {

    require(shares > 0 ,"ERROR"); //@audit changed here
    uint256 supply = totalSupply;
    
    return
      (supply == 0)
        ? (shares, 0)
        : (
          shares.mulDivDown(totalCollateral(), supply),
          shares.mulDivDown(_rdpxCollateral, supply)
        );
  }  
```

Please look into `//@audit changed here` comment in above function.

code snippet:-
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L219C1-L228C11