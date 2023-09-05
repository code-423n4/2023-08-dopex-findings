# Table of Contents
| Number | Issues Details                                                                                         | Count |
| :----- | :----------------------------------------------------------------------------------------------------- | :---- ||
| [L-1] | Implementing Maximum Fee Limit Instead of Allowing `100%` Fees | 1 |
| [N-1] | Approve `0` Before Modifying Allowance | 5 |
| [N-2] | Avoid Redefining Constants in Different Locations | 2 |
| [N-3] | Limit Usage of All-Capitalized Variable Names to `constant`/`immutable` Variables | 1 |
| [N-4] | Adopting `bytes.concat()` in Solidity 0.8.4 | 2 |


## [L-1]</a><a name="L-1"> Implementing Maximum Fee Limit Instead of Allowing `100%` Fees

To mitigate potential risks and maintain a balanced fee structure, it is advisable to disallow the setting of fees to `100%` altogether. Allowing fees to be set at such a high percentage poses a considerable risk and may have unintended consequences.

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: RdpxV2Core.sol
function setRdpxFeePercentage
```
</details>


## [N-1]</a><a name="N-1"> Approve `0` Before Modifying Allowance

<details>
<summary><i>There are 5 instances of this issue:</i></summary>

```solidity
File: UniV3LiquidityAmo.sol
IERC20WithBurn(_token).approve(_target, _amount);
```

```solidity
File: RdpxV2Core.sol
IERC20WithBurn(_token).approve(_spender, _amount);
```

</details>



## [N-2]</a><a name="N-2"> Avoid Redefining Constants in Different Locations

Consider defining constants in a single contract to prevent synchronization issues when only one instance is updated. A cost-efficient way to store constants in a single location involves creating an `internal constant` in a `library` as suggested <a href="https://medium.com/coinmonks/gas-cost-of-solidity-library-functions-dbe0cedd4678">here</a>. If the variable serves as a local cache of a value from another contract, contemplate making the cache variable internal or private. This would compel external users to interact with the contract holding the original value, thus preventing discrepancies among callers.


<details>
<summary><i>There are 2 instances of this issue:</i></summary>

```solidity
File: UniV2LiquidityAmo.sol
uint256 public constant DEFAULT_PRECISION = 1e8;
```

```solidity
File: ReLPContract.sol
uint256 public constant DEFAULT_PRECISION = 1e8;
```
</details>


## [N-4]</a><a name="N-4"> Adopting `bytes.concat()` in Solidity 0.8.4

With the introduction of Solidity version 0.8.4, it is recommended to utilize the `bytes.concat()` function instead of `abi.encodePacked(<bytes>, <bytes>)`. The `bytes.concat()` function provides a more explicit and concise approach for concatenating byte arrays. By adopting `bytes.concat()`, you can enhance code readability and ensure compatibility with the latest version of Solidity. It is advisable to update the code accordingly to take advantage of this new feature.

<details>
<summary><i>There are 2 instances of this issue:</i></summary>

```solidity
File: RdpxV2Core.sol
keccak256(abi.encodePacked(asset.tokenSymbol)) ==
        keccak256(abi.encodePacked(_token)),
```

```solidity
File: UniV3LiquidityAmo.sol
keccak256(abi.encodePacked(address(this), _tickLower, _tickUpper))
```
</details>