# Overall Code Quality: AVERAGE

# Centralization Risks
They are using a centralized oracle, which is very risky. The protocol should consider using a TWAP or chainlink oracle as it is better than having a centralized oracle at any time.

# Testing
The protocol has not done enough testing. We definitely recommend to do more testing. It is so low , that we recommend another audit after this review.
There are no fuzz tests. 

# Gas Optimisations
The contracts aren't gas optimized at all as evident by the large no of issues discovered by the bot and a protocol with a high complexity factor should have done better when it comes to gas optimizations

## Complications
We noticed the devs have made unnecessary complications due to separate accounting for balances. This leads to many uninviting bugs.


Overall it seems that the protocol hurried into the audit as evident by the lack of tests and gas optimisations



### Time spent:
40 hours