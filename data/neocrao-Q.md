# Low/QA by NeoCrao

## Add checks to `UniV2LiquidityAmo.addLiquidity()` to ensure min amount <= amount

The `addLiquidity` method takes in the min amount and the amount for the two tokens, and then the Uniswap V2 Router's `addLiquidity` function is called with these values.
But there are no checks to ensure that `min amount <= amount`.

Code Reference: https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L189

```
  function addLiquidity(
    uint256 tokenAAmount,
    uint256 tokenBAmount,
    uint256 tokenAAmountMin,
    uint256 tokenBAmountMin
  )
  ...
  ...
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
      ...
      ...
```

The IUniswapV2Router's `addLiquidity` requires that the `min amount <= desired amount`.
Reference: https://docs.uniswap.org/contracts/v2/reference/smart-contracts/router-02#addliquidity

Because the checks are missing, the `addLiquidity` function can be called with `min amount > amount` which may lead to unexpected reverts.

### Suggested Fix

Add checks like the following:

```
require(tokenAAmount >= tokenAAmountMin);
require(tokenBAmount >= tokenBAmountMin);
```

## Add a check in `UniV2LiquidityAmo.liquidityInPool()` to ensure that the pool exists

The result of `univ3_factory.getPool()` can be address 0 if the pool doesnt exist.
Reference: https://docs.uniswap.org/contracts/v3/reference/core/interfaces/IUniswapV3Factory#getpool

If the response is address 0, then it can lead to unexpected and unhandled reverts.

Code Reference: https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L100-L102

```
    IUniswapV3Pool get_pool = IUniswapV3Pool(
      univ3_factory.getPool(address(rdpx), _collateral_address, _fee)
    );
```

### Similar issues from the past
* https://code4rena.com/reports/2021-07-spartan#l-25-check-if-pool-exists-in-getpool-

### Suggested Fix

Add a check to ensure `getPool()` returns a non-zero address.

## Add checks to `UniV3LiquidityAmo.addLiquidity()` to ensure min amount does not exceed the desired amount

Code Reference: https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L155

```
  function addLiquidity(
    AddLiquidityParams memory params
  ) public onlyRole(DEFAULT_ADMIN_ROLE) {
```

The `addLiquidity()` methods takes in `AddLiquidityParams` as the input argument as `params`. The struct is defined as follows:

Code Reference: https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L50

```
  struct AddLiquidityParams {
    address _tokenA;
    address _tokenB;
    int24 _tickLower;
    int24 _tickUpper;
    uint24 _fee;
    uint256 _amount0Desired;
    uint256 _amount1Desired;
    uint256 _amount0Min;
    uint256 _amount1Min;
  }
```

There are not checks in the `addLiquidity()` function to ensure that `_amount0Desired >= _amount0Min` and `_amount1Desired >= _amount1Min`.
If the wrong arguments are passed to the `addLiquidity()` function, then unexpected reverts may happen, which might be hard to troubleshoot.

### Suggested Fix

Add the following checks to the function:

```
require(params._amount0Desired >= params._amount0Min);
require(params._amount1Desired >= params._amount1Min);
```

## Adding assets to token reserves can hit block gas limits

The `RdpxV2Core.addAssetTotokenReserves()` function loops over the `reserveAsset` to ensure that the asset being added does not exist.
As the size of `reserveAsset` array grows, it may hit block gas limits.

Code Reference: https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L246

