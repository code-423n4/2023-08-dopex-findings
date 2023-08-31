1. https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1172
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1181
```solidity
 rdpxRequired =
        ((RDPX_RATIO_PERCENTAGE - (bondDiscount / 2)) *
          _amount *
          DEFAULT_PRECISION) /
        (DEFAULT_PRECISION * rdpxPrice * 1e2);
```
DEFAULT_PRECISION is 1e8 and multiplying it with the numerator and denominator in this context has no use to prevent precision lose and even gives higher chances of DOS due to multiplication overflow in numerator and it also uses more gas. My report is simply to say 30_000/2_000 is same and better written as 30/2, both gives 15 but the latter is more readable and efficient. A second instance of this can be found in L1181 of the same RdpxV2Core.sol contract.

2. No Zero check to validate value of rdpxAmount input parameter in the mint(...) function in 
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L117-L122
it is a role function but putting the check there keeps the function safer
```solidity
 function mint(
    address to,
    uint256 expiry,
    uint256 rdpxAmount
  ) external onlyRole(MINTER_ROLE) {
    _whenNotPaused();
    require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
 +++ require(rdpxAmount != 0 , "rdpxAmount input cannot be zero")
    uint256 bondId = _mintToken(to);
    bonds[bondId] = Bond(to, expiry, rdpxAmount);

    emit BondMinted(to, bondId, expiry, rdpxAmount);
  }
```