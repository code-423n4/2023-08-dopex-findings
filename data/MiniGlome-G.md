## Gas Optimizations
| |Issue|Instances|
|-|:-|:-:| 
| [GAS-01] | Remove unused logic | 3 | 
| [GAS-02] | Access control is checked twice | 5 | 
| [GAS-03] | Value used only in if statement should be computed only in the if statement | 2 | 

### [GAS-01] Remove unused logic
*Instances (3)*:
The `slippageTolerance` variable is not used in `UniV2LiquidityAMO.sol`. Thus, it can be removed alongside the `setSlippageTolerance()` function.
```solidity
File: amo/UniV2LiquidityAMO.sol

50:  /// @notice The slippage tolernce in swaps in 1e8 precision
  uint256 public slippageTolerance = 5e5; // 0.5%

109: function setSlippageTolerance( //@audit [GAS] slippageTolerance is never used
    uint256 _slippageTolerance
  ) external onlyRole(DEFAULT_ADMIN_ROLE) { //@audit can be frontrun
    require(
      _slippageTolerance > 0,
      "reLPContract: slippage tolerance must be greater than 0"
    );
    slippageTolerance = _slippageTolerance;
  }
```

```solidity
File: core/RdpxV2Core.sol

732:  uint256 bondAmountForUser = amount1; //@audit [GAS] useless variable, used only once
```


### [GAS-02] Access control is checked twice
*Instances (5)*:
The `mint()` function of `RdpxDecayingBonds.sol` checks that the sender has the `MINTER_ROLE` in the modifier and again in the function body.
```solidity
File: decaying-bonds/RdpxDecayingBonds.sol

114:  function mint(
    address to,
    uint256 expiry,
    uint256 rdpxAmount
  ) external onlyRole(MINTER_ROLE) {
    _whenNotPaused();
    require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter"); //@audit [GAS] role is already checked by modifier
```
The `DEFAULT_ADMIN_ROLE` is already cheched in the modifier, there is no need to check `_isEligibleSender()` because the `DEFAULT_ADMIN_ROLE` can set himself as an "eligible sender".
```solidity
File: core/RdpxV2Core.sol

1051:  function upperDepeg(
    uint256 _amount,
    uint256 minOut
  ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256 wethReceived) {
    _isEligibleSender(); //@audit [GAS] useless second access control check

1080:  function lowerDepeg(
    uint256 _rdpxAmount,
    uint256 _wethAmount,
    uint256 minamountOfWeth,
    uint256 minOut
  ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256 dpxEthReceived) {
    _isEligibleSender();
```
```solidity
File: perp-vault/PerpetualAtlanticVault.sol

215:  function settle(
    uint256[] memory optionIds
  )
    external
    nonReentrant
    onlyRole(RDPXV2CORE_ROLE)
    returns (uint256 ethAmount, uint256 rdpxAmount)
  {
    _whenNotPaused();
    _isEligibleSender(); //@audit [GAS] useless check because of onlyRole modifier

372: function payFunding() external onlyRole(RDPXV2CORE_ROLE) returns (uint256) {
    _whenNotPaused();
    _isEligibleSender();
```

### [GAS-03] Value used only in if statement should be computed only in the if statement
*Instances (2)*:
When a value is only used in a if statement, computing the vaue before entering the if statement can waste gas if the call does not enter the if statement.
The `mint()` function of `RdpxDecayingBonds.sol` checks that the sender has the `MINTER_ROLE` in the modifier and again in the function body.
```solidity
File: core/RdpxV2Core.sol

545:  uint256 minOut = _ethToDpxEth //@audit [GAS] this is used only if (minAmount > 0), so it should be calculated only in this case
      ? (((_amount * getDpxEthPrice()) / 1e8) -
        (((_amount * getDpxEthPrice()) * slippageTolerance) / 1e16))
      : (((_amount * getEthPrice()) / 1e8) -
        (((_amount * getEthPrice()) * slippageTolerance) / 1e16));

    // Swap the tokens
    amountOut = dpxEthCurvePool.exchange(
      _ethToDpxEth ? int128(int256(a)) : int128(int256(b)),
      _ethToDpxEth ? int128(int256(b)) : int128(int256(a)),
      _amount,
      minAmount > 0 ? minAmount : minOut
    );

1189: uint256 strike = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
      .roundUp(rdpxPrice - (rdpxPrice / 4)); // 25% below the current price //@audit [GAS] can only be calculated if putOptionsRequired

    uint256 timeToExpiry = IPerpetualAtlanticVault(
      addresses.perpetualAtlanticVault
    ).nextFundingPaymentTimestamp() - block.timestamp; //@audit [GAS] same
    if (putOptionsRequired) {
      wethRequired += IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
        .calculatePremium(strike, rdpxRequired, timeToExpiry, 0);
    }
```