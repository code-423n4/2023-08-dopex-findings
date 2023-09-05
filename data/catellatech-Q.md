# QA report for Dopex contest by Catellatech

## [01] In the contract RdpxDecayingBonds.mint the require is redundant
In the mint function, there is essentially a double requirement of the same thing. Consider removing it, as the modifier does what you wrote in the require statement. 

### PoC
We removed the require statement and run the tests and work as we expected.
```Solidity
    <!-- 1. We commented the require and run the test -->
    function mint(address to, uint256 expiry, uint256 rdpxAmount) external onlyRole(MINTER_ROLE) {
        _whenNotPaused();
        // require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
        uint256 bondId = _mintToken(to);
        bonds[bondId] = Bond(to, expiry, rdpxAmount);

        emit BondMinted(to, bondId, expiry, rdpxAmount);
    }
```
Expected output:
```terminal
[PASS] testMint() (gas: 208805)
Logs:
  the test is complete without the require in the function bacause its has the modifier

Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 1.87ms
Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

## [02] The RdpxV2Core.approveContractToSpend() function doesn't perform as described in its NatSpec and function name
In the `RdpxV2Core.approveContractToSpend()` function, the following is described:
```shell
@notice Approve a contract to spend a certain amount of tokens
```
This is not accurate, as the function does not perform a check to verify whether the `_spender` address is indeed a contract, with the assistance of libraries like [OpenZeppelin](https://docs.openzeppelin.com/contracts/3.x/api/utils#Address-isContract-address-) offers.

### this is the function
```solidity
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
    _validate(_spender != address(0), 17); // validate that the spender is a contract address
    _validate(_amount > 0, 17);
    IERC20WithBurn(_token).approve(_spender, _amount);
  }
```

### Proof of Concept
```solidity
     function testEOACanBeApproved() public {
        // Lets make the rdpxV2Core contract has balance
        weth.transfer(address(rdpxV2Core), 25 * 1e18);

        address alice = makeAddr("alice");
        
        console.log("Balance of rdpxV2Core before being approved to an EOA:", weth.balanceOf(address(rdpxV2Core)));
        rdpxV2Core.approveContractToSpend(address(weth), alice, 25 * 1e18);

        // now the address that the admin approved can steal the tokens approved by admin
        console.log("EOA make the tx");
        vm.startPrank(alice);
        weth.transferFrom(address(rdpxV2Core), alice, 25 * 1e18);
        console.log("rdpxV2Core balance after rugpull from the alice address :", weth.balanceOf(address(rdpxV2Core)));
        vm.stopPrank();
    }
```

the output is:

```terminal
Running 1 test for tests/rdpxV2-core/Unit.t.sol:Unit
[PASS] testEOACanBeApproved() (gas: 85071)
Logs:
  Balance of rdpxV2Core before being approved to an EOA: 25000000000000000000
  EOA make the tx
  rdpxV2Core balance after rugpull from the alice address : 0

Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 682.50ms
Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

### Recommended Mitigation Steps
Add another check to ensure that the `_spender` address is indeed a contract and aligns with the function name  `approveContractToSpend`, as well as with the documentation provided by the team: `Approve a contract to spend a certain amount of tokens`.


## [03] In the `RdpxDecayingBonds:emergencyWithdraw` function exist a wrong assumption
The contract does not allow receiving `native currency`, but the `RdpxDecayingBonds:emergencyWithdraw` function has a mechanism to be able to transfer native currency. Speaking with the sponsor, he let me know the following:

```shell
psytama — hoy a las 9:55
ya the function used is just a basic function we add so its not optimized but the contract is not supposed to handle funds so its only there incase someone sends it tokens by mistake.
```
But this is wrong because the contract does not allow receiving native currency through a `receive/fallback` function.

