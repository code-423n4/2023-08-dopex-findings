
### Low Issues
|Title|Issue|Instances|
|-|:-|:-:|
|[L-1] Burn functions must be protected with a modifier | [Burn functions must be protected with a modifier](#burn-functions-must-be-protected-with-a-modifier) | 1 |
|[L-2] Missing contract-existence checks before low-level calls | [Missing contract-existence checks before low-level calls](#missing-contract-existence-checks-before-low-level-calls) | 1 |
|[L-3] Direct `supportsInterface()` calls | [Direct `supportsInterface()` calls](#direct-supportsinterface-calls) | 3 |
|[L-4] Fee/Rate Caps Missing in Smart Contracts | [Fee/Rate Caps Missing in Smart Contracts](#feerate-caps-missing-in-smart-contracts) | 1 |
|[L-5] Reentrancy vulnerabilities | [Reentrancy vulnerabilities](#reentrancy-vulnerabilities) | 18 |
|[L-6] Solmate SafeTransferLib Usage Detection | [Solmate SafeTransferLib Usage Detection](#solmate-safetransferlib-usage-detection) | 1 |
|[L-7] Unsafe Cast Unsigned to Signed | [Unsafe Cast Unsigned to Signed](#unsafe-cast-unsigned-to-signed) | 2 |

Total: 7 issues

### Non-Critical Issues
|Title|Issue|Instances|
|-|:-|:-:|
|[N-1] Do not calculate constants | [Do not calculate constants](#do-not-calculate-constants) | 2 |
|[N-2] Contract Ordering Violation Style Guide | [Contract Ordering Violation Style Guide](#contract-ordering-violation-style-guide) | 1 |
|[N-3] Control structures do not follow the Solidity Style Guide | [Control structures do not follow the Solidity Style Guide](#control-structures-do-not-follow-the-solidity-style-guide) | 29 |
|[N-4] Avoid double casting | [Avoid double casting](#avoid-double-casting) | 2 |
|[N-5] Consider move Msg.sender checks to modifier | [Consider move Msg.sender checks to modifier](#consider-move-msgsender-checks-to-modifier) | 1 |
|[N-6] Redundant Inheritance | [Redundant Inheritance](#redundant-inheritance) | 4 |
|[N-7] Should Declare as View | [Should Declare as View](#should-declare-as-view) | 6 |
|[N-8] Variable names too similar | [Variable names too similar](#variable-names-too-similar) | 30 |
|[N-9] Functions that alter state should emit events | [Functions that alter state should emit events](#functions-that-alter-state-should-emit-events) | 34 |
|[N-10] Whitespace in Expressions | [Whitespace in Expressions](#whitespace-in-expressions) | 4 |

Total: 10 issues

## Missing contract-existence checks before low-level calls
- Severity: Low
- Confidence: High

### Note 
reported only on issue that was missied in the winning bot.

### Description
Low-level calls return success if there is no code present at the specified address. In addition to the zero-address checks, add a check to verify that `<address>.code.length > 0` or use the `extcodesize` assembly operation to verify the presence of contract code at the specified address. Both these methods ensure the existence of a contract before making a low-level call.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###


- File: contracts/decaying-bonds/RdpxDecayingBonds.sol
```
 
Line: 98          (bool success, ) = to.call{ value: amount, gas: gas }("")
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L98](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L98)


</details>

# 

## Fee/Rate Caps Missing in Smart Contracts
- Severity: Low
- Confidence: High

### Note 
There is only check that fee is greater than 0, I reccomend to add check the fee is less than X. 

### Description
Fees/rates charged by contracts should be capped at a certain limit, preferably below 100%, to ensure users are not exploited. This detector checks for the absence of such limitations. The presence of fees/rates without upper limits could pose financial risks to the users of the contract.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: contracts/core/RdpxV2Core.sol
```
 
Line: 197          rdpxFeePercentage = _rdpxFeePercentage
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L197](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L197)


</details>

# 


## Reentrancy vulnerabilities
- Severity: Low
- Confidence: Medium

### Description

Detection of the [reentrancy bug](https://github.com/trailofbits/not-so-smart-contracts/tree/master/reentrancy).
Only report reentrancy that acts as a double call (see `reentrancy-eth`, `reentrancy-no-eth`).

<details>

<summary>
There are 12 instances of this issue:

</summary>

###
- Reentrancy in File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 189          function addLiquidity(
    uint256 tokenAAmount,
    uint256 tokenBAmount,
    uint256 tokenAAmountMin,
    uint256 tokenBAmountMin
  )
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 tokenAUsed, uint256 tokenBUsed, uint256 lpReceived)
  
```
:
	External calls:
	- File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 200          IERC20WithBurn(addresses.tokenA).safeApprove(
      addresses.ammRouter,
      tokenAAmount
    )
```

	- File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 204          IERC20WithBurn(addresses.tokenB).safeApprove(
      addresses.ammRouter,
      tokenBAmount
    )
```

	- File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 210          IERC20WithBurn(addresses.tokenA).safeTransferFrom(
      addresses.rdpxV2Core,
      address(this),
      tokenAAmount
    )
```

	- File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 215          IERC20WithBurn(addresses.tokenB).safeTransferFrom(
      addresses.rdpxV2Core,
      address(this),
      tokenBAmount
    )
```

	- File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 222          (tokenAUsed, tokenBUsed, lpReceived) = IUniswapV2Router(addresses.ammRouter)
      .addLiquidity(
        addresses.tokenA,
        addresses.tokenB,
        tokenAAmount,
        tokenBAmount,
        tokenAAmountMin,
        tokenBAmountMin,
        address(this),
        block.timestamp + 1
      )
```

	State variables written after the call(s):
	- File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 235          lpTokenBalance += lpReceived
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L189-L250](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L189-L250)


- Reentrancy in File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 258          function removeLiquidity(
    uint256 lpAmount,
    uint256 tokenAAmountMin,
    uint256 tokenBAmountMin
  )
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 tokenAReceived, uint256 tokenBReceived)
  
```
:
	External calls:
	- File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 268          IERC20WithBurn(addresses.pair).safeApprove(addresses.ammRouter, lpAmount)
```

	- File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 271          (tokenAReceived, tokenBReceived) = IUniswapV2Router(addresses.ammRouter)
      .removeLiquidity(
        addresses.tokenA,
        addresses.tokenB,
        lpAmount,
        tokenAAmountMin,
        tokenBAmountMin,
        address(this),
        block.timestamp + 1
      )
```

	State variables written after the call(s):
	- File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 283          lpTokenBalance -= lpAmount
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L258-L296](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L258-L296)


- Reentrancy in File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 155          function addLiquidity(
    AddLiquidityParams memory params
  ) public onlyRole(DEFAULT_ADMIN_ROLE) 
```
:
	External calls:
	- File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 158          IERC20WithBurn(params._tokenA).transferFrom(
      rdpxV2Core,
      address(this),
      params._amount0Desired
    )
```

	- File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 163          IERC20WithBurn(params._tokenB).transferFrom(
      rdpxV2Core,
      address(this),
      params._amount1Desired
    )
```

	- File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 169          IERC20WithBurn(params._tokenA).approve(
      address(univ3_positions),
      params._amount0Desired
    )
```

	- File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 173          IERC20WithBurn(params._tokenB).approve(
      address(univ3_positions),
      params._amount1Desired
    )
```

	- File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 193          (uint256 tokenId, uint128 amountLiquidity, , ) = univ3_positions.mint(
      mintParams
    )
```

	State variables written after the call(s):
	- File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 206          positions_array.push(pos)
```

	- File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 207          positions_mapping[tokenId] = pos
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAMO.sol#L155-L211](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAMO.sol#L155-L211)


- Reentrancy in File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 213          function removeLiquidity(
    uint256 positionIndex,
    uint256 minAmount0,
    uint256 minAmount1
  ) public onlyRole(DEFAULT_ADMIN_ROLE) 
```
:
	External calls:
	- File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 253          univ3_positions.decreaseLiquidity(decreaseLiquidityParams)
```

	- File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 255          univ3_positions.collect(collect_params)
```

	- File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 257          univ3_positions.burn(pos.token_id)
```

	State variables written after the call(s):
	- File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 263          delete positions_mapping[pos.token_id]
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAMO.sol#L213-L270](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAMO.sol#L213-L270)


- Reentrancy in File: contracts/core/RdpxV2Core.sol
```
 
Line: 495          function _issueBond(
    address _to,
    uint256 _amount
  ) internal returns (uint256 bondId) 
```
:
	External calls:
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 499          bondId = RdpxV2Bond(addresses.receiptTokenBonds).mint(_to)
```

	State variables written after the call(s):
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 500          bonds[bondId] = Bond({
      amount: _amount,
      maturity: block.timestamp + bondMaturity,
      timestamp: block.timestamp
    })
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L495-L505](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L495-L505)


- Reentrancy in File: contracts/core/RdpxV2Core.sol
```
 
Line: 471          function _purchaseOptions(
    uint256 _amount
  ) internal returns (uint256 premium) 
```
:
	External calls:
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 481          (premium, optionId) = IPerpetualAtlanticVault(
      addresses.perpetualAtlanticVault
    ).purchase(_amount, address(this))
```

	State variables written after the call(s):
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 485          optionsOwned[optionId] = true
```

	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 486          reserveAsset[reservesIndex["WETH"]].tokenBalance -= premium
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L471-L487](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L471-L487)


- Reentrancy in File: contracts/core/RdpxV2Core.sol
```
 
Line: 566          function _stake(
    address _to,
    uint256 _amount
  ) internal returns (uint256 receiptTokenAmount) 
```
:
	External calls:
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 572          IDpxEthToken(reserveAsset[reservesIndex["DPXETH"]].tokenAddress).mint(
      address(this),
      _amount / 2
    )
```

	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 578          receiptTokenAmount = IRdpxV2ReceiptToken(addresses.rdpxV2ReceiptToken)
      .deposit(_amount / 2)
```

	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 582          _issueBond(_to, receiptTokenAmount)
```

		- File: contracts/core/RdpxV2Core.sol
```
 
Line: 499          bondId = RdpxV2Bond(addresses.receiptTokenBonds).mint(_to)
```

	State variables written after the call(s):
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 582          _issueBond(_to, receiptTokenAmount)
```

		- File: contracts/core/RdpxV2Core.sol
```
 
Line: 500          bonds[bondId] = Bond({
      amount: _amount,
      maturity: block.timestamp + bondMaturity,
      timestamp: block.timestamp
    })
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L566-L583](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L566-L583)


- Reentrancy in File: contracts/core/RdpxV2Core.sol
```
 
Line: 624          function _transfer(
    uint256 _rdpxAmount,
    uint256 _wethAmount,
    uint256 _bondAmount,
    uint256 _bondId
  ) internal 
```
:
	External calls:
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 631          (, uint256 expiry, uint256 amount) = IRdpxDecayingBonds(
        addresses.rdpxDecayingBonds
      ).bonds(_bondId)
```

	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 643          IRdpxDecayingBonds(addresses.rdpxDecayingBonds).decreaseAmount(
        _bondId,
        amount - _rdpxAmount
      )
```

	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 648          IRdpxReserve(addresses.rdpxReserve).withdraw(_rdpxAmount)
```

	State variables written after the call(s):
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 650          reserveAsset[reservesIndex["RDPX"]].tokenBalance += _rdpxAmount
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L624-L685](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L624-L685)


- Reentrancy in File: contracts/core/RdpxV2Core.sol
```
 
Line: 941          function addToDelegate(
    uint256 _amount,
    uint256 _fee
  ) external returns (uint256) 
```
:
	External calls:
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 953          IERC20WithBurn(weth).safeTransferFrom(msg.sender, address(this), _amount)
```

	State variables written after the call(s):
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 961          delegates.push(delegatePosition)
```

	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 964          totalWethDelegated += _amount
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L941-L968](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L941-L968)


- Reentrancy in File: contracts/core/RdpxV2Core.sol
```
 
Line: 790          function provideFunding()
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 fundingAmount)
  
```
:
	External calls:
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 800          fundingAmount = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
      .payFunding()
```

	State variables written after the call(s):
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 803          reserveAsset[reservesIndex["WETH"]].tokenBalance -= fundingAmount
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L790-L808](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L790-L808)


- Reentrancy in File: contracts/core/RdpxV2Core.sol
```
 
Line: 764          function settle(
    uint256[] memory optionIds
  )
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 amountOfWeth, uint256 rdpxAmount)
  
```
:
	External calls:
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 772          (amountOfWeth, rdpxAmount) = IPerpetualAtlanticVault(
      addresses.perpetualAtlanticVault
    ).settle(optionIds)
```

	State variables written after the call(s):
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 776          optionsOwned[optionIds[i]] = false
```

	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 779          reserveAsset[reservesIndex["WETH"]].tokenBalance += amountOfWeth
```

	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 780          reserveAsset[reservesIndex["RDPX"]].tokenBalance -= rdpxAmount
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L764-L783](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L764-L783)


- Reentrancy in File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 372          function payFunding() external onlyRole(RDPXV2CORE_ROLE) returns (uint256) 
```
:
	External calls:
	- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 382          collateralToken.safeTransferFrom(
      addresses.rdpxV2Core,
      address(this),
      totalFundingForEpoch[latestFundingPaymentPointer]
    )
```

	State variables written after the call(s):
	- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 387          _updateFundingRate(totalFundingForEpoch[latestFundingPaymentPointer])
```

		- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 603          fundingRates[latestFundingPaymentPointer] =
        (amount * 1e18) /
        (endTime - startTime)
```

		- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 610          fundingRates[latestFundingPaymentPointer] =
        fundingRates[latestFundingPaymentPointer] +
        ((amount * 1e18) / (endTime - startTime))
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L372-L396](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L372-L396)


</details>

# 


## Burn functions must be protected with a modifier
- Severity: Low
- Confidence: High

### Description
If burn functions are not protected by a modifier, any address may be able to burn tokens, potentially leading to financial loss. A common modifier to use is `onlyOwner`.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: contracts/uniswap_v2/IUniswapV2Pair.sol
```
 
Line: 27          function burn(address to) external returns (uint256 amount0, uint256 amount1);
```
Burn function without a protective modifier.
[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/uniswap_v2/IUniswapV2Pair.sol#L27](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/uniswap_v2/IUniswapV2Pair.sol#L27)


</details>

# 


## Direct `supportsInterface()` calls
- Severity: Low
- Confidence: High

### Description
The `supportsInterface()` function is part of the ERC165 standard in Ethereum. It allows for the interrogation of a contract to determine if it implements a particular interface according to its specified ID. This is typically implemented in a contract by overriding the `supportsInterface()` function and returning true for the interface IDs that the contract supports.

However, if a contract directly calls the `supportsInterface()` function on another contract that has not properly implemented it, the function call may result in a revert, interrupting the execution of the transaction. This happens because if the called contract does not have `supportsInterface()` implemented, the call will fail.

This is why using `ERC165Checker` from OpenZeppelin library when checking for interface support is recommended. The `ERC165Checker`'s `supportsInterface()` function employs a low-level `staticcall` that does not revert if the call fails, instead, it returns false.

This detector flags instances of direct calls to `supportsInterface()`, which can potentially cause the calling function to revert, particularly if the target contract does not properly implement the `supportsInterface()` function or does not implement it at all. 

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
- File: contracts/core/RdpxV2Bond.sol
```
 
Line: 65          return super.supportsInterface(interfaceId)
```
 `ERC165Checker` from OpenZeppelin library instead.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L65](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L65)


- File: contracts/decaying-bonds/RdpxDecayingBonds.sol
```
 
Line: 182          return super.supportsInterface(interfaceId)
```
 `ERC165Checker` from OpenZeppelin library instead.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L182](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L182)


- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 653          return super.supportsInterface(interfaceId)
```
 `ERC165Checker` from OpenZeppelin library instead.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L653](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L653)


</details>

# 


## Solmate SafeTransferLib Usage Detection
- Severity: Low
- Confidence: Medium

### Description
This detector flags contracts that use Solmate's SafeTransferLib, which does not check whether the ERC20 contract exists. It is recommended to use the OpenZeppelin's SafeERC20 library instead.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
 
Line: 12          import { SafeTransferLib } from "solmate/src/utils/SafeTransferLib.sol";
```
  don't use SafeTransferLib
[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L12](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L12)


</details>

# 



## Unsafe Cast Unsigned to Signed
- Severity: Low
- Confidence: High

### Description
An unsafe cast occurs when a variable or expression of an unsigned/signed integer is converted to a signed/unsigned integer without proper checks. This type of cast can lead to unexpected behavior and vulnerabilities in the code.

When an unsigned integer is cast to a signed integer, the underlying bit representation remains the same, but the interpretation of the value changes. This can result in overflow or wraparound issues, leading to incorrect calculations or security vulnerabilities.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: contracts/core/RdpxV2Core.sol
```
 
Line: 553          int256(a)
```
 usafe cast from uint256 to int256 

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L553](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L553)


- File: contracts/core/RdpxV2Core.sol
```
 
Line: 554          int256(b)
```
 usafe cast from uint256 to int256 

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L554](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L554)



</details>

# 


## Do not calculate constants
- Severity: Non-Critical
- Confidence: High

### Description
Due to how constant variables are implemented in Solidity (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas. While in most cases, the compiler will optimize these computations away, it is considered a best practice to write code that does not rely on the compiler optimization.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: contracts/core/RdpxV2Core.sol
```
 
Line: 88          uint256 public constant RDPX_RATIO_PERCENTAGE = 25 * DEFAULT_PRECISION
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L88](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L88)


- File: contracts/core/RdpxV2Core.sol
```
 
Line: 91          uint256 public constant ETH_RATIO_PERCENTAGE = 75 * DEFAULT_PRECISION
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L91](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L91)


</details>

# 



## Whitespace in Expressions
- Severity: Non-Critical
- Confidence: High

### Description
Detects when whitespace usage in expressions does not conform to the Solidity style guide.

<details>

<summary>
There are 4 instances of this issue:

</summary>

###
- File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 28          contract UniV3LiquidityAMO is AccessControl, ERC721Holder 
```

```
// @audit: whitespace inside parenthesis
Line: 105        (uint128 liquidity, , , , ) = get_pool.positions(


// @audit: whitespace inside parenthesis
Line: 193        (uint256 tokenId, uint128 amountLiquidity, , ) = univ3_positions.mint(


```
[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAMO.sol#L28-L380](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAMO.sol#L28-L380)


- File: contracts/amo/UniV3LiquidityAmo.sol
```
 
Line: 28          contract UniV3LiquidityAMO is AccessControl, ERC721Holder 
```

```
// @audit: whitespace inside parenthesis
Line: 105        (uint128 liquidity, , , , ) = get_pool.positions(


// @audit: whitespace inside parenthesis
Line: 193        (uint256 tokenId, uint128 amountLiquidity, , ) = univ3_positions.mint(


```
[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L28-L380](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L28-L380)


- File: contracts/decaying-bonds/RdpxDecayingBonds.sol
```
 
Line: 18          contract RdpxDecayingBonds is
  ERC721,
  ERC721Enumerable,
  ERC721Burnable,
  Pausable,
  AccessControl

```

```
// @audit: whitespace inside parenthesis
Line: 90          (bool success, ) = to.call{ value: amount, gas: gas }("");


```
[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L18-L184](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L18-L184)


- File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
 
Line: 21          contract PerpetualAtlanticVaultLP is ERC20, IPerpetualAtlanticVaultLP 
```

```
// @audit: whitespace inside parenthesis
Line: 268      function beforeWithdraw(uint256 assets, uint256 ) internal {


```
[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L21-L302](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L21-L302)


</details>

# 



## Avoid double casting
- Severity: Non-Critical
- Confidence: High

### Description
This detector identifies instances where values are double casted in Solidity. Double casting can introduce unexpected truncations and/or rounding issues. Additionally, it reduces the readability of the code and can make it harder to maintain. It's recommended to avoid double casting to ensure clearer, safer, and more maintainable smart contracts.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: contracts/core/RdpxV2Core.sol
```
 
Line: 552          amountOut = dpxEthCurvePool.exchange(
      _ethToDpxEth ? int128(int256(a)) : int128(int256(b)),
      _ethToDpxEth ? int128(int256(b)) : int128(int256(a)),
      _amount,
      minAmount > 0 ? minAmount : minOut
    )
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L552-L557](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L552-L557)




</details>

# 


## Contract Ordering Violation Style Guide
- Severity: Non-Critical
- Confidence: High

### Description
The contract ordering detector identifies cases where the order of contract layout in a Solidity contract does not follow the recommended style guide. According to the Solidity style guide, contract should be laid out in a specific order: 1) Type declarations, 2) State variables, 3) Events, 4) Modifiers, and 5) Functions.

By adhering to the recommended contract ordering, code readability and maintainability can be improved.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
 
Line: 21          contract PerpetualAtlanticVaultLP is ERC20, IPerpetualAtlanticVaultLP 
```
-	This event `Deposit`  should come after state variable `rdpxRdpxV2Core`

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L21-L302](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L21-L302)


</details>

# 


## Control structures do not follow the Solidity Style Guide
- Severity: Non-Critical
- Confidence: High

### Description
Control structures should follow the One True Brace [OTB style](https://docs.soliditylang.org/en/latest/style-guide.html#control-structures). This means: 1. Opening braces should be on the same line as the declaration. 
2. Closing braces should be on their own line, at the same indentation level as the beginning of the declaration. 
3. The opening brace should be preceded by a single space.

<details>

<summary>
There are 29 instances of this issue:

</summary>

###
- File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 74          function setAddresses(
    address _tokenA,
    address _tokenB,
    address _pair,
    address _rdpxV2Core,
    address _rdpxOracle,
    address _ammFactory,
    address _ammRouter
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
 These control structures in the function `setAddresses` should follow the One True Brace (OTB) style:

    - Line: 93:     addresses = Addresses({
    - Line: 101:     });

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L74-L102](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L74-L102)


- File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 189          function addLiquidity(
    uint256 tokenAAmount,
    uint256 tokenBAmount,
    uint256 tokenAAmountMin,
    uint256 tokenBAmountMin
  )
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 tokenAUsed, uint256 tokenBUsed, uint256 lpReceived)
  
```
 These control structures in the function `addLiquidity` should follow the One True Brace (OTB) style:

    - Line: 198:   {

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L189-L250](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L189-L250)


- File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 258          function removeLiquidity(
    uint256 lpAmount,
    uint256 tokenAAmountMin,
    uint256 tokenBAmountMin
  )
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 tokenAReceived, uint256 tokenBReceived)
  
```
 These control structures in the function `removeLiquidity` should follow the One True Brace (OTB) style:

    - Line: 266:   {

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L258-L296](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L258-L296)


- File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 74          function setAddresses(
    address _tokenA,
    address _tokenB,
    address _pair,
    address _rdpxV2Core,
    address _rdpxOracle,
    address _ammFactory,
    address _ammRouter
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
 These control structures in the function `setAddresses` should follow the One True Brace (OTB) style:

    - Line: 93:     addresses = Addresses({
    - Line: 101:     });

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L74-L102](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L74-L102)


- File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 189          function addLiquidity(
    uint256 tokenAAmount,
    uint256 tokenBAmount,
    uint256 tokenAAmountMin,
    uint256 tokenBAmountMin
  )
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 tokenAUsed, uint256 tokenBUsed, uint256 lpReceived)
  
```
 These control structures in the function `addLiquidity` should follow the One True Brace (OTB) style:

    - Line: 198:   {

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L189-L250](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L189-L250)


- File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 258          function removeLiquidity(
    uint256 lpAmount,
    uint256 tokenAAmountMin,
    uint256 tokenBAmountMin
  )
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 tokenAReceived, uint256 tokenBReceived)
  
```
 These control structures in the function `removeLiquidity` should follow the One True Brace (OTB) style:

    - Line: 266:   {

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L258-L296](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L258-L296)


- File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 339          function execute(
    address _to,
    uint256 _value,
    bytes calldata _data
  ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (bool, bytes memory) 
```
 These control structures in the function `execute` should follow the One True Brace (OTB) style:

    - Line: 344:     (bool success, bytes memory result) = _to.call{ value: _value }(_data);

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAMO.sol#L339-L346](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAMO.sol#L339-L346)


- File: contracts/amo/UniV3LiquidityAmo.sol
```
 
Line: 339          function execute(
    address _to,
    uint256 _value,
    bytes calldata _data
  ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (bool, bytes memory) 
```
 These control structures in the function `execute` should follow the One True Brace (OTB) style:

    - Line: 344:     (bool success, bytes memory result) = _to.call{ value: _value }(_data);

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L339-L346](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L339-L346)


- File: contracts/core/RdpxV2Bond.sol
```
 
Line: 57          function supportsInterface(
    bytes4 interfaceId
  )
    public
    view
    override(ERC721, ERC721Enumerable, AccessControl)
    returns (bool)
  
```
 These control structures in the function `supportsInterface` should follow the One True Brace (OTB) style:

    - Line: 64:   {

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L57-L66](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L57-L66)


- File: contracts/core/RdpxV2Core.sol
```
 
Line: 124          constructor(address _weth) 
```
 These control structures in the function `constructor` should follow the One True Brace (OTB) style:

    - Line: 129:     ReserveAsset memory zeroAsset = ReserveAsset({
    - Line: 133:     });

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L124-L136](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L124-L136)


- File: contracts/core/RdpxV2Core.sol
```
 
Line: 240          function addAssetTotokenReserves(
    address _asset,
    string memory _assetSymbol
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
 These control structures in the function `addAssetTotokenReserves` should follow the One True Brace (OTB) style:

    - Line: 253:     ReserveAsset memory asset = ReserveAsset({
    - Line: 257:     });

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L240-L264](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L240-L264)


- File: contracts/core/RdpxV2Core.sol
```
 
Line: 304          function setAddresses(
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
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
 These control structures in the function `setAddresses` should follow the One True Brace (OTB) style:

    - Line: 327:     addresses = Addresses({
    - Line: 338:     });

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L304-L350](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L304-L350)


- File: contracts/core/RdpxV2Core.sol
```
 
Line: 358          function setPricingOracleAddresses(
    address _rdpxPriceOracle,
    address _dpxEthPriceOracle
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
 These control structures in the function `setPricingOracleAddresses` should follow the One True Brace (OTB) style:

    - Line: 365:     pricingOracleAddresses = PricingOracleAddresses({
    - Line: 368:     });

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L358-L371](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L358-L371)


- File: contracts/core/RdpxV2Core.sol
```
 
Line: 495          function _issueBond(
    address _to,
    uint256 _amount
  ) internal returns (uint256 bondId) 
```
 These control structures in the function `_issueBond` should follow the One True Brace (OTB) style:

    - Line: 500:     bonds[bondId] = Bond({
    - Line: 504:     });

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L495-L505](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L495-L505)


- File: contracts/core/RdpxV2Core.sol
```
 
Line: 764          function settle(
    uint256[] memory optionIds
  )
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 amountOfWeth, uint256 rdpxAmount)
  
