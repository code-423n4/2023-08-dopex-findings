## 1. Summary of Codebase
rDPXV2 primarily focuses on the new synthetic coin dpxETH that is pegged to ETH, which is minted during the bonding process and will be used to earn boosted yields via staking in curve pools. Dopex aims to use dpxETH as a staple collateral token for future Dopex Options Products.

### 1.1 Bonding Mechanism
The bonding process is where new dpxETH tokens are minted, where a user bonds with the rDPX V2 contract and receives a receipt token. A receipt token represents ETH and dpxETH LP on curve. They can then subsequently redeem back this receipt tokens after bond maturity ends which can then be used to retrieve the dpxETH and ETH associated with amount of receipt tokens minted.

There are 3 ways a user can bond on the Bonding contract; regular bonding, delegate bonding or using a decaying bond.

### 1.1.1 Regular Bonding
The regular bonding mechanism can be summarised in to 6 detailed steps below:

1. User transfer required rdpx and weth + the premium for the size of the rDPX provided for an rDPX Atlantic Perpetual PUT option which is 25% OTM after accounting for bond discount, which is dependent on the balance of rDPX in the treasury reserves. (Note that this will be intially set to 12.5%) This is computed by `RdpxV2Core.calculateBondCost()`. 

<br/>

2. Within `RdpxV2Core.bond()`, the amount of weth required including premium is first transferred to core contract

<br/>

3. The APP options will then be minted to the core contract via `RdpxV2Core._purchaseOptions()`.

<br/>

4. After that, the amount of rdpx required to be transferred by user bonding will be performed via `RdpxV2Core._transfer()`. Here, rdpx transferred by user to core contract is redirected where 50% of the rDPX provided for bonding is burnt from the Treasury Reserve and another 50% is sent to the`feeDistributor` as emissions for veDPX holders. These percentages are variable and can be controlled by governance. In addition, discount provided to user + the amount of rdpx sent by user will be sent back to the backing reserve.

<br/>

5. After transferring of funds by user, the dpxETH will be minted where half of the bond amount worth of dpxETH will be minted to be paired with equaivalent amount of wETH to LP on dpxETH/ETH curve pool. The remaining wETH and rDPX not used for LPing will remain in the backing reserves as backing for dpxETH minted. Once LP is performed, users will be issued the the bond amount with equivalent receipt token amounts, that can subsequently be redeemed after bond maturity (5 days) passes. This is handled in `RdpxV2Core._stake()`

<br/>

6. If reLP process is present, the reLP process will be intiated where part of the Uniswap V2 liquidity is removed, then the ETH received is swapped to rDPX. This essentially increases the price of rDPX and also increases the Backing Reserve of rDPX. This is handled by `ReLPContract.relp()`.

### 1.1.2 Delegate Bonding
Delegate bonding is a system where users come in with ETH and deposit it in a pool (via the Core Contract). They set a fee for the usage of their ETH, where users holding rDPX can use this pool of ETH to bond (using the regular bonding mechanism). This way users can bond with only rdpx or only ETH, allowing both to come together and bond to receive their share of receipt tokens.

The steps for delegate bonding can be summarised below:

1. Delegates first call `RdpxV2Core.addToDelegate()` to deposit wETH (minimum amount of 0.01 wETH to avoid spamming) and sets a fee (minimum of 1% but cannot be more than 100%). The `totalWethDelegated` state variable is updated, which represents the wETH in the core contract that is available for use in delegate bonding.

<br/>

2. Interested users then call `RdpxCore.bondWithDelegate()`, supplying the relevant delegateIds, where a similar bonding mechanism to regular bonding is performed via `_bondWithDelegate()`. The only difference between `_bond()` and `_bondWithDelegate()` is that two seperate bonds with receipt token amounts of ratio 75:25 is minted to delegate  and delegatee via `_stake()` after accounting for fees set by delegate. The amount of receipt tokens associated with bonds for delegate and delegatees is computed by `RdpxV2Core._calculateAmounts()`, called within `_bondWithDelegate()`.

