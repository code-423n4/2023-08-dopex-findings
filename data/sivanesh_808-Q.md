| Issue Count | Issues | Instances |  
|----------|----------|----------|
| [L-1]  |``abi.encodePacked()`` should not be used with dynamic types when passing the result to a hash function such as ``keccak256()``  | 2  |
| [L-2]  | Unused EVENT.  | 1 |
| [L-3]  | Unused Emit Functions.| 22  |
| [L-4]  | Large approvals may not work with some ERC20 tokens| 3   |
| [L-5]  | ``approve`` can revert if the current approval is not zero | 3 |
| [L-6]  | Return values of ``approve`` not checked | 3 |
| [L-7]  | Lack of two-step update for critical addresses| 5   | 


## [L-1] ``abi.encodePacked()`` should not be used with dynamic types when passing the result to a hash function such as ``keccak256()``

Use ``abi.encode()`` instead which will pad items to 32 bytes, which will prevent hash collisions ``(e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456),`` but ``abi.encode(0x123,0x456) => 0x0...1230...456)``. “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32()




```diff
FILE: contracts/amo/UniV3LiquidityAmo.sol


    106.  keccak256(abi.encodePacked(address(this), _tickLower, _tickUpper))


```


https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L106C1-L107C1


```diff
FILE: contracts/core/RdpxV2Core.sol


    1141.        keccak256(abi.encodePacked(asset.tokenSymbol)) ==
    1142.           keccak256(abi.encodePacked(_token)),

```


https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1141C1-L1142C45


## [L-2] Unused EVENT

Within the contract, there exists an event declaration ``event EmergencyWithdraw(address sender);`` that remains unused throughout the entire contract. By eliminating this function, we can conserve gas resources.




```diff
FILE: contracts/decaying-bonds/RdpxDecayingBonds.sol


  53. event EmergencyWithdraw(address sender);


```


https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L53C1-L54C1

## [L-3] Unused Emit Functions

In this contract, I have observed that 22 emit functions have been utilized without declaring the corresponding event functions. This redundancy will inflate deployment gas costs and should be eliminated as it serves no purpose in the code. It's essential to either remove the unused emit functions or declare the associated event functions to maintain code clarity and efficiency.




```diff
FILE: contracts/core/RdpxV2Core.sol

172.     emit LogEmergencyWithdraw(msg.sender, tokens);
185.    emit LogSetRdpxBurnPercentage(_rdpxBurnPercentage);
198.    emit LogSetRdpxFeePercentage(_rdpxFeePercentage);
208.    emit LogSetIsReLPActive(_isReLPActive);
220.    emit LogSetputOptionsRequired(_putOptionsRequired);
223.    emit LogSetBondMaturity(_bondMaturity);
263.    emit LogAssetAddedTotokenReserves(_asset, _assetSymbol);
289.    emit LogAssetRemovedFromtokenReserves(_assetSymbol, index);
349.    emit LogSetAddresses(addresses);
370.    emit LogSetPricingOracleAddresses(pricingOracleAddresses);
447.    emit LogSetBondDiscountFactor(_bondDiscountFactor);
461.    emit LogSetSlippageTolerance(_slippageTolerance);
782.    emit LogSettle(optionIds);
807.    emit LogProvideFunding(pointer, fundingAmount);
875.    emit LogBondWithDelegate(
932.    emit LogBond(rdpxRequired, wethRequired, receiptTokenAmount);
966.    emit LogAddToDelegate(_amount, _fee, delegates.length - 1);
989.    emit LogDelegateWithdraw(delegateId, amountWithdrawn);
1007.    emit LogSync();
1041.    emit LogRedeem(to, receiptTokenAmount);
1069.    emit LogUpperDepeg(_amount, wethReceived);
1123.    emit LogLowerDepeg(_rdpxAmount, _wethAmount, dpxEthReceived);



```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol

## [L-4] Large approvals may not work with some ERC20 tokens


Not all ``IERC20`` implementations are totally compliant, and some ``(e.g UNI, COMP)`` may fail if the valued passed is larger than ``uint96``

Source:https://github.com/d-xo/weird-erc20#revert-on-large-approvals--transfers

```diff
FILE: contracts/amo/UniV2LiquidityAmo.sol

134.     IERC20WithBurn(_token).approve(_spender, _amount)

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L134C5-L134C55


```diff
 FILE: contracts/amo/UniV3LiquidityAmo.sol

 150.      IERC20WithBurn(_token).approve(_target, _amount);

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L150C1-L151C1



```diff
FILE: contracts/core/RdpxV2Core.sol

411.    IERC20WithBurn(_token).approve(_spender, _amount);

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L411


## [L-5] ``approve`` can revert if the current approval is not zero


Some tokens like USDT check for the current approval, and they revert if it's not zero. While Tether is known to do this, it applies to other tokens as well, which are trying to protect against this attack vector.

Source: https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/edit#heading=h.m9fhqynw2xvt

```diff
FILE: contracts/amo/UniV2LiquidityAmo.sol

134.     IERC20WithBurn(_token).approve(_spender, _amount)

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L134C5-L134C55


```diff
FILE: contracts/amo/UniV3LiquidityAmo.sol

150.      IERC20WithBurn(_token).approve(_target, _amount);

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L150C1-L151C1



```diff
FILE: contracts/core/RdpxV2Core.sol