```
 These control structures in the function `settle` should follow the One True Brace (OTB) style:

    - Line: 770:   {

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L764-L783](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L764-L783)


- File: contracts/core/RdpxV2Core.sol
```
 
Line: 790          function provideFunding()
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 fundingAmount)
  
```
 These control structures in the function `provideFunding` should follow the One True Brace (OTB) style:

    - Line: 794:   {

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L790-L808](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L790-L808)


- File: contracts/core/RdpxV2Core.sol
```
 
Line: 941          function addToDelegate(
    uint256 _amount,
    uint256 _fee
  ) external returns (uint256) 
```
 These control structures in the function `addToDelegate` should follow the One True Brace (OTB) style:

    - Line: 955:     Delegate memory delegatePosition = Delegate({
    - Line: 960:     });

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L941-L968](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L941-L968)


- File: contracts/core/RdpxV2Core.sol
```
 
Line: 1260          function getDelegatePosition(
    uint256 _delegateId
  )
    public
    view
    returns (
      address delegate,
      uint256 amount,
      uint256 fee,
      uint256 activeCollateral
    )
  
```
 These control structures in the function `getDelegatePosition` should follow the One True Brace (OTB) style:

    - Line: 1271:   {

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1260-L1279](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1260-L1279)


- File: contracts/decaying-bonds/RdpxDecayingBonds.sol
```
 
