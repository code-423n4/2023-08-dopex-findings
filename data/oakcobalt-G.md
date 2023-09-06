### Gas -1 Wasteful operations in `updateFunding()` in PerpetualAtlanticVault.sol
`updateFunding()` updates transfers funding into PerpetualAtlanticVaultLP.sol and is used in two ways: (1) called publicly by anyone at any time; (2) used as a hook in key functions related to depositing and purchasing options though `deposit()` in PerpetualAtlanticVaultLP.sol and `purchase()` in PerpetualAtlanticVault.sol. 

Depending on time and context where `updateFunding()` is called, there might not be any funding to be sent to PerpetualAtlanticVaultLP.sol at times. However, even when there is no funding to be sent, `updateFunding()` will still perform all state changes including sending '0' tokens, adding '0' to state variables, which is a waste of gas.

```solidity
//PerpetualAtlanticVault.sol
  function updateFunding() public {
    updateFundingPaymentPointer();
    uint256 currentFundingRate = fundingRates[latestFundingPaymentPointer];
    uint256 startTime = lastUpdateTime == 0
      ? (nextFundingPaymentTimestamp() - fundingDuration)
      : lastUpdateTime;
    lastUpdateTime = block.timestamp;

|>  collateralToken.safeTransfer(
      addresses.perpetualAtlanticVaultLP,
      (currentFundingRate * (block.timestamp - startTime)) / 1e18
    );

|>  IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addProceeds(
      (currentFundingRate * (block.timestamp - startTime)) / 1e18
    );

    emit FundingPaid(
      msg.sender,
      ((currentFundingRate * (block.timestamp - startTime)) / 1e18),
      latestFundingPaymentPointer
    );
  }
```
(https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L502-L524)
(a) Transfer '0' token
When `currentFundingRate` is '0', and this is possible when a new epoch (`latestFundingPaymentPointer`) starts and `fundingRates` is not updated per the new epoch. this means `collateralToken.safeTransfer()` will send 0 amount token to LP contract. This is wasteful. 

Or when block.timestamp is the same as `startTime` and this can happen when `updateFunding()` is called too soon, this will also result in 0 amount token being sent.

(b) Add '0'
When the above two scenarios happen, the function will also pass '0' to `IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addProceeds()`, and inside `addProceeds()` 0 will be added to state variable `_totalCollateral`. This is wasteful.
```solidity
    /// @inheritdoc	IPerpetualAtlanticVaultLP
    function addProceeds(uint256 proceeds) public onlyPerpVault {
        require(
            collateral.balanceOf(address(this)) >= _totalCollateral + proceeds,
            "Not enough collateral token was sent"
        );
        _totalCollateral += proceeds;
    }
```
Recommendations:
Add a condition if statement to bypass 0 value transfer or adding '0' to make sure only when there is non-zero amount of token to transfer will token and proceeds be updated.

### GAS-2 Wasteful math operations in `calculateFunding()` on PerpetualAtlanticVault.sol
In PerpetualAtlanticVault.sol `calculateFunding()`, `timeToExpiry` is calculated in a wasteful way.

```solidity
  function calculateFunding(
    uint256[] memory strikes
  ) external nonReentrant returns (uint256 fundingAmount) {
...
      uint256 timeToExpiry = nextFundingPaymentTimestamp() -
        (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration));
...
```
(https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L426-L427)
As seen above, since the `nextFundingPaymentTimestamp()` be design will always be `genesis + (latestFundingPaymentPointer * fundingDuration)`. At any given time, `timeToExpiry` will always be `fundingDuration`. There is no need for the math operation here.

```solidity
  function nextFundingPaymentTimestamp()
    public
    view
    returns (uint256 timestamp)
  {
    return genesis + (latestFundingPaymentPointer * fundingDuration);
  }
```
Recommendations:
Remove the math operations as stated above.

### Gas -3 Wasteful zero value change state modification in PerpetualAtlanticVault in `calculateFunding()`
In PerpetualAtlanticVault.sol `calculateFunding()`, there are scenarios where 0 is added to a state variable which is a waste of gas. `calculateFunding()` is a public function and can be called by anyone and at any time. 

As seen from below, when `amount` is '0'. Zero value operations and state change will still proceeds in `calculatePremium()` , updating state variables `fundingPaymentsAccountedFor` and `fundingPaymentsAccountedForPerStrike` and `totalFundingForEpoch`. 

And `amount` can be '0' when `fundingPaymentsAccountedForPerStrike` has already taken into account `optionsPerStrike[strike]` in earlier transactions that might have be submitted by other users or admin. 

