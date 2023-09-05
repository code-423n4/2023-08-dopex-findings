### Low -1 Approve Racing condition for approve function
ERC20 approve has a know risk of approval [racing](https://swcregistry.io/docs/SWC-114/#:~:text=The%20race%20condition%20that%20happens,spend%20tokens%20on%20their%20behalf.), where a spender can over spend their allowance approved. This vulnerability is enabled by typical ERC20 approve function and there are (3) occurrences in the protocol found.
```solidity
//UniV2LiquidityAmo.sol
    function approveContractToSpend(
        address _token,
        address _spender,
        uint256 _amount
    ) external onlyRole(DEFAULT_ADMIN_ROLE) {
...

        IERC20WithBurn(_token).approve(_spender, _amount);
...}
```
(https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L134)
```solidity
//UniV3LiquidityAmo.sol
    function approveTarget(
        address _target,
        address _token,
        uint256 _amount,
        bool use_safe_approve
    ) public onlyRole(DEFAULT_ADMIN_ROLE) {
...
        IERC20WithBurn(_token).approve(_target, _amount);
...}
```
(https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L150)
```solidity
//RdpxV2Core.sol 
        function approveContractToSpend(
        address _token,
        address _spender,
        uint256 _amount
    ) external onlyRole(DEFAULT_ADMIN_ROLE) {
...
        IERC20WithBurn(_token).approve(_spender, _amount);
...}
```
(https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L411)
Recommendations: 
Whenever compatible with target token, use openzeppelin's `increaseAllowance` or `decreaseAllowance`. 
### Low -2 UniV2LiquidityAmo.sol accounting might be temporarily out of sync
In UniV2LiquidityAmo.sol, `sync()` is an external function that can be called by anyone to update the `lpTokenBalance`. And `lpTokenBalance` is modified in `addLiquidity()` and `removeLiquidity()`.

The issue is `addLiquidity()` and `removeLiquidity()` won't `sync()` the token balance before updating the token balance. In the case that `lpTokenBalance` is changed by either one of the function, but `sync()` is not called in time by users, and when the other function is modifying `lpTokenBalance` again the change in balance will be added or subtracted on an outdated number. if sync() is never called for a long time by users or anyone, these changes will continuously be added or substracted based on an out-of-sync `lpTokenBalance`.

```solidity
//UniV2LiquidityAmo.sol
    function addLiquidity(
        uint256 tokenAAmount,
        uint256 tokenBAmount,
        uint256 tokenAAmountMin,
        uint256 tokenBAmountMin
    )
        external
        onlyRole(DEFAULT_ADMIN_ROLE)
        returns (uint256 tokenAUsed, uint256 tokenBUsed, uint256 lpReceived)
    {
...
        // update LP token Balance
        lpTokenBalance += lpReceived;
...}

    function removeLiquidity(
        uint256 lpAmount,
        uint256 tokenAAmountMin,
        uint256 tokenBAmountMin
    )
        external
        onlyRole(DEFAULT_ADMIN_ROLE)
        returns (uint256 tokenAReceived, uint256 tokenBReceived)
    {
...
        lpTokenBalance -= lpAmount;
...}
```
(https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L235)
(https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L283)
```solidity
    function sync() external {
        lpTokenBalance = IERC20WithBurn(addresses.pair).balanceOf(
            address(this)
        );
    }
```

Recommendations:
Consider calling `sync()` inside `addLiquidity()` and `removeLiquidity` before updating `lpTokenBalance`.
### Low -3 UniV3LiquidityAmo.sol swap and addLiquidity deadlines are set unrealistically loose increasing the risk of stale transaction
(1) In UniV3LiquidityAmo.sol `swap()` calls UniswapV3 and is set with a deadline as `2105300114` - a hard-coded timestamp. Even if the timestamp is large enough, it has no effect on realistically preventing a stale transaction. 

(2) In UniV3LiquidityAmo.sol `addLiquidity()` calls UniswapV3 and is set with a deadline as `type(uint256).max`. This is too big for the set deadline to prevent a stale transaction.

```solidity
//UniV3LiquidityAmo.sol
  function swap(
    address _tokenA,
    address _tokenB,
    uint24 _fee_tier,
    uint256 _amountAtoB,
    uint256 _amountOutMinimum,
    uint160 _sqrtPriceLimitX96
  ) public onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256) {
...
    ISwapRouter.ExactInputSingleParams memory swap_params = ISwapRouter
      .ExactInputSingleParams(
        _tokenA,
        _tokenB,
        _fee_tier,
        address(this),
|>     2105300114, // Expiration: a long time from now
        _amountAtoB,
        _amountOutMinimum,
        _sqrtPriceLimitX96
      );
...
```
(https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L289-L300)
```solidity
  function addLiquidity(
    AddLiquidityParams memory params
  ) public onlyRole(DEFAULT_ADMIN_ROLE) {
...
    INonfungiblePositionManager.MintParams
      memory mintParams = INonfungiblePositionManager.MintParams(
        params._tokenA,
        params._tokenB,
        params._fee,
        params._tickLower,
        params._tickUpper,
        params._amount0Desired,
        params._amount1Desired,
        params._amount0Min,
        params._amount1Min,
        address(this),
|>      type(uint256).max
      );
...}
```
(https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L178-L191)

Recommendations:
Consider change the deadline to more realistic number based on `block.timestamp`.

### Low -4 In RdpxV2Core.sol, when removing an asset, `reservesIndex` mapping will not be updated correctly when the asset removed is the last member in `reserveAsset` array
In RdpxV2Core.sol `removeAssetFromtokenReserves()`, when the asset removed is the last member of `reserveAsset` array, `reservesIndex` mapping is not going to be updated correctly. The index of the asset being removed will not be reset to '0' in `reservesIndex` as intended. Instead the old index will still remain. Although in this case, `reserveAsset` array will still be updated. This leaves a 'ghost' index inside `reservesIndex` mapping.
```solidity
//RdpxV2Core.sol
  function removeAssetFromtokenReserves(
    string memory _assetSymbol
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    uint256 index = reservesIndex[_assetSymbol];
    _validate(index != 0, 18);

    // remove the asset from the mapping
    reservesIndex[_assetSymbol] = 0;

    // add new index for the last element
|>  reservesIndex[reserveTokens[reserveTokens.length - 1]] = index;

    // update the index of reserveAsset with the last element
    reserveAsset[index] = reserveAsset[reserveAsset.length - 1];

    // remove the last element
    reserveAsset.pop();
    reserveTokens.pop();

    emit LogAssetRemovedFromtokenReserves(_assetSymbol, index);
  }
```
(https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L280)
As seen above, when an asset is being removed, its index in `reservesIndex` mapping will be set to 0. However, if the asset being removed is the last asset, `reservesIndex` will immediately re-assign the asset to be removed to its original index. Now the asset's index will remain in the `reservesIndex` mapping.

Recommendations:
Consider before re-assigning the index, add a check to verify if the asset being removed is the last member, if so, skip re-assigning the index.

### Low -5 Unnecessary require statement
In RdpxDecayingBonds.sol `mint()`, the function already has an `onlyRole(MINTER_ROLE)` modifier restrict access to `MINTER_ROLE`. However, the function adds a `require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");` in the body to check the minter role a second time, which is unnecesary.

```solidity
//RdpxDecayingBonds.sol
  function mint(
    address to,
    uint256 expiry,
    uint256 rdpxAmount
  ) external onlyRole(MINTER_ROLE) {
    _whenNotPaused();
|>  require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
    uint256 bondId = _mintToken(to);
    bonds[bondId] = Bond(to, expiry, rdpxAmount);

    emit BondMinted(to, bondId, expiry, rdpxAmount);
  }
```
(https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L120)

### Low -5 RdpxDecayingBonds will start to mint where tokenId is 2
In RdpxDecayingBonds.sol `_mintToken()`, the first tokenId to be minted is 2. This is because `constructor()` already increments `tokenId` to 1 at the contract deployment. And `_mintToken()` will first increment tokenId before minting the token, which causes 2 to be the first id to be minted instead of 1.

```solidity
  constructor(
    string memory _name,
    string memory _symbol
  ) ERC721(_name, _symbol) {
    // Grant the minter role and admin role to deployer
    _setupRole(MINTER_ROLE, msg.sender);
    _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
|>  _tokenIdCounter.increment();
  }
```
```solidity
  function _mintToken(address to) private returns (uint256 tokenId) {
    tokenId = _tokenIdCounter.current();
|>  _tokenIdCounter.increment();
    _mint(to, tokenId);
  }
```
(https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L131)

Recommendations:
Consider shift `increment()` after `_mint(to,tokenId)`.

### NC-1 Approve contract to spend does not verify if spender is a contract
In UniV2LiquidityAmo.sol `approveContractToSpend()`, there is no verification is spender is a contract as intended by this function.

```solidity
  function approveContractToSpend(
    address _token,
    address _spender,
    uint256 _amount
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(_token != address(0), "reLPContract: token cannot be 0");
    require(_spender != address(0), "reLPContract: spender cannot be 0");
    require(_amount > 0, "reLPContract: amount must be greater than 0");
    IERC20WithBurn(_token).approve(_spender, _amount);
  }
```
Recommendations:
Consider using openzeppelin's address library `isContract` to verify if the spender is contract.

### NC-2 Consider adding a check to ensure `_assetSymbol` match the asset added
In RdpxV2Core.sol `addAssetTotokenReserves()` , when adding an asset, the asset symbol and the asset address are input separately by the admin. In the function body, there is no check to ensure that `_assetSymbol` matches `_asset` address. In the case where the admin made a mistake, wrong `_assetSymbol` will be added.

```solidity
//RdpxV2Core.sol
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
(https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L240)
Recommendations:
Consider adding a check in the function body to make sure asset symbol matches the address.

### NC -3 Unnecessary operations performed in `calculateBondCost()` when `putOptionsRequired` is false
In RdpxV2Core.sol `calculateBondCost()`, `timeToExpiry` is only needed when `putOptionRequired` is true.
However, `timeToExpiry` is calculated and assigned even when `putOptionRequired` is false. 
```solidity
  function calculateBondCost(
    uint256 _amount,
    uint256 _rdpxBondId
  ) public view returns (uint256 rdpxRequired, uint256 wethRequired) {
...
|>  uint256 timeToExpiry = IPerpetualAtlanticVault(
      addresses.perpetualAtlanticVault
    ).nextFundingPaymentTimestamp() - block.timestamp;
    if (putOptionsRequired) {
      wethRequired += IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
        .calculatePremium(strike, rdpxRequired, timeToExpiry, 0);
    }
}
```
Recommendations:
Consider wrap the calculation of `timeToExpirty` inside `if (putOptionsRequired)` statement.

### NC -4 wrong comment in `_transfer` in RdpxV2Core.sol
In RdpxV2Core.sol `_transfer()` when `_bondID` is '0'. The contract will transfer RDPX from the user in this function. However, the comment incorrectly states that ETH will also be transferred from the user in the function, which is a mistake and reduces code readability.
```solidity
    function _transfer(
        uint256 _rdpxAmount,
        uint256 _wethAmount,
        uint256 _bondAmount,
        uint256 _bondId
    ) internal {
...
        } else {
|>          // Transfer rDPX and ETH token from user
            IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress)
                .safeTransferFrom(msg.sender, address(this), _rdpxAmount);
...
```
(https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L652)

Recommendations:
Correct the comment.



