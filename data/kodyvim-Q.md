# counters.sol is deprecated
counters used in [RdpxDecayingBonds](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L13) and [perpetualAtlanticVault](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L39) would be removed from recent [openzepplin release](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/4233)

# Missing call to `sync` in UniV2LiquidityAmo#_sendTokensToRdpxV2Core
balance of reserves should be updated following any changes to the contract balances
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L160

# usage of abi.encodePacked to compare string
hash collision could be possible due to how bytes are packed when using encodePacked.
```solidity
function getReserveTokenInfo(
    string memory _token
  ) public view returns (address, uint256, string memory) {
    ReserveAsset memory asset = reserveAsset[reservesIndex[_token]];

    _validate(
      keccak256(abi.encodePacked(asset.tokenSymbol)) ==
        keccak256(abi.encodePacked(_token)),
      19
    );//@audit

    return (asset.tokenAddress, asset.tokenBalance, asset.tokenSymbol);
  }
```
## Recommendation
use `abi.encode` instead of `abi.encodePacked`

# Hardcoded deadline does not work
When interacting with AMM deadline happens to terminate a transaction which has been pending on the mempool which might have been executed on an unfavorable condition or timing.
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L295
```solidity
ISwapRouter.ExactInputSingleParams memory swap_params = ISwapRouter
      .ExactInputSingleParams(
        _tokenA,
        _tokenB,
        _fee_tier,
        address(this),
        2105300114, // Expiration: a long time from now@audit invalid deadline
        _amountAtoB,
        _amountOutMinimum,
        _sqrtPriceLimitX96
      );
```
## Recommendation
Allow the Admin to pass a deadline deemed reasonable to `swap`.

# Redundant checks should be removed
Within rdpxV2 there are some redundant checks which should be refactored.
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L120
```solidity
function mint(
    address to,
    uint256 expiry,
    uint256 rdpxAmount
  ) external onlyRole(MINTER_ROLE) {<@
    _whenNotPaused();
    require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");//@audit this is a redundant check
  }
```
other Instances:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1055
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1086

# leftover tokenB from swaps could be swept to rdpxV2Core when reLPing.
When reLping part of tokenB is swapped for tokenA and the other part readded as Liquidity, within does swaps tokenB could have leftover from the swap with could accumulate and left stuck within the `reLpContract`. These leftovers could be transferred to `rdpxV2Core` as well.
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L290

# setBondDiscount should be constrained
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L441 
```solidity
function setBondDiscount(
    uint256 _bondDiscountFactor
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_bondDiscountFactor > 0, 3);
@>  bondDiscountFactor = _bondDiscountFactor;

    emit LogSetBondDiscountFactor(_bondDiscountFactor);
  }
```
`bondDiscountFactor` could be set to a value that could DoS bonding.
```solidity
function calculateBondCost(
    uint256 _amount,
    uint256 _rdpxBondId
  ) public view returns (uint256 rdpxRequired, uint256 wethRequired) {
    ...SNIP
    if (_rdpxBondId == 0) {
@>    uint256 bondDiscount = (bondDiscountFactor *
        Math.sqrt(IRdpxReserve(addresses.rdpxReserve).rdpxReserve()) *
        1e2) / (Math.sqrt(1e18)); // 1e8 precision

@>     _validate(bondDiscount < 100e8, 14); 
	..SNIP
}
