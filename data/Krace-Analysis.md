Author: Krace

# 1. Overview
According to the doc, Deopx upgraded the rDPX token and introduced AMO ideology from Frax Finance.
My main concern is the logic problem in the contract. Any part that does not conform to the original design intention or code comments may be at risk, so I audited the code according to the given scope. First of all, AMO with a small amount of code, and The token implementation part, followed by the perpetual contract part, and finally the core rdpxV2Core part of the entire contract.



# 2. Code Analysis

## 2.1 UniV2LiquidityAmo
### Function
UniV2LiquidityAmo is an uniswap v2 liquidity amo contract. It inherits from `AccessControl`, only the `DEFAULT_ADMIN_ROLE` can perform its operations, except for one function `sync`, which is used to update the balance of `lpToken`.

`DEFAULT_ADMIN_ROLE` could set the addresses and slippage, approve another contract to use any token in this contract, and get any token held by this contract. The addresses include two base tokens `tokenA` and `tokenB` to control the liquidity.

The main function of this contract includes `addLiquidity`, `removeLiquidity` and `swap`, `DEFAULT_ADMIN_ROLE` could transfer `tokenA` and `tokenB` from `rdpxV2Core` to `IUniswapV2Router` to add liquidity and withdraw to remove liquidity. Admin could also swap `tokenA` and `tokenB` by function `swap`. Note that this contract will not hold any `tokenA` and `tokenB`, any left tokens will be transferred back to `rdpxV2Core`.


### Potential Risks
The `DEFAULT_ADMIN_ROLE` seems too powerful and may lead to centralization risks. For example,  `DEFAULT_ADMIN_ROLE` could withdraw all tokens in the contract, which may lead to user distrust.



## 2.2 UniV3LiquidityAmo
### Function
It's similar to the UniV2LiquidityAmo, but its liquidity is minted by `INonfungiblePositionManager univ3_positions`.

This contract also has three main functions: `addLiquidity`, `removeLiquidity` and `swap`, which are similar to UniV2LiquidityAmo. Besides, it uses `univ3_positions` to collect fees accumulated. 

## Potential Risks
1. First, the address of `univ3_factory`, `univ3_positions` and `univ3_router` are set in Constructor, and is not changeable, maybe you should set them to `immutable` or add a function to let admin change them.
2. Second, the admin can `execute` any external contract, which will also lead to centralization risks.
3. Third, the comment of `numPositions` said that it `Only counts non-withdrawn positions`, but it just returns the length of `positions_array` directly. It seems that there is no code that could judge whether a position is `non-withdrawn` or not.



## 2.3 RdpxDecayingBonds
### Function
RdpxDecayingBonds mints rDPX decaying bonds, it's an ERC721 token. The creator owns `MINTER_ROLE` and `DEFAULT_ADMIN_ROLE`. 

`DEFAULT_ADMIN_ROLE` could `pause` and `unpause` this contract, and withdraw any token in this contract when it is paused.

`MINTER_ROLE` will bind some amounts of rdpx to a minted NTF `RDPXDB`, rdpxV2Core (`RDPXV2CORE_ROLE`) could change the amount of rdpx bound to an NFT.

## Potential Risks
`decreaseAmount` actually just sets the `rdpxAmount` of the bond, I think it might be more appropriate to change the function name to `setAmount`.


## 2.4 DpxEthToken
Dopex Synthetic ETH token contract.
It's a normal ERC20 token.


## 2.5 ReLPContract
### Function
Contract to perform the reLP process on the Uniswap V2 AMO.
Providing `reLP` function for `RDPXV2CORE_ROLE`.
`reLP`  calculates the amount of `lpToRemove` according to the given `_amount`, then it `removeLiquidity` with `lpToRemove` to get `tokenA` and `tokenB`. Next, it `swap` half of `tokenB` to `tokenA` and `addLiquidity` with the half `tokenB` and the swapped `tokenA`. Finally, it will return the lp to `amo` and return the `tokenA` left to the `RDPXV2CORE`. In conclusion, `RDPXV2CORE_ROLE` could only get `tokenA` by calling this function.

### Potential Risk
There is a `(Math.sqrt(1e18))` in function `reLp`, maybe you should use 1e9 directly.


## 2.6 PerpetualAtlanticVaultLP
### Function
PerpetualAtlanticVaultLP allows users to deposit or redeem collaterals, when depositing, users will get xxx-LP token to stand for their shares.
The collateral includes normal collateral and rdpx token. The rdpx token can only added by `perpetualAtlanticVault`.
And this contract uses a `_totalCollateral` to store the amount of collateral deposited.


### Potential Risk
`_totalCollateral` will not change if users transfer collateral directly to this contract, so the real balance of collateral in this contract may not equal to `_totalCollateral`. This may lead to Dos in function `subtractLoss` as I submitted in my other report.


## 2.7 PerpetualAtlanticVault
### Function
It's also a Perpetual Atlantic Vault, but it mints ERC721 token.
`RDPXV2CORE_ROLE` could `purchase`, `settle` and `payFunding` by this contract.

> `purchase` accepts `collateralToken` from `RDPXV2CORE_ROLE` according to the premium of option and mints `PAV` to stand for an option. And the corresponding `requiredCollateral` will be locked in `PerpetualAtlanticVaultLP`.

> `settle` accepts a set of options and burns the corresponding NFT. `ethAmount` and `rdpxAmount` are calculated based on the options' amount and strike, `PerpetualAtlanticVaultLP` will transfer `ethAmount` of collateral to `RDPXV2CORE_ROLE`, `RDPXV2CORE_ROLE` will transfer `rdpxAmount` of rdpx to `PerpetualAtlanticVaultLP`

> `payFunding` is responsible for paying the funding.



## 2.8 RdpxV2Core.sol
### Function
The core code of the contract.
> `DEFAULT_ADMIN_ROLE`
  `settle` sends out rdpx and receives weth.
  `provideFunding` pays for funding in `perpetualAtlanticVault`
 
> Normal External
  `bondWithDelegate` locks a set of dpxEth to different delegates.
  `bond` - get the bond directly without delegate.
  `addToDelegate` - anyone can become a delegate by sending weth to this
  `withdraw` - delegate can withdraw WETH that is not used
  `sync` - update the balance of the reserve Asset
  `redeem` - withdraw matured bond, burn the bond NFT.



### Potential Risk
1. The contract uses `optionsOwned` to log whether an option id is owned or not. But it seems to be useless, when `settle` is invoked, `perpetualAtlanticVault` will burn the corresponding option Id, so there is no need to log the state of option Id.
2. The value of `rdpxBurnPercentage` and `rdpxFeePercentage` is set by `DEFAULT_ADMIN_ROLE` and can be greater than 100%. If `DEFAULT_ADMIN_ROLE` wrongly sets these two values, `_transfer` will burn or transfer more rdpx than this contract received, which will let the `reserveAsset[reservesIndex["RDPX"]].tokenBalance` not equal to the rdpx balance of this. You must call `sync` to update this value. Maybe you should update the `reserveAsset[reservesIndex["RDPX"]].tokenBalance` in `_transfer` after burn and transfer.



# 3. Conclusion
Generally speaking, the contract code is highly standardized, but the design is too centralized, which may be because I do not understand the goal of the project. In addition, there are some optimization problems and some useless codes, which should continue to be iterated in subsequent versions.

# 4. Total Time spent
About 20 hours

### Time spent:
20 hours