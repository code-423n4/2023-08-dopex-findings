## \[L01\] Dopex is UNLICENSED it's better to choose a proper LICENSE.

`// SPDX-License-Identifier: UNLICENSED`

> all contest

## \[L‑02\] abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()

Use abi.encode() instead which will pad items to 32 bytes, which will [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() [instead](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739).

```
(uint128 liquidity, , , , ) = get_pool.positions(
      keccak256(abi.encodePacked(address(this), _tickLower, _tickUpper))
    );
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L105C4-L107C7

# Non-Critical

## \[N-01\] Showing the actual values of numbers in NatSpec comments makes checking and reading code easier

```
48: uint256 public constant DEFAULT_PRECISION = 1e8; // 100_000_000
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L48

```
85:  uint256 public constant DEFAULT_PRECISION = 1e8;   // 100_000_000
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L85

## \[N-02\] Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

## \[S-03\] Generate perfect code headers every time

Description:
I recommend using header for Solidity code layout and readability

```
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

## \[N-04\] NatSpec comments should be increased in contracts

> all contest

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity Official documentation

## \[N-05\] Use return named variables or explicit returns consistently

Some functions are declaring returned named variables on the function header, while other functions are not defining returned named variables.

*The following function is using a return named variable in the function header:*

```
function execute(
    address _to,
    uint256 _value,
    bytes calldata _data
  ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (bool, bytes memory) {
    (bool success, bytes memory result) = _to.call{ value: _value }(_data);
    return (success, result);
  }
```

*The following function is using a return named variable in the function header:*

```
function _sendTokensToRdpxV2Core(address tokenA, address tokenB) internal {
    uint256 tokenABalance = IERC20WithBurn(tokenA).balanceOf(address(this));
    uint256 tokenBBalance = IERC20WithBurn(tokenB).balanceOf(address(this));
```

Consider adopting the same approach throughout the codebase to improve the explicitness and readability of the code.

[](https://docs.soliditylang.org/en/v0.8.15/natspec-format.html)

## \[N-06\] Function writing that does not comply with the Solidity Style Guide

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

## \[N‑07\] Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)

While the compiler knows to optimize away the exponentiation, it’s still better coding practice to use idioms that do not require compiler optimization, if they exist.

## \[N-08\] Showing the actual values of numbers in NatSpec comments makes checking and reading code easier

## \[N-09\] Use constants for numbers

In several locations in the code, numbers like 1e36, 100e18, 1e27 are used...

## \[N-10\] Assembly Codes Specific – Should Have Comments

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Assembly removes several important security features of
Solidity, which can make the code more insecure and more error-prone.

## \[N-11\] Use the delete keyword instead of assigning a value of 0

Using the ‘delete’ keyword instead of assigning a ‘0’ value is a detailed optimization that increases code readability and audit quality, and clearly indicates the intent.

Other hand, if use delete instead 0 value assign , it will be gas saved.

\[N-12\] Function writing that does not comply with the Solidity Style Guide

Context
All Contracts

Description
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
within a grouping, place the view and pure functions last

## \[S-01\] You can explain the operation of critical functions in NatSpec with an infographic.