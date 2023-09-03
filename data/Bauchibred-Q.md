# DopeX QA Report

|       | Issue                                                                                                                                  |
| ----- | -------------------------------------------------------------------------------------------------------------------------------------- |
| QA-01 | RdpxV2Core's `addToDelegate()` should be refactored                                                                                    |
| QA-02 | Swaps are made on curve without a deadline                                                                                             |
| QA-03 | Users are presently not allowed to withdraw DSC when a bond just matured                                                               |
| QA-04 | `RdpxDecayingBonds.decreaseAmount()` needs to be refactored                                                                             |
| QA-05 | `calculatePremium()` wouldn't work for call option types since `isPut` bool is hardcoded to true value breaks multiple functionalities |
| QA-06 | Getting prices would always return stale price                                                                                        |
| QA-07 | Unfulfilled heartbeat prices could be used                                                                                          |

## QA 01 RdpxV2Core's `addToDelegate()` should be refactored

### Impact

Medium to low, since this easily curbs user's ability to properly delegate their WETH, this is cause if user follows the recommendation of the function the problem never gets fixed and just leads to more failed attempts

### Proof of Concept

Take a look at [addToDelegate()](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L941-L969)

```solidity

  function addToDelegate(
    uint256 _amount,
    uint256 _fee
  ) external returns (uint256) {
    _whenNotPaused();
    // fee less than 100%
    _validate(_fee < 100e8, 8);
    // amount greater than 0.01 WETH
    _validate(_amount > 1e16, 4);
    // fee greater than 1%
    //@audit
    _validate(_fee >= 1e8, 8);

    IERC20WithBurn(weth).safeTransferFrom(msg.sender, address(this), _amount);

    Delegate memory delegatePosition = Delegate({
      amount: _amount,
      fee: _fee,
      owner: msg.sender,
      activeCollateral: 0
    });
    delegates.push(delegatePosition);

    // add amount to total weth delegated
    totalWethDelegated += _amount;

    emit LogAddToDelegate(_amount, _fee, delegates.length - 1);
    return (delegates.length - 1);
  }

```

As seen, the function is a pretty straight forward one, where it just allows the user to delegate theur their weth, the issue is that while validating the fees being greater than 1%, if it's invalid the error "8" is returned, but erro E8 states the below:

> // E8: "Fee cannot be more than 100"

Which would even confuse the user and make them think they are _setting fee to a value higher than the ax 100% value_ and users would then go ahead and reduce the fee to bypass this but the execution still return the same error, leading to an endless loop of failed attempts for users to delegate WETH

### Recommended Mitigation Steps

Create a new error message for when the fee is less than 1%

> //E20: "Fee cannot be less than 1"

Then refactor this [line](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L951) to

```solidity
_validate(_fee >= 1e8, 20);
```

## QA 02 Swaps are made on curve without a deadline

### Impact

Easily leads to trade staying valid for too llong and when it gets executed, funds received could lower in _dollar value_

### Proof of Concept

