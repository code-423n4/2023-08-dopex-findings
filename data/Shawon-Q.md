* `setSlippageTolerance` function of `UniV2LiquidityAmo.sol` does critical state changes which other smart contracts interact with.
As Emitting an event when critical state changes occur can be extremely helpful for external observers and users of the contract. 
It provides transparency and allows them to track what's happening on the blockchain.Emitting an event whenever this function is called is recommended.

*In `UniV3LiquidityAmo.sol`There are some hardcoded state variables e.g:uniV3_factory,univ3_positions,univ3_router.Hardcoding addresses directly in a smart contract's constructor 
 is generally not recommended for several reasons:

1.Immutability: Ethereum contracts are immutable once deployed. 
If you hardcode addresses, changing them would require deploying a new contract, which can be costly and impractical.

2.Upgradability: For future-proofing and upgradability, it's often better to use upgradeable patterns like Proxy Contracts or Dependency Injection.
This allows you to replace or upgrade individual components without changing the entire contract.

3.Security: Hardcoding addresses makes the contract less flexible and potentially less secure.
 In case of a vulnerability or a need to upgrade the contract, it's harder to make changes.

Use storage patterns that allow you to upgrade components or contracts without changing the core logic. 
Examples include the Proxy Pattern (such as the OpenZeppelin upgradeable contracts) or the Eternal Storage Pattern.