### the function looks like
```solidity
/**
     * @notice Transfers all funds to msg.sender
     * @dev    Can only be called by the owner
     * @param  tokens The list of erc20 tokens to withdraw
     * @param  transferNative Whether should transfer the native currency
     * @param  to The address to transfer the funds to
     * @param  amount The amount to transfer
     * @param  gas The gas to use for the transfer
     *
     */
    function emergencyWithdraw(
        address[] calldata tokens,
        bool transferNative,
        address payable to,
        uint256 amount,
        uint256 gas
    ) external onlyRole(DEFAULT_ADMIN_ROLE) {
        _whenPaused();
        if (transferNative) {
            (bool success,) = to.call{value: amount, gas: gas}("");
            require(success, "RdpxReserve: transfer failed");
        }

        IERC20WithBurn token;

        for (uint256 i = 0; i < tokens.length; i++) {
            token = IERC20WithBurn(tokens[i]);
            token.safeTransfer(msg.sender, token.balanceOf(address(this)));
        }
    }
```

### PoC to proof that the contract cannot receive "accidentaly"
```solidity
    function testCannotReceiveNativeCurrency() public {
        // The contract cannot receive native currency "accidentaly"
        (bool revertsAsExpected, ) = address(rdpxDecayingBonds).call{value: 1 ether}("");
        assertFalse(revertsAsExpected);
    } 
```
the output is:
```terminal
Running 1 test for tests/RdpxDecayingBondsTest.t.sol:RdpxDecayingBondsTest
[PASS] testLowLevelCallRevert() (gas: 11852)
Traces:
  [11852] RdpxDecayingBondsTest::testLowLevelCallRevert()
    ├─ [45] RdpxDecayingBonds::fallback{value: 1000000000000000000}()
    │   └─ ← "EvmError: Revert"
    └─ ← ()

Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 2.90ms
Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

## [04] Remove/resolve the comments like the following
Reviewing the contract tests, I was able to notice a comment in the unit tests for the vault contracts, specifically in the `PerpetualAtlanticVault.settle()` function's test. The comment in the test looks like this:

```solidity
/tests/perp-vault/Unit.t.sol

129: // bug: updateUnderlyingPrice does not update price, getUnderlyingPrice() will use amm price
130:  function testSettle() public 
```

According to the comments in the function, this is what the function does:

```soldity
   @notice   The function asset settles the options by transferring rdpx 
              from the rdpxV2Core and transfers eth to the rdpx rdpxV2Core 
              contract at the option strike price. Will also the burn the 
              option tokens from the user.
```

It is recommended to fix or remove this comment, in case the team knows whether the `updateUnderlyingPrice` function works or not.

## [05] Potential accidental inclusion of ERC20Burnable in xETHDpxEthToken contract
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L14

The xETHDpxEthToken contract inherits from `ERC20Burnable`, which defines public methods to allow burning of tokens.

Presumably, this was added accidentally as a mistake in order to implement the burn() function in xETHDpxEthToken. The internal function _burn is already provided in the base implementation of ERC20 (OpenZepplin library), and doesn’t need any extra implementation, which may increase the attack surface of the protocol.

The recommendation is to remove ERC20Burnable from the inheritance list of xETHDpxEthToken.

## [06] Multiple uint mappings can be combined into a single mapping of a uint to a struct, where appropriate
Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to [not having to recalculate the key’s keccak256 hash](https://gist.github.com/catellaTech/725eb34b6b12ebb39d44f1b80578f9e5) (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

### Contract that we recomment to apply the change
```Solidity
contracts/perp-vault/PerpetualAtlanticVault.sol

