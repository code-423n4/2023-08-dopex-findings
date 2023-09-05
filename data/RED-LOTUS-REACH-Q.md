# Low Risk and Non-Critical Issues
# Some values cannot be amended without deploying the contract
Critical values may be set to nonsense values during deployment.  

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L67
```
address public weth;
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L124-L126
```
constructor(address _weth) {
    _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
    weth = _weth;
```
## Risk
Because the audit does not include the actual final deploy scripts, there is a greater than 0 chance that there are errors there.  For values that are deployed incorrectly, that would cause a need to redeploy those contracts, possibly incurring a not-insignificant financial cost.

## Mitigation
Add a permissioned setter for critical (yet underlooked) values 

# Old version of OpenZeppelin Contracts used
Using old versions of 3rd-party libraries in the build process can introduce vulnerabilities to the protocol code. 

- Latest is 4.9.3 (as of August 28, 2023), version used in project is 4.9.0
### Risks

1. Older versions of OpenZeppelin contracts might not have fixes for known security vulnerabilities.
2. Older versions might contain features or functionality that have been deprecated in recent versions.
3. Newer versions often come with new features, optimizations, and improvements that are not available in older versions.

## Recommendation

Review the latest version of the OpenZeppelin contracts and upgrade.

# Redundant checks  for roles and whitelist
Calls to `mint()` are already guarded against addresses lacking the `MINTER_ROLE`, but there is a redundant `require` for the same condition that can be removed

https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/decaying-bonds/RdpxDecayingBonds.sol#L120
```
require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
```

These checks to `_isEligibleSender` are redundant, especially to functions that are already limited to admin roles.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1080-L1086
```
  function lowerDepeg(
    uint256 _rdpxAmount,
    uint256 _wethAmount,
    uint256 minamountOfWeth,
    uint256 minOut
  ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256 dpxEthReceived) {
    _isEligibleSender();
```

### Risk
- The intent of the code is obfuscated, making further updates and audits challenging when the historical or immediate context behind a redundant check is unclear.
### Recommendation
Remove unnecessary checks to reduce gas costs.
# Magic numbers obfuscate intention
To improve readability and maintainability, a constant can be used instead of the magic number. Consider replacing the magic numbers used in the codebase with constants.
In the below snippet, comments are used, however those comments can become stale and further confuse the context of the magic numbers.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L946-L951
```
// fee less than 100%
_validate(_fee < 100e8, 8);
// amount greater than 0.01 WETH
_validate(_amount > 1e16, 4);
// fee greater than 1%
_validate(_fee >= 1e8, 8);
```

### Risk
- The intent of the code is obfuscated, making further updates and audits challenging when the historical or immediate context behind those values is unclear.

### Recommendation
Save the magic numbers in a clearly-named constant according to the [Solidity Style Guide](https://solidity.readthedocs.io/en/latest/style-guide.html)

# Possible "out of gas" error when retrieving all bonds owned by a user
The getBondsOwned function iterates over all the bonds owned by a user. If a user owns a large number of bonds, this could exceed the block gas limit, causing the function to fail.

https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/decaying-bonds/RdpxDecayingBonds.sol#L156
```
for (uint256 i; i < ownerTokenCount; i++) {
  tokenIds[i] = tokenOfOwnerByIndex(_address, i);
}
```

### Risk
Unforeseen use-cases for the protocol may create perverse incentives for more entries to be used than was intended, creating cases where common operations return with Out of Gas errors thus bricking that functionality for specific users.

### Recommendation
Ensure there is an upper bounds to loops to prevent Out of Gas errors. 


# `approveContractToSpend` has no check that approved is a contract
The name of the function does not reflect the actual operation because any kind of address can be used, whether it is a contract or an EOA.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L397-L412
```
/**
 * @notice Approve a contract to spend a certain amount of tokens
 * @dev    Can only be called by admin
 * @param  _token the address of the token to approve
 * @param  _spender the address of the contract to approve
 * @param  _amount the amount to approve
 */
function approveContractToSpend(
  address _token,
  address _spender,
  uint256 _amount
) external onlyRole(DEFAULT_ADMIN_ROLE) {
  _validate(_token != address(0), 17);
  _validate(_spender != address(0), 17);
  _validate(_amount > 0, 17);

  IERC20WithBurn(_token).approve(_spender, _amount);
}
```

### Risk
- Future developers being onboarded to the protocol team may misunderstand the actual intention of the function, causing further vulnerabilities.

### Mitigation
Either change the naming of the function, or add a check that the target address is a contract.

# Zero address check is done wrong

In the constructor of PerpetualAtlanticVaultLP.sol there's a zero address check. It uses `||` to check but really it should be `&&`.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L94-L97
```
require(
  _perpetualAtlanticVault != address(0) || _rdpx != address(0),
  "ZERO_ADDRESS"
);
```

### Risk
- In the case where the deploy script is written with an uninitialized value for `_rdpx` but not `_perpetualAtlanticVault`, the deploy will continue. Because there is no function to update that value, the contract may need to be deployed for a not-inconsiderable cost.

### Mitigation
Change the conditional to use `&&`.

# Function casing is confusing for some calls

For some functions, casing follows a mixed case of mixed case but keeps the capitalization of specific variables. This breaks expectations from the [Solidity Style Guide](https://solidity.readthedocs.io/en/latest/style-guide.html), and breaks expectations when writing test code and reading for the purpose of auditing.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L240
```
function addAssetTotokenReserves(...)
```

### Risk
- Writing new code that  makes calls to functions as these could cause time lost in the development cycle.

### Recommendation
Keep a consistent naming style according to the [Solidity Style Guide](https://solidity.readthedocs.io/en/latest/style-guide.html).

# Inconsistent naming style

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L699-L702
```
  function _bondWithDelegate(
    uint256 _amount,
    uint256 rdpxBondId,
    uint256 delegateId
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L819-L823
```
  function bondWithDelegate(
    address _to,
    uint256[] memory _amounts,
    uint256[] memory _delegateIds,
    uint256 rdpxBondId
```

### Risk
- The inconsistent use of underscores makes it confusing whether the intent is to mark local variables or not. 

### Recommendation
Keep a consistent naming style according to the [Solidity Style Guide](https://solidity.readthedocs.io/en/latest/style-guide.html).

# Visibility too loose on various public functions
A handful of functions are declared public that are never called internally. It is good practice to mark such functions as  external, as this saves gas, especially in the case where the function  takes arguments, since external functions can read arguments directly from call data instead of having to allocate memory.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L819-L824
```
  function bondWithDelegate(
    address _to,
    uint256[] memory _amounts,
    uint256[] memory _delegateIds,
    uint256 rdpxBondId
  ) public returns (uint256 receiptTokenAmount, uint256[] memory) {
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L139-L142
```
  function decreaseAmount(
    uint256 bondId,
    uint256 amount
  ) public onlyRole(RDPXV2CORE_ROLE) {
```

### Risk
- Extra gas spent by the users of the protocol will be lose

### Recommendation  
These functions can be marked `external` for gas optimization.

# Usage of Comments and NatSpec
Comments and NatSpec are meant to be helpful and describe the intent of the code block. However, in multiple parts of the codebase, comments are not consistent with the code that is written or do not add any value to the reader.
### Risks

1. Lack of understanding of code without adequate documentation.
2. Difficulty in understanding, upgrading, and fixing code without documentation.
### Recommendation

1. Practice consistent use of NatSpec on all parts of the project codebase.
2. Include the need for NatSpec comments for code reviews to pass.

## Issues
Below are various issues related to comments and NatSpec.
### Comments are out of sync with code

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1047C10-L1047C96
```
@notice Lets users mint DpxEth at a 1:1 ratio when DpxEth pegs above 1.01 of the ETH token
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L397
```
@notice Approve a contract to spend a certain amount of tokens
```

### Comments generated from template
There are instances of comments being generated with template strings that serve no role in explaining to the user the context of whatever is being commented on:

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L85-L87
```
/// @notice Explain to an end user what this does
/// @dev Explain to a developer any extra details
/// @return Documents the return variables of a contract’s function state variable
```

### Vague terms in comments 
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1011
```
* @notice Withdraws DSC from a matured bond
```

### Redundant comments add no value
Comments are generally important to give context to the code being written.  Redundant comments detract from the readability of a codebase. 

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L900
```
// Validate amount
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L903
```
// Compute the bond cost
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L915
```
// update weth reserve
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L918
```
// purchase options
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L926
```
// Stake the ETH in the ReceiptToken contract
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L929
```
// reLP
```

### Inconsistent use of `@inheritdoc` 
NatSpec is very useful when it comes to providing an inline documentation of a codebase.  `@inheritdoc` are also very useful when use consistently in a codebase - for example: having the documentation on an interface and `@inheritdoc` in the implementation, and vice versa.  When used sporadically, it breaks up the ability to see useful context in a single place, and can be frustrating.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L371
```
/// @inheritdoc       IPerpetualAtlanticVault
```
