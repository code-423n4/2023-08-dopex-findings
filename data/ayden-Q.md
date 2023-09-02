1.when set isreLP should ensure the value being set is different from the current value
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L206#L206
```solidity
  function setIsreLP(bool _isReLPActive) external onlyRole(DEFAULT_ADMIN_ROLE) {
+   require(_isReLPActive ！= isReLPActive,"same value"); 
    isReLPActive = _isReLPActive; //@audit avoid set same value.
    emit LogSetIsReLPActive(_isReLPActive);
  }
```

2.`MINTER_ROLE` has been checked twice and `to` address lack of zero address check and we also need to check the token balance if bigger than zero in this function.
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L114#L125
```solidity
  function mint(
    address to,
    uint256 expiry,
    uint256 rdpxAmount
  ) external onlyRole(MINTER_ROLE) {
    _whenNotPaused();
-   require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");//@audit already in onlyRole modifier
    uint256 bondId = _mintToken(to);
    bonds[bondId] = Bond(to, expiry, rdpxAmount);

    emit BondMinted(to, bondId, expiry, rdpxAmount);
  }
```

3.while setting Hearbeat ensure `_heartbeat` is bigger than zero
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/oracles/DpxEthOracle.sol#L67#L73
```solidity
  function setHeartbeat(
    uint256 _heartbeat
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
+   require(_heartbeat>0);
    heartbeat = _heartbeat;//@audit should ensure heartbeat >0 .

    emit SetHeartbeat(_heartbeat);
  }
```

4.while `getPriceUpdates` ensure `_endIndex` is bigger than `_startIndex`
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/oracles/DpxEthOracle.sol#L130#L144
```solidity
  function getPriceUpdates(
    uint256 _startIndex,
    uint256 _endIndex
  ) external view returns (PriceObj[] memory result) {
+   require(_endIndex>_startIndex);
    uint256 resultLength = (_endIndex - _startIndex) + 1;//@audit should ensure end>=start.
    result = new PriceObj[](resultLength);

    for (uint256 i; i < resultLength; ) {
      result[i] = priceUpdates[_startIndex + i];

      unchecked {
        ++i;
      }
    }
  }
```