### 1.1.3 Bonding via Decaying Bonds
Decaying bonds are bonds carrying a specific amount of rDPX minted to users as a form of rebate for losses incurred on the Dopex Options Protocol or simply for incentives. This amount is decided when these decaying bonds are minted to users by the minter.

The steps for bonding via decaying bond can be summarised below:

1. The minter with the `MINTER_ROLE` first mints a decaying bond with associated rDPX via `RdpxDecayingBonds.mint()`, specifying the expiry and amount of rDPX.

<br/>

2. User with specific bondID of decaying bond minted to them can then call `RdpxV2Core.bond()` and specify the bondID. This initiates a similar bonding mechanism as regular bonding, with the difference lying in the amount of rDPX and wETH required computed via `RdpxV2Core.calculateBondCost()` where no bond discount is applied. Subsequently in the `RdpxV2Core._transfer()` function, since the bonds assigned to them is a form of rebate/incentive, the user do not need to pay the rDPX required for bonding, and thus only deposit to the core contract the wETH required for bonding + premium for the 25% OTM APP.


### 1.2 Atlantic Perpetual PUTS Options
A perpetual options pools for rDPX options the Core Contract uses for the bonding process. The liquidity for this will be provided via the open market and will be incentivized via DPX earned when shares received from providing collateral is staked. The APP contract follows a epoch system where each epoch is 1 week long.

A detailed summary of how APP system works can be summarised below:
1. Bond writers deposit collateral required to write options via `PerpetualAtlanticVaultLP.deposit()` into LP Vault, receiving associated amount of shares based on amount deposited.

<br/>

2. When bonding via `RdpxV2Core.bond()/bondWithDelegate()`, 25% OTM APP options is purchased for core contract via `RdpxV2Core._purchaseOptions()`, which in turn calls `PerpetualAtlanticVault.purchase()`. 

<br/>


3. At the start of each epoch, the contract admin will call `PerpetualAtlanticVault.calculateFunding()` to calculate the amount of wETH funding required based on new APP puts strike price values minted via bonding as well as existing APP puts not settled.

<br/>

4. After funding is calculated, admin will call `RdpxV2Core.provideFunding()`, which in turn calls `PerpetualAtlanticVault.payFunding()`, and pays the amount of collateral required to write APP puts provided by backing reserve in core contract. This funding is paid by the treasurys backing reserve.

<br/>

5. Once treasury decides based on market conditions to settle APP options via `RdpxV2Core.settle()`, collateral associated with relevant optionIds of bond writers will be unlocked for them to redeem.

<br/>

6. Bond writers can then redeem shares via `PerpetualAtlanticVaultLP.redeem()` to receive back the associated amount of rDPX and wETH based on amount of shares and total supply of shares, collateral and rdPX collateral available.


Note that `PerpetualAtlanticVaultLP.updateFunding()` which in turn calls `PerpetualAtlanticVaultLP.updateFundingPaymentPointer()` is called when actions are performed, which implements a drip wise funding method to transfer required collateral to write APP to PerpetualAtlanticVaultLP for each epoch based on the amount of exisitng and newly minted APP options. It also updates the epoch represented by `latestFundingPaymentPointer` after each epoch ends.

### 1.3 Automated Market Operator (AMO)
The automated market operator (AMO) aims to perform market operations of adding liquidity, removing liquidity and swapping within the rDPX/ETH pools but ensuring that dpxETH price peg is not changed. Similar to FRAX, dpxETH cannot be minted out of thin air that can potentially break the break, allowing a stable price peg of dpxETH.

Traditionally, the frax AMO holds 4 key components:
- Decollateralize - the portion of the strategy which lowers the CR
- Market operations - the portion of the strategy that is run in equilibrium and doesn't change the CR
- Recollateralize - the portion of the strategy which increases the CR
- FXS1559 - a formalized accounting of the balance sheet of the AMO which defines exactly how much FXS can be burned with profits above the target CR

In rDPXV2, the decollaterize and recollaterize components are integrated into the `RdpxV2Core.sol` core contract represented by the functions `RdpxV2Core.upperDepeg()` and `RdpxV2Core.lowerDepeg()` respectively. The equivalent FXS1559 accounting mechanism is also part of the core contract, and the market operations will be performed by the AMOs represented by `UniV2LiquidityAmo.sol ` and `UniV3LiquidityAmo.sol`.