Line: 89          function emergencyWithdraw(
    address[] calldata tokens,
    bool transferNative,
    address payable to,
    uint256 amount,
    uint256 gas
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
 These control structures in the function `emergencyWithdraw` should follow the One True Brace (OTB) style:

    - Line: 98:       (bool success, ) = to.call{ value: amount, gas: gas }("");

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L89-L107](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L89-L107)


- File: contracts/decaying-bonds/RdpxDecayingBonds.sol
```
 
Line: 174          function supportsInterface(
    bytes4 interfaceId
  )
    public
    view
    override(ERC721, ERC721Enumerable, AccessControl)
    returns (bool)
  
```
 These control structures in the function `supportsInterface` should follow the One True Brace (OTB) style:

    - Line: 181:   {

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L174-L183](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L174-L183)


- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 181          function setAddresses(
    address _optionPricing,
    address _assetPriceOracle,
    address _volatilityOracle,
    address _feeDistributor,
    address _rdpx,
    address _perpetualAtlanticVaultLP,
    address _rdpxV2Core
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
 These control structures in the function `setAddresses` should follow the One True Brace (OTB) style:

    - Line: 198:     addresses = Addresses({
    - Line: 206:     });

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L181-L212](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L181-L212)


- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 255          function purchase(
    uint256 amount,
    address to
  )
    external
    nonReentrant
    onlyRole(RDPXV2CORE_ROLE)
    returns (uint256 premium, uint256 tokenId)
  
