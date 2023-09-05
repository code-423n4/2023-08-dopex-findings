# Findings Summary

| ID     | Title                                                                 | Severity     |
| ------ | --------------------------------------------------------------------- | ------------ |
| [L-01] | RemoveAssetFromtokenReserves remove wrong token symbol                | Low          |
| [L-02] | The expiration time of the uniswap interactive setting is invalid     | Low          |
| [L-03] | safeApprove implementation does not meet expectations                 | Low          |
| [L-04] | ReLPContract.reLP doesn't consider token's decimals                   | Low          |

# Detailed Findings

# [L-01] RemoveAssetFromtokenReserves remove wrong token symbol

## Description

```solidity
    function removeAssetFromtokenReserves(
        string memory _assetSymbol
    ) external onlyRole(DEFAULT_ADMIN_ROLE) {
        uint256 index = reservesIndex[_assetSymbol];
        _validate(index != 0, 18);

        // remove the asset from the mapping
        reservesIndex[_assetSymbol] = 0;

        // add new index for the last element
        reservesIndex[reserveTokens[reserveTokens.length - 1]] = index;

        // update the index of reserveAsset with the last element
        reserveAsset[index] = reserveAsset[reserveAsset.length - 1];

        // remove the last element
        reserveAsset.pop();
        // @audit Not swap reserveTokens
        reserveTokens.pop();

        emit LogAssetRemovedFromtokenReserves(_assetSymbol, index);
    }
```

when removeAssetFromtokenReserves, it swap `reserveAsset` and pop the last element. but not do the same for `reserveTokens`, which causes removing the wrong token symbol.        

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.19;

import { Test, console } from "forge-std/Test.sol";
import { RdpxV2Core } from "contracts/core/RdpxV2Core.sol";

contract TestRemoveAssetFromtokenReserves is Test {
    RdpxV2Core core;

    function setUp() public {
        core = new RdpxV2Core(address(0));
    }

    function testRemoveAssetFromtokenReserves2() public {
        address asset1 = address(1);
        string memory assetSymbol1 = "S1";
        address asset2 = address(2);
        string memory assetSymbol2 = "S2";
        core.addAssetTotokenReserves(asset1, assetSymbol1);
        core.addAssetTotokenReserves(asset2, assetSymbol2);
        core.removeAssetFromtokenReserves(assetSymbol1);
        string memory symbol = core.reserveTokens(0);
        // @audit reserveToken should be assetSymbol2
        assertEq(symbol, assetSymbol1);
    }
}
```

## Recommendations

swap `reserveTokens` and then pop the last element

# [L-02] The expiration time of the uniswap interactive setting is invalid

## Description

The expiration time of uniswap interaction settings is invalid, and the user's transactions may remain in mempool for a long time and they can still be executed.    
The token price or pool status usually changes during this period, so even if the final result is within the slippage range, it can still cause a loss to the user.    
Let's take the most common swap example:    
1. The initial price 1A = 1B, the user initiates the transaction, swap 100 A for 100B    
2. The transaction stays in mempool for a long time due to network congestion, during which the price of B drops and 1A = 1.2B  
3. The searcher finds the arbitrage opportunity, executes tx with the sandwich, the end user gets 100B, and the searcher gets 20B 

The problem occurs because tx has no expiration time protection, and the state of executed and user-initiated transactions has changed a lot after a long time. even if the final result is within the slippage range, there is still a loss to the user.  

## Recommendations

Pass the expiration time through a parameter instead of using the current time or hardcode time

# [L-03] safeApprove implementation does not meet expectations

## Description

```solidity
  function approveTarget(
    address _target,
    address _token,
    uint256 _amount,
    bool use_safe_approve
  ) public onlyRole(DEFAULT_ADMIN_ROLE) {
    if (use_safe_approve) {
      // safeApprove needed for USDT and others for the first approval
      // You need to approve 0 every time beforehand for USDT: it resets
      TransferHelper.safeApprove(_token, _target, _amount);
    } else {
      IERC20WithBurn(_token).approve(_target, _amount);
    }
  }

  function safeApprove(address token, address to, uint value) internal {
    // bytes4(keccak256(bytes('approve(address,uint256)')));
    (bool success, bytes memory data) = token.call(
      abi.encodeWithSelector(0x095ea7b3, to, value)
    );
    require(
      success && (data.length == 0 || abi.decode(data, (bool))),
      "TransferHelper: APPROVE_FAILED"
    );
  }
```

According to the code comments, for tokens such as USDT, the allowance should be reset to 0 before approval.    
But the implementation of safeApprove does not meet expectations, it only checks the return value of approve, and does not reset the allowance.

## Recommendations

set allowance to zero before approve

# [L-04] ReLPContract.reLP doesn't consider token's decimals

## Description

```solidity
  // calculate min amounts to remove
  uint256 mintokenAAmount = tokenAToRemove -
    ((tokenAToRemove * liquiditySlippageTolerance) / 1e8);
  uint256 mintokenBAmount = ((tokenAToRemove * tokenAInfo.tokenAPrice) /
    1e8) -
    ((tokenAToRemove * tokenAInfo.tokenAPrice) * liquiditySlippageTolerance) /
    1e16;
```

When the number of tokenA and tokenB is converted in ReLPContract.reLP, it does not take into account that the token's precision may not be equal.     
For dpxETH/WETH, their precision is 18, and there is no problem in conversion. But if it is other token groups, if the accuracy is not the same, there will be serious problems with the quantity conversion.   

## Recommendations

Consider differences in precision when converting quantities, or when initializing the contract, verify whether the token precision is equal.