```
  /**
   * @notice Adds a asset to the reserves tokens
   * @dev    Can only be called by admin
   **/
  function addAssetTotokenReserves(
    address _asset,
    string memory _assetSymbol
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");

@>  for (uint256 i = 1; i < reserveAsset.length; i++) {
@>    require(
@>      reserveAsset[i].tokenAddress != _asset,
@>      "RdpxV2Core: asset already exists"
@>    );
@>  }

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

### Suggested Fix

Maintain another mapping like the following to keep track of what assets are already part of the reserve.

```
mapping(address assetAddress => bool exists) public reserveAssetAddresses;
```

And replace the `for..loop` with:

```
require(!reserveAssetAddresses[_asset], "RdpxV2Core: asset already exists")
```

## Missing checks to ensure `rdpxAmount != 0`

In the `RdpxDecayingBonds.mint()` function, there are no checks to ensure that `rdpxAmount != 0`

Code Reference: https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L114

```
  /// @notice Mints decaying rdpx bonds
  /// @dev Can only be called by the minter
  /// @param to address of the user to mint the bonds for
  /// @param expiry timestamp of the bond expiry
  /// @param rdpxAmount amount of rdpx to bond
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

### Suggested Fix

Add a `require` statement like the following which checks that `rdpxAmount != 0`

```
require(rdpxAmount != 0, "The amount cannot be 0");
```

## Missing checks to ensure `expiry` is not in the past

In the `RdpxDecayingBonds.mint()` function, there are no checks to ensure that `expiry` is not in the past

Code Reference: https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L114

```
  /// @notice Mints decaying rdpx bonds
  /// @dev Can only be called by the minter
  /// @param to address of the user to mint the bonds for
  /// @param expiry timestamp of the bond expiry
  /// @param rdpxAmount amount of rdpx to bond
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

### Suggested Fix

Add a `require` statement like the following which checks that `expiry` is not in the past

```
require(expiry >= block.timestamp, "The expiration cannot be in the past");
```

## Documentation only has placeholder text

The documentation only has placeholder text, and no real documentation

Code Reference: https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L84-L88

```
  /// @dev the pointer to the lattest funding payment timestamp
  /// @notice Explain to an end user what this does
  /// @dev Explain to a developer any extra details
  /// @return Documents the return variables of a contract’s function state variable
  /// @inheritdoc	IPerpetualAtlanticVault
  uint256 public latestFundingPaymentPointer = 0;
```

## `safeApproval` not needed if the address updates are the same

The `PerpetualAtlanticVault.setAddresses()` method performs `safeApproval` on the new `perpetualAtlanticVaultLP` address. This is not needed if the previous and the new `perpetualAtlanticVaultLP` are the same.

Code Reference: https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L207

```
    collateralToken.safeApprove(
      addresses.perpetualAtlanticVaultLP,
      type(uint256).max
    );
```

## The `PerpetualAtlanticVaultLP.convertToShares()` method can be simplified to return early if `supply` is 0.

If the `supply` is 0, then the `PerpetualAtlanticVaultLP.convertToShares()` method just returns the `assets` that were provided as input.

Code Reference: https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L274

```
  function convertToShares(
    uint256 assets
  ) public view returns (uint256 shares) {
    uint256 supply = totalSupply;
    uint256 rdpxPriceInAlphaToken = perpetualAtlanticVault.getUnderlyingPrice();

    uint256 totalVaultCollateral = totalCollateral() +
      ((_rdpxCollateral * rdpxPriceInAlphaToken) / 1e8);
    return
      supply == 0 ? assets : assets.mulDivDown(supply, totalVaultCollateral);
  }
```

The method can be updated to be as:

```
  function convertToShares(
    uint256 assets
  ) public view returns (uint256 shares) {
    uint256 supply = totalSupply;

+   if (supply == 0) {
+       return assets;
+   }

    uint256 rdpxPriceInAlphaToken = perpetualAtlanticVault.getUnderlyingPrice();

    uint256 totalVaultCollateral = totalCollateral() +
      ((_rdpxCollateral * rdpxPriceInAlphaToken) / 1e8);
-   return
-     supply == 0 ? assets : assets.mulDivDown(supply, totalVaultCollateral);
+   return assets.mulDivDown(supply, totalVaultCollateral);
  }
