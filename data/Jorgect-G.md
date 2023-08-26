# GAS REPORT

## [G-01] The RpdxV2Core.sol contract is performing aditional calculation in some cases.

The _curveSwap is calculating the minOut always but this depend of minAmount input so if the minAmount is set the minOut is gonna be calculated but never used.

```
file:contracts/core/RdpxV2Core.sol
function _curveSwap(
    uint256 _amount,
    bool _ethToDpxEth,
    bool validate,
    uint256 minAmount
  ) internal returns (uint256 amountOut) {
    ...
    // calculate minimum amount out
    uint256 minOut = _ethToDpxEth
      ? (((_amount * getDpxEthPrice()) / 1e8) -
        (((_amount * getDpxEthPrice()) * slippageTolerance) / 1e16))
      : (((_amount * getEthPrice()) / 1e8) -
        (((_amount * getEthPrice()) * slippageTolerance) / 1e16));

    // Swap the tokens
    amountOut = dpxEthCurvePool.exchange(
      _ethToDpxEth ? int128(int256(a)) : int128(int256(b)),
      _ethToDpxEth ? int128(int256(b)) : int128(int256(a)),
      _amount,
      minAmount > 0 ? minAmount : minOut
    );
  }

```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L515C3-L559C1

### Recomendation

Calculate minOut just when minAmount is set to 0;

## [G-02] The RpdxV2Core.sol contract is performing an adittional call that is not need it.

When the bonds function is called it is returning the owner and is no used, but then is calling ownerOf of function to  get the owner

```
function _transfer(uint256 _rdpxAmount, uint256 _wethAmount, uint256 _bondAmount, uint256 _bondId) internal {
        if (_bondId != 0) {
            (, uint256 expiry, uint256 amount) = IRdpxDecayingBonds(addresses.rdpxDecayingBonds).bonds(_bondId); //@audit (gas) you can get the owner here and not call ownerOf

            _validate(amount >= _rdpxAmount, 1);
            _validate(expiry >= block.timestamp, 2);
            _validate(IRdpxDecayingBonds(addresses.rdpxDecayingBonds).ownerOf(_bondId) == msg.sender, 9);
            ...
}
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L638

### Recomendation

User the owner returned by bonds function insted of call ownerOf.

## [G-03] The payFunding function in PerpetualAtlanticVault.sol contract is reading three time a variable from storge.

The payFunding function in PerpetualAtlanticVault.sol contract is reading two time a variable from storge insted of kepping in a memory variable.

```
function payFunding() external onlyRole(RDPXV2CORE_ROLE) returns (uint256) {
    _whenNotPaused();
    _isEligibleSender();

    _validate(
      totalActiveOptions ==
        fundingPaymentsAccountedFor[latestFundingPaymentPointer],
      6
    );

    collateralToken.safeTransferFrom(
      addresses.rdpxV2Core,
      address(this),
      totalFundingForEpoch[latestFundingPaymentPointer]
    );
    _updateFundingRate(totalFundingForEpoch[latestFundingPaymentPointer]);

    emit PayFunding(
      msg.sender,
      totalFundingForEpoch[latestFundingPaymentPointer],
      latestFundingPaymentPointer
    );

    return (totalFundingForEpoch[latestFundingPaymentPointer]);
  }
```

### Recomendation

Keep in a memory variable  totalFundingForEpoch[latestFundingPaymentPointer] and use it.


