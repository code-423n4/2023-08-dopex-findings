|     |     |
| --- | --- |
| \[G-01\] | Use hardcode address instead `address(this)` |
| \[G-02\] | `+=` Costs More Gas Than `=+` For State Variables |
| \[G-03\] | Events are not indexed |
| \[G-04\] | Make 3 event parameters indexed when possible |
| \[G‑05\] | Structs can be packed into fewer storage slots in the following way |
| \[G-06\] | \[G-06\] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead |
| \[G-07\] | Do not calculate constants |
| \[G‑08\] | Using `calldata` instead of `memory` for read-only arguments in external functions saves gas |
| \[G-09\] | Make fewer external calls. |
| \[G-10\] | Use `assembly` to check for `address(0)` |
| \[G-11\] | Expressions for constant values such as a call to `keccak256()`, should use immutable rather than constant |
| \[G-12\] | Use `ERC721A` instead `ERC721` |
| \[G‑13\] | Avoid contract existence checks by using low level calls |
| \[G-14\] | Redundant zero initialization |
| \[G-15\] | `++i` costs less gas compared to `i++` or `i += 1` |
| \[G-16\] | Return values from external calls can be cached to avoid unnecessary call |
| \[G-17\] | Use assembly for math (add, sub, mul, div) |

## \[G-01\] Use hardcode address instead `address(this)`

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded `address`. Foundry’s script.sol and solmate’s `LibRlp.sol` contracts can help achieve this.

References: <ins>https://book.getfoundry.sh/reference/forge-std/compute-create-address</ins>

<ins>https://twitter.com/transmissions11/status/1518507047943245824</ins>

```
function _sendTokensToRdpxV2Core() internal {
    uint256 tokenABalance = IERC20WithBurn(addresses.tokenA).balanceOf(
      address(this)
    );
    uint256 tokenBBalance = IERC20WithBurn(addresses.tokenB).balanceOf(
      address(this)
    );
```

### https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L160C2-L166C7

```
IERC20WithBurn(addresses.tokenA).safeTransferFrom(
      addresses.rdpxV2Core,
      address(this),
      tokenAAmount
    );
    IERC20WithBurn(addresses.tokenB).safeTransferFrom(
      addresses.rdpxV2Core,
      address(this),
      tokenBAmount
    );
```

### https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L210C1-L219C7

```
address(this),
```

### https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L230

```
address(this),
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L323

```
address(this),
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L341

```
function sync() external {
    lpTokenBalance = IERC20WithBurn(addresses.pair).balanceOf(address(this));
  }
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L362C3-L365C1

```
keccak256(abi.encodePacked(address(this), _tickLower, _tickUpper))
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L106

```
address(this),
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L189

```
address(this),
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L285

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L294

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L169

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L105

### Recommended Mitigation Steps

Use hardcoded `address`.

## \[G-02\] `+=` Costs More Gas Than `=+` For State Variables

### Mitigation

When you use the `+=` operator on a state variable, the EVM has to perform three operations:

- Load the current value of the state variable.
- Add the new value to it.
- Then store the result back in the state variable.

On the other hand, when you use the `=` operator and then add the values separately, the EVM only needs to perform two operations:

- Load the current value of the state variable.
- Add the new value to it.

Better use `=+` For State Variables.

```
235: lpTokenBalance += lpReceived;
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L235

```
283: lpTokenBalance -= lpAmount;
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L283

```
reserveAsset[reservesIndex["WETH"]].tokenBalance += amountOfWeth;
    reserveAsset[reservesIndex["RDPX"]].tokenBalance -= rdpxAmount;
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L779C5-L781C1

```
userTotalBondAmount += returnValues.bondAmountForUser;
      totalBondAmount += _amounts[i];
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L851C6-L852C38

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L916

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1067

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1196

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L302C5-L304C40

## <ins>\[G-03\] Events are not indexed</ins>

The emitted events are not indexed, making off-chain scripts such as front-ends of dApps difficult to filter the events efficiently.

Recommend adding the `indexed` keyword in each event,

```Solidity
event LogAddLiquidity(
    address indexed sender,
    uint256 tokenAAmount,
    uint256 tokenBAmount,
    uint256 tokenAAmountMin,
    uint256 tokenBAmountMin,
    uint256 tokenAUsed,
    uint256 tokenBUsed,
    uint256 lpReceived
  );

  event LogRemoveLiquidity(
    address indexed sender,
    uint256 lpAmount,
    uint256 tokenAAmountMin,
    uint256 tokenBAmountMin,
    uint256 tokenAReceived,
    uint256 tokenBReceived
  );

  event LogSwap(
    address indexed sender,
    uint256 token1Amount,
    uint256 token2AmountOutMin,
    bool swapTokenAForTokenB,
    uint256 token2Amount
  );

  event LogAssetsTransfered(
    address indexed sender,
    uint256 tokenAAmount,
    uint256 tokenBAmount
  );

  event LogEmergencyWithdraw(address sender, address[] tokens);
}
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L387C2-L422C2

```
event RecoveredERC20(address token, uint256 amount);
  event RecoveredERC721(address token, uint256 id);
  event LogAssetsTransfered(
    uint256 tokenAAmount,
    uint256 tokenBAmount,
    address tokenAAddress,
    address tokenBAddress
  );
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L368C1-L375C5

```
event log(uint);
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L379

## \[G-04\] Make 3 event parameters indexed when possible

It’s the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

[Reference](https://code4rena.com/reports/2023-01-drips#g-05-make-3-event-parameters-indexed-when-possible)

```
421: event LogEmergencyWithdraw(address sender, address[] tokens);
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L421