```
 These control structures in the function `purchase` should follow the One True Brace (OTB) style:

    - Line: 263:   {
    - Line: 296:     optionPositions[tokenId] = OptionPosition({
    - Line: 300:     });

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L255-L312](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L255-L312)


- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 315          function settle(
    uint256[] memory optionIds
  )
    external
    nonReentrant
    onlyRole(RDPXV2CORE_ROLE)
    returns (uint256 ethAmount, uint256 rdpxAmount)
  
```
 These control structures in the function `settle` should follow the One True Brace (OTB) style:

    - Line: 322:   {

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L315-L369](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L315-L369)


- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 563          function nextFundingPaymentTimestamp()
    public
    view
    returns (uint256 timestamp)
  
```
 These control structures in the function `nextFundingPaymentTimestamp` should follow the One True Brace (OTB) style:

    - Line: 567:   {

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L563-L569](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L563-L569)


- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 645          function supportsInterface(
    bytes4 interfaceId
  )
    public
    view
    override(ERC721, ERC721Enumerable, AccessControl)
    returns (bool)
  
```
 These control structures in the function `supportsInterface` should follow the One True Brace (OTB) style:

    - Line: 652:   {

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L645-L654](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L645-L654)


- File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
 
Line: 81          constructor(
    address _perpetualAtlanticVault,
    address _rdpxRdpxV2Core,
    address _collateral,
    address _rdpx,
    string memory _collateralSymbol
  )
    ERC20(
      "PerpetualAtlanticVault LP Token",
      _collateralSymbol,
      ERC20(_collateral).decimals()
    )
  
```
 These control structures in the function `constructor` should follow the One True Brace (OTB) style:

    - Line: 93:   {

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L81-L108](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L81-L108)


- File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
 
Line: 231          function _beforeTokenTransfer(
    address from,
    address,
    uint256
  ) internal virtual 
```
 These control structures in the function `_beforeTokenTransfer` should follow the One True Brace (OTB) style:

    - Line: 235:   ) internal virtual {}

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L231-L235](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L231-L235)


- File: contracts/reLP/ReLPContract.sol
```
 
Line: 115          function setAddresses(
    address _tokenA,
    address _tokenB,
    address _pair,
    address _rdpxV2Core,
    address _tokenAReserve,
    address _amo,
    address _rdpxOracle,
    address _ammFactory,
    address _ammRouter
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
 These control structures in the function `setAddresses` should follow the One True Brace (OTB) style:

    - Line: 138:     addresses = Addresses({
    - Line: 148:     });

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L115-L164](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L115-L164)


- File: contracts/relp/ReLPContract.sol
```
 
