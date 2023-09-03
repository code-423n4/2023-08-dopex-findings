# gas

 ### Note: last 6 types ([G-18],[G-19],[G-20],[G-21],[G-22],[G-23]) that the automated report missed.


# summary 
|      | issue | instance |
|------|-------|----------|
|[G-01]|Use calldata instead of memory for function arguments that do not get mutated|3|
|[G-02]|Avoid contract existence checks by using low level calls|93|
|[G-03]|Can Make The Variable Outside The Loop To Save Gas |7|
|[G-04]|Make 3 event parameters indexed when possible|10|
|[G-05]|Amounts should be checked for 0 before calling a transfer|13|
|[G-06]|Do not calculate constants|2|
|[G-07]|Use hardcode address instead address(this)|10|
|[G-08]|Use Assembly To Check For address(0)|19|
|[G-09]|Use constants instead of type(uintx).max|17|
|[G-10]|Use ERC721A instead ERC721|3|
|[G-11]|Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if-statement|1|
|[G-12]|Use assembly for math (add, sub, mul, div)|4|
|[G-13]|Use assembly to perform efficient back-to-back calls|6|
|[G-14]|Empty blocks should be removed to save gas|1|
|[G-15]|Avoid emitting storage values|10|
|[G-16]|Use Modifiers Instead of Functions To Save Gas|2|
|[G-17]|Gas savings can be achieved by changing the model for assigning value to the structure (260 gas)|8|
|[G-18]|Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate|1|
|[G-19]|Using storage instead of memory for structs/arrays saves gas|3|
|[G-20]|State variables should be cached in stack variables rather than re-reading them from storage|54|
|[G-21]|internal functions only called once can be inlined to save gas|4|
|[G-22]|Use assembly for small keccak256 hashes, in order to save gas|1|
|[G-23]|>= costs less gas than >|5|




## [G-01] Use calldata instead of memory for function arguments that do not get mutated
Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.

```solidity
File: core/RdpxV2Core.sol
765       uint256[] memory optionIds
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L765

```solidity
File: perp-vault/PerpetualAtlanticVault.sol
316   uint256[] memory optionIds

406   uint256[] memory strikes
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L316

## [G‑02] Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

```solidity
File: amo/UniV2LiquidityAmo.sol
134   IERC20WithBurn(_token).approve(_spender, _amount);

161   uint256 tokenABalance = IERC20WithBurn(addresses.tokenA).balanceOf(
      address(this)
    );

164   uint256 tokenBBalance = IERC20WithBurn(addresses.tokenB).balanceOf(
      address(this)
    );    

168  IERC20WithBurn(addresses.tokenA).safeTransfer(
      addresses.rdpxV2Core,
      tokenABalance
    );  

172 IERC20WithBurn(addresses.tokenB).safeTransfer(
      addresses.rdpxV2Core,
      tokenBBalance
    );

200 IERC20WithBurn(addresses.tokenA).safeApprove(
      addresses.ammRouter,
      tokenAAmount
    );

204  IERC20WithBurn(addresses.tokenB).safeApprove(
      addresses.ammRouter,
      tokenBAmount
    );

210 IERC20WithBurn(addresses.tokenA).safeTransferFrom(
      addresses.rdpxV2Core,
      address(this),
      tokenAAmount
    );

215 IERC20WithBurn(addresses.tokenB).safeTransferFrom(
      addresses.rdpxV2Core,
      address(this),
      tokenBAmount
    ); 

222   (tokenAUsed, tokenBUsed, lpReceived) = IUniswapV2Router(addresses.ammRouter)
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

268  IERC20WithBurn(addresses.pair).safeApprove(addresses.ammRouter, lpAmount);

271  (tokenAReceived, tokenBReceived) = IUniswapV2Router(addresses.ammRouter)
      .removeLiquidity(
        addresses.tokenA,
        addresses.tokenB,
        lpAmount,
        tokenAAmountMin,
        tokenBAmountMin,
        address(this),
        block.timestamp + 1
      );

321 IERC20WithBurn(token1).safeTransferFrom(
      addresses.rdpxV2Core,
      address(this),
      token1Amount
    );  

328 IERC20WithBurn(token1).safeApprove(addresses.ammRouter, token1Amount);

336 token2Amount = IUniswapV2Router(addresses.ammRouter)

363  lpTokenBalance = IERC20WithBurn(addresses.pair).balanceOf(address(this));

382  return IRdpxEthOracle(addresses.rdpxOracle).getLpPriceInEth();
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L134

```solidity
File: amo/UniV3LiquidityAmo.sol
150   IERC20WithBurn(_token).approve(_target, _amount);

158   IERC20WithBurn(params._tokenA).transferFrom(
      rdpxV2Core,
      address(this),
      params._amount0Desired
    );

163  IERC20WithBurn(params._tokenB).transferFrom(
      rdpxV2Core,
      address(this),
      params._amount1Desired
    );

169 IERC20WithBurn(params._tokenA).approve(
      address(univ3_positions),
      params._amount0Desired
    );

173 IERC20WithBurn(params._tokenB).approve(
      address(univ3_positions),
      params._amount1Desired
    );

283     IERC20WithBurn(_tokenA).transferFrom(
      rdpxV2Core,
      address(this),
      _amountAtoB
    );

330 INonfungiblePositionManager(tokenAddress).safeTransferFrom(
      address(this),
      rdpxV2Core,
      token_id
    );    

354     uint256 tokenABalance = IERC20WithBurn(tokenA).balanceOf(address(this));

355     uint256 tokenBBalance = IERC20WithBurn(tokenB).balanceOf(address(this));  

357     IERC20WithBurn(tokenA).safeTransfer(rdpxV2Core, tokenABalance);