```

## `RdpxV2Core` needs to add `reservesIndex[<TOKEN>]` checks throughout the contract

In the `RdpxV2Core` contract, its assumed that the following always exist in the `reserveAsset` array.

Code Reference: https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L52-L61

```
  /* Inital tokens in the reserve
     index0: ZERO address
     index1: weth
     index2: rdpx
     index3: dpxEth
     index4: crv
  */

  /// @notice Array containg the reserve assets
  ReserveAsset[] public reserveAsset;
```

Now, throughout the code, there are accesses like `reserveAsset[reservesIndex["WETH"]]`, `reserveAsset[reservesIndex["DPXETH"]]` etc, based on the above assumption.

However, these initial tokens can be removed from `reserveAsset`, or the contract might even get initialized incorrectly. If such cases, all such accesses would fail, and cause unexpected behaviors.

### Suggested Fix

Before performing any access like `reserveAsset[reservesIndex[<TOKEN>]]`, ensure that `reservesIndex[<TOKEN>]` is not `0`, and `reserveAsset[reservesIndex[<TOKEN>]]` is not `0`.

## Missing check in `PerpetualAtlanticVaultLP.redeem()` can lead to underflows

In the `PerpetualAtlanticVaultLP.redeem()` function, if the `msg.sender` is not the `owner`, then the allowance granted by the `owner` to the `msg.sender` is retrieved. If this `allowance` is not `uint256 max`, then the allowance is reduced by `shares`.
The underflow happens if the `allowance` is 0 or less than shares. This should be handled using a `require` statement, otherwise unexpected reverts will happen.

Code Reference: https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L153

```
  function redeem(
    uint256 shares,
    address receiver,
    address owner
  ) public returns (uint256 assets, uint256 rdpxAmount) {
    perpetualAtlanticVault.updateFunding();

    if (msg.sender != owner) {
      uint256 allowed = allowance[owner][msg.sender]; // Saves gas for limited approvals.

      if (allowed != type(uint256).max) {
@>      allowance[owner][msg.sender] = allowed - shares;
      }
    }
    ...
    ...
```

### Suggested Fix

Update the code above to include a `require` check as follows:

```
      ...
      ...
      if (allowed != type(uint256).max) {
+       require(allowed >= shares, "Not enough allowance");
        allowance[owner][msg.sender] = allowed - shares;
      }
      ...
      ...
```

## Add checks that `positionIndex` is valid in `UniV3LiquidityAmo.removeLiquidity()`

The `positionIndex` could be out of bound, which will revert the transaction.

Code Reference: https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L218

```
  function removeLiquidity(
    uint256 positionIndex,
    uint256 minAmount0,
    uint256 minAmount1
  ) public onlyRole(DEFAULT_ADMIN_ROLE) {
    Position memory pos = positions_array[positionIndex];
    INonfungiblePositionManager.CollectParams
      memory collect_params = INonfungiblePositionManager.CollectParams(
        pos.token_id,
        rdpxV2Core,
        type(uint128).max,
        type(uint128).max
      );
    ...
    ...
```

### Suggested Fix
Add a require to ensure `positionIndex` is in bounds:

```
require(positionIndex < positions_array.length);
```

## Cache results of `nextFundingPaymentTimestamp()` in `PerpetualAtlanticVault._updateFundingRate()`

In the `PerpetualAtlanticVault._updateFundingRate()` function, the `nextFundingPaymentTimestamp()` is called several times, and the return value can be cached to avoid calling it multiple times.

There are two such instances:

### Instance 1
Code Reference: https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L594

```
  function _updateFundingRate(uint256 amount) private {
    if (fundingRates[latestFundingPaymentPointer] == 0) {
      uint256 startTime;
      if (lastUpdateTime > nextFundingPaymentTimestamp() - fundingDuration) {
        startTime = lastUpdateTime;
      } else {
        startTime = nextFundingPaymentTimestamp() - fundingDuration;
      }
      uint256 endTime = nextFundingPaymentTimestamp();
      fundingRates[latestFundingPaymentPointer] =
        (amount * 1e18) /
        (endTime - startTime);
    } else {
      uint256 startTime = lastUpdateTime;
      uint256 endTime = nextFundingPaymentTimestamp();
      if (endTime == startTime) return;
      fundingRates[latestFundingPaymentPointer] =
        fundingRates[latestFundingPaymentPointer] +
        ((amount * 1e18) / (endTime - startTime));
    }
  }
```

### Instance 2

Code Reference: https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L462

```
  function updateFundingPaymentPointer() public {
    while (block.timestamp >= nextFundingPaymentTimestamp()) {
      if (lastUpdateTime < nextFundingPaymentTimestamp()) {
        uint256 currentFundingRate = fundingRates[latestFundingPaymentPointer];

        uint256 startTime = lastUpdateTime == 0
          ? (nextFundingPaymentTimestamp() - fundingDuration)
          : lastUpdateTime;

        lastUpdateTime = nextFundingPaymentTimestamp();

        collateralToken.safeTransfer(
          addresses.perpetualAtlanticVaultLP,
          (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
            1e18
        );

        IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP)
          .addProceeds(
            (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
              1e18
          );

        emit FundingPaid(
          msg.sender,
          ((currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
            1e18),
          latestFundingPaymentPointer
        );
      }

      latestFundingPaymentPointer += 1;
      emit FundingPaymentPointerUpdated(latestFundingPaymentPointer);
    }
  }
```

## `timeToExpiry` calculation can be simplified in `PerpetualAtlanticVault.calculateFunding()` function

Code Reference: https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L426

```
      uint256 timeToExpiry = nextFundingPaymentTimestamp() -
        (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration));
```

The `nextFundingPaymentTimestamp()` function is as follows:
```
  /// @inheritdoc	IPerpetualAtlanticVault
  function nextFundingPaymentTimestamp()
    public
    view
    returns (uint256 timestamp)
  {
    return genesis + (latestFundingPaymentPointer * fundingDuration);
  }
```

So, `nextFundingPaymentTimestamp()` will be: `genesis + (latestFundingPaymentPointer * fundingDuration)`

Now, lets try to substitute the above in the `timeToExpiry` calculation:

```
uint256 timeToExpiry = nextFundingPaymentTimestamp() - (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration));
or, uint256 timeToExpiry = genesis + (latestFundingPaymentPointer * fundingDuration) - (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration));
or, uint256 timeToExpiry = genesis + (latestFundingPaymentPointer * fundingDuration) - genesis - ((latestFundingPaymentPointer - 1) * fundingDuration);
or, uint256 timeToExpiry = (latestFundingPaymentPointer * fundingDuration) - ((latestFundingPaymentPointer - 1) * fundingDuration);
or, uint256 timeToExpiry = fundingDuration * ((latestFundingPaymentPointer) - ((latestFundingPaymentPointer - 1)));
or, uint256 timeToExpiry = fundingDuration * (latestFundingPaymentPointer - latestFundingPaymentPointer + 1);
or, uint256 timeToExpiry = fundingDuration * (1);
or, uint256 timeToExpiry = fundingDuration;
```

All the tests pass as well with the change.

## Consider adding a check to `RdpxDecayingBonds.decreaseAmount()` to ensure the bond exists

The `bondId` provided might not exist in the `bonds` mapping. In that case a new entry will be created with owner as 0 address, expiry as 0, and the rdpxAmount as the amount provided.

Code Reference: https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L139

```
  function decreaseAmount(
    uint256 bondId,
    uint256 amount
  ) public onlyRole(RDPXV2CORE_ROLE) {
    _whenNotPaused();
    bonds[bondId].rdpxAmount = amount;
  }
```

## Typo in function name `RdpxV2Core.addAssetTotokenReserves()`

The function name should be:
```
                   V
function addAssetToTokenReserves(
```

Code Reference: https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L240C3-L240C32

```
                    Typo
                     V
  function addAssetTotokenReserves(
```