Line: 115          function setAddresses(
    address _tokenA,
    address _tokenB,
    address _pair,
    address _rdpxV2Core,
    address _tokenAReserve,
    address _amo,
    address _rdpxOracle,
    address _ammFactory,
    address _ammRouter
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
 These control structures in the function `setAddresses` should follow the One True Brace (OTB) style:

    - Line: 138:     addresses = Addresses({
    - Line: 148:     });

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/relp/ReLPContract.sol#L115-L164](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/relp/ReLPContract.sol#L115-L164)


</details>

# 


## Consider move Msg.sender checks to modifier
- Severity: Non-Critical
- Confidence: Medium

### Note 
report only on instance that was missing in the winning bot. 

### Description
This detector identifies functions where checks for `msg.sender` authorization are not made in a modifier. Using modifiers for authorization checks can improve code readability and maintainability.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: contracts/decaying-bonds/RdpxDecayingBonds.sol
```
 
Line: 120          require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter")
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L120](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L120)


</details>

# 


## Redundant Inheritance
- Severity: Non-Critical
- Confidence: High

### Description
Detects when a contract inherits from another contract that it already indirectly inherits from.

<details>

<summary>
There are 4 instances of this issue:

</summary>

###
- File: contracts/core/RdpxV2Bond.sol
```
 
Line: 11          contract RdpxV2Bond is
  ERC721,
  ERC721Enumerable,
  Pausable,
  AccessControl,
  ERC721Burnable

```
 The contract `RdpxV2Bond` has redundant inheritance: `ERC721` is already inherited indirectly.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L11-L67](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L11-L67)


- File: contracts/decaying-bonds/RdpxDecayingBonds.sol
```
 
Line: 18          contract RdpxDecayingBonds is
  ERC721,
  ERC721Enumerable,
  ERC721Burnable,
  Pausable,
  AccessControl

```
 The contract `RdpxDecayingBonds` has redundant inheritance: `ERC721` is already inherited indirectly.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L18-L184](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L18-L184)


- File: contracts/dpxETH/DpxEthToken.sol
```
 
Line: 12          contract DpxEthToken is
  ERC20,
  ERC20Burnable,
  Pausable,
  AccessControl,
  IDpxEthToken

```
 The contract `DpxEthToken` has redundant inheritance: `ERC20` is already inherited indirectly.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L12-L63](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L12-L63)


- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 28          contract PerpetualAtlanticVault is
  IPerpetualAtlanticVault,
  ReentrancyGuard,
  Pausable,
  ERC721,
  ERC721Enumerable,
  ERC721Burnable,
  AccessControl,
  ContractWhitelist

```
 The contract `PerpetualAtlanticVault` has redundant inheritance: `ERC721` is already inherited indirectly.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L28-L655](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L28-L655)


</details>

# 



## Event Emission Preceding External Calls: A Best Practice
- Severity: Non-Critical
- Confidence: Medium

### Description

Ensure that events follow the best practice of check-effects-interaction, and are emitted before external calls


<details>

<summary>
There are 24 instances of this issue:

</summary>

###
- Reentrancy in File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 160          function _sendTokensToRdpxV2Core() internal 
```
:
	External calls:
	- File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 168          IERC20WithBurn(addresses.tokenA).safeTransfer(
      addresses.rdpxV2Core,
      tokenABalance
    )
```

	- File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 172          IERC20WithBurn(addresses.tokenB).safeTransfer(
      addresses.rdpxV2Core,
      tokenBBalance
    )
```

	Event emitted after the call(s):
	- File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 177          emit LogAssetsTransfered(msg.sender, tokenABalance, tokenBBalance)
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L160-L178](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L160-L178)


- Reentrancy in File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 142          function emergencyWithdraw(
    address[] calldata tokens
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
:
	External calls:
	- File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 149          token.safeTransfer(msg.sender, token.balanceOf(address(this)))
```

	Event emitted after the call(s):
	- File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 152          emit LogEmergencyWithdraw(msg.sender, tokens)
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L142-L153](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L142-L153)


- Reentrancy in File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 160          function _sendTokensToRdpxV2Core() internal 
```
:
	External calls:
	- File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 168          IERC20WithBurn(addresses.tokenA).safeTransfer(
      addresses.rdpxV2Core,
      tokenABalance
    )
```

	- File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 172          IERC20WithBurn(addresses.tokenB).safeTransfer(
      addresses.rdpxV2Core,
      tokenBBalance
    )
```

	Event emitted after the call(s):
	- File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 177          emit LogAssetsTransfered(msg.sender, tokenABalance, tokenBBalance)
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L160-L178](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L160-L178)


- Reentrancy in File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 142          function emergencyWithdraw(
    address[] calldata tokens
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
:
	External calls:
	- File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 149          token.safeTransfer(msg.sender, token.balanceOf(address(this)))
```

	Event emitted after the call(s):
	- File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 152          emit LogEmergencyWithdraw(msg.sender, tokens)
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L142-L153](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L142-L153)


- Reentrancy in File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 353          function _sendTokensToRdpxV2Core(address tokenA, address tokenB) internal 
```
:
	External calls:
	- File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 357          IERC20WithBurn(tokenA).safeTransfer(rdpxV2Core, tokenABalance)
```

	- File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 358          IERC20WithBurn(tokenB).safeTransfer(rdpxV2Core, tokenBBalance)
```

	- File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 361          IRdpxV2Core(rdpxV2Core).sync()
```

	Event emitted after the call(s):
	- File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 363          emit LogAssetsTransfered(tokenABalance, tokenBBalance, tokenA, tokenB)
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAMO.sol#L353-L364](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAMO.sol#L353-L364)


- Reentrancy in File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 313          function recoverERC20(
    address tokenAddress,
    uint256 tokenAmount
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
:
	External calls:
	- File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 319          TransferHelper.safeTransfer(tokenAddress, rdpxV2Core, tokenAmount)
```

	Event emitted after the call(s):
	- File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 321          emit RecoveredERC20(tokenAddress, tokenAmount)
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAMO.sol#L313-L322](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAMO.sol#L313-L322)


- Reentrancy in File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 324          function recoverERC721(
    address tokenAddress,
    uint256 token_id
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
:
	External calls:
	- File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 330          INonfungiblePositionManager(tokenAddress).safeTransferFrom(
      address(this),
      rdpxV2Core,
      token_id
    )
```

	Event emitted after the call(s):
	- File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 335          emit RecoveredERC721(tokenAddress, token_id)
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAMO.sol#L324-L336](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAMO.sol#L324-L336)


- Reentrancy in File: contracts/amo/UniV3LiquidityAmo.sol
```
 
Line: 353          function _sendTokensToRdpxV2Core(address tokenA, address tokenB) internal 
```
:
	External calls:
	- File: contracts/amo/UniV3LiquidityAmo.sol
```
 
Line: 357          IERC20WithBurn(tokenA).safeTransfer(rdpxV2Core, tokenABalance)
```

	- File: contracts/amo/UniV3LiquidityAmo.sol
```
 
Line: 358          IERC20WithBurn(tokenB).safeTransfer(rdpxV2Core, tokenBBalance)
```

	- File: contracts/amo/UniV3LiquidityAmo.sol
```
 
Line: 361          IRdpxV2Core(rdpxV2Core).sync()
```

	Event emitted after the call(s):
	- File: contracts/amo/UniV3LiquidityAmo.sol
```
 
Line: 363          emit LogAssetsTransfered(tokenABalance, tokenBBalance, tokenA, tokenB)
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L353-L364](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L353-L364)


- Reentrancy in File: contracts/amo/UniV3LiquidityAmo.sol
```
 
Line: 313          function recoverERC20(
    address tokenAddress,
    uint256 tokenAmount
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
:
	External calls:
	- File: contracts/amo/UniV3LiquidityAmo.sol
```
 
Line: 319          TransferHelper.safeTransfer(tokenAddress, rdpxV2Core, tokenAmount)
```

	Event emitted after the call(s):
	- File: contracts/amo/UniV3LiquidityAmo.sol
```
 
Line: 321          emit RecoveredERC20(tokenAddress, tokenAmount)
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L313-L322](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L313-L322)


- Reentrancy in File: contracts/amo/UniV3LiquidityAmo.sol
```
 
Line: 324          function recoverERC721(
    address tokenAddress,
    uint256 token_id
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
:
	External calls:
	- File: contracts/amo/UniV3LiquidityAmo.sol
```
 
Line: 330          INonfungiblePositionManager(tokenAddress).safeTransferFrom(
      address(this),
      rdpxV2Core,
      token_id
    )
```

	Event emitted after the call(s):
	- File: contracts/amo/UniV3LiquidityAmo.sol
```
 
Line: 335          emit RecoveredERC721(tokenAddress, token_id)
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L324-L336](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L324-L336)


- Reentrancy in File: contracts/core/RdpxV2Core.sol
```
 
Line: 941          function addToDelegate(
    uint256 _amount,
    uint256 _fee
  ) external returns (uint256) 
```
:
	External calls:
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 953          IERC20WithBurn(weth).safeTransferFrom(msg.sender, address(this), _amount)
```

	Event emitted after the call(s):
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 966          emit LogAddToDelegate(_amount, _fee, delegates.length - 1)
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L941-L968](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L941-L968)


- Reentrancy in File: contracts/core/RdpxV2Core.sol
```
 
Line: 161          function emergencyWithdraw(
    address[] calldata tokens
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
:
	External calls:
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 169          token.safeTransfer(msg.sender, token.balanceOf(address(this)))
```

	Event emitted after the call(s):
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 172          emit LogEmergencyWithdraw(msg.sender, tokens)
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L161-L173](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L161-L173)


- Reentrancy in File: contracts/core/RdpxV2Core.sol
```
 
Line: 1080          function lowerDepeg(
    uint256 _rdpxAmount,
    uint256 _wethAmount,
    uint256 minamountOfWeth,
    uint256 minOut
  ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256 dpxEthReceived) 
```
:
	External calls:
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 1097          amountOfWethOut = IUniswapV2Router(addresses.dopexAMMRouter)
        .swapExactTokensForTokens(
          _rdpxAmount,
          minamountOfWeth,
          path,
          address(this),
          block.timestamp + 10
        )[path.length - 1]
```

	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 1112          dpxEthReceived = _curveSwap(
      _wethAmount + amountOfWethOut,
      true,
      true,
      minOut
    )
```

		- File: contracts/core/RdpxV2Core.sol
```
 
Line: 552          amountOut = dpxEthCurvePool.exchange(
      _ethToDpxEth ? int128(int256(a)) : int128(int256(b)),
      _ethToDpxEth ? int128(int256(b)) : int128(int256(a)),
      _amount,
      minAmount > 0 ? minAmount : minOut
    )
```

		- File: contracts/core/RdpxV2Core.sol
```
 
Line: 552          amountOut = dpxEthCurvePool.exchange(
      _ethToDpxEth ? int128(int256(a)) : int128(int256(b)),
      _ethToDpxEth ? int128(int256(b)) : int128(int256(a)),
      _amount,
      minAmount > 0 ? minAmount : minOut
    )
```

	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 1119          IDpxEthToken(reserveAsset[reservesIndex["DPXETH"]].tokenAddress).burn(
      dpxEthReceived
    )
```

	Event emitted after the call(s):
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 1123          emit LogLowerDepeg(_rdpxAmount, _wethAmount, dpxEthReceived)
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1080-L1124](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1080-L1124)


- Reentrancy in File: contracts/core/RdpxV2Core.sol
```
 
Line: 790          function provideFunding()
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 fundingAmount)
  