358     IERC20WithBurn(tokenB).safeTransfer(rdpxV2Core, tokenBBalance);

361     IRdpxV2Core(rdpxV2Core).sync();
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L150


```solidity
File: core/RdpxV2Core.sol
339 IERC20WithBurn(weth).approve(
      addresses.perpetualAtlanticVault,
      type(uint256).max
    );

343  IERC20WithBurn(weth).approve(addresses.dopexAMMRouter, type(uint256).max);

344  IERC20WithBurn(weth).approve(addresses.dpxEthCurvePool, type(uint256).max);

345  IERC20WithBurn(weth).approve(
      addresses.rdpxV2ReceiptToken,
      type(uint256).max
    );    

411 IERC20WithBurn(_token).approve(_spender, _amount);

481 (premium, optionId) = IPerpetualAtlanticVault(
      addresses.perpetualAtlanticVault
    ).purchase(_amount, address(this));

499  bondId = RdpxV2Bond(addresses.receiptTokenBonds).mint(_to);

529  uint256 ethBalance = IStableSwap(addresses.dpxEthCurvePool).balances(a);

530  uint256 dpxEthBalance = IStableSwap(addresses.dpxEthCurvePool).balances(
        b
      );

572   IDpxEthToken(reserveAsset[reservesIndex["DPXETH"]].tokenAddress).mint(
      address(this),
      _amount / 2
    );    

578  receiptTokenAmount = IRdpxV2ReceiptToken(addresses.rdpxV2ReceiptToken)
      .deposit(_amount / 2);

631  (, uint256 expiry, uint256 amount) = IRdpxDecayingBonds(
        addresses.rdpxDecayingBonds
      ).bonds(_bondId);

638  IRdpxDecayingBonds(addresses.rdpxDecayingBonds).ownerOf(_bondId) ==
          msg.sender,
        9
      );

643    IRdpxDecayingBonds(addresses.rdpxDecayingBonds).decreaseAmount(
        _bondId,
        amount - _rdpxAmount
      );

648  IRdpxReserve(addresses.rdpxReserve).withdraw(_rdpxAmount);

653    IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress)
        .safeTransferFrom(msg.sender, address(this), _rdpxAmount);

657   IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress).burn(
        (_rdpxAmount * rdpxBurnPercentage) / 1e10
      );
677   IRdpxReserve(addresses.rdpxReserve).withdraw(
        _rdpxAmount + extraRdpxToWithdraw
      );

772 (amountOfWeth, rdpxAmount) = IPerpetualAtlanticVault(
      addresses.perpetualAtlanticVault
    ).settle(optionIds); 

796  uint256 pointer = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
      .latestFundingPaymentPointer();

800  fundingAmount = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
      .payFunding();


873  if (isReLPActive) IReLP(addresses.reLPContract).reLP(totalBondAmount);

909  IERC20WithBurn(weth).safeTransferFrom(
      msg.sender,
      address(this),
      wethRequired
    );

930   if (isReLPActive) IReLP(addresses.reLPContract).reLP(_amount);

953   IERC20WithBurn(weth).safeTransferFrom(msg.sender, address(this), _amount);

987  IERC20WithBurn(weth).safeTransfer(msg.sender, amountWithdrawn);

997   uint256 balance = IERC20WithBurn(reserveAsset[i].tokenAddress).balanceOf(
        address(this)
      );

1026  msg.sender == RdpxV2Bond(addresses.receiptTokenBonds).ownerOf(id),

1032  RdpxV2Bond(addresses.receiptTokenBonds).burn(id);

1036      IERC20WithBurn(addresses.rdpxV2ReceiptToken).safeTransfer(
      to,
      receiptTokenAmount
    );

1059      IDpxEthToken(reserveAsset[reservesIndex["DPXETH"]].tokenAddress).mint(
      address(this),
      _amount
    );

1097         amountOfWethOut = IUniswapV2Router(addresses.dopexAMMRouter)
        .swapExactTokensForTokens(
          _rdpxAmount,
          minamountOfWeth,
          path,
          address(this),
          block.timestamp + 10
        )[path.length - 1];

1119      IDpxEthToken(reserveAsset[reservesIndex["DPXETH"]].tokenAddress).burn(
      dpxEthReceived
    );

1164  Math.sqrt(IRdpxReserve(addresses.rdpxReserve).rdpxReserve()) *

1189  uint256 strike = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
      .roundUp(rdpxPrice - (rdpxPrice / 4)); // 25% below the current price

1192    uint256 timeToExpiry = IPerpetualAtlanticVault(
      addresses.perpetualAtlanticVault
    ).nextFundingPaymentTimestamp() - block.timestamp;

1196    wethRequired += IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
        .calculatePremium(strike, rdpxRequired, timeToExpiry, 0);

1208   IRdpxEthOracle(pricingOracleAddresses.rdpxPriceOracle).getLpPriceInEth();

1218     IDpxEthOracle(pricingOracleAddresses.dpxEthPriceOracle)
        .getDpxEthPriceInEth();

1229     IDpxEthOracle(pricingOracleAddresses.dpxEthPriceOracle)
        .getEthPriceInDpxEth();

1240     IRdpxEthOracle(pricingOracleAddresses.rdpxPriceOracle)
        .getRdpxPriceInEth();                        
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L339

```solidity
File: perp-vault/PerpetualAtlanticVault.sol
359      IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).subtractLoss(
      ethAmount
    );

362  IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP)
      .unlockLiquidity(ethAmount);

364  IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addRdpx(
      rdpxAmount
    );

515   IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addProceeds(
      (currentFundingRate * (block.timestamp - startTime)) / 1e18
    );

