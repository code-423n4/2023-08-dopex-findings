## vulnerability details with code :

in the  collectFees function there  is  a loop that iterates through positions to collect fees. 
and Gas consumption increases with the number of iterations so If the gas limit is exceeded, the transaction fails.
let's say there are 10,000 positions in positions_array, and each iteration consumes a certain amount of gas. Ethereum has a block gas limit, let's say 12,500,000 gas units for this example. 
If each iteration consumes 50,000 gas, and there are 10,000 iterations, the total gas consumption would be 500,000,000 gas units, which exceeds the block's gas limit.

- Here is the code :https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L120C5-L133C4

- Here is the vulnerable part in code that can cause the issue that already explain in the vulnerability details:
```solidity
for (uint i = 0; i < positions_array.length; i++) {
      Position memory current_position = positions_array[i];
      INonfungiblePositionManager.CollectParams
        memory collect_params = INonfungiblePositionManager.CollectParams(
          current_position.token_id,
          rdpxV2Core,
          type(uint128).max,
          type(uint128).max
        );

      // Send to custodian address
      univ3_positions.collect(collect_params);
    }
```
## impact

If gas consumption exceeds the block's gas limit, the collectFees transaction will fail, preventing the collection of fees for all positions in the loop.
## recommendation
to fix this i think it's implement  a batch Instead of collecting fees for all positions in a single loop