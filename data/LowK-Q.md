https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L339-L348

The contract RdpxV2Core allows an admin user changes a list of addresses of a `addresses` state via a function `setAddresses` includes `perpetualAtlanticVault`, `dopexAMMRouter`, `dpxEthCurvePool` and `rdpxV2ReceiptToken`, so on.

After setting these addresses to a new list, the contract will approve maximum allowance for `perpetualAtlanticVault`, `dopexAMMRouter`, `dpxEthCurvePool`, and `rdpxV2ReceiptToken`.

Everything is okay if the admin calls the first time. But if after that, the admin calls to changes to a new list then the contract was not changed allowance of the old addresses to zero.

This bug will able to increase risks. If any bad thing occurs such as hack accidents in those old addresses will permit the drain of all funds in the contract.

The vulnerable code:

```
function setAddresses(
    address _dopexAMMRouter,
    address _dpxEthCurvePool,
    address _rdpxDecayingBonds,
    address _perpetualAtlanticVault,
    address _perpetualAtlanticVaultLP,
    address _rdpxReserve,
    address _rdpxV2ReceiptToken,
    address _feeDistributor,
    address _reLPContract,
    address _receiptTokenBonds
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_dopexAMMRouter != address(0), 17);
    _validate(_dpxEthCurvePool != address(0), 17);
    _validate(_rdpxDecayingBonds != address(0), 17);
    _validate(_perpetualAtlanticVault != address(0), 17);
    _validate(_perpetualAtlanticVaultLP != address(0), 17);
    _validate(_rdpxReserve != address(0), 17);
    _validate(_rdpxV2ReceiptToken != address(0), 17);
    _validate(_feeDistributor != address(0), 17);
    _validate(_reLPContract != address(0), 17);
    _validate(_receiptTokenBonds != address(0), 17);

    addresses = Addresses({
      dopexAMMRouter: _dopexAMMRouter,
      dpxEthCurvePool: _dpxEthCurvePool,
      rdpxDecayingBonds: _rdpxDecayingBonds,
      perpetualAtlanticVault: _perpetualAtlanticVault,
      perpetualAtlanticVaultLP: _perpetualAtlanticVaultLP,
      rdpxReserve: _rdpxReserve,
      rdpxV2ReceiptToken: _rdpxV2ReceiptToken,
      feeDistributor: _feeDistributor,
      reLPContract: _reLPContract,
      receiptTokenBonds: _receiptTokenBonds
    });
    IERC20WithBurn(weth).approve(
      addresses.perpetualAtlanticVault,
      type(uint256).max
    );
    IERC20WithBurn(weth).approve(addresses.dopexAMMRouter, type(uint256).max);
    IERC20WithBurn(weth).approve(addresses.dpxEthCurvePool, type(uint256).max);
    IERC20WithBurn(weth).approve(
      addresses.rdpxV2ReceiptToken,
      type(uint256).max
    );
    emit LogSetAddresses(addresses);
  }
```

I recommended adding a line to change the allowance of old addresses to zero.