530  return IRdpxEthOracle(addresses.assetPriceOracle).getRdpxPriceInEth();

535  return IVolatilityOracle(addresses.volatilityOracle).getVolatility(_strike);

545    premium = ((IOptionPricing(addresses.optionPricing).getOptionPrice(
      _strike,
      _price > 0 ? _price : getUnderlyingPrice(),
      getVolatility(_strike),
      timeToExpiry
    ) * _amount) / 1e8);    
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L359

```solidity
File: perp-vault/PerpetualAtlanticVaultLP.sol
210   IERC20WithBurn(rdpx).balanceOf(address(this)) >= _rdpxCollateral + amount,
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L210

```solidity
File: reLP/ReLPContract.sol
150    IERC20WithBurn(addresses.pair).safeApprove(
      addresses.ammRouter,
      type(uint256).max
    );

155    IERC20WithBurn(addresses.tokenA).safeApprove(
      addresses.ammRouter,
      type(uint256).max
    );

160    IERC20WithBurn(addresses.tokenB).safeApprove(
      addresses.ammRouter,
      type(uint256).max
    );

217      tokenAInfo.tokenAReserve = IRdpxReserve(addresses.tokenAReserve)
      .rdpxReserve(); // rdpx reserves

221    tokenAInfo.tokenAPrice = IRdpxEthOracle(addresses.rdpxOracle)
      .getRdpxPriceInEth();

237    uint256 totalLpSupply = IUniswapV2Pair(addresses.pair).totalSupply();

243     IERC20WithBurn(addresses.pair).transferFrom(
      addresses.amo,
      address(this),
      lpToRemove
    );