411.    IERC20WithBurn(_token).approve(_spender, _amount);

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L411

### Recommended Mitigation Steps

Consider always calling token.safeApprove(addr, 0) before changing the approval, or better yet, use safeIncreaseAllowance/safeDecreaseAllowance.

## [L-6] Return values of ``approve`` not checked


Not all ``IERC20`` implementations ``revert`` when there's a failure in ``approve``. The function signature has a boolean return value and they indicate errors that way instead.

By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything.

```diff
FILE: contracts/amo/UniV2LiquidityAmo.sol

134.     IERC20WithBurn(_token).approve(_spender, _amount)

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L134C5-L134C55


```diff
FILE: contracts/amo/UniV3LiquidityAmo.sol

150.      IERC20WithBurn(_token).approve(_target, _amount);

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L150C1-L151C1



```diff
FILE: contracts/core/RdpxV2Core.sol

411.    IERC20WithBurn(_token).approve(_spender, _amount);

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L411


## [L-7] Lack of two-step update for critical addresses


Add a two-step process for any critical address changes.


```diff
FILE: contracts/amo/UniV2LiquidityAmo.sol

74.  function setAddresses(
    address _tokenA,
    address _tokenB,
    address _pair,
    address _rdpxV2Core,
    address _rdpxOracle,
    address _ammFactory,
    address _ammRouter
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(
      _tokenA != address(0) &&
        _tokenB != address(0) &&
        _pair != address(0) &&
        _rdpxV2Core != address(0) &&
        _rdpxOracle != address(0) &&
        _ammFactory != address(0) &&
        _ammRouter != address(0),
      "reLPContract: address cannot be 0"
    );
    addresses = Addresses({
      tokenA: _tokenA,
      tokenB: _tokenB,
      pair: _pair,
      rdpxV2Core: _rdpxV2Core,
      rdpxOracle: _rdpxOracle,
      ammFactory: _ammFactory,
      ammRouter: _ammRouter
    });
  }

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L74C2-L102C4

```diff
FILE: contracts/core/RdpxV2Core.sol

304. function setAddresses(
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

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L304C2-L350C4


```diff
FILE: contracts/core/RdpxV2Core.sol

358.function setPricingOracleAddresses(
    address _rdpxPriceOracle,
    address _dpxEthPriceOracle
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_rdpxPriceOracle != address(0), 17);
    _validate(_dpxEthPriceOracle != address(0), 17);

    pricingOracleAddresses = PricingOracleAddresses({
      rdpxPriceOracle: _rdpxPriceOracle,
      dpxEthPriceOracle: _dpxEthPriceOracle
    });


```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L358C3-L369C1


```diff
FILE: contracts/perp-vault/PerpetualAtlanticVault.sol

181. function setAddresses(
    address _optionPricing,
    address _assetPriceOracle,
    address _volatilityOracle,
    address _feeDistributor,
    address _rdpx,
    address _perpetualAtlanticVaultLP,
    address _rdpxV2Core
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_optionPricing != address(0), 1);
    _validate(_assetPriceOracle != address(0), 1);
    _validate(_volatilityOracle != address(0), 1);
    _validate(_feeDistributor != address(0), 1);
    _validate(_rdpx != address(0), 1);
    _validate(_perpetualAtlanticVaultLP != address(0), 1);
    _validate(_rdpxV2Core != address(0), 1);

    addresses = Addresses({
      optionPricing: _optionPricing,
      assetPriceOracle: _assetPriceOracle,
      volatilityOracle: _volatilityOracle,
      feeDistributor: _feeDistributor,
      rdpx: _rdpx,
      perpetualAtlanticVaultLP: _perpetualAtlanticVaultLP,
      rdpxV2Core: _rdpxV2Core
    });
    collateralToken.safeApprove(
      addresses.perpetualAtlanticVaultLP,
      type(uint256).max
    );
    emit AddressesSet(addresses);
  }



```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L181C3-L213C1


```diff
FILE: contracts/perp-vault/PerpetualAtlanticVault.sol

115. function setAddresses(
    address _tokenA,
    address _tokenB,
    address _pair,
    address _rdpxV2Core,
    address _tokenAReserve,
    address _amo,
    address _rdpxOracle,
    address _ammFactory,
    address _ammRouter
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(
      _tokenA != address(0) &&
        _tokenB != address(0) &&
        _pair != address(0) &&
        _rdpxV2Core != address(0) &&
        _tokenAReserve != address(0) &&
        _amo != address(0) &&
        _rdpxOracle != address(0) &&
        _ammFactory != address(0) &&
        _ammRouter != address(0),
      "reLPContract: address cannot be 0"
    );
    addresses = Addresses({
      tokenA: _tokenA,
      tokenB: _tokenB,
      pair: _pair,
      rdpxV2Core: _rdpxV2Core,
      tokenAReserve: _tokenAReserve,
      amo: _amo,
      rdpxOracle: _rdpxOracle,
      ammFactory: _ammFactory,
      ammRouter: _ammRouter
    });

    IERC20WithBurn(addresses.pair).safeApprove(
      addresses.ammRouter,
      type(uint256).max
    );

    IERC20WithBurn(addresses.tokenA).safeApprove(
      addresses.ammRouter,
      type(uint256).max
    );

    IERC20WithBurn(addresses.tokenB).safeApprove(
      addresses.ammRouter,
      type(uint256).max
    );
  }


```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L115C2-L165C1
