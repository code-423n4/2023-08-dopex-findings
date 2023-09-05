# Approach taken in evaluating the codebase
Steps:

- `Use a static code analysis tool:` Static code analysis tools can scan the code for potential bugs and vulnerabilities. These tools can be used to identify a wide range of issues, including:

  -  Insecure coding practices
  -  Common vulnerabilities
  -  Code that is not compliant with security standards
- `Read the documentation:` The documentation for Lybra Finance should provide a detailed overview of the protocol and its codebase. This documentation can be used to understand the purpose of the code and to identify potential areas of concern.

- `Scope the analysis:` Once you have a basic understanding of the protocol and its codebase, you can start to scope the analysis. This involves identifying the specific areas of code that you want to focus on. For example, you may want to focus on the code that handles user input, the code that interacts with external APIs, or the code that stores sensitive data.

- `Manually review the code:` Once you have scoped the analysis, you can start to manually review the code. This involves reading the code line-by-line and looking for potential problems. Some of the things you should look for include:

   - Unvalidated user input
   - Hardcoded credentials
   - Insecure cryptographic functions
   - Unsafe deserialization

- `Mark vulnerable code parts with @audit tags:` Once you have identified any potential vulnerabilities, you should mark them with @audit tags. This will help you to identify the vulnerable code parts later on.

- `Dig deep into vulnerable code parts and compare with documentations:` For each vulnerable code part, you should dig deep to understand how it works and why it is vulnerable. You should also compare the code with the documentation to see if there are any discrepancies.

- `Perform a series of tests:` Once you have finished reviewing the code, you should perform a series of tests to ensure that it works as intended. These tests should cover a wide range of scenarios, including:

  - Valid and invalid user input
  - Different types of attacks
  - Different operating systems and hardware platforms
- `Report any problems:` If you find any problems with the code, you should report them to the developers of Lybra Finance. The developers will then be able to fix the problems and release a new version of the protocol.


# Codebase quality analysis
`UniV2LiquidityAmo.sol`

- `Emergency Withdraw:` This issue is found in the emergencyWithdraw function. This function allows the contract admin to withdraw all funds, which could be potentially abused.

- `Block Timestamp Manipulation:` This issue is found in the addLiquidity, removeLiquidity, and swap functions. These functions use block.timestamp for setting deadlines in Uniswap interactions, which could potentially be manipulated by miners.

- `Lack of Input Validation:` This issue is found in the swap function. This function does not validate its inputs, which could lead to unexpected behavior if incorrect values are passed.

- `Potential Front-Running:` This issue is found in the addLiquidity, removeLiquidity, and swap functions. These functions could potentially be front-run by bots due to the use of block.timestamp.

`UniV3LiquidityAmo.sol`

* `No checks for zero addresses:` The constructor does not check if the _rdpx and _rdpxV2Core addresses are non-zero. This could lead to potential issues if zero addresses are accidentally passed.

- `No checks for contract existence:` The constructor does not check if the _rdpx and _rdpxV2Core addresses are actually contracts. This could lead to potential issues if non-contract addresses are accidentally passed.



- `Reentrancy guard:` The contract does not use a reentrancy guard. This could potentially allow for reentrancy attacks, especially in functions that involve external calls such as addLiquidity, removeLiquidity, and swap.

- `No checks for overflows/underflows:` Although the contract uses SafeMath for uint256 operations, it does not check for overflows/underflows in operations involving other data types.


- `No checks for successful external calls:` The contract does not check if external calls are successful. This could lead to unexpected behavior if an external call fails.

- `Use of type(uint128).max and type(uint256).max:` These could potentially lead to overflow issues.


- The execute function allows arbitrary calls with arbitrary data to be executed. This is a major security risk as it could allow the contract to be manipulated in unexpected ways.

`RdpxV2Bond.sol`

- `No Rate Limiting:` The mint function does not have any rate limiting. This means if an address with the MINTER_ROLE wanted to, they could mint an unlimited number of tokens, potentially diluting the value of existing tokens. This issue is found in the mint function.

- `No Input Validation:` The mint function does not perform any checks on the 'to' address. If a zero address is passed, the function will throw an exception and revert, wasting gas. This issue is found in the mint function.

`RdpxDecayingBonds.sol`

- `Re-entrancy Attack:` The emergencyWithdraw function (line 89-107) makes external calls inside a loop. This could potentially lead to a re-entrancy attack if one of the token contracts has a malicious safeTransfer function.