257  (, uint256 amountB) = IUniswapV2Router(addresses.ammRouter).removeLiquidity(

277   uint256 tokenAAmountOut = IUniswapV2Router(addresses.ammRouter)

286  (, , uint256 lp) = IUniswapV2Router(addresses.ammRouter).addLiquidity(

298  IERC20WithBurn(addresses.pair).safeTransfer(addresses.amo, lp);

299  IUniV2LiquidityAmo(addresses.amo).sync();    

302  IERC20WithBurn(addresses.tokenA).safeTransfer(

304  IERC20WithBurn(addresses.tokenA).balanceOf(address(this))

306  IRdpxV2Core(addresses.rdpxV2Core).sync();
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L150

## [G-03] Can Make The Variable Outside The Loop To Save Gas 
When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

```
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;
        
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        
        return total;
    }
}
```


```solidity
File: core/RdpxV2Core.sol
997   uint256 balance = IERC20WithBurn(reserveAsset[i].tokenAddress).balanceOf(
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L997

```solidity
File: perp-vault/PerpetualAtlanticVault.sol
329   uint256 strike = optionPositions[optionIds[i]].strike;

330   uint256 amount = optionPositions[optionIds[i]].amount;

419   uint256 strike = strikes[i];

421   uint256 amount = optionsPerStrike[strike] -

426   uint256 timeToExpiry = nextFundingPaymentTimestamp() -

429   uint256 premium = calculatePremium(
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L329

## [G-04] Make 3 event parameters indexed when possible
 events are used to emit information about state changes in a contract. When defining an event, it's important to consider the gas cost of emitting the event, as well as the efficiency of searching for events in the Ethereum blockchain.

Making event parameters indexed can help reduce the gas cost of emitting events and improve the efficiency of searching for events. When an event parameter is marked as indexed, its value is stored in a separate data structure called the event topic, which allows for more efficient searching of events.
Exmple:
```
• event UserMetadataEmitted(uint256 indexed userId, bytes32 indexed key, bytes value);

• event UserMetadataEmitted(uint256 indexed userId, bytes32 indexed key, bytes indexed value);
```

```solidity
File: amo/UniV2LiquidityAmo.sol
387    event LogAddLiquidity(
    address indexed sender,
    uint256 tokenAAmount,
    uint256 tokenBAmount,
    uint256 tokenAAmountMin,
    uint256 tokenBAmountMin,
    uint256 tokenAUsed,
    uint256 tokenBUsed,
    uint256 lpReceived
  );

  event LogRemoveLiquidity(
    address indexed sender,
    uint256 lpAmount,
    uint256 tokenAAmountMin,
    uint256 tokenBAmountMin,
    uint256 tokenAReceived,
    uint256 tokenBReceived
  );

  event LogSwap(
    address indexed sender,
    uint256 token1Amount,
    uint256 token2AmountOutMin,
    bool swapTokenAForTokenB,
    uint256 token2Amount
  );

  event LogAssetsTransfered(
    address indexed sender,
    uint256 tokenAAmount,
    uint256 tokenBAmount
  );

  event LogEmergencyWithdraw(address sender, address[] tokens);
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L387-L421

```solidity
File: amo/UniV3LiquidityAmo.sol
  event RecoveredERC20(address token, uint256 amount);
  event RecoveredERC721(address token, uint256 id);
  event LogAssetsTransfered(
    uint256 tokenAAmount,
    uint256 tokenBAmount,
    address tokenAAddress,
    address tokenBAddress
  );
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L367-L375

```solidity
File: decaying-bonds/RdpxDecayingBonds.sol
  event BondMinted(
    address to,
    uint256 bondId,
    uint256 expiry,
    uint256 rdpxAmount
  );

  event EmergencyWithdraw(address sender);
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L46-L53

```solidity
File: perp-vault/PerpetualAtlanticVaultLP.sol
  event Deposit(
    address indexed caller,
    address indexed owner,
    uint256 assets,
    uint256 shares
  );
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L28-L33

## [G-05] Amounts should be checked for 0 before calling a transfer
It is generally a good practice to check for zero values before making any transfers in smart contract functions. This can help to avoid unnecessary external calls and can save gas costs.

Checking for zero values is especially important when transferring tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can check whether a value is zero by using the == operator. Here's an example of how you can check for a zero value before making a transfer:

```
function transfer(address payable recipient, uint256 amount) public {
    require(amount > 0, "Amount must be greater than zero");
    recipient.transfer(amount);
}
```
In the above example, we check to make sure that the amount parameter is greater than zero before making the transfer to the recipient address. If the amount is zero or negative, the function will revert and the transfer will not be made.

```solidity
File: amo/UniV2LiquidityAmo.sol
149   token.safeTransfer(msg.sender, token.balanceOf(address(this)));

168   IERC20WithBurn(addresses.tokenA).safeTransfer(
      addresses.rdpxV2Core,
      tokenABalance
    );

172   IERC20WithBurn(addresses.tokenB).safeTransfer(
      addresses.rdpxV2Core,
      tokenBBalance
    ); 

210   IERC20WithBurn(addresses.tokenA).safeTransferFrom(
      addresses.rdpxV2Core,
      address(this),
      tokenAAmount
    );

215    IERC20WithBurn(addresses.tokenB).safeTransferFrom(
      addresses.rdpxV2Core,
      address(this),
      tokenBAmount
    );

321    IERC20WithBurn(token1).safeTransferFrom(
      addresses.rdpxV2Core,
      address(this),
      token1Amount
    );               
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L149

```solidity
File: amo/UniV3LiquidityAmo.sol
158 IERC20WithBurn(params._tokenA).transferFrom(
      rdpxV2Core,
      address(this),
      params._amount0Desired
    );
163     IERC20WithBurn(params._tokenB).transferFrom(
      rdpxV2Core,
      address(this),
      params._amount1Desired
    );

302  TransferHelper.safeApprove(_tokenA, address(univ3_router), _amountAtoB);

319  TransferHelper.safeTransfer(tokenAddress, rdpxV2Core, tokenAmount);

330   INonfungiblePositionManager(tokenAddress).safeTransferFrom(
      address(this),
      rdpxV2Core,
      token_id
    );

357     IERC20WithBurn(tokenA).safeTransfer(rdpxV2Core, tokenABalance);

358     IERC20WithBurn(tokenB).safeTransfer(rdpxV2Core, tokenBBalance);    
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L158

```solidity
File: decaying-bonds/RdpxDecayingBonds.sol
105   token.safeTransfer(msg.sender, token.balanceOf(address(this)));
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L105

## [G-06] Do not calculate constants
Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas

```solidity
File: core/RdpxV2Core.sol
88   uint256 public constant RDPX_RATIO_PERCENTAGE = 25 * DEFAULT_PRECISION;

91   uint256 public constant ETH_RATIO_PERCENTAGE = 75 * DEFAULT_PRECISION;
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L88

## [G-07] Use hardcode address instead address(this)
it can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

Here's an example of how you can use a hardcoded address instead of address(this):

```
contract MyContract {
    address public myAddress = 0x1234567890123456789012345678901234567890;
    #L
    function doSomething() public {
        // Use myAddress instead of address(this)
        require(msg.sender == myAddress, "Caller is not authorized");
        
        // Do something
    }
}
```
In the above example, we have a contract MyContract with a public address variable myAddress. Instead of using address(this) to retrieve the contract's address, we have pre-calculated and hardcoded the address in the variable. This can help to reduce the gas cost of our contract and make our code more efficient.

[References](https://book.getfoundry.sh/reference/forge-std/compute-create-address)


```solidity
File: amo/UniV2LiquidityAmo.sol
212   address(this),

217   address(this),

230   address(this),
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L212

```solidity
File: amo/UniV3LiquidityAmo.sol
160  address(this),

165  address(this),

189  address(this),
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L160

```solidity
File: reLP/ReLPContract.sol
242  address(this),

263  address(this),

282  address(this),

293  address(this),

304  IERC20WithBurn(addresses.tokenA).balanceOf(address(this))
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L242


## [G-08] Use Assembly To Check For address(0)
it's generally more gas-efficient to use assembly to check for a zero address (address(0)) than to use the == operator.

The reason for this is that the == operator generates additional instructions in the EVM bytecode, which can increase the gas cost of your contract. By using assembly, you can perform the zero address check more efficiently and reduce the overall gas cost of your contract.

Here's an example of how you can use assembly to check for a zero address:

```
contract MyContract {
    function isZeroAddress(address addr) public pure returns (bool) {
        uint256 addrInt = uint256(addr);
        
        assembly {
            // Load the zero address into memory
            let zero := mload(0x00)
            
            // Compare the address to the zero address
            let isZero := eq(addrInt, zero)
            
            // Return the result
            mstore(0x00, isZero)
            return(0, 0x20)
        }
    }
}
```
In the above example, we have a function isZeroAddress that takes an address as input and returns a boolean value indicating whether the address is equal to the zero address. Inside the function, we convert the address to an integer using uint256(addr), and then use assembly to compare the integer to the zero address.

By using assembly to perform the zero address check, we can make our code more gas-efficient and reduce the overall cost of our contract. It's important to note that assembly can be more difficult to read and maintain than Solidity code, so it should be used with caution and only when necessary


```solidity
File: amo/UniV2LiquidityAmo.sol
83   require(
      _tokenA != address(0) &&
        _tokenB != address(0) &&
        _pair != address(0) &&
        _rdpxV2Core != address(0) &&
        _rdpxOracle != address(0) &&
        _ammFactory != address(0) &&
        _ammRouter != address(0),
      "reLPContract: address cannot be 0"
    );

131   require(_token != address(0), "reLPContract: token cannot be 0");

132   require(_spender != address(0), "reLPContract: spender cannot be 0");    
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L83

```solidity
File: core/RdpxV2Core.sol
244   require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L244


```solidity
File: perp-vault/PerpetualAtlanticVaultLP.sol
94   require(
      _perpetualAtlanticVault != address(0) || _rdpx != address(0),
      "ZERO_ADDRESS"
    );   
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L94

```solidity
File: reLP/ReLPContract.sol
126  require(
      _tokenA != address(0) &&
        _tokenB != address(0) &&
        _pair != address(0) &&
        _rdpxV2Core != address(0) &&
        _tokenAReserve != address(0) &&
        _amo != address(0) &&
        _rdpxOracle != address(0) &&
        _ammFactory != address(0) &&
        _ammRouter != address(0),
      "reLPContract: address cannot be 0"
    );
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L126-L137


## [G-09] Use constants instead of type(uintx).max
 it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

Here's an example of how you can use a constant instead of type(uintX).max:
```
contract MyContract {
    uint120 constant MAX_VALUE = 2**120 - 1;
    
    function doSomething(uint120 value) public {
        require(value <= MAX_VALUE, "Value exceeds maximum");
        
        // Do something
    }
}
```
In the above example, we have a contract with a constant MAX_VALUE that represents the maximum value of a uint120. When the doSomething function is called with a value parameter, it checks whether the value is less than or equal to MAX_VALUE using the <= operator.

By using a constant instead of type(uint120).max, we can make our code more efficient and reduce the gas cost of our contract.

It's important to note that using constants can make your code more readable and maintainable, since the value is defined in one place and can be easily updated if necessary. However, constants should be used with caution and only when their value is known at compile-time.


```solidity
File: amo/UniV3LiquidityAmo.sol
126     type(uint128).max,
127     type(uint128).max

190    type(uint256).max

223        type(uint128).max,
224        type(uint128).max
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L126

```solidity
File: core/RdpxV2Core.sol
341  type(uint256).max

343    IERC20WithBurn(weth).approve(addresses.dopexAMMRouter, type(uint256).max);

344    IERC20WithBurn(weth).approve(addresses.dpxEthCurvePool, type(uint256).max);

347  type(uint256).max
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L341

```solidity
File: perp-vault/PerpetualAtlanticVault.sol
209   type(uint256).max

247   type(uint256).max
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L209

```solidity
File: perp-vault/PerpetualAtlanticVaultLP.sol
106    collateral.approve(_perpetualAtlanticVault, type(uint256).max);

107    ERC20(rdpx).approve(_perpetualAtlanticVault, type(uint256).max);

155    if (allowed != type(uint256).max) {
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L106

```solidity
File: reLP/ReLPContract.sol
152  type(uint256).max

157  type(uint256).max

162  type(uint256).max
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L152



## [G-10] Use ERC721A instead ERC721
ERC721A standard, ERC721A is an improvement standard for ERC721 tokens. It was proposed by the Azuki team and used for developing their NFT collection. Compared with ERC721, ERC721A is a more gas-efficient standard to mint a lot of of NFTs simultaneously. It allows developers to mint multiple NFTs at the same gas price. This has been a great improvement due to Ethereum's sky-rocketing gas fee.

[Reffrence](https://nextrope.com/erc721-vs-erc721a-2/)


```solidity
File: core/RdpxV2Bond.sol
4   import { ERC721 } from "@openzeppelin/contracts/token/ERC721/ERC721.sol";
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L4

```solidity
File: decaying-bonds/RdpxDecayingBonds.sol
9   import { ERC721 } from "@openzeppelin/contracts/token/ERC721/ERC721.sol";
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L9

```solidity
File: perp-vault/PerpetualAtlanticVault.sol
9   import { ERC721 } from "@openzeppelin/contracts/token/ERC721/ERC721.sol";
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L9

## [G‑11] Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if-statement
require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }.

```solidity
File: perp-vault/PerpetualAtlanticVaultLP.sol
287 require(
      assets <= totalAvailableCollateral(),
      "Not enough available assets to satisfy withdrawal"
    );
    _totalCollateral -= assets;
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L287

## [G-12] Use assembly for math (add, sub, mul, div)
Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety.

Addition:
```
//addition in SolidityCache multiple accesses of a mapping/array
function addTest(uint256 a, uint256 b) public pure {
    uint256 c = a + b;
}
```
Gas: 303
```
//addition in assembly
function addAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := add(a, b)
        if lt(c, a) {
            mstore(0x00, "overflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 263
```
Subtraction
```
//subtraction in Solidity
function subTest(uint256 a, uint256 b) public pure {
  uint256 c = a - b;
}
```
Gas: 300
```
//subtraction in assembly
function subAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := sub(a, b)
        if gt(c, a) {
            mstore(0x00, "underflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 263
```
Multiplication
```
//multiplication in Solidity
function mulTest(uint256 a, uint256 b) public pure {
    uint256 c = a * b;
}
```
Gas: 325
```
//multiplication in assembly
function mulAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := mul(a, b)
        if lt(c, a) {
            mstore(0x00, "overflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 265

Division
```
//division in Solidity
function divTest(uint256 a, uint256 b) public pure {
    uint256 c = a * b;
}
```
Gas: 325

```
//division in assembly
function divAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := div(a, b)
        if gt(c, a) {
            mstore(0x00, "underflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 265

```solidity
File: perp-vault/PerpetualAtlanticVault.sol
568   return genesis + (latestFundingPaymentPointer * fundingDuration);
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L568

```solidity
File: perp-vault/PerpetualAtlanticVault.sol
581   return _strike - remainder + roundingPrecision;
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L681

```solidity
File: perp-vault/PerpetualAtlanticVaultLP.sol
195  _totalCollateral += proceeds;

213  _rdpxCollateral += amount;
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L195

## [G-13] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.
Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.
[Reffrence](https://github.com/code-423n4/2023-06-stader-findings/blob/main/data/JCN-G.md#gasreport-output-with-all-optimizations-applied)

```solidity
File: amo/UniV2LiquidityAmo.sol
161 uint256 tokenABalance = IERC20WithBurn(addresses.tokenA).balanceOf(
      address(this)
    );
    uint256 tokenBBalance = IERC20WithBurn(addresses.tokenB).balanceOf(
      address(this)
    );
    // transfer token A and B from this contract to the rdpxV2Core
    IERC20WithBurn(addresses.tokenA).safeTransfer(
      addresses.rdpxV2Core,
      tokenABalance
    );
    IERC20WithBurn(addresses.tokenB).safeTransfer(
      addresses.rdpxV2Core,
      tokenBBalance
    );

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L161

```solidity
File: core/RdpxV2Core.sol
1189  uint256 strike = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
      .roundUp(rdpxPrice - (rdpxPrice / 4)); // 25% below the current price

    uint256 timeToExpiry = IPerpetualAtlanticVault(
      addresses.perpetualAtlanticVault
    ).nextFundingPaymentTimestamp() - block.timestamp;
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1189-L1194


## [G-14] Empty blocks should be removed to save gas
Empty blocks in smart contracts can save gas by minimizing unnecessary computations. Gas optimization is crucial for reducing transaction costs and contract efficiency. However, the decision to include empty blocks should consider the trade-off between gas savings and code readability. Striking a balance is essential for maintaining a clear and maintainable codebase.

```solidity
File: perp-vault/PerpetualAtlanticVaultLP.sol
231    function _beforeTokenTransfer(
    address from,
    address,
    uint256
  ) internal virtual {}
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L231-L235

## [G-15] Avoid emitting storage values
Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. We can avoid unecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.

```solidity
File: core/RdpxV2Core.sol
349  emit LogSetAddresses(addresses);

370  emit LogSetPricingOracleAddresses(pricingOracleAddresses);

966  emit LogAddToDelegate(_amount, _fee, delegates.length - 1);
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L349

```solidity
File: perp-vault/PerpetualAtlanticVault.sol
211   emit AddressesSet(addresses);

389   emit PayFunding(
      msg.sender,
      totalFundingForEpoch[latestFundingPaymentPointer],
      latestFundingPaymentPointer
    );

485     emit FundingPaid(
          msg.sender,
          ((currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
            1e18),
          latestFundingPaymentPointer
        );
494   emit FundingPaymentPointerUpdated(latestFundingPaymentPointer);

519     emit FundingPaid(
      msg.sender,
      ((currentFundingRate * (block.timestamp - startTime)) / 1e18),
      latestFundingPaymentPointer
    );
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L211


```solidity
File: amo/UniV3LiquidityAmo.sol
268    emit log(positions_array.length);
269    emit log(positions_mapping[pos.token_id].token_id);
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L268-L269

## [G-16] Use Modifiers Instead of Functions To Save Gas
Using modifiers instead of separate functions can indeed help save gas in smart contracts. Modifiers allow you to modify the behavior of functions in a reusable and efficient manner. Here's an example of how the provided function could be refactored using a modifier:
```solidity
modifier validate(bool _clause, uint256 _errorCode) {
  if (!_clause) revert PerpetualAtlanticVaultError(_errorCode);
  _;
}
```
```solidity
function myFunction() public validate(someCondition, errorCode) {
  // Function body
}
```
By using modifiers, the validation logic is integrated directly into the function declaration. This eliminates the need for a separate validation function and reduces gas consumption by avoiding an additional function call.


```solidity
File: core/RdpxV2Core.sol
751  function _validate(bool _clause, uint256 _errorCode) internal pure {
    if (!_clause) revert RdpxV2CoreError(_errorCode);
  }
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L751

```solidity
File: perp-vault/PerpetualAtlanticVault.sol
621   function _validate(bool _clause, uint256 _errorCode) private pure {
    if (!_clause) revert PerpetualAtlanticVaultError(_errorCode);
  }
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L621

note:This with 0.8.9 compiler and optimization enabled. As you can see it's cheaper to deploy with a modifier, and it will save you about 30 gas. But sometimes modifiers increase code size of the contract.

## [G-17] Gas savings can be achieved by changing the model for assigning value to the structure (260 gas)

```solidity
File: amo/UniV2LiquidityAmo.sol
  addresses = Addresses({
      tokenA: _tokenA,
      tokenB: _tokenB,
      pair: _pair,
      rdpxV2Core: _rdpxV2Core,
      rdpxOracle: _rdpxOracle,
      ammFactory: _ammFactory,
      ammRouter: _ammRouter
    });
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L93-L101

`change to this` 

```
addresses.tokenA = _tokenA;
addresses.tokenB = _tokenB;
addresses.pair = _pair;
addresses.rdpxV2Core = _rdpxV2Core;
addresses.rdpxOracle = _rdpxOracle;
addresses.ammFactory = _ammFactory;
addresses.ammRouter = _ammRouter;
```

```solidity
File: core/RdpxV2Core.sol
1:
129 ReserveAsset memory zeroAsset = ReserveAsset({
      tokenAddress: address(0),
      tokenBalance: 0,
      tokenSymbol: "ZERO"
    });

2:
253 ReserveAsset memory asset = ReserveAsset({
      tokenAddress: _asset,
      tokenBalance: 0,
      tokenSymbol: _assetSymbol
    });    

3:
327   addresses = Addresses({
      dopexAMMRouter: _dopexAMMRouter,
      dpxEthCurvePool: _dpxEthCurvePool,
      rdpxDecayingBonds: _rdpxDecayingBonds,
      perpetualAtlanticVault: _perpetualAtlanticVault,
      perpetualAtlanticVaultLP: _perpetualAtlanticVaultLP,
      rdpxReserve: _rdpxReserve,
      rdpxV2ReceiptToken: _rdpxV2ReceiptToken,
      feeDistributor: _feeDistributor,
      reLPContract: _reLPContract,
      receiptTokenBonds: _receiptTokenBonds
    });

4:
365   pricingOracleAddresses = PricingOracleAddresses({
      rdpxPriceOracle: _rdpxPriceOracle,
      dpxEthPriceOracle: _dpxEthPriceOracle
    });

5:
955  Delegate memory delegatePosition = Delegate({
      amount: _amount,
      fee: _fee,
      owner: msg.sender,
      activeCollateral: 0
    });
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L129-L133

`change to this` 

```solidity
1:
ReserveAsset memory zeroAsset;
zeroAsset.tokenAddress = address(0);
zeroAsset.tokenBalance = 0;
zeroAsset.tokenSymbol = "ZERO";

2:
ReserveAsset memory asset;
asset.tokenAddress = _asset;
asset.tokenBalance = 0;
asset.tokenSymbol = _assetSymbol;

3:
addresses.dopexAMMRouter = _dopexAMMRouter;
addresses.dpxEthCurvePool = _dpxEthCurvePool;
addresses.rdpxDecayingBonds = _rdpxDecayingBonds;
addresses.perpetualAtlanticVault = _perpetualAtlanticVault;
addresses.perpetualAtlanticVaultLP = _perpetualAtlanticVaultLP;
addresses.rdpxReserve = _rdpxReserve;
addresses.rdpxV2ReceiptToken = _rdpxV2ReceiptToken;
addresses.feeDistributor = _feeDistributor;
addresses.reLPContract = _reLPContract;
addresses.receiptTokenBonds = _receiptTokenBonds;

4:
pricingOracleAddresses.rdpxPriceOracle = _rdpxPriceOracle;
pricingOracleAddresses.dpxEthPriceOracle = _dpxEthPriceOracle;

5:
Delegate memory delegatePosition;
delegatePosition.amount = _amount;
delegatePosition.fee = _fee;
delegatePosition.owner = msg.sender;
delegatePosition.activeCollateral = 0;
```


```solidity
File:  perp-vault/PerpetualAtlanticVault.sol
198 addresses = Addresses({
      optionPricing: _optionPricing,
      assetPriceOracle: _assetPriceOracle,
      volatilityOracle: _volatilityOracle,
      feeDistributor: _feeDistributor,
      rdpx: _rdpx,
      perpetualAtlanticVaultLP: _perpetualAtlanticVaultLP,
      rdpxV2Core: _rdpxV2Core
    });
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L198-L206

`change to this` 

```solidity
addresses.optionPricing = _optionPricing;
addresses.assetPriceOracle = _assetPriceOracle;
addresses.volatilityOracle = _volatilityOracle;
addresses.feeDistributor = _feeDistributor;
addresses.rdpx = _rdpx;
addresses.perpetualAtlanticVaultLP = _perpetualAtlanticVaultLP;
addresses.rdpxV2Core = _rdpxV2Core;
```


```solidity
File: reLP/ReLPContract.sol
138  addresses = Addresses({
      tokenA: _tokenA,
      tokenB: _tokenB,
      pair: _pair,
      rdpxV2Core: _rdpxV2Core,
      tokenAReserve: _tokenAReserve,
      amo: _amo,
      rdpxOracle: _rdpxOracle,
      ammFactory: _ammFactory,
      ammRouter: _ammRouter
    });
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L138-L148

`change to this` 


```solidity
addresses.tokenA = _tokenA;
addresses.tokenB = _tokenB;
addresses.pair = _pair;
addresses.rdpxV2Core = _rdpxV2Core;
addresses.tokenAReserve = _tokenAReserve;
addresses.amo = _amo;
addresses.rdpxOracle = _rdpxOracle;
addresses.ammFactory = _ammFactory;
addresses.ammRouter = _ammRouter;
```

## [G‑18] Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate
Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to [not having to recalculate the key's keccak256 hash](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0) (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

Note: These are instances that the automated report missed.
Total Instances: `1`

```solidity
File: core/RdpxV2Core.sol
76 mapping(uint256 => Bond) public bonds;

  /// @dev Option id => owned or not (boolean)
  mapping(uint256 => bool) public optionsOwned;

  /// @dev Funding paid for epoch
  mapping(uint256 => bool) public fundingPaidFor;
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L76-L82


## [G‑19] Using storage instead of memory for structs/arrays saves gas
When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

Note: These are instances that the automated report missed.
Total Instances: `3`

```solidity
File: amo/UniV3LiquidityAmo.sol
218  Position memory pos = positions_array[positionIndex];
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L218

```solidity
File: core/RdpxV2Core.sol
1138   ReserveAsset memory asset = reserveAsset[reservesIndex[_token]];

1272   Delegate memory delegatePosition = delegates[_delegateId];
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1138

## [G‑20] State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.
Note: These are instances that the automated report missed.
Total Instances: `54`

```solidity
File: amo/UniV2LiquidityAmo.sol
1: in _sendTokensToRdpxV2Core function addresses is state variable
161  uint256 tokenABalance = IERC20WithBurn(addresses.tokenA).balanceOf(

164  uint256 tokenBBalance = IERC20WithBurn(addresses.tokenB).balanceOf(

168  IERC20WithBurn(addresses.tokenA).safeTransfer(
169  addresses.rdpxV2Core,

172  IERC20WithBurn(addresses.tokenB).safeTransfer(
173  addresses.rdpxV2Core,  


2: in addLiquidity function addresses is state variable
200  IERC20WithBurn(addresses.tokenA).safeApprove(
201  addresses.ammRouter,
  
204  IERC20WithBurn(addresses.tokenB).safeApprove(
205  addresses.ammRouter,

210  IERC20WithBurn(addresses.tokenA).safeTransferFrom(
211  addresses.rdpxV2Core,

215  IERC20WithBurn(addresses.tokenB).safeTransferFrom(
216  addresses.rdpxV2Core,

222    (tokenAUsed, tokenBUsed, lpReceived) = IUniswapV2Router(addresses.ammRouter)
      .addLiquidity(
        addresses.tokenA,
        addresses.tokenB,


3: in removeLiquidity function addresses is state variable

268    IERC20WithBurn(addresses.pair).safeApprove(addresses.ammRouter, lpAmount);

    // remove liquidity
    (tokenAReceived, tokenBReceived) = IUniswapV2Router(addresses.ammRouter)
      .removeLiquidity(
        addresses.tokenA,
274     addresses.tokenB,

314 token1 = addresses.tokenA;
      token2 = addresses.tokenB;
    } else {
      token1 = addresses.tokenB;
      token2 = addresses.tokenA;

322 addresses.rdpxV2Core,

328 IERC20WithBurn(token1).safeApprove(addresses.ammRouter, token1Amount);

336 token2Amount = IUniswapV2Router(addresses.ammRouter)
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L161

```solidity
File: reLP/ReLPContract.sol
205   addresses.tokenA,
206   addresses.tokenB

209   addresses.ammFactory,

217   tokenAInfo.tokenAReserve = IRdpxReserve(addresses.tokenAReserve)

221   tokenAInfo.tokenAPrice = IRdpxEthOracle(addresses.rdpxOracle)

224   tokenAInfo.tokenALpReserve = addresses.tokenA == tokenASorted

237   uint256 totalLpSupply = IUniswapV2Pair(addresses.pair).totalSupply();

243  IERC20WithBurn(addresses.pair).transferFrom(
      addresses.amo,

257   (, uint256 amountB) = IUniswapV2Router(addresses.ammRouter).removeLiquidity(
      addresses.tokenA,
      addresses.tokenB,

269 path[0] = addresses.tokenB;
    path[1] = addresses.tokenA;     

277 uint256 tokenAAmountOut = IUniswapV2Router(addresses.ammRouter)

286  (, , uint256 lp) = IUniswapV2Router(addresses.ammRouter).addLiquidity(
      addresses.tokenA,
      addresses.tokenB,

298  IERC20WithBurn(addresses.pair).safeTransfer(addresses.amo, lp);
    IUniV2LiquidityAmo(addresses.amo).sync();

    // transfer rdpx to rdpxV2Core
    IERC20WithBurn(addresses.tokenA).safeTransfer(
      addresses.rdpxV2Core,
      IERC20WithBurn(addresses.tokenA).balanceOf(address(this))
    );
    IRdpxV2Core(addresses.rdpxV2Core).sync();      
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L205

## [G‑21] internal functions only called once can be inlined to save gas
Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.
Note: These are instances that the automated report missed.
Total Instances: `4`
```solidity
File: core/RdpxV2Bond.sol
45    function _beforeTokenTransfer(
    address from,
    address to,
    uint256 tokenId,
    uint256 batchSize
  ) internal override(ERC721, ERC721Enumerable) {
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/#L45-L50

```solidity
File: decaying-bonds/RdpxDecayingBonds.sol
162 function _beforeTokenTransfer(
    address from,
    address to,
    uint256 tokenId,
    uint256 batchSize
  ) internal override(ERC721, ERC721Enumerable) {
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L162-L167


```solidity
File: dpxETH/DpxEthToken.sol
55  function _beforeTokenTransfer(
    address from,
    address to,
    uint256 amount
  ) internal override {
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L55-L59

```solidity
File: perp-vault/PerpetualAtlanticVault.sol
635  function _beforeTokenTransfer(
    address from,
    address to,
    uint256 tokenId,
    uint256 batchSize
  ) internal override(ERC721, ERC721Enumerable) {
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L635-L640

## [G‑22] Use assembly for small keccak256 hashes, in order to save gas
If the arguments to the encode call can fit into the scratch space (two words or fewer), then it's more efficient to use assembly to generate the hash (80 gas): keccak256(abi.encodePacked(x, y)) -> assembly {mstore(0x00, a); mstore(0x20, b); let hash := keccak256(0x00, 0x40); }

Note: These are instances that the automated report missed.
Total Instances: `1`
```solidity
File: amo/UniV3LiquidityAmo.sol
106  keccak256(abi.encodePacked(address(this), _tickLower, _tickUpper))
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L106

## [G‑23] >= costs less gas than >
The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas. If < is being used, the condition can be inverted.
Note: These are instances that the automated report missed.
Total Instances: `5`
```solidity
File: amo/UniV2LiquidityAmo.sol
112    require(
      _slippageTolerance > 0,
      "reLPContract: slippage tolerance must be greater than 0"
    );
133  require(_amount > 0, "reLPContract: amount must be greater than 0");    
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L112

```solidity
File: reLP/ReLPContract.sol
93      require(
      _reLPFactor > 0,
      "reLPContract: reLP factor must be greater than 0"
    );
174      require(
      _liquiditySlippageTolerance > 0,
      "reLPContract: liquidity slippage tolerance must be greater than 0"
    );

189    require(
      _slippageTolerance > 0,
      "reLPContract: slippage tolerance must be greater than 0"
    );        
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L93