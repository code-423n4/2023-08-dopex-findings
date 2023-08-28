`sync` function is public and can be spammed, potentially leading to higher gas costs for genuine users.  

In this contract, there's a function called sync() which is used to reconcile the balances of the two assets in a liquidity pool. If this function is public, anyone can call it.
Vulnerable Code:
```
function sync() public {
    xxx
}
```

Impact
Malicious actors can spam this function, causing unnecessary writes to the Ethereum state, which in turn increases the gas costs. Genuine users who subsequently interact with the contract might then experience higher gas fees.
