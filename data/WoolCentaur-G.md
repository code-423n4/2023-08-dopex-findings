## [G1] Use external in place of public
Found on line 824 at contracts/core/RdpxV2Core.sol
```
  function bondWithDelegate(
    address _to,
    uint256[] memory _amounts,
    uint256[] memory _delegateIds,
    uint256 rdpxBondId
  ) public returns (uint256 receiptTokenAmount, uint256[] memory) {
```
Found on line 898 at contracts/core/RdpxV2Core.sol
```
  function bond(
    uint256 _amount,
    uint256 rdpxBondId,
    address _to
  ) public returns (uint256 receiptTokenAmount) {
```

**Mitigation**
Change public to external

## [G2] Set local variables in place of multiple function calls
Found on line 669 at contracts/core/RdpxV2Core.sol
`uint256 rdpxAmountInWeth = (_rdpxAmount * getRdpxPrice()) / 1e8;`
Found on line 674 at contracts/core/RdpxV2Core.sol
`getRdpxPrice();`

**Mitigation**
Set a local variable to save the result of `getRdpxPrice()` to.  Saves approx. 650 gas

Found on line 475 at contracts/perp-vault/PerpetualAtlanticVault.sol
`(currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /`
Found on line 481 at contracts/perp-vault/PerpetualAtlanticVault.sol
`(currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /`
Found on line 487 at contracts/perp-vault/PerpetualAtlanticVault.sol
`((currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /`

**Mitigation**
Set a local variable to save the result of `currentFundingRate * (nextFundingPaymentTimestamp() - startTime)` to.  Saves approx. 963 gas

Found on line 512 at contracts/perp-vault/PerpetualAtlanticVault.sol
`(currentFundingRate * (block.timestamp - startTime)) / 1e18`
Found on line 516 at contracts/perp-vault/PerpetualAtlanticVault.sol
`(currentFundingRate * (block.timestamp - startTime)) / 1e18`
Found on line 521 at contracts/perp-vault/PerpetualAtlanticVault.sol
`((currentFundingRate * (block.timestamp - startTime)) / 1e18),`

**Mitigation**
Set a local variable to save the result of `(currentFundingRate * (block.timestamp - startTime)) / 1e18` to.  Saves approx. 490 gas

## [G3] Use internal in place of public
Found on line 276 at contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
  function convertToShares(
    uint256 assets
  ) public view returns (uint256 shares) {
```

**Mitigation**
Change public to internal

## [G4] Check if tokenA or tokenB is zero before transferring
Found on line 168 at contracts/amo/UniV2LiquidityAmo.sol
`IERC20WithBurn(addresses.tokenA).safeTransfer(`
Found on line 172 at contracts/amo/UniV2LiquidityAmo.sol
`IERC20WithBurn(addresses.tokenB).safeTransfer(`

**Description**
When calling `swap()` on UniV2LiquidityAmo.sol, it calls the internal function `_sendTokensToRdpxV2Core()` where it is possible for either the tokenA or tokenB to be zero. As of now, the safeTransfer is always initiated, regardless of whether or not it is actually transferring anything.

**Mitigation**
Implement a check to see if tokenA or tokenB greater than zero. If so, proceed with the `safeTransfer` calls.  Each transfer of `0`, costs approx. 3315 gas.
