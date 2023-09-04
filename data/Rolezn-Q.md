## QA Summary<a name="QA Summary">
Low Severity Risks

### Low Risk Issues
| |Issue|Contexts|
|-|:-|:-:|
| [LOW&#x2011;1](#LOW&#x2011;1) | Consider checking that `mint` to `address(this)` is disabled | 4 |
| [LOW&#x2011;2](#LOW&#x2011;2) | Consider implementing two-step procedure for updating protocol addresses | 6 |
| [LOW&#x2011;3](#LOW&#x2011;3) | Constant decimal values | 16 |
| [LOW&#x2011;4](#LOW&#x2011;4) | Large transfers may not work with some `ERC20` tokens | 34 |
| [LOW&#x2011;5](#LOW&#x2011;5) | Minting tokens to the zero address should be avoided | 1 |
| [LOW&#x2011;6](#LOW&#x2011;6) | OpenZeppelin's `Counters.sol` is deprecated | 3 |
| [LOW&#x2011;7](#LOW&#x2011;7) | Privileged roles can change critical parameter even though `contract` is in `paused` mode | 1 |
| [LOW&#x2011;8](#LOW&#x2011;8) | Solmate's SafeTransferLib doesn't check whether the ERC20 contract exists | 1 |

Total: 66 contexts over 8 issues

### Non-critical Issues
| |Issue|Contexts|
|-|:-|:-:|
| [NC&#x2011;1](#NC&#x2011;1) | Empty `bytes` check is missing | 4 |
| [NC&#x2011;2](#NC&#x2011;2) | Overly complicated arithmetic | 1 |

Total: 5 contexts over 2 issues

## Low Risk Issues

### <a href="#qa-summary">[LOW&#x2011;1]</a><a name="LOW&#x2011;1"> Consider checking that `mint` to `address(this)` is disabled

Functions allowing users to specify the receiving address should check whether the referenced address is not `address(this)`. This would prevent tokens to remain lock inside the contract.  An example of the potential for loss by leaving this open is the EOS token smart contract where more than 90,000 tokens are stuck at the contract address.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: RdpxV2Bond.sol

37: function mint(
    address to
  ) public onlyRole(MINTER_ROLE) returns (uint256 tokenId) {
    tokenId = _tokenIdCounter.current();
    _tokenIdCounter.increment();
    _mint(to, tokenId);
  }

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Bond.sol#L37

```solidity
File: RdpxDecayingBonds.sol

114: function mint(
    address to,
    uint256 expiry,
    uint256 rdpxAmount
  ) external onlyRole(MINTER_ROLE) {
    _whenNotPaused();
    require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
    uint256 bondId = _mintToken(to);
    bonds[bondId] = Bond(to, expiry, rdpxAmount);

    emit BondMinted(to, bondId, expiry, rdpxAmount);
  }

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L114

```solidity
File: RdpxDecayingBonds.sol

129: function _mintToken(address to) private returns (uint256 tokenId) {
    tokenId = _tokenIdCounter.current();
    _tokenIdCounter.increment();
    _mint(to, tokenId);
  }

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L129


```solidity
File: DpxEthToken.sol

47: function burnFrom(
    address account,
    uint256 amount
  ) public override onlyRole(BURNER_ROLE) {
    _spendAllowance(account, _msgSender(), amount);
    _burn(account, amount);
  }

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/dpxETH/DpxEthToken.sol#L47



</details>




### <a href="#qa-summary">[LOW&#x2011;3]</a><a name="LOW&#x2011;3"> Constant decimal values

The use of fixed decimal values such as `1e18` or `1e16` in Solidity contracts can lead to inaccuracies, bugs, and vulnerabilities, particularly when interacting with tokens having different decimal configurations. Not all ERC20 tokens follow the standard 18 decimal places, and assumptions about decimal places can lead to miscalculations.

Resolution: Always retrieve and use the `decimals()` function from the token contract itself when performing calculations involving token amounts. This ensures that your contract correctly handles tokens with any number of decimal places, mitigating the risk of numerical errors or under/overflows that could jeopardize contract integrity and user funds.

#### <ins>Proof Of Concept</ins>

<details>


```solidity
File: ReLPContract.sol

232: uint256 tokenAToRemove = ((((_amount * 4) * 1e18) /
254: ((tokenAToRemove * tokenAInfo.tokenAPrice) * liquiditySlippageTolerance) /
      1e16;
275: (((amountB / 2) * tokenAInfo.tokenAPrice * slippageTolerance) / 1e16);

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L232

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L254

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L275





</details>










### <a href="#qa-summary">[LOW&#x2011;8]</a><a name="LOW&#x2011;8"> `ERC20` tokens may not be compatible with significantly large approvals

Not all `IERC20` implementations are totally compliant, and some (e.g `UNI`, `COMP`) may fail if the valued passed is larger than `uint96`. [Source](https://github.com/d-xo/weird-erc20#revert-on-large-approvals--transfers)

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: UniV2LiquidityAmo.sol

134: IERC20WithBurn(_token).approve(_spender, _amount);

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L134

```solidity
File: UniV3LiquidityAmo.sol

150: IERC20WithBurn(_token).approve(_target, _amount);

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L150

```solidity
File: UniV3LiquidityAmo.sol

169: IERC20WithBurn(params._tokenA).approve(
173: IERC20WithBurn(params._tokenB).approve(

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L169

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L173


```solidity
File: RdpxV2Core.sol

411: IERC20WithBurn(_token).approve(_spender, _amount);

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L411



</details>



### <a href="#qa-summary">[LOW&#x2011;4]</a><a name="LOW&#x2011;4"> Large transfers may not work with some `ERC20` tokens

Some `IERC20` implementations (e.g `UNI`, `COMP`) may fail if the valued transferred is larger than `uint96`. [Source](https://github.com/d-xo/weird-erc20#revert-on-large-approvals--transfers)

#### <ins>Proof Of Concept</ins>

<details>


```solidity
File: UniV2LiquidityAmo.sol

168: IERC20WithBurn(addresses.tokenA).safeTransfer(
172: IERC20WithBurn(addresses.tokenB).safeTransfer(

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L168

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L172



```solidity
File: UniV2LiquidityAmo.sol

210: IERC20WithBurn(addresses.tokenA).safeTransferFrom(
215: IERC20WithBurn(addresses.tokenB).safeTransferFrom(

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L210

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L215



```solidity
File: UniV2LiquidityAmo.sol

321: IERC20WithBurn(token1).safeTransferFrom(

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L321

```solidity
File: UniV3LiquidityAmo.sol

158: IERC20WithBurn(params._tokenA).transferFrom(
163: IERC20WithBurn(params._tokenB).transferFrom(

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L158

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L163



```solidity
File: UniV3LiquidityAmo.sol

283: IERC20WithBurn(_tokenA).transferFrom(

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L283

```solidity
File: PerpetualAtlanticVault.sol

289: collateralToken.safeTransferFrom(
347: collateralToken.safeTransferFrom(
382: collateralToken.safeTransferFrom(

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L289

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L347

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L382



</details>



### <a href="#qa-summary">[LOW&#x2011;5]</a><a name="LOW&#x2011;5"> Minting tokens to the zero address should be avoided

The core function `mint` is used by users to mint an option position by providing token1 as collateral and borrowing the max amount of liquidity. `Address(0)` check is missing in both this function and the internal function `_mint`, which is triggered to mint the tokens to the to address. Consider applying a check in the function to ensure tokens aren't minted to the zero address.

#### <ins>Proof Of Concept</ins>


```solidity
File: RdpxDecayingBonds.sol


function mint(
    address to,
    uint256 expiry,
    uint256 rdpxAmount
  ) external onlyRole(MINTER_ROLE) {
    _whenNotPaused();
    require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
    uint256 bondId = _mintToken(to);
    bonds[bondId] = Bond(to, expiry, rdpxAmount);

    emit BondMinted(to, bondId, expiry, rdpxAmount);
  }

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L114






### <a href="#qa-summary">[LOW&#x2011;6]</a><a name="LOW&#x2011;6"> OpenZeppelin's `Counters.sol` is deprecated

See https://github.com/OpenZeppelin/openzeppelin-contracts/pull/4289 for details regarding deprecation of `Counters.sol`

#### <ins>Proof Of Concept</ins>

```solidity
File: RdpxV2Bond.sol

9: import { Counters } from "@openzeppelin/contracts/utils/Counters.sol";

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Bond.sol#L9

```solidity
File: RdpxDecayingBonds.sol

13: import { Counters } from "@openzeppelin/contracts/utils/Counters.sol";

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L13

```solidity
File: PerpetualAtlanticVault.sol

14: import { Counters } from "@openzeppelin/contracts/utils/Counters.sol";

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L14






### <a href="#qa-summary">[LOW&#x2011;7]</a><a name="LOW&#x2011;7"> Privileged roles can change critical parameter even though `contract` is in `paused` mode

There should be `whenNotPaused` modifier present

#### <ins>Proof Of Concept</ins>

```solidity
File: PerpetualAtlanticVault.sol

237: function updateFundingDuration(
    uint256 _fundingDuration
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    fundingDuration = _fundingDuration;
  }

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L237





### <a href="#qa-summary">[LOW&#x2011;8]</a><a name="LOW&#x2011;8"> Solmate's SafeTransferLib doesn't check whether the ERC20 contract exists

Solmate's SafeTransferLib, which is often used to interact with non-compliant/unsafe ERC20 tokens, does not check whether the ERC20 contract exists. The following code will not revert in case the token doesn't exist (yet).

This is stated in the Solmate library: https://github.com/transmissions11/solmate/blob/main/src/utils/SafeTransferLib.sol#L9

#### <ins>Proof Of Concept</ins>


```solidity
File: PerpetualAtlanticVaultLP.sol

12: import { SafeTransferLib } from "solmate/src/utils/SafeTransferLib.sol";

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L12




#### <ins>Recommended Mitigation Steps</ins>
Add a contract exist control in functions that use solmate's `SafeTransferLib`

```solidity
pragma solidity >=0.8.0;

function isContract(address _addr) private returns (bool isContract) {
    isContract = _addr.code.length > 0;
}
```


## Non Critical Issues

### <a href="#qa-summary">[NC&#x2011;1]</a><a name="NC&#x2011;1"> Empty `bytes` check is missing

To avoid mistakenly accepting empty `bytes` as a valid value, consider to check for empty `bytes`. This helps ensure that empty `bytes` are not treated as valid inputs, preventing potential issues.

#### <ins>Proof Of Concept</ins>


```solidity
File: UniV3LiquidityAmo.sol

339: function execute(
    address _to,
    uint256 _value,
    bytes calldata _data
  ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (bool, bytes memory) {
```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L339


```solidity
File: RdpxV2Bond.sol

57: function supportsInterface(
    bytes4 interfaceId
  )
    public
    view
    override(ERC721, ERC721Enumerable, AccessControl)
    returns (bool)
  {
```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Bond.sol#L57


```solidity
File: RdpxDecayingBonds.sol

174: function supportsInterface(
    bytes4 interfaceId
  )
    public
    view
    override(ERC721, ERC721Enumerable, AccessControl)
    returns (bool)
  {
```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L174


```solidity
File: PerpetualAtlanticVault.sol

645: function supportsInterface(
    bytes4 interfaceId
  )
    public
    view
    override(ERC721, ERC721Enumerable, AccessControl)
    returns (bool)
  {
```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L645




### <a href="#qa-summary">[NC&#x2011;2]</a><a name="NC&#x2011;2"> Overly complicated arithmetic

To maintain readability in code, particularly in Solidity which can involve complex mathematical operations, it is often recommended to limit the number of arithmetic operations to a maximum of 2-3 per line. Too many operations in a single line can make the code difficult to read and understand, increase the likelihood of mistakes, and complicate the process of debugging and reviewing the code. Consider splitting such operations over more than one line, take special care when dealing with division however. Try to limit the number of arithmetic operations to a maximum of 3 per line.

#### <ins>Proof Of Concept</ins>


```solidity
File: ReLPContract.sol

275: (((amountB / 2) * tokenAInfo.tokenAPrice * slippageTolerance) / 1e16);

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L275