```
:
	External calls:
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 800          fundingAmount = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
      .payFunding()
```

	Event emitted after the call(s):
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 807          emit LogProvideFunding(pointer, fundingAmount)
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L790-L808](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L790-L808)


- Reentrancy in File: contracts/core/RdpxV2Core.sol
```
 
Line: 1016          function redeem(
    uint256 id,
    address to
  ) external returns (uint256 receiptTokenAmount) 
```
:
	External calls:
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 1032          RdpxV2Bond(addresses.receiptTokenBonds).burn(id)
```

	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 1036          IERC20WithBurn(addresses.rdpxV2ReceiptToken).safeTransfer(
      to,
      receiptTokenAmount
    )
```

	Event emitted after the call(s):
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 1041          emit LogRedeem(to, receiptTokenAmount)
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1016-L1042](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1016-L1042)


- Reentrancy in File: contracts/core/RdpxV2Core.sol
```
 
Line: 304          function setAddresses(
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
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
:
	External calls:
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 339          IERC20WithBurn(weth).approve(
      addresses.perpetualAtlanticVault,
      type(uint256).max
    )
```

	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 343          IERC20WithBurn(weth).approve(addresses.dopexAMMRouter, type(uint256).max)
```

	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 344          IERC20WithBurn(weth).approve(addresses.dpxEthCurvePool, type(uint256).max)
```

	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 345          IERC20WithBurn(weth).approve(
      addresses.rdpxV2ReceiptToken,
      type(uint256).max
    )
```

	Event emitted after the call(s):
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 349          emit LogSetAddresses(addresses)
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L304-L350](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L304-L350)


- Reentrancy in File: contracts/core/RdpxV2Core.sol
```
 
Line: 764          function settle(
    uint256[] memory optionIds
  )
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 amountOfWeth, uint256 rdpxAmount)
  
```
:
	External calls:
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 772          (amountOfWeth, rdpxAmount) = IPerpetualAtlanticVault(
      addresses.perpetualAtlanticVault
    ).settle(optionIds)
```

	Event emitted after the call(s):
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 782          emit LogSettle(optionIds)
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L764-L783](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L764-L783)


- Reentrancy in File: contracts/core/RdpxV2Core.sol
```
 
Line: 1051          function upperDepeg(
    uint256 _amount,
    uint256 minOut
  ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256 wethReceived) 
```
:
	External calls:
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 1059          IDpxEthToken(reserveAsset[reservesIndex["DPXETH"]].tokenAddress).mint(
      address(this),
      _amount
    )
```

	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 1065          wethReceived = _curveSwap(_amount, false, true, minOut)
```

		- File: contracts/core/RdpxV2Core.sol
```
 
Line: 552          amountOut = dpxEthCurvePool.exchange(
      _ethToDpxEth ? int128(int256(a)) : int128(int256(b)),
      _ethToDpxEth ? int128(int256(b)) : int128(int256(a)),
      _amount,
      minAmount > 0 ? minAmount : minOut
    )
```

		- File: contracts/core/RdpxV2Core.sol
```
 
Line: 552          amountOut = dpxEthCurvePool.exchange(
      _ethToDpxEth ? int128(int256(a)) : int128(int256(b)),
      _ethToDpxEth ? int128(int256(b)) : int128(int256(a)),
      _amount,
      minAmount > 0 ? minAmount : minOut
    )
```

	Event emitted after the call(s):
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 1069          emit LogUpperDepeg(_amount, wethReceived)
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1051-L1070](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1051-L1070)


- Reentrancy in File: contracts/core/RdpxV2Core.sol
```
 
Line: 975          function withdraw(
    uint256 delegateId
  ) external returns (uint256 amountWithdrawn) 
```
:
	External calls:
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 987          IERC20WithBurn(weth).safeTransfer(msg.sender, amountWithdrawn)
```

	Event emitted after the call(s):
	- File: contracts/core/RdpxV2Core.sol
```
 
Line: 989          emit LogDelegateWithdraw(delegateId, amountWithdrawn)
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L975-L990](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L975-L990)


- Reentrancy in File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 219          function emergencyWithdraw(
    address[] calldata tokens
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
:
	External calls:
	- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 227          token.safeTransfer(msg.sender, token.balanceOf(address(this)))
```

	Event emitted after the call(s):
	- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 230          emit EmergencyWithdraw(msg.sender, tokens)
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L219-L231](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L219-L231)


- Reentrancy in File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 372          function payFunding() external onlyRole(RDPXV2CORE_ROLE) returns (uint256) 
```
:
	External calls:
	- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 382          collateralToken.safeTransferFrom(
      addresses.rdpxV2Core,
      address(this),
      totalFundingForEpoch[latestFundingPaymentPointer]
    )
```

	Event emitted after the call(s):
	- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 389          emit PayFunding(
      msg.sender,
      totalFundingForEpoch[latestFundingPaymentPointer],
      latestFundingPaymentPointer
    )
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L372-L396](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L372-L396)


- Reentrancy in File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 181          function setAddresses(
    address _optionPricing,
    address _assetPriceOracle,
    address _volatilityOracle,
    address _feeDistributor,
    address _rdpx,
    address _perpetualAtlanticVaultLP,
    address _rdpxV2Core
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
:
	External calls:
	- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 207          collateralToken.safeApprove(
      addresses.perpetualAtlanticVaultLP,
      type(uint256).max
    )
```

	Event emitted after the call(s):
	- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 211          emit AddressesSet(addresses)
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L181-L212](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L181-L212)


- Reentrancy in File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 462          function updateFundingPaymentPointer() public 
```
:
	External calls:
	- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 473          collateralToken.safeTransfer(
          addresses.perpetualAtlanticVaultLP,
          (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
            1e18
        )
```

	- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 479          IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP)
          .addProceeds(
            (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
              1e18
          )
```

	Event emitted after the call(s):
	- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 485          emit FundingPaid(
          msg.sender,
          ((currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
            1e18),
          latestFundingPaymentPointer
        )
```

	- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 494          emit FundingPaymentPointerUpdated(latestFundingPaymentPointer)
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L462-L496](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L462-L496)


