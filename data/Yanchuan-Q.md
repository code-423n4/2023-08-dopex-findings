## 1.token_id on removeLiquidity() is always 0
removeLiquidity() is used to remove liquidity from Uniswap V3. pos.token_id is the nft token id of the liquidity to be removed. At the end of the function, pos.token_id is deleted from positions_mapping (line 263), so  positions_mapping[pos.token_id].token_id must be 0. line 269 should be changed to **emit log(pos.token_id)**
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L269
```
262    positions_array.pop();
263    delete positions_mapping[pos.token_id];
264
265    // send tokens to rdpxV2Core
266    _sendTokensToRdpxV2Core(tokenA, tokenB);
267
268    emit log(positions_array.length);
269    emit log(positions_mapping[pos.token_id].token_id);
```

## 2. repeated checks
RdpxDecayingBonds can only be minted when the contract isn't paused. This checking is added in _beforeTokenTransfer(), there is no need to perform another check within the mint function. (line 119)
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L119
```
114  function mint(
115    address to,
116    uint256 expiry,
117    uint256 rdpxAmount
118  ) external onlyRole(MINTER_ROLE) {
119    _whenNotPaused();
120    require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
121    uint256 bondId = _mintToken(to);
122    bonds[bondId] = Bond(to, expiry, rdpxAmount);
123
124    emit BondMinted(to, bondId, expiry, rdpxAmount);
125  }
```

```
  function _beforeTokenTransfer(
    address from,
    address to,
    uint256 tokenId,
    uint256 batchSize
  ) internal override(ERC721, ERC721Enumerable) {
    _whenNotPaused();
    super._beforeTokenTransfer(from, to, tokenId, batchSize);
  }
```

## 3. Logical "or" should be changed to "and"
The logical "or" should be changed to "and" because the logical "or" implies that as long as one address is not equal to address(0), the check will pass. Furthermore, since there are four parameters of address type, all four of these parameters need to be checked.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L94-L97
```
  constructor(
    address _perpetualAtlanticVault,
    address _rdpxRdpxV2Core,
    address _collateral,
    address _rdpx,
    string memory _collateralSymbol
  )
    ERC20(
      "PerpetualAtlanticVault LP Token",
      _collateralSymbol,
      ERC20(_collateral).decimals()
    )
  {
    require(
      _perpetualAtlanticVault != address(0) || _rdpx != address(0),
      "ZERO_ADDRESS"
    );
```

## 4. Incorrect error codes
In RdpxV2Core.sol, error codes are defined to represent the reasons for transaction failures. However, it seems that the error codes used in the following _validate() functions are not appropriate.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L410
should be changed to E4

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L533-L542
chould be changed to E4

```
// E1: "Insufficient bond amount",
// E2: "Bond has expired",
// E3: "Invalid parameters",
// E4: "Invalid amount",
// E5: "Not enough ETH available from the delegate",
// E6: "Invalid bond ID",
// E7: "Bond has not matured",
// E8: "Fee cannot be more than 100",
// E9: "msg.sender is not the owner",
// E10: "Price is not above the upper peg",
// E11: "Amount greater that max permissible amount",
// E12: "Price is not lower than first lower peg",
// E13: "Amount greater than max permissible amount"
// E14: "Invalid delegate Id"
// E15: "Invalid amount"
// E16: "Funding already paid for the epoch"
// E17: "Zero address"
// E18: "Asset not found"
// E19: "Token not found"
```
