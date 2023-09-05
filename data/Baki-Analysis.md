# Overall Code Quality : AVERAGE

# Centralization Risks
They are using a centralized oracle, which is very risky. The protocol should consider using a TWAP or chainlink oracle. As it is better than having a centralized oracle at any time.

# Testing
Protocol has not done enough testing. We definitely recommend to do more testing. It is so low , that we recommend another audit after this review.
There are no fuzz tests. 

## Complications
We noticed the devs have made unnecessary complications due to separate accounting for balances. This leads to many uninviting bugs.

### Time spent:
40 hours