- Reentrancy in File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
 
Line: 145          function redeem(
    uint256 shares,
    address receiver,
    address owner
  ) public returns (uint256 assets, uint256 rdpxAmount) 
```
:
	External calls:
	- File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
 
Line: 150          perpetualAtlanticVault.updateFunding()
```

	- File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
 
Line: 170          collateral.transfer(receiver, assets)
```

	- File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
 
Line: 172          IERC20WithBurn(rdpx).safeTransfer(receiver, rdpxAmount)
```

	Event emitted after the call(s):
	- File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
 
Line: 174          emit Withdraw(msg.sender, receiver, owner, assets, shares)
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L145-L175](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L145-L175)


</details>

# 


## Should Declare as View
- Severity: Non-Critical
- Confidence: High

### Description
Detects when functions do not change the state of the contract and can be declared as view.

<details>

<summary>
There are 6 instances of this issue:

</summary>

###
- File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 160          function _sendTokensToRdpxV2Core() internal 
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L160-L178](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L160-L178)


- File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 160          function _sendTokensToRdpxV2Core() internal 
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L160-L178](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L160-L178)


- File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 353          function _sendTokensToRdpxV2Core(address tokenA, address tokenB) internal 
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAMO.sol#L353-L364](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAMO.sol#L353-L364)


- File: contracts/amo/UniV3LiquidityAmo.sol
```
 
Line: 353          function _sendTokensToRdpxV2Core(address tokenA, address tokenB) internal 
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L353-L364](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L353-L364)


- File: contracts/core/RdpxV2Core.sol
```
 
Line: 515          function _curveSwap(
    uint256 _amount,
    bool _ethToDpxEth,
    bool validate,
    uint256 minAmount
  ) internal returns (uint256 amountOut) 
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L515-L558](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L515-L558)


- File: contracts/core/RdpxV2Core.sol
```
 
Line: 1016          function redeem(
    uint256 id,
    address to
  ) external returns (uint256 receiptTokenAmount) 
```

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1016-L1042](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1016-L1042)


</details>

# 


## Variable names too similar
- Severity: Non-Critical
- Confidence: Medium

### Description
Detect variables with names that are too similar.

<details>

<summary>
There are 30 instances of this issue:

</summary>

###
- Variable File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 305          uint256 token1Amount
```
 is too similar to File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 190          uint256 tokenAAmount
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L305](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L305)


- Variable File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 305          uint256 token1Amount
```
 is too similar to File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 308          uint256 token2Amount
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L305](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L305)


- Variable File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 308          uint256 token2Amount
```
 is too similar to File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 190          uint256 tokenAAmount
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L308](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L308)


- Variable File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 308          uint256 token2Amount
```
 is too similar to File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 191          uint256 tokenBAmount
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L308](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L308)


- Variable File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 260          uint256 tokenAAmountMin
```
 is too similar to File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 193          uint256 tokenBAmountMin
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L260](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L260)


- Variable File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 192          uint256 tokenAAmountMin
```
 is too similar to File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 193          uint256 tokenBAmountMin
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L192](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L192)


- Variable File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 161          uint256 tokenABalance = IERC20WithBurn(addresses.tokenA).balanceOf(
      address(this)
    )
```
 is too similar to File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 164          uint256 tokenBBalance = IERC20WithBurn(addresses.tokenB).balanceOf(
      address(this)
    )
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L161-L163](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L161-L163)


- Variable File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 265          uint256 tokenAReceived
```
 is too similar to File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 265          uint256 tokenBReceived
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L265](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L265)


- Variable File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 305          uint256 token1Amount
```
 is too similar to File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 191          uint256 tokenBAmount
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L305](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L305)


- Variable File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 190          uint256 tokenAAmount
```
 is too similar to File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 191          uint256 tokenBAmount
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L190](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L190)


- Variable File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 260          uint256 tokenAAmountMin
```
 is too similar to File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 261          uint256 tokenBAmountMin
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L260](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L260)


- Variable File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 192          uint256 tokenAAmountMin
```
 is too similar to File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 261          uint256 tokenBAmountMin
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L192](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L192)


- Variable File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 305          uint256 token1Amount
```
 is too similar to File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 308          uint256 token2Amount
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L305](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L305)


- Variable File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 305          uint256 token1Amount
```
 is too similar to File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 190          uint256 tokenAAmount
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L305](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L305)


- Variable File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 308          uint256 token2Amount
```
 is too similar to File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 190          uint256 tokenAAmount
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L308](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L308)


- Variable File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 192          uint256 tokenAAmountMin
```
 is too similar to File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 193          uint256 tokenBAmountMin
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L192](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L192)


- Variable File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 260          uint256 tokenAAmountMin
```
 is too similar to File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 193          uint256 tokenBAmountMin
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L260](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L260)


- Variable File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 192          uint256 tokenAAmountMin
```
 is too similar to File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 261          uint256 tokenBAmountMin
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L192](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L192)


- Variable File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 161          uint256 tokenABalance = IERC20WithBurn(addresses.tokenA).balanceOf(
      address(this)
    )
```
 is too similar to File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 164          uint256 tokenBBalance = IERC20WithBurn(addresses.tokenB).balanceOf(
      address(this)
    )
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L161-L163](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L161-L163)


- Variable File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 265          uint256 tokenAReceived
```
 is too similar to File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 265          uint256 tokenBReceived
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L265](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L265)


- Variable File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 305          uint256 token1Amount
```
 is too similar to File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 191          uint256 tokenBAmount
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L305](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L305)


- Variable File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 308          uint256 token2Amount
```
 is too similar to File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 191          uint256 tokenBAmount
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L308](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L308)


- Variable File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 190          uint256 tokenAAmount
```
 is too similar to File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 191          uint256 tokenBAmount
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L190](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L190)


- Variable File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 260          uint256 tokenAAmountMin
```
 is too similar to File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 261          uint256 tokenBAmountMin
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L260](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L260)


- Variable File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 354          uint256 tokenABalance = IERC20WithBurn(tokenA).balanceOf(address(this))
```
 is too similar to File: contracts/amo/UniV3LiquidityAMO.sol
```
 
Line: 355          uint256 tokenBBalance = IERC20WithBurn(tokenB).balanceOf(address(this))
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAMO.sol#L354](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAMO.sol#L354)


- Variable File: contracts/amo/UniV3LiquidityAmo.sol
```
 
Line: 354          uint256 tokenABalance = IERC20WithBurn(tokenA).balanceOf(address(this))
```
 is too similar to File: contracts/amo/UniV3LiquidityAmo.sol
```
 
Line: 355          uint256 tokenBBalance = IERC20WithBurn(tokenB).balanceOf(address(this))
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L354](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L354)


- Variable File: contracts/reLP/ReLPContract.sol
```
 
Line: 250          uint256 mintokenAAmount = tokenAToRemove -
      ((tokenAToRemove * liquiditySlippageTolerance) / 1e8)
```
 is too similar to File: contracts/reLP/ReLPContract.sol
```
 
Line: 252          uint256 mintokenBAmount = ((tokenAToRemove * tokenAInfo.tokenAPrice) /
      1e8) -
      ((tokenAToRemove * tokenAInfo.tokenAPrice) * liquiditySlippageTolerance) /
      1e16
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L250-L251](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L250-L251)


- Variable File: contracts/reLP/ReLPContract.sol
```
 
Line: 204          address tokenASorted
```
 is too similar to File: contracts/reLP/ReLPContract.sol
```
 
Line: 204          address tokenBSorted
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L204](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L204)


- Variable File: contracts/relp/ReLPContract.sol
```
 
Line: 250          uint256 mintokenAAmount = tokenAToRemove -
      ((tokenAToRemove * liquiditySlippageTolerance) / 1e8)
```
 is too similar to File: contracts/relp/ReLPContract.sol
```
 
Line: 252          uint256 mintokenBAmount = ((tokenAToRemove * tokenAInfo.tokenAPrice) /
      1e8) -
      ((tokenAToRemove * tokenAInfo.tokenAPrice) * liquiditySlippageTolerance) /
      1e16
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/relp/ReLPContract.sol#L250-L251](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/relp/ReLPContract.sol#L250-L251)


- Variable File: contracts/relp/ReLPContract.sol
```
 
Line: 204          address tokenASorted
```
 is too similar to File: contracts/relp/ReLPContract.sol
```
 
Line: 204          address tokenBSorted
```