```solidity
  function calculateFunding(
    uint256[] memory strikes
  ) external nonReentrant returns (uint256 fundingAmount) {
...
|>    uint256 amount = optionsPerStrike[strike] -
        fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
          strike
        ];

      uint256 timeToExpiry = nextFundingPaymentTimestamp() -
        (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration));

 |>   uint256 premium = calculatePremium(
        strike,
        amount,
        timeToExpiry,
        getUnderlyingPrice()
      );

      latestFundingPerStrike[strike] = latestFundingPaymentPointer;
      fundingAmount += premium;

      // Record number of options that funding payments were accounted for, for this epoch
 |>   fundingPaymentsAccountedFor[latestFundingPaymentPointer] += amount;

      // record the number of options funding has been accounted for the epoch and strike
 |>   fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
        strike
      ] += amount;

      // Record total funding for this epoch
      // This does not need to be done in purchase() since it's already accounted for using `addProceeds()`
|>    totalFundingForEpoch[latestFundingPaymentPointer] += premium;
...
```
(https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L405-L459)
Recommendations:
Add a '0' value bypass, and only proceed for state changes when `amount` is not zero.

### GAS -4 Wasteful 0 token transfers in UniV3LiquidityAmo.sol
In UniV3LiquidityAmo.sol `removeLiquidity()`, after decreasing liquidity from UniswapV3 and collect tokens  through `decreaseLiquidity()` and `collect()`, tokens collected (tokenA and tokenB) are directly sent from UniswapV3 to RdpxV2Core.sol. 

However, `removeLiquidity()` will call `_sendTokensToRdpxV2Core()` which under the hood send tokenA and tokenB balance to RdpxV2Core.sol. This is unnecessary operation because UniswapV3 will not send tokenA and tokenB to UniV3LqiduityAmo.sol during `removeLiquidity()`. And this is because `collect()` param. will pass RdpxV2Core.sol as the recipient in the first place. In this case, `_sendTokensToRdpxV2Core()` will transfer '0' amount tokens.

In addition, `_sendTokensToRdpxV2Core()` although intended as a token sweeping function, is already included in `swap()` and `addLiquidity()` where tokenA and tokenB were actually transferred to UniV3LiquidityAmo.sol contract. Since (A) `removeLiquidity()` is different from other functions where tokenA and tokenB are not transferred to the contract itself , and (B) dust tokens would have been already taken care of when tokenA and tokenB are transferred in other functions, and (C) there is a ERC20 token recovery function in place `recoverERC20()` for any donations, there is no need to transfer tokenA and tokenB balance in `removeLiquidity()` and it would be wasteful to do so. 

Instead, we only need to call `IRdpxV2Core(rdpxV2Core).sync();` to make sure that RdpxV2Core.sol will sync the tokens transferred directly from UniswapV3.
```solidity
function removeLiquidity(
    uint256 positionIndex,
    uint256 minAmount0,
    uint256 minAmount1
  ) public onlyRole(DEFAULT_ADMIN_ROLE) {
    Position memory pos = positions_array[positionIndex];
    INonfungiblePositionManager.CollectParams
      memory collect_params = INonfungiblePositionManager.CollectParams(
        pos.token_id,
 |>     rdpxV2Core,
        type(uint128).max,
        type(uint128).max
      );

    (
      ,
      ,
      address tokenA,
      address tokenB,
      ,
      ,
      ,
      uint128 liquidity,
      ,
      ,
      ,

    ) = univ3_positions.positions(pos.token_id);

    // remove liquidity
    INonfungiblePositionManager.DecreaseLiquidityParams
      memory decreaseLiquidityParams = INonfungiblePositionManager
        .DecreaseLiquidityParams(
          pos.token_id,
          liquidity,
          minAmount0,
          minAmount1,
          block.timestamp
        );

    univ3_positions.decreaseLiquidity(decreaseLiquidityParams);

    univ3_positions.collect(collect_params);

    univ3_positions.burn(pos.token_id);

    positions_array[positionIndex] = positions_array[
      positions_array.length - 1
    ];
    positions_array.pop();
    delete positions_mapping[pos.token_id];

    // send tokens to rdpxV2Core
|>  _sendTokensToRdpxV2Core(tokenA, tokenB);

    emit log(positions_array.length);
    emit log(positions_mapping[pos.token_id].token_id);
  }
```
```solidity
  function _sendTokensToRdpxV2Core(address tokenA, address tokenB) internal {
    uint256 tokenABalance = IERC20WithBurn(tokenA).balanceOf(address(this));
    uint256 tokenBBalance = IERC20WithBurn(tokenB).balanceOf(address(this));
    // transfer token A and B from this contract to the rdpxV2Core
|>  IERC20WithBurn(tokenA).safeTransfer(rdpxV2Core, tokenABalance);
|>  IERC20WithBurn(tokenB).safeTransfer(rdpxV2Core, tokenBBalance);

    // sync token balances
    IRdpxV2Core(rdpxV2Core).sync();

    emit LogAssetsTransfered(tokenABalance, tokenBBalance, tokenA, tokenB);
  }
```
Recommendations:
In `removeLiquidity()`, consider replacing `_sendTokensToRdpxV2Core()` with `IRdpxV2Core(rdpxV2Core).sync();`.
