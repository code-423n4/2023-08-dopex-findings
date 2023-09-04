## Suggestions to consider including in your analysis

First think I've noticed that code was written by different people with different codestyle,
next observation -- Time to market was minimal, because here obvious typos and code-smells which
should be fixed during review. I'm about typos, copy-paste, un-optimal calls and so on.

## Any comments for the judge to contextualize your findings

My review I started from small contracts, just to ensure that they are ok.
Next, I went through main logic: Core, Vault and VaultLP. At the last I considered ReLP and Uniswap
contracts.

## Approach taken in evaluating the codebase

To review the codebase I used manual review with echidna to figure out typical improvements -- like
variable which can be immutable.
Next, I used authors explanation,
see: https://dopex.notion.site/rDPX-V2-RI-b45b5b402af54bcab758d62fb7c69cb4#a7503a215a984244b3a54021259f6ae9
And Dopex's site with option fundamentals,
see: https://docs.dopex.io/option-fundamentals/option-settlement
Moreover I used RemixIDE to run small parts of code, to ensure their correctness.

## Architecture recommendations

Current architecture has one main advantages -- it is real to understand.
The components are strongly connected with each other, but it's still real to figure out what's
going on.
Main disadvantages -- all features placed in same contract. This makes it difficult to understand.
For example, option pricing can be separated from Vault to independent library.

Next, PUT Option in Core contract -- at the first, this is not an option -- neither american no
european.
I'm really surprised that a project dealing with options has implemented european options in such
way.
I suggest to separate option storage and handling from Core contract to independent contract, which
has manager -- Core address.

Next, suggestion -- Now creating contract is too overcomplicated and too unstable to rights
assignment errors.
Because manager access granted during contract creation, and in some cases contract creator got
role, but hasn't.
So, do next -- Create Factory contract, like in UniswapV3, which should work like this:
https://github.com/Uniswap/v3-core/blob/main/contracts/UniswapV3PoolDeployer.sol
https://github.com/Uniswap/v3-core/blob/main/contracts/UniswapV3Factory.sol
It creates contract, then initializes fields in predefined way.

Advantages:

* Easy to deploy.
* Easy to figure out who is responsible for code part.
* Easy to figure out linked params, like tokens, oracles.

Errors. Don't do like this. I've found a lot of discrepancies with error code and what
really happened. It seems that the developers themselves do not know which error code carries what
meaning.

Instead of just constant you can have the same constants in a separate file (library for example)

```solidity

val ZERO_ADDRESS = 0
...
```

Why is it better — you decided that `ZERO_ADDRESS` is now not 0, but 1
in the current solution, you need to go through all the files — carefully replace them from 0 with

1.

Here you can just replace 0 with 1 in one place and everything will work.

Next proposal -- all protocol depends on admin calls.

## Codebase quality analysis

I've found tests, but they are not exhausted, and not covered all cases.
As I mentioned above:

* There was a staff turnover.
* I suppose it was possible to merge into main without review.
* Copy-paste in obvious places.

## Centralization risks

Whole protocol depends on admin:

* Liquidity providers won't get any income if manager won't call `settle` function.
* Admin can withdraw all money from all contracts
* Admin can settle any option as he wishes.
* Admin must call `provideFunding()` at the beginning of each epoch.
* Only admin can handle depeg.

So, all points I denoted above -- indicate that the contract is very centralized and completely
dependent on the manager.

## Systemic risks

There a risk of possibility to stop whole platform with not frequent calling of some functions,
which may cause an out of gas error and the entire contract will get stuck.

Admin handles token peg to ether, this lead to possibility to depeg and as result collapse of the
entire protocol and loss of users' money.

### Time spent:
60 hours