## L-01 `removeAssetFromTokenReserves` function does not remove assets correctly from the `reserveTokens` array

[Link to the code](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L287)

`removeAssetFromtokenReserves` function removes tokens from `reserveAsset`, `reserveIndex`, and `reserveTokens`. But from the `reserveTokens` array it is not removed correctly - function removes **last** token no matter what token is intented to be removed.

```solidity
    reservesIndex[reserveTokens[reserveTokens.length - 1]] = index;

    // update the index of reserveAsset with the last element
    reserveAsset[index] = reserveAsset[reserveAsset.length - 1];

    // remove the last element
    reserveAsset.pop();
    reserveTokens.pop();
```

### Recommended solution
Update the code to the following

```solidity
    reservesIndex[reserveTokens[reserveTokens.length - 1]] = index;

    // update the index of reserveAsset with the last element
    string memory lastElement = reserveTokens[reserveTokens.length - 1];
    reservesIndex[lastElement] = index;
    reserveTokens[index] = lastElement;

    // remove the last element
    reserveAsset.pop();
    reserveTokens.pop();
```

