#### NC-1 Use named return values consistently 

**Description** => There are contracts that have functions that make use of named return values e.g returns (uint256 premium) in the majority of cases for functions but some functions do not use this without any logical reasoning, policy, guide or documentation as to why some use named return variables and others do not. 

It is important for code to be consistent. Other contracts do not use named return variables at all whereas others do and within same contract some functions use named returns others do not.

Examples below:

[Majority functions in UniV2LiquidityAmo.sol use named return variables ](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol)
However, below functions do not:
[function getLpPrice() public view returns (uint256) { ..line 381 ](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L381C3-L381C56)
[function getLpTokenBalanceInWeth() external view returns (uint256) { ..line 372 ](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L372) 

[UniV3LiquidityAmo.sol does not use named return variables at all](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol)

[Majority functions in RdpxV2Core.sol use named return variables ](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol)
However, below functions do not:
[....) external returns (uint256) { { ..RdpxV2Core.sol line 944 ](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L944)
[public view returns (address, uint256, string memory) { ..RdpxV2Core.sol line 1137 ](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1137C5-L1137C60) 
[..function getEthPrice() public view returns (uint256 ethPrice) {...RdpxV2Core.sol line 1227 ](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1227C3-L1227C66)
[..function getRdpxPrice() public view returns (uint256) { {...RdpxV2Core.sol line 1238 ](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1238)
[function getDelegatesLength() external view returns (uint256) ...RdpxV2Core.sol line 1248 ](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1248C3-L1248C66)

[See Below ..RdpxV2Bond.sol functions one makes use of named return variables another does not](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol)
[..public onlyRole(MINTER_ROLE) returns (uint256 tokenId)  line 39](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L39C3-L39C60)
[...returns (bool) ...line 63](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L63)

[See below function ...RdpxDecayingBonds.sol that uses named return variable whereas rest of functions don't](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol)
[ function _mintToken(address to) private returns (uint256 tokenId) line 129](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L129C2-L129C68)


[PerpetualAtlanticVault.sol functions also mix functions with named return variables and others without with many instances](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol)

[PerpetualAtlanticVaultLP.sol also mixes functions with named return variables and others without]
You may think its to do with e.g public functions don't use named returns but see examples below 
```solidity 
  // ================================ PUBLIC VIEW FUNCTIONS ================================ //

  /// @notice Returns the amount of collateral and rdpx per share
  function redeemPreview(
    uint256 shares
  ) public view returns (uint256, uint256) {
    return _convertToAssets(shares);
  }

  /// @notice Returns the amount of shares recieved for a given amount of assets
  function previewDeposit(uint256 assets) public view returns (uint256) {
    return convertToShares(assets);
  }

  /// @notice Returns the amount of shares recieved for a given amount of assets
  function convertToShares(
    uint256 assets
  ) public view returns (uint256 shares) {
    uint256 supply = totalSupply;
    uint256 rdpxPriceInAlphaToken = perpetualAtlanticVault.getUnderlyingPrice();
```
convertToShares is under public functions but uses named return variables whereas others do not?

**Impact** => This can lead to problems with code readability, auditing, maintaning, or even errors if devs misinterprets or misuses these differences 

**Tools Used** => Manual Analysis 

**Recommendation** => It is recommended to be consistent with named return variables or set policies as to which types of functions will have named return variables and which ones will not. Fix all instances in examples above and all others not mentioned but relevant to this issue. 