- `Gas Limit Issues:` The emergencyWithdraw function (line 89-107) could potentially run out of gas if the list of tokens is very large, as it makes external calls in a loop.

`DpxEthToken.sol`

- `No Burner Role Assignment:` The contract has a BURNER_ROLE defined, but there is no function to grant this role to any address. This means that no one will be able to call the burn and burnFrom functions. This is found in the BURNER_ROLE constant and the burn and burnFrom functions.

- `No Role Revocation:` There are no functions to revoke roles. If an address is compromised, there is no way to remove its privileges. This is a general issue with the contract.

- `No Checks for Zero Address:` The mint function does not check if the 'to' address is the zero address. If the zero address is passed in, tokens will be minted to the zero address and effectively be burned. This is found in the mint function.

`PerpetualAtlanticVault.sol`

- `Front-Running Attack:` The purchase function is vulnerable to front-running attacks. An attacker can observe a transaction that calls this function and then front-run it by offering a higher gas price. This could allow the attacker to manipulate the price of the underlying asset.

- `Lack of Input Validation:` In the setAddresses function, there is no validation to check if the input addresses are contract addresses. This could lead to potential issues if a user accidentally inputs an EOA (Externally Owned Account) address instead of a contract address.


`PerpetualAtlanticVaultLP.sol`

- `Integer Underflow:` In the redeem function, the allowed variable could underflow if shares is greater than allowed. This is mitigated by the SafeMath library which prevents underflows and overflows.

- `No Access Control:` The contract does not have any access control mechanisms. This means that anyone can call any function. This is not necessarily a security issue, but it could lead to unexpected behavior if not handled correctly.

- `Potential for Front-Running:` The deposit and redeem functions are public and do not have any mechanisms to prevent front-running. This means that a malicious actor could potentially manipulate the price of the LP token by front-running transactions.

`ReLPContract.sol`

- `No Input Validation:` In the setAddresses function, there is no validation to check if the input addresses are contract addresses. This could lead to potential issues if a non-contract address is passed. This issue is found in the setAddresses function.

- `No Checks on External Calls:` In the reLP function, there are several external calls to other contracts without any checks on their return values. This could lead to unexpected behavior if these calls fail. This issue is found in the reLP function.

- `Potential Reentrancy Attack:` The reLP function makes external calls to other contracts and then continues to execute further logic. This could potentially lead to a reentrancy attack if the called contract is malicious. This issue is found in the reLP function.


# Centralization risks
A single point of failure is not acceptable for this project. Centrality risk is high in the project as the role of onlyRole detailed below has very critical and important powers:

Project and funds may be compromised by a malicious or stolen private key onlyRole msg.sender

