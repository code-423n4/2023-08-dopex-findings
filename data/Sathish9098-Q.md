# LOW FINDINGS


| LOW COUNT | Issues  | Instances  |
|---------- |---------|----------- |
| [L-1] | The ``DEFAULT_ADMIN_ROLE`` to withdraw tokens, regardless of whether the situation is genuinely an ``emergency`` or ``not ``  | 2 |
| [L-2] | Using a very short deadline ``(block.timestamp + 1)`` can lead to ``Reverts``  | 3  |
| [L-3] | Ensuring ``sufficient gas`` to prevent reverts in transactions involving ``UniswapV2Router`` and ``UniswapV2Pair ``  | 1 |
| [L-4] | ``DEFAULT_ADMIN_ROLE`` can be single point of failure   | 10  |
| [L-5] | Failure to check ``_whenNotPaused()`` in the ``redeem()``,``mint()`` function results in ``unintended behaviors`` and poses a ``potential risk``  | 2 |
| [L-6] | Revert on ``approval`` to ``Zero Address`` | 1  |
| [L-7] | Divide by zero should be avoided   | 3 |
| [L-8] | Signature use at deadlines should be allowed  | 2  |
| [L-9] | Vulnerable version of openzeppelin-Contracts used   | 1  |
| [L-10] | The absence of robust ``uint256`` value ``sanity checks`` for critical parameter assignments poses a technical risk   | 1  |
| [L-11] | Add ``blocklist`` function for ``NFT ``  | -  |
| [L-12] | Don't use deprecated ``Counters.sol`` library  | 1 |
| [L-13] | Unchecked Return Value in ``swapExactTokensForTokens()`` Function May Lead to ``Loss of Funds``  | 2 |
| [L-14] | Contracts Fail to check ``Token Blacklist`` before executing token transfers    | -  |
| [L-15] | Add setter function to ``rdpxV2Core`` variable   | 1 |
| [L-16] | Lack of symbol existence check for ``_assetSymbol`` Allows for ``Duplicate Symbols ``  | 1  |
| [L-17] | Critical address changes should use ``two-step procedure``   | 2 |
| [L-18] | Unused ``EnumerableSet`` library   | 1 |



##

## [L-1] The ``DEFAULT_ADMIN_ROLE`` to withdraw tokens, regardless of whether the situation is ``genuinely`` an ``emergency`` or ``not ``

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


https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L161-L173

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

## [L-3] Ensuring ``sufficient gas`` to prevent reverts in transactions involving ``UniswapV2Router`` and ``UniswapV2Pair ``

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

## [L-4] ``DEFAULT_ADMIN_ROLE`` can be single point of failure 

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

##

## [L-5] Failure to check ``_whenNotPaused()`` in the ``redeem()``,mint() function results in ``unintended behaviors`` and poses a ``potential risk``

### Impact
The primary technical risk arises from the lack of a pause check. Without this check, the function allows for `redemptions`` even when the contract is ``intentionally paused``. This ``contradicts`` the typical design of ``pausing`` a contract to prevent actions during certain conditions, such as ``security vulnerabilities`` or ``emergencies``.

which can lead to ``financial`` and ``security implications`` that may not align with the contract's ``intended functionality`` and ``objectives``.

### POC

```solidity
FILE: Breadcrumbs2023-08-dopex/contracts/core/RdpxV2Core.sol

 function redeem(
    uint256 id,
    address to
  ) external returns (uint256 receiptTokenAmount) {
    // Validate bond ID
    _validate(bonds[id].timestamp > 0, 6);
    // Validate if bond has matured
    _validate(block.timestamp > bonds[id].maturity, 7);

    _validate(
      msg.sender == RdpxV2Bond(addresses.receiptTokenBonds).ownerOf(id),
      9
    );

    // Burn the bond token
    // Note requires an approval of the bond token to this contract
    RdpxV2Bond(addresses.receiptTokenBonds).burn(id);

    // transfer receipt tokens to user
    receiptTokenAmount = bonds[id].amount;
    IERC20WithBurn(addresses.rdpxV2ReceiptToken).safeTransfer(
      to,
      receiptTokenAmount
    );

    emit LogRedeem(to, receiptTokenAmount);
  }

```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1016-L1019

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L37-L39

### Recommended Mitigation
Add ``_whenNotPaused()`` check to redeem() function

```solidity

_whenNotPaused() ;

```

##

## [L-6] Revert on ``approval`` to ``Zero Address``

### Impact

Some tokens (e.g. OpenZeppelin) will revert if trying to approve the zero address to spend tokens (i.e. a call to approve(address(0), amt)).

Integrators may need to add special cases to handle this logic if working with such a token.

### POC

```solidity
FILE: 2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol

150: IERC20WithBurn(_token).approve(_target, _amount);

