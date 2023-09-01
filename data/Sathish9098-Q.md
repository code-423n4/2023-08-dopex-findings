# LOW FINDINGS

##

## [L-] The ``DEFAULT_ADMIN_ROLE`` to withdraw tokens, regardless of whether the situation is ``genuinely`` an ``emergency`` or ``not ``

### Impact
Allowing ``administrators`` to withdraw tokens without restriction can create a ``conflict of interest``, where they may prioritize their own interests over the ``protocol's well-being`` . This will create unexpected behavior of the protocol 


### POC
```solidity
FILE: 2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol

function emergencyWithdraw(
    address[] calldata tokens
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    IERC20WithBurn token;

    for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }

    emit LogEmergencyWithdraw(msg.sender, tokens);
  }

```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L142-L153

### Recommended Mitigation
Create a ``state variable`` that represents the emergency state of the contract. This variable can be a boolean flag (e.g., bool public emergencyActive) that is initially set to ``false``. Only authorized addresses can change this variable to ``true`` when an emergency situation occurs.

```solidity

Require(emergencyActive, "Protocol is not in emergency situation");

```

##

## [L-2] Using a very short deadline ``(block.timestamp + 1)`` can lead to ``Reverts``

### Impact

- If the deadline expires ```repeatedly``, it may discourage other users from adding ``liquidity`` to the ``pool``. This could lead to a ``liquidity crisis in the pool``.

- The deadline is the time at which the transaction will revert if it is not mined. If the ``network is congested``, the transaction may not be mined before the ``deadline`` expires.

- The deadline does not account for the time it takes for the transaction to propagate through the network. This means that the transaction may not be mined until after the deadline has expired.

### Impact

```solidity
FILE : Breadcrumbs2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol

 // add Liquidity
    (tokenAUsed, tokenBUsed, lpReceived) = IUniswapV2Router(addresses.ammRouter)
      .addLiquidity(
        addresses.tokenA,
        addresses.tokenB,
        tokenAAmount,
        tokenBAmount,
        tokenAAmountMin,
        tokenBAmountMin,
        address(this),
        block.timestamp + 1 //@audit not enough time 
      );

```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L231

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L279

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L342

### Recommended Mitigation
Use a ``longer deadlines ``

##

## [L-] Ensuring ``sufficient gas`` to prevent reverts in transactions involving ``UniswapV2Router`` and ``UniswapV2Pair ``

### Impact
The gas price of the transactions ``addLiquidity()``, ``removeLiquidity()``, and ``swap()`` must be high enough to ensure that they are mined quickly. If the gas price is too low, the transactions may not be mined before the ``deadline expires``, and they will be reverted. This is because the ``UniswapV2Router`` and ``UniswapV2Pair`` contracts use a ``complex pricing mechanism`` that ``requires`` a ``significant amount of gas to execute``

### POC

```solidity
FILE: Breadcrumbs2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol

 // swap token A for token B
    token2Amount = IUniswapV2Router(addresses.ammRouter)
      .swapExactTokensForTokens(
        token1Amount,
        token2AmountOutMin,
        path,
        address(this),
        block.timestamp + 1
      )[path.length - 1];


// remove liquidity
    (tokenAReceived, tokenBReceived) = IUniswapV2Router(addresses.ammRouter)
      .removeLiquidity(
        addresses.tokenA,
        addresses.tokenB,
        lpAmount,
        tokenAAmountMin,
        tokenBAmountMin,
        address(this),
        block.timestamp + 1
      );

// add Liquidity
    (tokenAUsed, tokenBUsed, lpReceived) = IUniswapV2Router(addresses.ammRouter)
      .addLiquidity(
        addresses.tokenA,
        addresses.tokenB,
        tokenAAmount,
        tokenBAmount,
        tokenAAmountMin,
        tokenBAmountMin,
        address(this),
        block.timestamp + 1
      );

