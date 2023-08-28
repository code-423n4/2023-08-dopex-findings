## 1. Spelling error in UniV3LiquidityAmo.sol

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L11

**Uniswap V3** is spelt as **Uniswamp V3**

#### Recommendation

Change Uniswamp V3 in Line 11 to Uniswap V3

## 2. Improper comment placement on line 376
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L376

The comment: 

`
/*
   **  burn tokenAmount from the recipient and send tokens to the receipient
   */
`
on line **376** to **378** is placed below the event **LogAssetTransfered** which it is meant to explain, this is quite confusing as comments are naturally placed above the line of code its meant to clarify

#### Recommendation

Move the comment directly above line **370** which is directly above the code its meant to explain

 