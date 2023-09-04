# [L-01] `removeLiquidity()` always emit 0 for tokenId

```solidity
    FILE: 2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol
    
    263: delete positions_mapping[pos.token_id];

    265: // send tokens to rdpxV2Core
    266: _sendTokensToRdpxV2Core(tokenA, tokenB);

    268: emit log(positions_array.length);
    269: emit log(positions_mapping[pos.token_id].token_id); // @audit emitting deleted mapping value
```
[UniV3LiquidityAmo.sol#L263-L269](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L263-L269)

# [N-01] Check implemented twice to check role
```solidity
  FILE: 2023-08-dopex/contracts/decaying-bonds/RdpxDecayingBonds.sol
  118: ) external onlyRole(MINTER_ROLE) {
  119:  _whenNotPaused();
  120:  require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
```

[RdpxDecayingBonds.sol#L118-L120](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L118-L120)