```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L150

### Recommended Mitigation
Add address(0) before approve tokens 

```solidity

Require(_target != address(0), "Address can't be address(0)");

```
##
## [L-7] Divide by zero should be avoided 

### Impact
The divisions below take an input parameter which does not have any zero-value checks, which may lead to the functions reverting when zero is passed.

### POC

if the ``rdpxRequiredInWeth`` ``_wethRequired`` value can be a 0 . So before ``division operation`` the non zero must be checked 

```solidity
FILE: 2023-08-dopex/contracts/core/RdpxV2Core.sol

uint256 extraRdpxToWithdraw = (discountReceivedInWeth * 1e8) /
        getRdpxPrice();

1172: DEFAULT_PRECISION) /
        (DEFAULT_PRECISION * rdpxPrice * 1e2);

1182:  (DEFAULT_PRECISION * rdpxPrice * 1e2);

```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L608-L609

```solidity
FILE: 2023-08-dopex/contracts/reLP/ReLPContract.sol

232: uint256 tokenAToRemove = ((((_amount * 4) * 1e18) /
      tokenAInfo.tokenAReserve) *

239:  uint256 lpToRemove = (tokenAToRemove * totalLpSupply) /
      tokenAInfo.tokenALpReserve;

```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L239

### Recommended Mitigation
Add non zero check before division 

```solidity

require(getRdpxPrice() > 0 , " Not zero" );

```

##

## [L-8] Signature use at deadlines should be allowed

According to [EIP-2612](https://github.com/ethereum/EIPs/blob/71dc97318013bf2ac572ab63fab530ac9ef419ca/EIPS/eip-2612.md?plain=1#L58), signatures used on exactly the deadline timestamp are supposed to be allowed. While the signature may or may not be used for the exact EIP-2612 use case (transfer approvals), for consistency's sake, all deadlines should follow this semantic. If the timestamp is an expiration rather than a deadline, consider whether it makes more sense to include the expiration timestamp as a valid timestamp, as is done for deadlines.

### POC
```solidity
FILE: 2023-08-dopex/contracts/core/RdpxV2Core.sol

636: _validate(expiry >= block.timestamp, 2);

1023:  _validate(block.timestamp > bonds[id].maturity, 7);

```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L636

##

## [L-9] Vulnerable version of openzeppelin-Contracts used 

### Impact

Known issues:

  - Improper Encoding or Escaping of Output
  - Improper Input Validation
  - Missing Authorization

### POC

```
// OpenZeppelin Contracts (last updated v4.9.0) (token/ERC20/utils/SafeERC20.sol)

```
### Recommended Mitigation
Use  latest ``OpenZeppelin version V4.9.3`` to avoid unexpected errors

##

## [L-10] The absence of robust ``uint256`` value ``sanity checks`` for critical parameter assignments poses a technical risk 

### Impact 
Incorrect parameter ``assignments`` can lead to ``data corruption`` or ``inconsistency`` within the protocol, affecting its ``functionality`` and ``integrity``.

### POC
```solidity
FILE: Breadcrumbs2023-08-dopex/contracts/perp-vault/PerpetualAtlanticVault.sol

124:  genesis = _gensis;

```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L124

### Recommended Mitigation
Add validity checks

 - > 0
 - > MIN
 - < MAX

##

## [L-11] Add ``blocklist`` function for ``NFT ``

As stated in the project:

```
Check all that apply (e.g. timelock, ``NFT``, AMM, ERC20, rollups, etc.): Timelock function, NFT, AMM, ERC-20 Token
```
NFT thefts have increased recently, so with the addition of hacked NFTs to the platform, NFTs can be converted into liquidity. To prevent this, I recommend adding the blacklist function.

Marketplaces such as Opensea have a blacklist feature that will not list NFTs that have been reported theft, NFT projects such as Manifold have blacklist functions in their smart contracts.

Here is the project example; Manifold

```solidity

Manifold Contract https://etherscan.io/address/0xe4e4003afe3765aca8149a82fc064c0b125b9e5a#code

     modifier nonBlacklistRequired(address extension) {
         require(!_blacklistedExtensions.contains(extension), "Extension blacklisted");
         _;
     }

```
### Recommended Mitigation Steps
Add to ``Blacklist`` function and ``modifier``.

##

## [L-12] Don't use deprecated ``Counters.sol`` library

### Impact

[Remove Discussions](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/4233). This library is not recommended by OpenZeppelin.

Problems:
There are a few issues though. This ``guarantee`` was always a soft guarantee, because the inner struct members are directly modifiable bypassing the struct interface. Additionally, a Counter being a ``uint256`` value occupies an entire storage slot whereas in most occasions packing a counter in storage with other values will be a far larger optimization than removing overflow checks; the significance of this has also changed since the introduction of Counters due to changes in the EVM gas prices.

### POC
```solidity
FILE: Breadcrumbs2023-08-dopex/contracts/core/RdpxV2Bond.sol

29: Counters.Counter private _tokenIdCounter;

```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L20

### Recommended Mitigation 

```solidity

uint256 private _tokenIdCounter;

 _safeMint(_receiver, _tokenIdCounter++)

```

##

## [L-13] Unchecked Return Value in ``swapExactTokensForTokens()`` Function May Lead to ``Loss of Funds``

### Impact
The code you have provided also does not check if the ``swap`` works as expected. This means that the ``caller`` cannot be sure that they will receive the ``amount`` of ``token B` that they ``requested``. It is important to check the amount of ``token B`` received to make sure that it is at least equal to the amount that was requested.

### POC

```solidity
FILE: 2023-08-dopex/contracts/amo/UniV2LiquidityAmo.sol

335: // swap token A for token B
    token2Amount = IUniswapV2Router(addresses.ammRouter)
      .swapExactTokensForTokens(
        token1Amount,
        token2AmountOutMin,
        path,
        address(this),
        block.timestamp + 1
      )[path.length - 1];

```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L335-L343

### Recommended Mitigation
Add checks to avoid unexpected amount 

```
 // Check if the swap worked as expected.
  require(token2Amount  >= token2AmountOutMin, "Swap failed");

```

##

## [L-14] Contracts Fail to check ``Token Blacklist`` before executing token transfers   

### Impact
If a smart contract does not check the ``token blacklist`` before ``token transfers``, then it is possible for ``malicious`` or ``compromised token`` owners to trap funds in the ``contract`` by adding the ``contract addres``s to the ``blocklist``.

This is a serious ``security vulnerability`` that can have a significant impact on the users of the contract. For example, if a ``Uniswap LP`` is blocked, then the ``LP`` will be unable to withdraw their funds from the ``pool``. This could potentially result in significant ``financial losses`` for the ``LP``

### Recommended Mitigation
Implement ``isBlacklisted()`` function . . If the address is ``blacklisted``, then the transfer should be ``reverted``

```Solidity

 mapping(address => bool) private _isBlacklisted;

function isBlacklisted(address token) public view returns (bool) {
    return _isBlacklisted[token];
  }

```

##

## [L-15] Add setter function to ``rdpxV2Core`` variable 

### Impact

``rdpxV2Core`` is critical ``address`` to dealing funds.

If the address stored in the ``rdpxV2Core`` variable (presumably a critical component or contract within the protocol) were to become ``compromised`` or needed to be changed for ``any reason``, not having a ``setter function`` to update it could pose a ``significant risk`` to the ``overall protocol``. In such a situation, the ``inability`` to update this ``address`` could result in problems for the ``entire protocol``.

### POC
```solidity
FILE: 2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol

78: rdpxV2Core = _rdpxV2Core;

```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L78

### Recommended Mitigation
Add setter function

```solidity

function setRdpxV2Core(address _newRdpxV2Core) public {
        require(msg.sender == owner, "Only the owner can update rdpxV2Core");
        rdpxV2Core = _newRdpxV2Core;
    }

```
##

## [L-16] Lack of symbol existence check for ``_assetSymbol`` Allows for ``Duplicate Symbols ``

### Impact
The ``addAssetTotokenReserves`` function does not check if the ``asset symbol`` already exists. This means that it is possible to add the ``same asset symbol`` to the token reserves ``multiple times``.

### POC
```solidity
FILE: Breadcrumbs2023-08-dopex/contracts/core/RdpxV2Core.sol

function addAssetTotokenReserves(
    address _asset,
    string memory _assetSymbol
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");

    for (uint256 i = 1; i < reserveAsset.length; i++) {
      require(
        reserveAsset[i].tokenAddress != _asset,
        "RdpxV2Core: asset already exists"
      );
    }

    ReserveAsset memory asset = ReserveAsset({
      tokenAddress: _asset,
      tokenBalance: 0,
      tokenSymbol: _assetSymbol
    });
    reserveAsset.push(asset);
    reserveTokens.push(_assetSymbol);

    reservesIndex[_assetSymbol] = reserveAsset.length - 1;

    emit LogAssetAddedTotokenReserves(_asset, _assetSymbol);
  }

```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L240-L264

### Recommended Mitigation

```solidity

require(
  !reserveTokens.includes(_assetSymbol),
  "RdpxV2Core: asset symbol already exists"
);

```

##

## [L-17] Critical changes should use two-step procedure

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two-step procedure on the critical functions.

Consider adding a two-steps pattern on critical changes to avoid mistakenly transferring ownership of roles or critical functionalities to the wrong address.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L358-L370

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L378-L381

## [L-18] Unused ``EnumerableSet`` library 

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L26








