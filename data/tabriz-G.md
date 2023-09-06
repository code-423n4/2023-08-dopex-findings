## [G-01] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

The EVM operates with 32 byte words. Therefore, if you declare state variables less than 32 bytes the EVM will need to perform extra operations to cast your value to the specified size.

```
int24 tickLower; // the tick range of the position
    int24 tickUpper;
    uint24 fee_tier;
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L44C3-L46C21

```
int24 _tickLower;
    int24 _tickUpper;
    uint24 _fee;
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L53C3-L55C17

## [G-02] Redundant zero initialization

Solidity does not recognize null as a value, so uint variables are initialized to zero. Setting a uint variable to zero is redundant and can waste gas.

There are several places where an int is initialized to zero, which looks like:

```
uint256 public latestFundingPaymentPointer = 0;
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L89

## [G-03] Events are not indexed

The emitted events are not indexed, making off-chain scripts such as front-ends of dApps difficult to filter the events efficiently.

Recommend adding the `indexed` keyword in each event,

```Solidity
event LogAddLiquidity(
    address indexed sender,
    uint256 tokenAAmount,
    uint256 tokenBAmount,
    uint256 tokenAAmountMin,
    uint256 tokenBAmountMin,
    uint256 tokenAUsed,
    uint256 tokenBUsed,
    uint256 lpReceived
  );

  event LogRemoveLiquidity(
    address indexed sender,
    uint256 lpAmount,
    uint256 tokenAAmountMin,
    uint256 tokenBAmountMin,
    uint256 tokenAReceived,
    uint256 tokenBReceived
  );

  event LogSwap(
    address indexed sender,
    uint256 token1Amount,
    uint256 token2AmountOutMin,
    bool swapTokenAForTokenB,
    uint256 token2Amount
  );

  event LogAssetsTransfered(
    address indexed sender,
    uint256 tokenAAmount,
    uint256 tokenBAmount
  );

  event LogEmergencyWithdraw(address sender, address[] tokens);
}
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L387C2-L422C2

```
event RecoveredERC20(address token, uint256 amount);
  event RecoveredERC721(address token, uint256 id);
  event LogAssetsTransfered(
    uint256 tokenAAmount,
    uint256 tokenBAmount,
    address tokenAAddress,
    address tokenBAddress
  );
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L368C1-L375C5

```
event log(uint);
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L379

## [G-04] Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

```
uint256 public constant RDPX_RATIO_PERCENTAGE = 25 * DEFAULT_PRECISION;

  /// @notice ETH Ratio required when bonding
  uint256 public constant ETH_RATIO_PERCENTAGE = 75 * DEFAULT_PRECISION;
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L88C1-L91C73

## [G-05] Use the unchecked keyword to save gas
 
 Using the unchecked keyword to avoid redundant arithmetic underflow/overflow checks to save gas when an underflow/overflow cannot happen.
 
 We can apply the unchecked keyword in the following line of code since ther are require statements before to ensure the arithmetic operations would not cause an integer underflow or overflow.
 
 For example, change the code at line 276 to:
 ```
 unchecked{
   uint256 requiredCollateral = (amount * strike) / 1e8;
 }
 
```
 
 ```
 uint256 requiredCollateral = (amount * strike) / 1e8;
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L276C3-L276C58

```
 uint256 timeToExpiry = nextFundingPaymentTimestamp() - block.timestamp;
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L283C4-L283C76

```

 uint256 amount = optionsPerStrike[strike] -
        fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
          strike
        ];
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L421C6-L424C11

```
 uint256 timeToExpiry = nextFundingPaymentTimestamp() -
        (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration));
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L426C6-L428C1

```
  uint256 startTime = lastUpdateTime == 0
          ? (nextFundingPaymentTimestamp() - fundingDuration)
          : lastUpdateTime;
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L467C7-L469C28

so on...

## [G-06] Use calldata instead of memory for funcation arguments that do not get mutated
```
  function collectFees() external onlyRole(DEFAULT_ADMIN_ROLE) {
    for (uint i = 0; i < positions_array.length; i++) {
      Position memory current_position = positions_array[i];
      INonfungiblePositionManager.CollectParams
        memory collect_params = INonfungiblePositionManager.CollectParams(
          current_position.token_id,
          rdpxV2Core,
          type(uint128).max,
          type(uint128).max
        );

```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L121

```
function removeLiquidity(
    uint256 positionIndex,
    uint256 minAmount0,
    uint256 minAmount1
  ) public onlyRole(DEFAULT_ADMIN_ROLE) {
    Position memory pos = positions_array[positionIndex];
    INonfungiblePositionManager.CollectParams
      memory collect_params = INonfungiblePositionManager.CollectParams(
        pos.token_id,
        rdpxV2Core,
        type(uint128).max,
        type(uint128).max
      );
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L213C3-L225C9

## [G-07] Cache state variable outside of loop to avoid reading storage on every iteration
 
```
   for (uint i = 0; i < positions_array.length; i++) {
      Position memory current_position = positions_array[i];
      INonfungiblePositionManager.CollectParams
        memory collect_params = INonfungiblePositionManager.CollectParams(
          current_position.token_id,
          rdpxV2Core,
          type(uint128).max,
          type(uint128).max
        );
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L120

```
 for (uint256 i = 0; i < optionIds.length; i++) {
      uint256 strike = optionPositions[optionIds[i]].strike;
      uint256 amount = optionPositions[optionIds[i]].amount;

      // check if strike is ITM
      _validate(strike >= getUnderlyingPrice(), 7);

      ethAmount += (amount * strike) / 1e8;
      rdpxAmount += amount;
      optionsPerStrike[strike] -= amount;
      totalActiveOptions -= amount;

      // Burn option tokens from user
      _burn(optionIds[i]);

      optionPositions[optionIds[i]].strike = 0;
    }
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L328

## [G-08] Use do while loops instead of for loops
A do while loop will cost less gas since the condition is not being checked for the first iteration.

```
 for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L147C4-L150C6

```
 for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L225C4-L229C1