Here, the CR refers to the collateral ratio of rDPX to peg token of dpxETH, which is wETH. If dpxETH price is above the peg, the CR is lowered via `upperDepeg()`, dpxETH supply expands like usual, and AMOs keep running.

If the CR is lowered to the point that the peg slips, the AMOs have predefined recollateralize operations via `lowerDepeg()` which increases the CR. The system recollateralizes just like before as protocol rDPX/ETH LP tokens are redeemed and the CR goes up to return to the peg, thus also allowing continued operation of AMOs.

## 2. Codebase Quality
| Codebase Quality Categories  | Comments |
| :---:| --- |
| **Testing**  | Unit test and Integration tests were present using Foundry, allowing ease of testing for PoCs|
| **Documentation** | The documentation for Dopex has some inconsistencies, but overall provides a satisfactory overview of the protocol   |
| **Organization** | Codebase is decently organized with clear distinctions between the 9 contracts. |    

## 3. Improvements

### 3.1 Ensure LP logic is consistent with docs
In the protocol docs, it is mentioned that 

> The full amount of rDPX and 33% of the ETH provided (33% of 75% = 25%) is sent to the Backing Reserve and used to mint dpxETH which is then paired with the rest of the ETH to LP on curve.

But we can see in `RdpxV2Core._stake()` where the LP logic is performed, the core contract only mints half the amount of dpxETH associated with dpxETH amount to bond and pair with equivalent wETH. This means that there will be remaining wETH left in the backing reserve instead of the amount described in the DoCs where ALL of the wETH is used to LP on curve. Hence it is suggested to ensure the LP logic on curve pool is consistent with docs described.

### 3.2 Ensure maximum value is implemented when setting/changing sensitive protocol parameters
Add a maximum cap to ensure that sensitive system parameters are not accidentally set to a too large/small value.

- `slippageTolerance`: Slippage tolerance `RdpxV2Core.setSlippageTolerance()`

<br/>

- `bondDiscountFactor`: bond discount factor used to control bond discount given to user bonding set via `RdpxV2Core.setBondDiscount()`. 

### 3.3 Consider a brief timelock for delegated wETH 
Currently, delegated wETH by delegates can be withdrawn any time, and as such delegates can DoS delegatees trying to delegate bond anytime by withdrawing required wETH collateral. As such, consider implementing a brief timelock to avoid this scenario.

### 3.4 Consider separating reLP process from bonding process
Since the reLP process involves market operations of swapping, adding and removing liquidity from the rDPX/ETH pools, it might be better to allow admin to manually initiate the reLP process so that they can specify the slippage and deadline required to avoid MEV sandwich attacks.

### 3.5 Allow partial withdrawal of delegated collateral
Consider allowing partial withdrawal logic for collateral delegated so that users that only wants to withdraw part of the wETH collateral do not have to withdraw and redelegate.

### 3.6 Ensure that expiry of decaying bond minted is greater than current timestamp
Add a check (`expiry > block.timestamp`) to avoid setting expiry smaller than current timestamp and resulting in decaying bond not being able to be used.

### 3.7 Create a separate admin only function to recover dust amounts in PerpetualAtlanticVault
Since the PerpetualAtlanticVault employs a drip wise style funding approach to transfer required collateral to vaultLP, there maybe rounding errors associated with division of timestamps based on when funding rate is updated via `PerpetualAtlanticVault._updateFundingRate()` evident in the test files [here](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L594-L613). This can accumulate over time, so consider employing a mechanism to transfer out these dust amounts to the vaultLP.

### 3.8 Consider using multi-hop swaps in AMOs when performing market operations