```solidity
1-  UniV2LiquidityAmo.sol
  // Line 68-80
  function setAddresses(
    address _tokenA,
    address _tokenB,
    address _pair,
    address _rdpxV2Core,
    address _rdpxOracle,
    address _ammFactory,
    address _ammRouter
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    ...
  }

  // Line 86-92
  function setSlippageTolerance(
    uint256 _slippageTolerance
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    ...
  }

  // Line 100-109
  function approveContractToSpend(
    address _token,
    address _spender,
    uint256 _amount
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    ...
  }

  // Line 113-123
  function emergencyWithdraw(
    address[] calldata tokens
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    ...
  }

  // Line 176-209
  function addLiquidity(
    uint256 tokenAAmount,
    uint256 tokenBAmount,
    uint256 tokenAAmountMin,
    uint256 tokenBAmountMin
  )
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 tokenAUsed, uint256 tokenBUsed, uint256 lpReceived)
  {
    ...
  }

  // Line 213-239
  function removeLiquidity(
    uint256 lpAmount,
    uint256 tokenAAmountMin,
    uint256 tokenBAmountMin
  )
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 tokenAReceived, uint256 tokenBReceived)
  {
    ...
  }

  // Line 248-278
  function swap(
    uint256 token1Amount,
    uint256 token2AmountOutMin,
    bool swapTokenAForTokenB
  ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256 token2Amount) {
    ...
  }

2-UniV3LiquidityAmo.sol

97: function collectFees() external onlyRole(DEFAULT_ADMIN_ROLE) {
    // function body
}

139: function approveTarget(
    address _target,
    address _token,
    uint256 _amount,
    bool use_safe_approve
) public onlyRole(DEFAULT_ADMIN_ROLE) {
    // function body
}

155: function addLiquidity(
    AddLiquidityParams memory params
) public onlyRole(DEFAULT_ADMIN_ROLE) {
    // function body
}

221: function removeLiquidity(
    uint256 positionIndex,
    uint256 minAmount0,
    uint256 minAmount1
) public onlyRole(DEFAULT_ADMIN_ROLE) {
    // function body
}

269: function swap(
    address _tokenA,
    address _tokenB,
    uint24 _fee_tier,
    uint256 _amountAtoB,
    uint256 _amountOutMinimum,
    uint160 _sqrtPriceLimitX96
) public onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256) {
    // function body
}

297: function recoverERC20(
    address tokenAddress,
    uint256 tokenAmount
) external onlyRole(DEFAULT_ADMIN_ROLE) {
    // function body
}

305: function recoverERC721(
    address tokenAddress,
    uint256 token_id
) external onlyRole(DEFAULT_ADMIN_ROLE) {
    // function body
}

313: function execute(
    address _to,
    uint256 _value,
    bytes calldata _data
) external onlyRole(DEFAULT_ADMIN_ROLE) returns (bool, bytes memory) {
    // function body
}  

3 - RdpxV2Bond.sol
29  function pause() public onlyRole(DEFAULT_ADMIN_ROLE) {

33  function unpause() public onlyRole(DEFAULT_ADMIN_ROLE) {

39  ) public onlyRole(MINTER_ROLE) returns (uint256 tokenId) {     

4- RdpxV2Core.sol
in RdpxV2Core contract 23 instance of onlyRole

5- RdpxDecayingBonds.sol
in RdpxDecayingBonds contract 5 instance of onlyRole

6- DpxEthToken.sol
in DpxEthToken contract 5 instance of onlyRole


8- PerpetualAtlanticVault.sol
in PerpetualAtlanticVault contract 11 instance of onlyRole

9- ReLPContract.sol
in ReLPContract contract 5 instance of onlyRole
```

# Gas Optimization

`UniV3LiquidityAmo.sol`
- `Use of IERC20WithBurn(params._tokenA).approve:` This is called every time in addLiquidity, even if the approval is not necessary. Consider checking if the approval is necessary before calling approve.

- `Use of positions_array.length in numPositions:` This could be replaced with a state variable that is incremented and decremented as positions are added and removed. This would save the gas cost of accessing the length of the array.

- `Use of delete in removeLiquidity:` delete does not actually save gas and can be removed.

- `Use of TransferHelper.safeTransfer:` If the tokens being transferred are known to be compliant with the ERC20 standard, consider using transfer instead to save gas.

- `Use of positions_array.push:` Consider using a state variable to keep track of the length of the array and manually setting the value at the index. This can save gas over using push.

`RdpxV2Bond.sol`

- `Batch Minting:` Instead of minting one token per transaction in the mint function, consider implementing a function that allows for minting multiple tokens at once. This could significantly reduce the gas cost per token.

- `Input Validation:` In the mint function, add a require statement to check the 'to' address. This will prevent transactions that are destined to fail from being submitted, saving gas.

- `Efficient Storage:` In the contract, you are using a Counter from OpenZeppelin library. If the counter is not necessary and a simple uint256 would suffice, consider replacing it to save gas.

- `Use External Instead of Public:` If a function is not called from within the contract, consider declaring it as external instead of public to save gas. For example, the mint function could be declared as external if it's not called internally.

- `Remove Unused Code:` If there are any functions or variables in the contract that are not being used, consider removing them. This can help to reduce the overall gas cost of deploying the contract. For example, if the pause and unpause functions are not used, they could be removed.

`RdpxDecayingBonds.sol`
- `Loop Optimization:` In the emergencyWithdraw function (line 89-107), there is a loop that makes external calls to safeTransfer function of the ERC20 token contracts. This could potentially consume a lot of gas if the list of tokens is very large. A more gas-efficient approach could be to require the admin to call the emergencyWithdraw function once for each token they want to withdraw, rather than passing in an array of tokens.

- `Redundant Role Check:` In the mint function (line 119-130), there is a redundant role check. The function modifier onlyRole(MINTER_ROLE) already ensures that the caller has the minter role, so the require statement require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter"); is unnecessary and consumes extra gas.

- `Use of Structs:`The contract uses a struct Bond to store bond information. While this is not necessarily a problem, it's worth noting that structs can be more gas-intensive than using separate mappings for each property. If gas optimization is a priority, you could consider using separate mappings for owner, expiry, and rdpxAmount.