Take a look at [RdpxV2Core.sol#L515-L558](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L515-L558)

```solidity
  function _curveSwap(
    uint256 _amount,
    bool _ethToDpxEth,
    bool validate,
    uint256 minAmount
  ) internal returns (uint256 amountOut) {
    IStableSwap dpxEthCurvePool = IStableSwap(addresses.dpxEthCurvePool);

    // First compute a reverse swapping of dpxETH to ETH to compute the amount of ETH required
    address coin0 = dpxEthCurvePool.coins(0);
    (uint256 a, uint256 b) = coin0 == weth ? (0, 1) : (1, 0);

    // validate the swap for peg functions
    if (validate) {
      uint256 ethBalance = IStableSwap(addresses.dpxEthCurvePool).balances(a);
      uint256 dpxEthBalance = IStableSwap(addresses.dpxEthCurvePool).balances(
        b
      );
      _ethToDpxEth
        ? _validate(
          ethBalance + _amount <= (ethBalance + dpxEthBalance) / 2,
          14
        )
        : _validate(
          dpxEthBalance + _amount <= (ethBalance + dpxEthBalance) / 2,
          14
        );
    }


    // calculate minimum amount out
    uint256 minOut = _ethToDpxEth
      ? (((_amount * getDpxEthPrice()) / 1e8) -
        (((_amount * getDpxEthPrice()) * slippageTolerance) / 1e16))
      : (((_amount * getEthPrice()) / 1e8) -
        (((_amount * getEthPrice()) * slippageTolerance) / 1e16));

    // Swap the tokens
    //@audit
    amountOut = dpxEthCurvePool.exchange(
      _ethToDpxEth ? int128(int256(a)) : int128(int256(b)),
      _ethToDpxEth ? int128(int256(b)) : int128(int256(a)),
      _amount,
      minAmount > 0 ? minAmount : minOut
    );
  }

```

As seen, while querying `dpxEthCurvePool.exchange()` this is done with no deadline leading to _Impact being possible_

### Recommended Mitigation Steps

Add a deadline protection

## QA 03 Users are presently not allowed to withdraw DSC when a bond just matured

### Impact

Low, this is cause the duration of this unfair DOS is relatively short

### Proof of Concept

Take a look at [redeem()](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1016-L1042)

```solidity
  function redeem(
    uint256 id,
    address to
  ) external returns (uint256 receiptTokenAmount) {
    // Validate bond ID
    _validate(bonds[id].timestamp > 0, 6);
    // Validate if bond has matured
    //@audit
    _validate(block.timestamp > bonds[id].maturity, 7);

    _validate(
      msg.sender == RdpxV2Bond(addresses.receiptTokenBonds).ownerOf(id),
      9
    );

    // Burn the bond token
    // Note requires an approval of the bond token to this contract
    RdpxV2Bond(addresses.receiptTokenBonds).burn(id);

    // transfer receipt tokens to user
    receiptTokenAmount = bonds[id].amount;
    IERC20WithBurn(addresses.rdpxV2ReceiptToken).safeTransfer(
      to,
      receiptTokenAmount
    );

    emit LogRedeem(to, receiptTokenAmount);
  }
```

As seen, the function is just used to withdraw DSC from any bond that's matured, the issue now is that if the bond just got matured, i.e `block.timestamp == bonds[id].maturity`, while validating, the function does not take into account that the present timestamp might be the bond's maturity timestamp and still wrongly errors out with the `// E7: "Bond has not matured",` revert message

### Recommended Mitigation Steps

Make the check inclusive to fix this, since if `block.timestamp == bonds[id].maturity` then bond already got matured and settlements should be allowed

## QA 04 `RdpxDecayingBonds.decreaseAmount()` needs to be refactored

### Impact

Current implementation does not do what's it's _commented_ to do

### Proof of Concept

Take a look at [RdpxDecayingBonds.sol#L139-L145](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L139-L145)

```solidity
  /// @notice Decreases the bond amount
  /// @dev Can only be called by the rdpxV2Core
  /// @param bondId id of the bond to decrease
  /// @param amount amount to decrease
  //@audit
  function decreaseAmount(
    uint256 bondId,
    uint256 amount
  ) public onlyRole(RDPXV2CORE_ROLE) {
    _whenNotPaused();
    bonds[bondId].rdpxAmount = amount;
  }
```

As seen, the function does not _decrease_ the _amount to decrease_ as stated by the comments, but rather it sets the bond new rdpxAmount with provided amount... do note that where as this does not break contract's logic since the only instance of this being used from [RdpxV2Core.sol#L643-L646](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L643-L646), seen below, _where it was called with the already subtracted value_, this should still be fixed

```solidity
      IRdpxDecayingBonds(addresses.rdpxDecayingBonds).decreaseAmount(
        _bondId,
        amount - _rdpxAmount
      );
```

### Recommended Mitigation Steps

Code implementation should always match it's execution, helps improve readability and also stops wrong implementation from happening in the future

## QA 05 calculatePremium() would not work for call option types since `isPut` bool is hardcoded to true value breaks multiple functionalities including

### Impact

Break in an important aspect of contract's logic including [calculatePremium()](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L539-L551)

### Proof of Concept

Take a look at [calculatePremium()](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L539-L551)

```solidity
  function calculatePremium(
    uint256 _strike,
    uint256 _amount,
    uint256 timeToExpiry,
    uint256 _price
  ) public view returns (uint256 premium) {
    premium = ((IOptionPricing(addresses.optionPricing).getOptionPrice(
      _strike,
      _price > 0 ? _price : getUnderlyingPrice(),
      getVolatility(_strike),
      timeToExpiry
    ) * _amount) / 1e8);
  }
```

Now take a look at [getOptionPrice()](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/libraries/OptionPricingSimple.sol#L66)

```solidity
  function getOptionPrice(
    uint256 strike,
    uint256 lastPrice,
    uint256 volatility,
    uint256 expiry
  ) external view override returns (uint256) {
    uint256 timeToExpiry = (expiry * 100) / 86400;
    bool isPut = true;//@audit

    uint256 optionPrice = BlackScholes
      .calculate(
        // @audit
        isPut ? 1 : 0, // 0 - Put, 1 - Call
        lastPrice,
        strike,
        timeToExpiry, // Number of days to expiry mul by 100
        0,
        volatility
      )
      .div(BlackScholes.DIVISOR);

    uint256 minOptionPrice = lastPrice.mul(minOptionPricePercentage).div(1e10);

    if (minOptionPrice > optionPrice) {
      return minOptionPrice;
    }

    return optionPrice;
  }
```

As seen the `isPut` value is hardcoded for all executions, setting it to true, this means that this check `isPut ? ` would always be true and set the the option type to `1` from the `ABDKMathQuad.sol` we can see that this is accurate for puts, since `  uint8 internal constant OPTION_TYPE_PUT = 1;` exists.

Issue now is that due to the hardcoded value always being puts, option pricing for `call` option types are bricked

Additionally this section of `ABDKMathQuad.sol `does not do anything since option type has already been hardcoded for all `getOptionPrice()` calls

```
    if (optionType == OPTION_TYPE_CALL) {
      return
        ABDKMathQuad.toUInt(
          ABDKMathQuad.mul(
            _calculateCallTimeDecay(S, d1, X, r, T, d2),
            ABDKMathQuad.fromUInt(DIVISOR)
          )
        );
    } else if (optionType == OPTION_TYPE_PUT) {
      return
        ABDKMathQuad.toUInt(
          ABDKMathQuad.mul(
            _calculatePutTimeDecay(X, r, T, d2, S, d1),
            ABDKMathQuad.fromUInt(DIVISOR)
          )
        );
    }
```

Though the impact is a bit relaxed since on bonding only put options are required, nonethelless there is no need for the `isPut` check since it's alredy been hardcoded.

### Recommended Mitigation Steps

When needed, constantly pass the option type and do not hardcode `isPut`

## QA 06 Getting prices would always return stale price

### Impact

Returned prices would be relatively stale

### Proof of Concept

Take a look at this section of [DpxEthOracle.sol](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/oracles/DpxEthOracle.sol#L77-L123)

<details>
<summary>Click to see full code reference</summary>

```solidity
 /*==== KEEPER FUNCTIONS ====*/

  /// @notice Updates the last price of the token
  /// @param _price price
  /// @return priceUpdatesLength length of the priceUpdates
  function updatePrice(
    uint256 _price
  ) external onlyRole(KEEPER_ROLE) returns (uint256) {
    uint256 blockTimestamp = block.timestamp;

    priceUpdates[priceUpdatesLength] = PriceObj({
      price: _price,
      updatedAt: blockTimestamp
    });

    priceUpdatesLength += 1;

    lastPrice = _price;

    lastUpdated = blockTimestamp;

    emit PriceUpdate(_price, blockTimestamp);

    return priceUpdatesLength;
  }

  /*==== VIEWS ====*/

  /// @inheritdoc IDpxEthOracle
  function getDpxEthPriceInEth() external view returns (uint256) {
    require(lastPrice != 0, "Last price == 0");
//@audit
    if (block.timestamp > lastUpdated + heartbeat) {
      revert HeartbeatNotFulfilled();
    }

    return lastPrice;
  }

  /// @inheritdoc IDpxEthOracle
  function getEthPriceInDpxEth() external view returns (uint256) {
    require(lastPrice != 0, "Last price == 0");

    if (block.timestamp > lastUpdated + heartbeat) {
      revert HeartbeatNotFulfilled();
    }

    // To not lose any precision we square 'decimals' and divide it by lastPrice which is in 'decimals' precision
    return (DECIMALS * DECIMALS) / lastPrice;
  }

```

</details>
Attached above are 2 price getter functions and the keeper function to update prices, as seen current implementation hinders any one to update prices and as such could allow any one to interact with the protocol with prices as old as 34 minuted, and "price getting" naturally should be the _current_ price of an asset

### Recommended Mitigation Steps

Refactor how prices are updated in the protocol and ensure that returned prices are fresh.

## QA 07 Unfulfilled heartbeat prices could be used

### Impact

Prices that do not fulfill the heartbeat would be ingested

### Proof of Concept

Take a look at both price getters from DpxEthOracle.sol, seen [here](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/oracles/DpxEthOracle.sol#L104-L124)

```solidity
  function getDpxEthPriceInEth() external view returns (uint256) {
    require(lastPrice != 0, "Last price == 0");
//@audit
    if (block.timestamp > lastUpdated + heartbeat) {
      revert HeartbeatNotFulfilled();
    }

    return lastPrice;
  }

    function getEthPriceInDpxEth() external view returns (uint256) {
    require(lastPrice != 0, "Last price == 0");

    if (block.timestamp > lastUpdated + heartbeat) {
      revert HeartbeatNotFulfilled();
    }
    // To not lose any precision we square 'decimals' and divide it by lastPrice which is in 'decimals' precision
    return (DECIMALS * DECIMALS) / lastPrice;
  }
```

As seen, the check to see if the heartbeat is fulfilled or not is not correctly implemented, this is cause in the edge case where `block.timestamp == lastUpdated + heartbeat`, price should be considered as stale already but that's not the case since this would still be ingested

### Recommended Mitigation Steps

Barring how arbitrum's sequencer ingestion could affect this, the very least is to make the stale check inclusive, i.e make the below changes

```diff

  function getDpxEthPriceInEth() external view returns (uint256) {
    require(lastPrice != 0, "Last price == 0");
//@audit
-    if (block.timestamp > lastUpdated + heartbeat) {
+    if (block.timestamp >= lastUpdated + heartbeat) {
      revert HeartbeatNotFulfilled();
    }

    return lastPrice;
  }

    function getEthPriceInDpxEth() external view returns (uint256) {
    require(lastPrice != 0, "Last price == 0");

-    if (block.timestamp > lastUpdated + heartbeat) {
+    if (block.timestamp >= lastUpdated + heartbeat) {
      revert HeartbeatNotFulfilled();
    }
    // To not lose any precision we square 'decimals' and divide it by lastPrice which is in 'decimals' precision
    return (DECIMALS * DECIMALS) / lastPrice;
  }
```
