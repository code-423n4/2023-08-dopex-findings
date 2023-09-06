0. RdpxV2Core::settle() - Gas optimization possible in `for loop`.

https://github.com/code-423n4/2023-08-dopex/blob/e96aaa5ea21f11b29d828dbe2d0745974cd046ed/contracts/core/RdpxV2Core.sol#L775-L777

Current code:
```solidity
    for (uint256 i = 0; i < optionIds.length; i++) {  /// @audit-issue GAS
      optionsOwned[optionIds[i]] = false;
    }
```

Recommended gas optimizations:
```solidity
 ++ uint256 optionIdsLength = optionIds.length;
 ++ for (uint256 i; i < optionIdsLength;) {
      optionsOwned[optionIds[i]] = false;
      
 ++   unchecked {
 ++   	i++;
 ++   }
    }
```


1. RdpxV2Core::bondWithDelegate() - Gas optimization possible in `for loop`.

https://github.com/code-423n4/2023-08-dopex/blob/e96aaa5ea21f11b29d828dbe2d0745974cd046ed/contracts/core/RdpxV2Core.sol#L836-L867

Recommendation:

```solidity
 ++ uint256 _amountsLength = _amounts.length;
 ++ for (uint256 i; i < _amountsLength;) {
    
        /// `for loop` logic
    
 ++     unchecked {
 ++         i++;
 ++     }
    }
```

2. 

https://github.com/code-423n4/2023-08-dopex/blob/e96aaa5ea21f11b29d828dbe2d0745974cd046ed/contracts/core/RdpxV2Core.sol#L167-L170

Existing code/`for loop`:
```solidity
    for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }
```

Recommendation:
```solidity
 ++ uint256 tokensLength = tokens.length;
 ++ for (uint256 i; i < tokensLength;) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
 ++   unchecked {
 ++       i++;
 ++   }
    }
```