```
event RecoveredERC20(address token, uint256 amount);
  event RecoveredERC721(address token, uint256 id);
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L368C3-L369C52

```
event log(uint);
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L379

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L28C3-L41C5

## \[G‑05\] Structs can be packed into fewer storage slots in the following way

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

```
struct Position {
    uint256 token_id;
    uint128 liquidity; // the liquidity of the position
    int24 tickLower; // the tick range of the position
    int24 tickUpper;
    uint24 fee_tier;
    address collateral_address;
  }

  // Add liquidity param
  struct AddLiquidityParams {
    uint256 _amount0Desired;
    uint256 _amount1Desired;
    uint256 _amount0Min;
    uint256 _amount1Min;
    int24 _tickLower;
    int24 _tickUpper;
    uint24 _fee;
    address _tokenA;
    address _tokenB;
  }
```

## \[G-06\] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

The EVM operates with 32 byte words. Therefore, if you declare state variables less than 32 bytes the EVM will need to perform extra operations to cast your value to the specified size.

```
int24 tickLower; // the tick range of the position
    int24 tickUpper;
    uint24 fee_tier;
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L44C3-L46C21

```
int24 _tickLower;
    int24 _tickUpper;
    uint24 _fee;
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L53C3-L55C17

## \[G-07\] Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

```
uint256 public constant RDPX_RATIO_PERCENTAGE = 25 * DEFAULT_PRECISION;

  /// @notice ETH Ratio required when bonding
  uint256 public constant ETH_RATIO_PERCENTAGE = 75 * DEFAULT_PRECISION;
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L88C1-L91C73

## \[G‑08\] Using `calldata` instead of `memory` for read-only arguments in external functions saves gas

When a function with a `memory` array is called externally, the `abi.decode()` step has to use a for-loop to copy each index of the `calldata` to the `memory` index. Each iteration of this for-loop costs at least 60 gas (i.e. `60 * <mem_array>.length`). Using `calldata` directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having `memory` arguments, it’s still valid for implementation contracs to use `calldata` arguments instead.

If the array is passed to an `internal` function which passes the array to another internal function where the array is modified and therefore `memory` is used in the `external` call, it’s still more gass-efficient to use `calldata` when the `external` function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

```
function addAssetTotokenReserves(
    address _asset,
    string memory _assetSymbol
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L240C3-L243C44

```
function settle(
    uint256[] memory optionIds
  )
    external
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L315C2-L318C13

## \[G-09\] Make fewer external calls.

Every call to an external contract costs a decent amount of gas. For optimization of gas usage, It’s better to call one function and have it return all the data you need rather than calling a separate function for every piece of data. This might go against the best coding practices for other languages, but solidity is special.

## \[G-10\] Use `assembly` to check for `address(0)`

```
require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L244

```
_validate(_dopexAMMRouter != address(0), 17);
    _validate(_dpxEthCurvePool != address(0), 17);
    _validate(_rdpxDecayingBonds != address(0), 17);
    _validate(_perpetualAtlanticVault != address(0), 17);
    _validate(_perpetualAtlanticVaultLP != address(0), 17);
    _validate(_rdpxReserve != address(0), 17);
    _validate(_rdpxV2ReceiptToken != address(0), 17);
    _validate(_feeDistributor != address(0), 17);
    _validate(_reLPContract != address(0), 17);
    _validate(_receiptTokenBonds != address(0), 17);
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L316C4-L325C53

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L362C4-L363C53

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L379

### Mitigation

Using `assembly` to check for the zero address can result in significant gas savings compared to using a Solidity expression; especially if the check is performed frequently or in a loop. However, it’s important to note that using `assembly` can make the code less readable and harder to maintain, so it should be used judiciously and with caution.

## \[G-11\] Expressions for constant values such as a call to `keccak256()`, should use immutable rather than constant

```
bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
  bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/dpxETH/DpxEthToken.sol#L19C3-L21C66

```
bytes32 public constant MANAGER_ROLE = keccak256("MANAGER_ROLE");

  /// @dev Rdpx v2 core role which can purchase and settle options
  bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L45C1-L48C74

```
bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L22

## \[G-12\] Use `ERC721A` instead `ERC721`

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol

### Mitigation

`ERC721A` is an improvement standard for `ERC721` tokens. It was proposed by the Azuki team and used for developing their NFT collection. Compared with `ERC721`, `ERC721A` is a more gas-efficient standard to mint a lot of of NFTs simultaneously. It allows developers to mint multiple NFTs at the same gas price. This has been a great improvement due to Ethereum’s sky-rocketing gas fee. Reference: <ins>https://nextrope.com/erc721-vs-erc721a-2/</ins>.

## \[G‑13\] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

```
(bool success, ) = to.call{ value: amount, gas: gas }("");
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L98

## \[G-14\] Redundant zero initialization

Solidity does not recognize null as a value, so uint variables are initialized to zero. Setting a uint variable to zero is redundant and can waste gas.

There are several places where an int is initialized to zero, which looks like:

```
uint256 public latestFundingPaymentPointer = 0;
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L89

## \[G-15\] `++i` costs less gas compared to `i++` or `i += 1`

`++i` costs less gas compared to `i++` or `i += 1` for unsigned integer, as pre-increment is cheaper (about 5 gas per iteration). This statement is true even with the optimizer enabled.

`latestFundingPaymentPointer += 1;`

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L493

## \[G-16\] Return values from external calls can be cached to avoid unnecessary call

External calls are expensive as they use the `STATICCALL`/`CALL` opcode (~100 gas). If you are calling the same external function more than once you should cache the return value to avoid an unnecessary `STATICCALL`/`CALL`.