Currently, the swap functions in the respective AMOs uses a single hop fixed path swap from rDPX -> wETH or wETH -> rDPX. Rather than trading between a single pool, [smart routing could use multiple hops](https://docs.uniswap.org/sdk/v3/guides/routing) (as many as needed) to ensure that the end result of the swap is the optimal price. This prevents potential leakage of value when swapping by using only a single fixed path.

While currently swapping via uniswap v2 only allows a direct swap from rDPX -> wETH, in the future, if there are plans to launch other pools similar to UniV3 or even other pools pairing (such as with stablecoins), the `swap()` function will still be restricted to a single path swap. This also ensures other forms of liquidity to perform market operations in the event there is low liquidity in the primary rDPX/wETH pools.

### 3.9 Consider splitting premium payments between delegate and delegatee during delegate bonding
Currently when bonding via delegate bonding, ALL of the initial premium for the 25% OTM APP is paid for by the delegate. Since bonding via delegate bonding is a 2 party mechanism, the premium paid should also be split accordingly between delegatee and delegate and not solely be covered by the delegate in a similar 75:25 ratio between delegate and delegatee respectively.

While it is true that there is a fee incurred when bonding via delegate bond set by delegate, this fee may end up only covering the premium and will have no additional incentive to the delegate delegating ETH for bonding. Additionally, if fees are set too high to both cover the premium and the incentive towards delegate, it may disincentivize delegatees from utilizing the delegate bonding mechanism, causing dopex to lose out on potential customers using their option products.

## 4. Centralization Risks
### 4.1 System Parameters
The following system parameters set only has a non-zero value check but do not have a maximum value cap that should be enforced.
- `rdpxBurnPercentage`: rdpx to burn from treasury reserve during bonding set via `RdpxV2Core.setRdpxBurnPercentage()`. Excessive value can cause too much rdpx to be burned.

<br/>

- `rdpxFeePercentage`: Fees sent as emission to veDPX holders during bonding set via `RdpxV2Core.setRdpxFeePercentage()`. Excessive value can cause too much fees to be sent to veDPX holders

<br/>

- `bondMaturity`: Bond maturity set via `setBondMaturity()`, excessive values can prevent users from redeeming bonds and getting back receipt tokens


### 4.2 Emergency Withdrawals
- `RdpxV2Core.emergencyWithdraw()`: Protocol admin can withdraw ALL of the funds in the core contract backing reserves, griefing users funds

<br/>

- `PerpetualAtlanticVaultLP.emergencyWithdraw()`: Protocol admin can withdraw ALL of the initial premium provided by users bonding for APP options minted.

### 4.3 Pausing Mechanism
There are `pause()` functions present throughout the codebase where admin can pause ALL users from interacting with the protocol at any time. This includes the following contracts:

- `RdpxV2Bond.sol`, `RdpxV2Core.sol`, `DpxEthToken.sol`, `PerpetualAtlanticVault.sol`, `RdpxReserve.sol`

### 4.4 Others
The following priviledged functions can cause DoS of important
- `RdpxV2Core.removeAssetFromtokenReserves()`: Can remove any assets from interacting with rDPXV2 protocol

<br/>

- `RdpxV2Core.setPricingOracleAddresses()`: Can change pricing oracle address anytime, potentially influencing (increase.decrease) price of rDPX and dpxETH.

<br/>

- `RdpxV2Core.settle()`: Protocol treasury can withhold settling APP options can lock bond writers collateral.

<br/>

- `RdpxV2Core.provideFunding()`: Protocol admin can choose not to provide sufficient funding from backing reserves for APP puts written by bond writers depositing collateral required in vaultLP

## 5. Time Spent
| Day  | Scope | Details|
| :---: | :---: | --- |
| 1-2 | rDPXV2 Docs [Link](https://dopex.notion.site/rDPX-V2-RI-b45b5b402af54bcab758d62fb7c69cb4) | Review Docs and understand key components of protocol, namely: bonding, reLP, APP and AMO |
| 3-6 | rDPXV2 contracts [Link](https://github.com/code-423n4/2023-08-dopex) | Manual audits of contracts with the following flow,  `RdpxV2Core.sol` --> `PerpetualAtlanticVault.sol/PerpetualAtlanticVaultLP.sol` --> `ReLPContract.sol` --> `UniV2LiquidityAmo.sol/UniV3LiquidityAmo.sol`|
| 7 | - |Finish up Analysis report |



### Time spent:
56 hours