66: mapping(uint256 => uint256) public fundingPaymentsAccountedFor;
73: mapping(uint256 => uint256) public totalFundingForEpoch;
76: mapping(uint256 => uint256) public optionsPerStrike;
79: mapping(uint256 => uint256) public latestFundingPerStrike;
82: mapping(uint256 => uint256) public fundingRates;
```

### Recommended Mitigation Steps
We recommend applying your own best practices, as you did in the [RdpxDecayingBonds](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L40) contract.

## [07] In the PerpetualAtlanticVaultLP.constructor missing check of input validation
In the constructor, it is not checked that the address of  `_rdpxRdpxV2Core` is different from `address(0)`. It is recommended to follow your applied best practices and properly validate this important input.

The same recommendation can be applied on the `RdpxV2Core.constructor` with the `weth` address.

## [08] Inconsistent Use of NatSpec
NatSpec is a boon to all Solidity developers. Not only does it provide a structure for developers to document their code within the code itself, it encourages the practice of documenting code. When future developers read code documented with NatSpec, they are able to increase their capacity to understand, upgrade, and fix code. Without code documented with NatSpec, that capacity is hindered.

The DOPEX codebase does have a high level of documentation with NatSpec. However there are numerous instances of functions missing NatSpec.

### Possible Risks
1. Lack of understanding of code without adequate documentation.
2. Difficulty in understanding, upgrading, and fixing code without documentation.

### Recommended Mitigation Steps
- Practice consistent use of NatSpec on all parts of the project codebase.
- Include the need for NatSpec comments for code reviews to pass.


## [09] Inconsistent coding style
An inconsistent coding style can be found throughout the codebase. Some examples include:
Some of the **parameter names** in certain functions lack the underscores as seen in other cases. To enhance code `readability` and adhere to the Solidity style guide standards, it is recommended to incorporate underscores in the parameter names. **This practice contributes to maintaining a consistent and organized codebase.**

In some functions, underscores are used: 
```solidity
Contract: UniV2LiquidityAmo
 function approveContractToSpend(
    address _token,
    address _spender,
    uint256 _amount
  ) external onlyRole(DEFAULT_ADMIN_ROLE) 

```
while in others they are not present:

```solidity
Contract: UniV2LiquidityAmo
  function addLiquidity(
    uint256 tokenAAmount,
    uint256 tokenBAmount,
    uint256 tokenAAmountMin,
    uint256 tokenBAmountMin
  )
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 tokenAUsed, uint256 tokenBUsed, uint256 lpReceived)
```

To improve the overall consistency and readability of the codebase, consider adhering to a more consistent coding style by following a specific style guide.

## [10] Empty blocks should be removed or emit something
In Solidity, any function defined in a non-abstract contract must have a complete implementation that specifies what to do when that function is called.

```solidity
contracts/perp-vault/PerpetualAtlanticVaultLP.sol

231:function _beforeTokenTransfer(
        address from,
        address,
        uint256
    ) internal virtual {}
```

### Recommended Mitigation Steps
Emmit something in the function.

## [11] Old version of OpenZeppelin Contracts used
Using old versions of 3rd-party libraries in the build process can introduce vulnerabilities to the protocol code.

- Latest is 4.9.3 (as of July 28, 2023), version used in project is 4.9.0

### Risks
- Older versions of OpenZeppelin contracts might not have fixes for known security vulnerabilities.
- Older versions might contain features or functionality that have been deprecated in recent versions.
- Newer versions often come with new features, optimizations, and improvements that are not available in older versions.

### Recommendation
Review the latest version of the OpenZeppelin contracts and upgrade

## [12] In some important functions trough the scope should have better manage of events
In some functions where the `admin` can `set or update` certain sensitive information, only the new information is tracked in the event, omitting the previous information. It should be considered to add the input of the previous information in the event to maintain transparency.

### Function that we recomment to add the old information in the event
```solidity
contracts/core/RdpxV2Core.sol

180: function setRdpxBurnPercentage
193: function setRdpxFeePercentage
228: function setBondMaturity
441: function setBondDiscount
```

## [13] In the UniV2LiquidityAmo.approveContractToSpend() function should emit some event
Events provide a way for auditing and traceability, as they publicly record the actions and changes made in a smart contract. This can be useful to verify the authenticity of certain transactions or activities.
And in this case the function is very important:

### Function that we recomment to have event
```solidity
  function approveContractToSpend(
    address _token,
    address _spender,
    uint256 _amount
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(_token != address(0), "reLPContract: token cannot be 0");
    require(_spender != address(0), "reLPContract: spender cannot be 0");
    require(_amount > 0, "reLPContract: amount must be greater than 0");
    IERC20WithBurn(_token).approve(_spender, _amount);
  }