```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L221-L232

### Recommended Mitigation
Add gas check before execute transaction

```
Require(gasLeft() > expectedGas, ``Not sufficient gas to execute this transaction``);

```
##

## [L-] Excessive Dependency on ``DEFAULT_ADMIN_ROLE`` in ``UniV2LiquidityAmo`` Contract Functions

### Impact
All functions in a contract are too much dependent on the ``DEFAULT_ADMIN_ROLE``, it means that the contract is ``too centralized``. This is because the ``DEFAULT_ADMIN_ROLE`` is a single address that has control over ``all of the functions`` in the contract. This can be a ``security risk``, as it means that if the ``DEFAULT_ADMIN_ROLE`` is ``compromised``, then all of the functions in the contract can be controlled by the ``attacker``

```solidity
FILE: Breadcrumbs2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol

 function setSlippageTolerance(
    uint256 _slippageTolerance
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {

 function approveContractToSpend(
    address _token,
    address _spender,
    uint256 _amount
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {

 function emergencyWithdraw(
    address[] calldata tokens
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {

 function addLiquidity(
    uint256 tokenAAmount,
    uint256 tokenBAmount,
    uint256 tokenAAmountMin,
    uint256 tokenBAmountMin
  )
    external
    onlyRole(DEFAULT_ADMIN_ROLE)

 function swap(
    uint256 token1Amount,
    uint256 token2AmountOutMin,
    bool swapTokenAForTokenB
  ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256 token2Amount) {

```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L304-L308

### Recommended Mitigation
You can ``spread the control`` of the functions across ``multiple addresses``












1.Revert on Approval To Zero Address

Some tokens (e.g. OpenZeppelin) will revert if trying to approve the zero address to spend tokens (i.e. a call to approve(address(0), amt)).

Integrators may need to add special cases to handle this logic if working with such a token.

2.Low Decimals

Some tokens have low decimals (e.g. USDC has 6). Even more extreme, some tokens like Gemini USD only have 2 decimals.

This may result in larger than expected precision loss.

3.High Decimals

Some tokens have more than 18 decimals (e.g. YAM-V2 has 24).

This may trigger unexpected reverts due to overflow, posing a liveness risk to the contract.

4.Non string metadata
Some tokens (e.g. MKR) have metadata fields (name / symbol) encoded as bytes32 instead of the string prescribed by the ERC20 specification.

This may cause issues when trying to consume metadata from these tokens.

5.No Revert on Failure
Some tokens do not revert on failure, but instead return false (e.g. ZRX, EURS).

While this is technically compliant with the ERC20 standard, it goes against common solidity coding practices and may be overlooked by developers who forget to wrap their calls to transfer in a require.

6. push 0 problems when version more than 0.8.19

7. Divide by zero should be avoided 

8. latest openzepelin version 

9. All proxy contracts initialized in initialize function

10. initializer could be front run 

11) All hard coded values right ?

12. add blocklist function for NFT 

13) Is timeclock function implemented ?

14) is there any swap function . Slippage protection, deadline, Hardcoded Slippage ?, 

15. Can the 1st deposit raise a problem ?

16) The contract implement a white/blacklist ? or some kind of addresses check ? is blocklist and whitelist tokens checked

17) Solmate ERC20.sageTransferLib do not check the contract existence

18) msg.value not checked can have result in unexpected behaviour

19) Is the function refunds the extra amount paid ? when using msg.value 

Was disableInitializers() called ?

if any contract inheritance has a constructor (erc20, reentrancyGuard, Pausable…) is used : use the upgreadable version for initialize

Signature Malleability : do not use escrevover() but use the openzepplin/ECDSA.sol (The last version should be used here)

Take care if (receiver == caller) can have unexpected behaviour

Hash collisions are possible with abi.encodePacked (here)

Is this possible the oracle returns state data. LatestAnswer() 

Consider implementing two-step procedure for updating protocol addresses

Direct supportsInterface() calls may cause caller to revert