[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/relp/ReLPContract.sol#L204](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/relp/ReLPContract.sol#L204)


</details>

# 


## Functions that alter state should emit events
- Severity: Non-Critical
- Confidence: Medium

### Description
Functions that alter the state of the contract should emit an event to inform external observers of the change.

<details>

<summary>
There are 34 instances of this issue:

</summary>

###
- File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 74          function setAddresses(
    address _tokenA,
    address _tokenB,
    address _pair,
    address _rdpxV2Core,
    address _rdpxOracle,
    address _ammFactory,
    address _ammRouter
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
 The function `setAddresses` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L74-L102](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L74-L102)


- File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 109          function setSlippageTolerance(
    uint256 _slippageTolerance
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
 The function `setSlippageTolerance` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L109-L117](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L109-L117)


- File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 362          function sync() external 
```
 The function `sync` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L362-L364](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L362-L364)


- File: contracts/amo/UniV2LiquidityAMO.sol
```
 
Line: 22          contract UniV2LiquidityAMO is AccessControl 
```
 The function `slitherConstructorVariables` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L22-L422](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAMO.sol#L22-L422)


- File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 74          function setAddresses(
    address _tokenA,
    address _tokenB,
    address _pair,
    address _rdpxV2Core,
    address _rdpxOracle,
    address _ammFactory,
    address _ammRouter
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
 The function `setAddresses` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L74-L102](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L74-L102)


- File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 109          function setSlippageTolerance(
    uint256 _slippageTolerance
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
 The function `setSlippageTolerance` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L109-L117](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L109-L117)


- File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 362          function sync() external 
```
 The function `sync` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L362-L364](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L362-L364)


- File: contracts/amo/UniV2LiquidityAmo.sol
```
 
Line: 22          contract UniV2LiquidityAMO is AccessControl 
```
 The function `slitherConstructorVariables` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L22-L422](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L22-L422)


- File: contracts/core/RdpxV2Core.sol
```
 
Line: 378          function addAMOAddress(address _addr) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
 The function `addAMOAddress` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L378-L381](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L378-L381)


- File: contracts/core/RdpxV2Core.sol
```
 
Line: 388          function removeAMOAddress(
    uint256 _index
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
 The function `removeAMOAddress` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L388-L394](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L388-L394)


- File: contracts/core/RdpxV2Core.sol
```
 
Line: 471          function _purchaseOptions(
    uint256 _amount
  ) internal returns (uint256 premium) 
```
 The function `_purchaseOptions` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L471-L487](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L471-L487)


- File: contracts/core/RdpxV2Core.sol
```
 
Line: 495          function _issueBond(
    address _to,
    uint256 _amount
  ) internal returns (uint256 bondId) 
```
 The function `_issueBond` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L495-L505](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L495-L505)


- File: contracts/core/RdpxV2Core.sol
```
 
Line: 566          function _stake(
    address _to,
    uint256 _amount
  ) internal returns (uint256 receiptTokenAmount) 
```
 The function `_stake` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L566-L583](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L566-L583)


- File: contracts/core/RdpxV2Core.sol
```
 
Line: 624          function _transfer(
    uint256 _rdpxAmount,
    uint256 _wethAmount,
    uint256 _bondAmount,
    uint256 _bondId
  ) internal 
```
 The function `_transfer` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L624-L685](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L624-L685)


- File: contracts/core/RdpxV2Core.sol
```
 
Line: 699          function _bondWithDelegate(
    uint256 _amount,
    uint256 rdpxBondId,
    uint256 delegateId
  ) internal returns (BondWithDelegateReturnValue memory returnValues) 
```
 The function `_bondWithDelegate` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L699-L744](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L699-L744)


- File: contracts/core/RdpxV2Core.sol
```
 
Line: 33          contract RdpxV2Core is
  IRdpxV2Core,
  AccessControl,
  ContractWhitelist,
  ERC721Holder,
  Pausable

```
 The function `slitherConstructorVariables` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L33-L1287](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L33-L1287)


- File: contracts/decaying-bonds/RdpxDecayingBonds.sol
```
 
Line: 139          function decreaseAmount(
    uint256 bondId,
    uint256 amount
  ) public onlyRole(RDPXV2CORE_ROLE) 
```
 The function `decreaseAmount` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L139-L145](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L139-L145)


- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 237          function updateFundingDuration(
    uint256 _fundingDuration
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
 The function `updateFundingDuration` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L237-L241](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L237-L241)


- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 594          function _updateFundingRate(uint256 amount) private 
```
 The function `_updateFundingRate` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L594-L614](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L594-L614)


- File: contracts/perp-vault/PerpetualAtlanticVault.sol
```
 
Line: 28          contract PerpetualAtlanticVault is
  IPerpetualAtlanticVault,
  ReentrancyGuard,
  Pausable,
  ERC721,
  ERC721Enumerable,
  ERC721Burnable,
  AccessControl,
  ContractWhitelist

```
 The function `slitherConstructorVariables` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L28-L655](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L28-L655)


- File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
 
Line: 180          function lockCollateral(uint256 amount) public onlyPerpVault 
```
 The function `lockCollateral` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L180-L182](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L180-L182)


- File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
 
Line: 185          function unlockLiquidity(uint256 amount) public onlyPerpVault 
```
 The function `unlockLiquidity` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L185-L187](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L185-L187)


- File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
 
Line: 190          function addProceeds(uint256 proceeds) public onlyPerpVault 
```
 The function `addProceeds` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L190-L196](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L190-L196)


- File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
 
Line: 199          function subtractLoss(uint256 loss) public onlyPerpVault 
```
 The function `subtractLoss` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L199-L205](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L199-L205)


- File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
 
Line: 208          function addRdpx(uint256 amount) public onlyPerpVault 
```
 The function `addRdpx` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L208-L214](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L208-L214)


- File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol
```
 
Line: 286          function beforeWithdraw(uint256 assets, uint256 /*shares*/) internal 
```
 The function `beforeWithdraw` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L286-L292](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L286-L292)


- File: contracts/reLP/ReLPContract.sol
```
 
Line: 115          function setAddresses(
    address _tokenA,
    address _tokenB,
    address _pair,
    address _rdpxV2Core,
    address _tokenAReserve,
    address _amo,
    address _rdpxOracle,
    address _ammFactory,
    address _ammRouter
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
 The function `setAddresses` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L115-L164](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L115-L164)


- File: contracts/reLP/ReLPContract.sol
```
 
Line: 171          function setLiquiditySlippageTolerance(
    uint256 _liquiditySlippageTolerance
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
 The function `setLiquiditySlippageTolerance` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L171-L179](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L171-L179)


- File: contracts/reLP/ReLPContract.sol
```
 
Line: 186          function setSlippageTolerance(
    uint256 _slippageTolerance
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
 The function `setSlippageTolerance` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L186-L194](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L186-L194)


- File: contracts/reLP/ReLPContract.sol
```
 
Line: 25          contract ReLPContract is AccessControl 
```
 The function `slitherConstructorVariables` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L25-L312](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L25-L312)


- File: contracts/relp/ReLPContract.sol
```
 
Line: 115          function setAddresses(
    address _tokenA,
    address _tokenB,
    address _pair,
    address _rdpxV2Core,
    address _tokenAReserve,
    address _amo,
    address _rdpxOracle,
    address _ammFactory,
    address _ammRouter
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
 The function `setAddresses` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/relp/ReLPContract.sol#L115-L164](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/relp/ReLPContract.sol#L115-L164)


- File: contracts/relp/ReLPContract.sol
```
 
Line: 171          function setLiquiditySlippageTolerance(
    uint256 _liquiditySlippageTolerance
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
 The function `setLiquiditySlippageTolerance` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/relp/ReLPContract.sol#L171-L179](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/relp/ReLPContract.sol#L171-L179)


- File: contracts/relp/ReLPContract.sol
```
 
Line: 186          function setSlippageTolerance(
    uint256 _slippageTolerance
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 
```
 The function `setSlippageTolerance` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/relp/ReLPContract.sol#L186-L194](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/relp/ReLPContract.sol#L186-L194)


- File: contracts/relp/ReLPContract.sol
```
 
Line: 25          contract ReLPContract is AccessControl 
```
 The function `slitherConstructorVariables` changes state but does not emit an event.

[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/relp/ReLPContract.sol#L25-L312](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/relp/ReLPContract.sol#L25-L312)


</details>

# 