```

Other functions that should have events:

```solidity
contracts/reLP/ReLPContract.sol

171: function setLiquiditySlippageTolerance
186: function setSlippageTolerance
```

### Recommended Mitigation Steps
Add some event in the function to keep the transparency with the users.


## [14] Crucial information is missing on important functions in all contracts
In the constructor functions it is not specified in the documentation if the admin will be an EOA or a contract, for example in the UniV2LiquidityAmo and UniV3LiquidityAmo contract. Consider improving the docstrings to reflect the exact intended behaviour, and using Address.isContract function from OpenZeppelin’s library to detect if an address is effectively a contract.

## [15] In some contracts, we find inconsistencies regarding uint256
Some developers prefer to use uint256 because it is consistent with other uint data types, which also specify their size, and also because making the size of the data explicit reminds the developer and the reader how much data they've got to play with, which may help prevent or detect bugs. insecure and more error-prone.

### Recommended Mitigation Steps
Use `forge fmt` to format the contracts.

## [16] We suggest to use named parameters for mapping type
Consider using named parameters in mappings (e.g. mapping(address account => uint256 balance)) to improve readability. This feature is present since Solidity 0.8.18.
For example:

```Solidity 
mapping(address account => uint256 balance); 
```

## [17] We suggest using the OpenZeppelin SafeCast library
We have noticed that in the main contract [RdpxV2Core.__curveSwap()](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/core/RdpxV2Core.sol#L515-558) implements some type conversions. 
Explicited first converted `uint256 to int256` and then `int256 to int128`, use the safeCast and implement the following functions:

```solidity
----------------------- uint256 to int256 ----------------------------------

    /**
     * @dev Converts an unsigned uint256 into a signed int256.
     *
     * Requirements:
     *
     * - input must be less than or equal to maxInt256.
     */
    function toInt256(uint256 value) internal pure returns (int256) {
        // Note: Unsafe cast below is okay because `type(int256).max` is guaranteed to be positive
        if (value > uint256(type(int256).max)) {
            revert SafeCastOverflowedUintToInt(value);
        }
        return int256(value);
    }

----------------------- int256 to int128 ----------------------------------
    /**
     * @dev Returns the downcasted int128 from int256, reverting on
     * overflow (when the input is less than smallest int128 or
     * greater than largest int128).
     *
     * Counterpart to Solidity's `int128` operator.
     *
     * Requirements:
     *
     * - input must fit into 128 bits
     */
    function toInt128(int256 value) internal pure returns (int128 downcasted) {
        downcasted = int128(value);
        if (downcasted != value) {
            revert SafeCastOverflowedIntDowncast(128, value);
        }
    }
```

We recommend using the OpenZeppelin SafeCast library to make the project more robust and take advantage of the gas optimizations and best practices provided by OpenZeppelin.


## [18] Use a single file for all system-wide constants
In some contracts, there are many `constants` used in the system. It is recommended to store all constants in a single file (for example, Constants.sol) and use inheritance to access these values.

This helps improve readability and facilitates future maintenance. It also helps with any issues, as some of these hard-coded contracts are administrative contracts.

`Constants.sol` should be used and imported in contracts that require access to these values. This is just a suggestion.

## [19] Use of deprecated function
The team is working with OZ's Access Control, the `_setupRole` function is deprecated.
`OZ documentation`
```shell
    * NOTE: This function is deprecated in favor of {_grantRole}.
     */
    function _setupRole(bytes32 role, address account) internal virtual {
        _grantRole(role, account);
    }
```