Here several aspects about mechanism of the protocol are discussed. Topic 1, 2, 3 are related to each other, and the final suggestion is give users some freedom in pricing. Topic 4 and 5 also have similar suggestion for over protection. The rest topics are independent.

1. [HV derived IV and pricing](#hv-derived-iv-and-pricing)
2. [BSM and perpetual](#bsm-and-perpetual)
3. [incentive for perpetual writer](#incentive-for-perpetual-writer)
4. [depeg restore](#depeg-restore)
5. [price bounce back after settlement](#price-bounce-back-after-settlement)
6. [bond DpxEth is one way process](#bond-dpxeth-is-one-way-process)
7. [inflation/deflation of rDPX](#inflationdeflation-of-rdpx)


# HV derived IV and pricing

IV is an important metric in option pricing. In this contract, traders's flexibility to quote is limited due to the way IV is get. Further, the option liquidity will be affected, as well as the DpxEth bonding availability.

According to the doc in [option-pricing](https://docs.dopex.io/option-fundamentals/option-pricing), IV is based on HV:
> Volatility is based on the 30-day historical volatility of the underlying

The current way seems intended to provide some "fair" price for options. However when trade options, IV is the result, not the input.

Let's recall how IV is derived in option trading: 
1. traders provide bid ask quotes
2. based on the quoted price, IV is derived through BSM as a metric for human to get a quick feeling about the level of price

There is no guarantee that real volatility will converge to IV, or the other way.
Traders provide their quotes based on their risk estimate, their pricing are changing all the time, as a result, the IV smile and term structure are constantly changing. It could has nothing to do with past price path.

The quote also includes slippage, for far OTM options, the slippage is larger, this is the seller's "risk premium". However if everything is based on HV, the IV surface becomes rigid, sometimes the price could be too low, writer's won't be willing to take the downside risk. Or the price could be too high, someone can take advantage of it. However, overall, traders can not adjust the price at will. This mechanism could hinder lots of traders to participate. As a result, the liquidity of the perpetual vault is harder to grow. 

On the other hand, HV value also depends on the way to sample the data, with interval of 1 min, 5 min, 15 min, different HV values will be got. Using this as reference is also prone to error.


Suggestion:
Give the users some freedom on the bid ask price, instead of fixing IV (price). Maybe something similar to AMM could be designed, which can timely reflect the supply and demand of options.


# BSM and perpetual

BSM is used to calculate option premium, however the condition for BSM requires the option has some specific expiry date, which is not the case for perpetual. The time decay of BSM is not even, the time value decay at different speed when the expiry approaches.

As a result, the current formula for premium is not accurate, and inconsistent for different `fundingDuration`. Either the treasury or the users will have unfair gain/loss. 

See the discussion in [BSM has expiry with uneven time decay, it is not for perpetual](https://github.com/code-423n4/2023-08-dopex-findings/issues/1969).

Suggestion:
Empirical approach could be used and might be better, let the traders and market find the best price. Just like real option prices has IV smile (in BSM, IV smile does not exist, it should be flat). The smile is the dynamic equilibrium of market supply and demand. So some idea like AMM could be used to give users the freedom to discover the most appropriate price for perpetual options.



# incentive for perpetual writer

Apart from the flexibility discussed above, the benefits for writers needs attention as well.

25% OTM option premium is negligible. As rDPX price drops towards strike, the premium will be higher, but at the end the seller will incur some loss in face value after settlement.

Then why will the users be willing to write the perpetual put options? The fund efficiency is low with of full collateral requirement. And since this is perpetual, each portion of locked collateral can only be released when settled, which means exercised at a price higher then the market. The writers as a whole group will incur loss at every moment of settlement. It is hard to say whether options fee income can cover the loss. 

With the same amount of eth, if the writer sell puts in deribit:
- higher APR
- has portfolio margin, means much higher fund efficiency
- more ways to hedge, and control over the strategy
- much more flexible if want to exit

There could be 2 reasons for writers to lock collateral:
1. expect the price of rDPX will go up in the long term, since redeemed funds are weth and rDPX
2. some other rewards by the protocol, (currently is share in 3k $DPX/year for the first 3 years paid by the Dopex Treasury)

No 1 is not guaranteed, it won't be the reason for many users. The controllable method is No 2 to add enough incentive for the writers. However this could be tricky, if the rewards is too large, far beyond market iv, some users could take advantage of the protocol, at the end it will be the protocol to take the downside risk, in some sense. If the rewards is too low, not enough writers. Downside protection can not be provided for future bonders. 

Suggestion:
Maybe also consider some high rewards with strict condition such as vesting period.


# depeg restore

The logic to handle depeg might need more tweak. Currently, to prevent rDPX crash, 25% OTM put option is used as downside protection. However, when that happens, it is still using the crashed rDPX as the last resort to buy back weth to restore peg. 

See the discussion in [lower depeg could fail to restore if no enough rDPX](https://github.com/code-423n4/2023-08-dopex-findings/issues/1771).

Suggestion:
One solution is over protection for crash risk.


# price bounce back after settlement

Now we assume there is enough writer to take the downside risk, and settlement are done after some market crash. However the rDPX price soon bounce back. Then the loss is realized at the moment of settlement and becomes permanent. The 93.75% will gradually approaching. The core contract is equivalent to swap at 75% price. When the price bounce back, it's the perpetual LP's profit.

Although later the protocol has other income, and AMO can be used to restore the peg. I think some other way is to over protect the downside. When 25% rDPX is used, the put option can be more than that amount. As a result, if rDPX price drop below the strike, settlement can recover some loss by it's own.

Suggestion:
Same as above topic, over protection for rDPX.


# bond DpxEth is one way process

Current design is rDPX + weth -> DpxEth. 
The one way process could make many users concern whether to bond. After all, bond, waiting for the epoch, redeem is receiptToken, remove liquidity, and finally swap back to weth is a detour process. As such, the liquidity of DpxEth might be hard to scale.

Suggestion: 
Consider add method to convert back, the users could has less to worry when bond DpxEth.


# inflation/deflation of rDPX

If there is long term trend of appreciation/depreciation of rDPX, one of the backing asset of DpxEth, the bonding system may not be easy to maintain sustainability. If in the long term rDPX goes down, few people will be interested in writing perpetual puts. If rDPX always goes up, users may want to hold, rather than bond for DpxEth. 

Although rDPX is not expected to be something like stablecoin, the totalSupply still needs attention, as well as the volatility. Since it is the backing asset, it should maintain some value and reliability. 

Suggestion:
Monitor and control the supply of rDPX from protocol's side.

### Time spent:
30 hours