QA1. Some ERC20 tokens do not support allowance approval from non-zero to non-zero. So the following RdpxV2Core.approveContractToSpend() code will not work. 

[https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/core/RdpxV2Core.sol#L403-L412](https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/core/RdpxV2Core.sol#L403-L412)

Correction: approve to zero first before approving to the designated ``amount`` for a safe approval:

```diff
function approveContractToSpend(
    address _token,
    address _spender,
    uint256 _amount
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_token != address(0), 17);
    _validate(_spender != address(0), 17);
    _validate(_amount > 0, 17);

+     uint256 allowance = IERC20WithBurn(_token).allowance(msg.sender, _spender);
+    if(allowance != 0) IERC20WithBurn(_token).approve(_spender, 0);


    IERC20WithBurn(_token).approve(_spender, _amount);
  }

```

QA2. Possible divide by zero error in ``perpetualAtlanticVault._updateFundingRate()`` due to lack of check of wether ``endTime == startTime`` for the case of ``fundingRates[latestFundingPaymentPointer] == 0``.

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L594-L614](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L594-L614)

Mitigation: we need to check ``endTime == startTime`` for the case of ``fundingRates[latestFundingPaymentPointer] == 0`` as well.

```diff
 function _updateFundingRate(uint256 amount) private {
    if (fundingRates[latestFundingPaymentPointer] == 0) {
      uint256 startTime;
      if (lastUpdateTime > nextFundingPaymentTimestamp() - fundingDuration) {
        startTime = lastUpdateTime;
      } else {
        startTime = nextFundingPaymentTimestamp() - fundingDuration;
      }
      uint256 endTime = nextFundingPaymentTimestamp();
+      if (endTime == startTime) return;
      fundingRates[latestFundingPaymentPointer] =
        (amount * 1e18) /
        (endTime - startTime);
    } else {
      uint256 startTime = lastUpdateTime;
      uint256 endTime = nextFundingPaymentTimestamp();
      if (endTime == startTime) return;
      fundingRates[latestFundingPaymentPointer] =
        fundingRates[latestFundingPaymentPointer] +
        ((amount * 1e18) / (endTime - startTime));
    }
  }
```

QA3. UniV2LiquidityAmo.addLiquidity() (removeLiquidity() and swap()) might fail for some ERC20 tokens that revert on zero-transfer. The main reason is that it calls _sendTokensToRdpxV2Core() to send back ununsed tokenA and tokenB. However, there is no unused tokenA, then _sendTokensToRdpxV2Core() will faile for tokens that revert on zero-transfer.

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L189-L250](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L189-L250)

The following _sendTokensToRdpxV2Core() function will fail if there is zero balance for tokenA or tokenB.

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L160-L178](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L160-L178)

Mitigation: check whether the balance if zero or not, call safeTransfer only when the balance of non-zero:

```diff
function _sendTokensToRdpxV2Core() internal {
    uint256 tokenABalance = IERC20WithBurn(addresses.tokenA).balanceOf(
      address(this)
    );
    uint256 tokenBBalance = IERC20WithBurn(addresses.tokenB).balanceOf(
      address(this)
    );
    // transfer token A and B from this contract to the rdpxV2Core

+ if (tokenABalance > 0) 
    IERC20WithBurn(addresses.tokenA).safeTransfer(
      addresses.rdpxV2Core,
      tokenABalance
    );
  
+ if(tokenBBalance > 0) 
  IERC20WithBurn(addresses.tokenB).safeTransfer(
      addresses.rdpxV2Core,
      tokenBBalance
    );

    emit LogAssetsTransfered(msg.sender, tokenABalance, tokenBBalance);
  }
```

QA4.  UniV3LiquidityAmo.recoverERC20() fails to call IRdpxV2Core(rdpxV2Core).sync(), as a result, the balance of some reserve tokens in rdpxV2Core might not be synced after the calling of ``recoverERC20()``.

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L313-L322](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L313-L322)

Mitigation: 
We need to call IRdpxV2Core(rdpxV2Core).sync():

```diff

  function recoverERC20(
    address tokenAddress,
    uint256 tokenAmount
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    // Can only be triggered by owner or governance, not custodian
    // Tokens are sent to the custodian, as a sort of safeguard
    TransferHelper.safeTransfer(tokenAddress, rdpxV2Core, tokenAmount);

+   IRdpxV2Core(rdpxV2Core).sync();
    emit RecoveredERC20(tokenAddress, tokenAmount);
  }

```

QA5. The ReLPContract.reLP() function has the following problems.

First, it lacks a slippage control when calling ``IUniswapV2Router(addresses.ammRouter).addLiquidity()``:

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L286-L295](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L286-L295)

Second, it fails to send the dust tokenB to ``addresses.rdpxV2Core`` (only tokenA is considered) when there is any:

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L302-L305](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L302-L305)

Both of them are necessary to correct the function. 

QA6. An empty symbol reserve asset is added in the constructor, but two other data structures are not revised, including ``reserveTokens`` and ``reservesIndex``.

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L124-L136](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L124-L136)

We need to revise ``reserveTokens`` and ``reservesIndex`` as well to be consistent when add a new asset to reserve:

```diff
 constructor(address _weth) {
    _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
    weth = _weth;

    // add Zero asset to reserveAsset
    ReserveAsset memory zeroAsset = ReserveAsset({
      tokenAddress: address(0),
      tokenBalance: 0,
      tokenSymbol: "ZERO"
    });
    reserveAsset.push(zeroAsset);
    putOptionsRequired = true;

+    reserveTokens.push(_assetSymbol);
+    reservesIndex[_assetSymbol] = reserveAsset.length - 1;

  }
```

QA7. The emergency withdraw for RdpxDecayingBonds is supposed to send the tokens to ``Tor`` rather than ``msg.sender`` according to the NATSpec. 

```diff
function emergencyWithdraw(
    address[] calldata tokens,
    bool transferNative,
    address payable to,
    uint256 amount,
    uint256 gas
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _whenPaused();
    if (transferNative) {
      (bool success, ) = to.call{ value: amount, gas: gas }("");
      require(success, "RdpxReserve: transfer failed");
    }
    IERC20WithBurn token;

    for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
-      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
+      token.safeTransfer(to, token.balanceOf(address(this)));
    }
  }

```