- `Use of Events:` The contract emits an event BondMinted in the mint function (line 119-130). While events are useful for tracking contract activity, they do consume gas. If gas optimization is a priority, you could consider reducing the use of events or only emitting them in critical functions.

- `Use of SafeERC20:` The contract uses the SafeERC20 library for ERC20 token interactions. While this library provides additional safety checks, it can be more gas-intensive than using the standard ERC20 functions directly. If gas optimization is a priority, and you're confident in the safety of the ERC20 tokens you're interacting with, you could consider using the standard ERC20 functions directly.

`DpxEthToken.sol`

- `Redundant Role Assignment:` In the constructor, the deployer is granted the DEFAULT_ADMIN_ROLE, which is unnecessary because the deployer is automatically assigned this role. Removing this line can save gas. This is found in the constructor function.

- `Use of keccak256 for Role Definition:` The contract uses keccak256 to define roles, which is more expensive in terms of gas than simply using a bytes32 literal. This is found in the PAUSER_ROLE, MINTER_ROLE, and BURNER_ROLE constants.

- `Redundant _whenNotPaused Call:` The _beforeTokenTransfer function calls _whenNotPaused, which checks if the contract is paused. However, this check is unnecessary because the ERC20 _transfer function (which calls _beforeTokenTransfer) already checks if the contract is paused. Removing this call can save gas. This is found in the _beforeTokenTransfer function.

- `Use of _msgSender():` The burn function uses _msgSender() to get the sender's address, which is more expensive in terms of gas than simply using msg.sender. This is found in the burn function.

`PerpetualAtlanticVault.sol`

- `Redundant SafeMath:` In Solidity 0.8.0 and later, SafeMath functions are built into the language and are automatically applied to all arithmetic operations. The contract uses SafeMath for arithmetic operations, which is unnecessary and increases gas costs. For example, in the calculatePremium function, SafeMath is used for multiplication and division.

- `Redundant External Calls:` The calculateFunding function makes multiple external calls to the same contract which could be optimized. Instead of calling optionsPerStrike[strike] and fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][strike] multiple times, you could store the result in a local variable and reuse it.

- `Loop Optimization:` In the settle function, there is a loop that iterates over the optionIds array. If this array is large, it could cause the function to run out of gas. Consider using a pattern that allows users to settle their options in smaller batches.

- `Storage Optimization:` The contract uses multiple mappings to store data. For example, fundingPaymentsAccountedFor, fundingPaymentsAccountedForPerStrike, totalFundingForEpoch, optionsPerStrike, and latestFundingPerStrike. Some of this data could potentially be stored more efficiently, reducing the amount of storage used and therefore reducing gas costs.

- `Use of transferFrom:` In several places, the contract uses the safeTransferFrom function to move tokens. This function requires an approval transaction before it can be used, which doubles the gas cost. If possible, consider using transfer instead, which does not require an approval transaction.

`ReLPContract.sol`

- `Redundant Safe Approvals:` In the setAddresses function, the contract is setting unlimited allowances for the AMM router on behalf of itself for tokenA, tokenB, and the pair. If these tokens are not expected to change frequently, these approvals could be set once in the constructor, saving gas for future transactions. This issue is found in the setAddresses function.

- `Use of transferFrom instead of transfer:` In the reLP function, the contract is using transferFrom to move tokens from the AMO to itself. If the AMO is a trusted contract, it could call transfer directly, saving the gas cost of the approve call. This issue is found in the reLP function.

- `Redundant Balance Checks:` In the reLP function, the contract is checking the balance of tokenA in itself after the Uniswap operations. This could be optimized by storing the balance before the operations and then subtracting the used amount. This issue is found in the reLP function.

- `Use of block.timestamp:` The reLP function uses block.timestamp for setting the deadline of the Uniswap transactions. This could be optimized by using a fixed offset from block.number, which is cheaper to access. This issue is found in the reLP function.

- `Use of Structs:` The contract uses a struct to store tokenA information in the reLP function. This could be optimized by using local variables, as structs are more expensive in terms of gas. This issue is found in the reLP function.

# Other recommendations
- Regular code reviews and adherence to best practices.
Conduct external audits by security experts.
- Consider open sourcing the contract for community review.
Maintain comprehensive security documentation.
- Establish a responsible disclosure policy for vulnerabilities.
- Implement continuous monitoring for unusual activity.
Educate users about risks and best practices.


### Time spent:
10 hours