# Table Of Contents

- [Architectural Review and Feedback](#Architectural-Review-and-Feedback)
- [Attack Vectors to Depeg dpxETH-ETH](#Attack-Vectors-to-Depeg-dpxETH-ETH)
- [Discussion of Access Control & Centralization Risks](#Discussion-of-Access-Control-&-Centralization-Risks)
- [Market and Protocol Efficacy Risk Scenarios](#Market-and-Protocol-Efficacy-Risk-Scenarios)
- [Technical Architecture Documents and Diagrams](#Technical-Architecture-Documents-and-Diagrams)

# Introduction

This analysis report delves into various sections and modules of the rDPX V2 Product that the RED-LOTUS team audited, specific sections covered are listed within the table of contents above. 

Our approach included a thorough examination and testing of the codebase, research on wider security implications applicable to the protocol and expanded discussion of potential out of scope/known issues from the audit contest. 

A number of potential issues were identified related to centralization and systemic market risks applicable to the protocol were explored. Furthermore, as the Project Team specifically asked for Wardens to explore ways to break the dpxETH-ETH peg, the RED-LOTUS team explored a number of attack vectors and gave detailed description in the corresponding section of the report.

The team also supplied feedback on the rDPX V2 Product Specification from an Architectural perspective and demonstrated that there were inconsistencies between the supplied documentation and the production codebase. We hope that the Project Team will gain value from our Analysis to update their documentation where applicable.

Throughout our analysis, we aimed to provide a comprehensive understanding of the codebase, suggesting areas for possible improvement. To validate our insights, we supplemented our explanations with illustrative figures, demonstrating robust comprehension of Dopex's internal code functionality.

Over the course of the 14-day contest, we dedicated approximately 300+ hours to auditing the codebase. Our ultimate goal is to provide a report that will give the Project Team wider perspective and value from our research conducted to strengthen the security posture, usability and efficiency of the protocol.

This analysis report and included diagrams are free to be shared with other parties as the Project Team see fit.


# Architectural Review and Feedback

- [Introduction](#introduction)
- [Initial Observations / Out of Scope Contracts](#initial-observations--out-of-scope-contracts)
- [What is a synthetic coin?](#what-is-a-synthetic-coin)
- [What is the bonding process? How is it architecturally achieved?](#what-is-the-bonding-process-how-is-it-architecturally-achieved)
- [Atlantic Perpetual Puts Options](#atlantic-perpetual-puts-options)
- [Role of the AMO (Algorithmic Market Operations Controller)](#role-of-the-amo-algorithmic-market-operations-controller)

## Introduction

This Architectural Review and Feedback section will detail the ways in which the product specification is achieved and where necessary critical feedback will be given to relevant sections of the implemented codebase and systemic market, protocol and smart contract risks will be discussed and analysed. 

The contest brief and product specification provided by the Dopex team describes that: 

“rDPX V2 introduces a new synthetic coin dpxETH which is pegged to ETH. dpxETH will be used to earn boosted yields on ETH and will be a staple collateral token for future Dopex Options Products.

The rDPX bonding process represents the method in which new dpxETH tokens can be minted. When a user bonds with the rDPX V2 contract they receive a receipt token. A receipt token represents ETH and dpxETH LP on curve.

Via the bonding process new dpxETH is minted and its backing is maintained via a rDPX and ETH reserve (the Backing Reserves). These backing reserves are controlled via AMOs. To ensure a safe and controllable way to scale rDPX V2 and dpxETH together we have decided incorporate the AMO ideology from Frax Finance.”

## Initial Observations / Out of Scope Contracts

It should first be noted that in the ‘Contract Architecture’ section of the rDPX v2 Product Specification does not mention the `RdpxV2Bond.sol` contract, which is a key part of the architecture within the bonding process. Although the bonding ‘module’ is referenced in RpdxV2Core.sol contract description, it is suggested that the project team update the Contract Architecture section of the Product Specification to include how the `RpdxV2Bond.sol` contract is used.

It is also observed that the RpdxV2ReceiptToken contract are also out of scope for the contest, it is unclear what the reason for this is due to the Receipt Token also forming a core part of the bonding architecture. Without a security review including this contract, it is not possible to have quality assurance that the rDPX v2 system is operating without significant smart contract risk to the project team and end users.

There is also serious concern with the dpxETH-ETH price oracle being out of scope for the security review, this is covered in depth in the “Attack Vectors to Depeg dpxETH-ETH” section of the Analysis report.

## What is a synthetic coin?

A synthetic coin or synthetic asset refers to a financial instrument that represents the value of another asset, without requiring the holder to own the actual underlying asset. In the case of the new synthetic coin, dpxETH, it represents the current value of ETH. It is noted that, for dpxETH to represent the current value of ETH, a number of mechanisms are required to maintain this ‘peg’. The RED-LOTUS team explored ways in which this peg could be broken or manipulated in the ‘Attack Vectors to Depeg dpxETH-ETH’ section of the Analysis. Please see this section for further discussion.

## What is the bonding process? How is it architecturally achieved?

![https://i.imgur.com/8cZMmty.jpeg](https://i.imgur.com/8cZMmty.jpeg)

The bonding process to mint new tokens is a mechanism used in various DeFi protocols to create new tokens in exchange for providing collateral or locking up existing tokens. In the case of rDPX v2 there are 3 ways a user can bond on the Bonding contract; regular bonding, delegate bonding or using a decaying bond.

**Regular Bonding**

![https://i.imgur.com/TofJMx2.jpeg](https://i.imgur.com/TofJMx2.jpeg)

The first mechanism that can be used to bond is referred to within the rDPX V2 Product Specification as 'regular bonding'. This is architecutrally implemented primarily using the `RdpxV2Core.sol` and `RpdxV2Bond.sol` contracts.

Please see the "Technical Architecture Documents and Diagrams" section of the Analysis report for an in depth technical analysis of the `RdpxV2Core.sol` contract.

The rDPX V2 Product Specification gives an accurate description of the 'regular' bonding process and a detailed step-by-step process of how it is implemented. We do not find any discrepancies with what is described in the Product Specification and the codebase.

It is noted, however, that a number of the variables that are used within the bonding process such as the burn percentage, emission to veDPX holders and the 'toggleable' reLP process are set by the `DEFAULT_ADMIN_ROLE`, related discussion of associated centralization risks can be found in the "Access Control & Centralization Risks" section of the analysis.

**Delegate Bonding**

![https://i.imgur.com/DpSg3BW.jpeg](https://i.imgur.com/DpSg3BW.jpeg)

Similar to the regular bonding process, we do not find discrepancies between the process stated in the rDPX V2 Product Specification and the production codebase. 

**Bonding via the rDPX Decaying Bond**

![https://i.imgur.com/H4Gj7TL.jpeg](https://i.imgur.com/H4Gj7TL.jpeg)


Again, we agree that the process that is documentation in the Product Specification is accurate. However, we do note a specific concern with the manner in which decaying bonds are minted. 

The rDPX V2 Product Specification notes that "Decaying bonds can be minted to users as rebates for losses incurred on the Dopex Options Protocol or for incentives". However, it is not transparent what the criteria or threshold is that would qualify a user for receipt of a decaying bond. Without a clear process in which these criteria are set out, the process may be subject to abuse or manipulation. As such, we are not able to validate if the decaying bond process is secure from an external assurance perspective.

**Risks Associated with the Bonding Mechanism**

A risk scenario that the Project Team may consider is that `isReLPActive` is toggleable and the Admin can change this variable according to the market conditions of rDPX. Imagine a scenario when `isReLPActive` is set to true for a while. Bonding functions call the reLP function every time someone bonds, and according to docs, "reLP is the process by which part of the Uniswap V2 liquidity is removed, then the ETH received is swapped to rDPX. This essentially increases the price of rDPX and also increases the Backing Reserve of rDPX."

So it's safe to assume that the price of rDPX will increase with every bond when `isReLPActive` is set to true. Gradually, the price of rDPX will become a rather high number, which the protocol assume is healthy because the backing reserve of rDPX will increase. Now admin decides to toggle `isReLPActive` to false and it will stay false for a while. In this situation, the price of rDPX will potentially fall, because it was overly inflated before. With this unfortunate free fall of rDPX price, users who have active bonds will suffer a huge amount of loss and protocol will attract negative criticism and become infamous so less users will be willing to bond and use Dopex overall.

## Atlantic Perpetual Puts Options

The use of the Atlantic Perpetual Puts Options and wider Atlantic functionality in the rDPX V2 Product will be critically analysed in this section.

**Atlantic Liquidation Protection**

To first understand the use of Atlantic Perpetual Put Options, we first need to understand the concept of Atlantic Liquidation Protection. Dopex created a feature named "Atlantic Liquidation Protection" which allows users to hedge collateral in support of their position, via the means of purchasing "Atlantic puts to prevent liquidations on up to 10x leveraged long positions on $ETH". 

On the open market, if a leverage trader has a position open and their long-term analysis is correct (yet short-term volatility is at play) then they risk being liquidated due to the negative price movements of the underlying asset. Therefore, the Atlantic Liquidation Protection has the ability to save the trader from liquidation through position under the condition that the trader bought an Atlantic Perpetual Put Option.

However, based on our understanding, it seems that when a user enters a long leveraged trade and purchases the Atlantic Liquidation Protection, the current Dopex documentation fails to acknowledge that if the market overall is experiencing negative and volatile price action, then the user can still be liquidated due to the fact that their hedged collateral is losing value. As such, the Atlantic Put that they purchased for their long leveraged trade will be liquidated causing a more significant loss.

We deem it somewhat misleading to suggest to the end-user that they are fully protected from liquidation, we advise to provide more clarity within the documentation as to how the Atlantic Liquidation Protection will indeed protect the trader from liquidation in certain scenarios, and when the trader is still at risk of liquidation. This clarity would provide reputational assurance for the Dopex protocol as end-users would be fully aware of their risk exposure.

**Critical Analysis of PerpetualAtlanticVaultLP.sol**

Careful analysis of the vault system which Dopex utilises for their Perpetual Atlantic Options generated a few concerns in regards to:

1. The overall usage of variables (in the context of variables like `example` versus `_example`)
2. Dopex's ability to set the correct addresses in relation to oracles
3. Accurate accounting in relation to depositing, withdrawing and redeeming

There was also initial confusion relating to the collateral which is utilised at different times within the Perpetual Atlantic Vaults. Upon further investigation and communication with the Project Team we discovered that: 

- `WETH` is the main token of collateral
- `uint256 _totalCollateral` is **all** `WETH` collateral in the LP vault
- `uint256 _activeCollateral` consists of **all** `WETH` locked collateral in the LP vault
- `uint256 _rdpxCollateral` is **all** `rDPX` collateral in the LP vault

**Usage of Variables**

Regarding point 1, an instance of this is the variable `collateralSymbol` which holds a string based on the input from the constructor variable `_collateralSymbol`, but `collateralSymbol` is never actually utilised, whereas `_collateralSymbol` is on line 104.
```
[L100] collateralSymbol = _collateralSymbol;
...
[L104] symbol = string.concat(_collateralSymbol, "-LP");
```

Ultimately, assigning `_collateralSymbol` to `collateralSymbol` is useless and it is unnecessary gas spent UNLESS it is used accurately and appropriately. 

However, upon a deeper dive into the Atlantic Perpetual Vault contracts, it was revealed that the accurate logic is used within `PerpetualAtlanticVault.sol` as shown below:

```
constructor(
	string memory _name,
	string memory _symbol,
	address _collateralToken,
	uint256 _gensis
) ERC721(_name, _symbol) {
	_validate(_collateralToken != address (0), 1);

	collateralToken = IERC20WithBurn(_collateralToken);
	underlyingSymbol = collateralToken.symbol();
	collateralPrecision = 10 ** collateralToken.decimals();
	genesis = _genesis;
}
```

**Setting Oracle Addresses**

Regarding point 2, an example of this is `_assetPriceOracle` which is responsible for giving the correct asset price. Of course, this variable is manually set by the `setAddresses` function within the `PerpetualAtlanticVault.sol` as shown below:

```
function setAddresses(
	address _optionPricing,
	address _assetPriceOracle,
	address _volatilityOracle,
	address _feeDistributor,
	address _rdpx
	address _perpetualAtlanticVaultLP,
	address _rdpxV2Core
)   external onlyRole(DEFAULT_ADMIN_ROLE) {
		...
	}
)
```

Granted, there are checks for every single input within the function which are validated through the `_validate` function shown below:

```
function _validate(bool _clause, uint256 _errorCode) private pure {
	if (!_clause) revert PerpetualAtlanticVaultError(_errorCode);
}
```

However, there is no check that the addresses are actually correct or whether they even exist, which means that if a "human error" scenario were to occur, then the protocols credibility can be at stake, especially if we are linking this to protocol contract upgrades. Furthermore, the Dopex ecosystem will function based off of invalid oracles, which is not a great user experience nor will it allow the protocol to function as intended; ultimately damaging its credibility and integrity in the long term. To further strengthen this point, we would like to add a report from [Spearbit](https://solodit.xyz/issues/oraclev1getmemberreportstatus-returns-true-for-non-existing-oracles-spearbit-liquid-collective-pdf) on the protocol Liquid Collective, where Spearbit reported a value being returned as "true" for non-existing oracles. Even though the scenario is a little different here, the lack of oracle checking problem still exists and needs to be eliminated.

The same problem also exists within the core contract, `RdpxV2Core.sol`, in relation to the following function:
```
function setPricingOracleAddresses(
	address _rdpxPriceOracle,
	address _dpxEthPriceOracle,
)   external onlyRole(DEFAULT_ADMIN_ROLE) {
	_validate(_rdpxPriceOracle != address(0), 17);
	_validate(_dpxEthPriceOracle != address(0), 17);
		...
)
```

**Accurate Account Keeping**

In relation to point 3, we deemed the foundation of the `deposit()` function to have a well known security vulnerability pertaining to the ERC4626 Vault behaviour, also known as the inflation attack.  

It was stated in communications with the Project Team that the vault is "similar to 4626 in how it works but doesn't follow it". As such, the vault mechanics are not actually completely ERC4626, but simply behave like it. 

![https://i.imgur.com/46knTva.png](https://i.imgur.com/46knTva.png)

The vulnerability lies in how `assets` are deposited and exchanged for `shares`. The reason is due to vaults following the ERC4626 standard which are prone to inflation attacks. An inflation attack is where the exchange rate of assets which are deposited and ERC20 shares which are minted in return are not exchanged in a sufficient manner, due to the fact that the user whom deposited funds into the vault first was essentially sandwich attacked. If the pool does not have a minimum deposit amount (which we could not find within this code) then realistically speaking, upon a clear vault creation, the assets are a 1:1 rate and the first depositor can be manipulated by an attacker performing a sandwich attack, conclusively manipulating the exchange rate. 

It is known that when a vault contains collateral, then the balance between the assets and the shares reflects the vaults exchange rate. After the team spent a few days attempting to compromise the function (along with analysis of the behaviour of `convertToShares()`, `previewDeposit()` and `redeem()`) it was deemed that the function may be vulnerable, however we could not successfully exploit it. We still believe that the function needs to be deep-dived into with more time and resource, because the code logic in its current form demonstrates that an inflation attack is viable.

In addition to the analysis provided here, please see the "Technical Architecture Documents and Diagrams" for a detailed technical analysis of the `PerpetualAtlanticVaultLP.sol` contract.

**Critical Analysis of PerpetualAtlanticVault.sol**

Our review of PerpetualAtlanticVault identified a number of centralization and governance associated risks. These concerns were reported as audit findings within the Code4Arena contest. Although we understand that the Project Team and Admin is trusted it is still necessary to discuss potential risks and mitigations that can be implemented. We deep dive into this topic with the "Access Control & Centralization Risks" section of the Analysis report.

We observe that an `emergencyWithdraw` function has been implemented that will allow the Admin to withdraw all funds from the contract. We observe that this function is present in a number of contract in the rDPX V2 codebase. As aforementioned, we deep dive into this topic and available mitigations in the "Access Control & Centralization Risks" section of the Analysis report.

Furthermore, whilst the implementation of `updateFundingDuration` is in line with the intended business logic of the protocol, the wallet address which has the DEFAULT_ADMIN_ROLE will have the ability to update the funding duration along with a custom uint256 input at any time. We highlight this because in times of unprecedented market events, causing extreme volatility, the funding duration can be manually altered for for malicious purposes. 

```solidity

function updateFundingDuration(
	uint256 _fundingDuration
) external onlyRole(DEFAULT_ADMIN_ROLE) {
  fundingDuration = _fundingDuration;
}

```

We recommend that this function should be operated by governance proposals and support decentralized community decision making. However, we note that this is a architectural decision for the Project Team to decide on.

## Role of the AMO (Algorithmic Market Operations Controller)

It is noted that Dopex states in the rDPX V2 Product Specification ([https://dopex.notion.site/rDPX-V2-RI-b45b5b402af54bcab758d62fb7c69cb4#a7503a215a984244b3a54021259f6ae9](https://www.notion.so/rDPX-V2-RI-b45b5b402af54bcab758d62fb7c69cb4?pvs=21)) that “To ensure a safe and controllable way to scale rDPX V2 and dpxETH together we have decided incorporate the AMO ideology from Frax Finance ([https://docs.frax.finance/amo/overview](https://docs.frax.finance/amo/overview)).” However, it must be critically assessed to what extent the rDPX implementation actually mirrors the Frax ideology and codebase.

It is noted that there are 4 key properties to an AMO:

**Decollateralize** - the portion of the strategy which lowers the CR (Collateralisation Ratio)

**Market operations** - the portion of the strategy that is run in equilibrium and doesn't change the CR

**Recollateralize** - the portion of the strategy which increases the CR

**FXS1559** - a formalized accounting of the balance sheet of the AMO which defines exactly how much FXS can be burned with profits above the target CR.

It is noted from the Frax Finance documentation that the AMO is designed to serve as an “autonomous contract(s) that enacts arbitrary monetary policy so long as it does not change the FRAX price off its peg” and that Frax AMO’s serve as a complete "mechanism-in-a-box”. 

However, the RED-LOTUS-TEAM observes that the Dopex implementation of the AMO differs significantly from the Frax codebase and ideology.

**Decollateralize / Recollateralize**

Firstly, decollateralize and recollateralize operations are not contained within the AMO contracts, they are implemented in the form of the `upperDepeg` and `lowerDepeg` functions in the `RpdxV2Core.sol`. Although we observe this as implementing two of the core elements of the Frax AMO ideology it should be noted that the separation of functions from a single contract can be seen to contradict the Frax ideology of being a "mechanism in a box". Furthermore, it should be noted that the Frax implementation contains a timelock security mechanism that is not present within the Dopex implementation.

**FXS1559**

Furthermore, it is stated in the rDPX V2 Prthe FXS1559 is stated to be incorporated by “the mechanism by which rDPX would be burned with profits above the CR this too would remain a part of the Core Contract.” Upon first analysis of the RpdxV2Core.sol contract, it is unclear where this mechanism for burning profits above the CR is implemented. As such, the RED-LOTUS team reached out to the project team to clarify. The following response was received: “no we don’t use FXS1559”. 

![https://i.imgur.com/yiIauus.jpeg](https://i.imgur.com/yiIauus.jpeg)

This is another observed example of the rDPX V2 codebase not matching the supplied product documentation. At best this is confusing for auditors/external assessors and in the worst case could be seen as misleading to end-users regarding the security / infrastructural controls that are in place to manage risk. 

**Market Operations / reLP Incorporation**

It is observed that the 'Market Operations' element of the AMO ideology of FRAX is stated to be implemented within the `UniswapV2LiquidityAmo.sol` and `UniswapV3LiquidityAmo.sol`. This is as the rDPX V2 Product Specification describes, "The AMO contracts in rDPX V2 will solely be responsible to perform market operations". The Frax ideology states that relevant contracts "enact arbitrary policy so long as it does not change the FRAX price off its peg". Applied to rDPX V2, there are a number of factors to consider regarding if this element of the ideology is truly implemented.

Firstly, although the `UniswapV2LiquidityAmo.sol` and `UniswapV3LiquidityAmo.sol` does run a market operation strategy in regards to the process of adding and removing liquidity to Uniswap V2/V3 pools and then collecting fees - the `reLPContract.sol` is "applied on the Uniswap V2 liquidity added by the Uniswap V2 AMO". As such, it can be observed that portion of the market operations strategy is run outside of the AMO contracts and may certainly jeopardize the continued stability of dpxETH-ETH peg. This again demonstrates a differing implementation than the Frax "mechanism in a box" ideology.

This is due to the strategy of removing liquidity from the Uniswap V2 pool, swapping ETH for rDPX, which lead to a higher valuation of rDPX. As a result, less rDPX will be needed for users to bond and mint dpxETH. This will in turn increase the likelihood that the dpxETH-ETH will depeg and it will be neccessary to recollateralize (`lowerDepeg`). Usage of `lowerDepeg` will deplete backing reserves, leading to instability of the wider rDPX V2 product.

It is also noted that the recollateralize function is part of the 'Peg Stability Module' of which serious concerns of its efficacy are critically analysed in the "Attack Vectors to Depeg dpxETH-ETH".

Furthermore, it is stated in the rDPX V2 Product Specification that the reLP process is "toggleable" and "may or may not be used on every bond execution depending on the market condition of rDPX". However, these market condition criteria are not set out or even discussed in the documentation. As such, end-users are not aware of their true risk exposure. We direct the reader to the "Market and Protocol Efficacy Risk Scenarios" section of the Analysis report where we deeper dive into specific scenarios of market volatility and the risk exposure presented to the system and end-users. 

# Attack Vectors to Depeg dpxETH-ETH

It is noted that the Project Team have requested that Wardens specifically try to focus on breaking the dpxETH - ETH peg. As such, the RED-LOTUS team spent significant time and resource to explore different vectors in which it may be possible to compromise the peg or re-peg functionality. In summary, we have analysed that there are serious architectural and security concerns with the implementation of the ‘Peg Stability Module’.

1. **Introduction**

The ‘Peg Stability Module’ is contained within the RpdxV2Core.sol contract, comprising of the three core functions below:

- ***upperDepeg*** (if dpxETH > 1 ETH)**:** Mints new dpxETH to sell on the curve pool to bring back the peg to 1 ETH.

```solidity
function upperDepeg(
    uint256 _amount,
    uint256 minOut
  ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256 wethReceived) {
    _isEligibleSender();

    _validate(getDpxEthPrice() > 1e8, 10);

    IDpxEthToken(reserveAsset[reservesIndex["DPXETH"]].tokenAddress).mint(
      address(this),
      _amount
    );

    // Swap DpxEth for ETH token
    wethReceived = _curveSwap(_amount, false, true, minOut);

    reserveAsset[reservesIndex["WETH"]].tokenBalance += wethReceived;

    emit LogUpperDepeg(_amount, wethReceived);
  }
```

- ***lowerDepeg*** (if dpxETH < 1 ETH)**:** The backing reserves are used to buy and burn dpxETH from the curve pool to bring back the peg to 1 ETH.

```solidity
function lowerDepeg(
    uint256 _rdpxAmount,
    uint256 _wethAmount,
    uint256 minamountOfWeth,
    uint256 minOut
  ) external onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256 dpxEthReceived) {
    _isEligibleSender();
    _validate(getDpxEthPrice() < 1e8, 13);

    uint256 amountOfWethOut;

    if (_rdpxAmount != 0) {
      address[] memory path;
      path = new address[](2);
      path[0] = reserveAsset[reservesIndex["RDPX"]].tokenAddress;
      path[1] = weth;

      amountOfWethOut = IUniswapV2Router(addresses.dopexAMMRouter)
        .swapExactTokensForTokens(
          _rdpxAmount,
          minamountOfWeth,
          path,
          address(this),
          block.timestamp + 10
        )[path.length - 1];

      reserveAsset[reservesIndex["RDPX"]].tokenBalance -= _rdpxAmount;
    }

    // update Reserves
    reserveAsset[reservesIndex["WETH"]].tokenBalance -= _wethAmount;

    dpxEthReceived = _curveSwap(
      _wethAmount + amountOfWethOut,
      true,
      true,
      minOut
    );

    IDpxEthToken(reserveAsset[reservesIndex["DPXETH"]].tokenAddress).burn(
      dpxEthReceived
    );

    emit LogLowerDepeg(_rdpxAmount, _wethAmount, dpxEthReceived);
  }
```

- ***settle:*** When any rDPX APPs can be exercised, they are exercised to bring back the peg and back the backing reserves more in ETH.

```solidity
function settle(
    uint256[] memory optionIds
  )
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 amountOfWeth, uint256 rdpxAmount)
  {
    _whenNotPaused();
    (amountOfWeth, rdpxAmount) = IPerpetualAtlanticVault(
      addresses.perpetualAtlanticVault
    ).settle(optionIds);
    for (uint256 i = 0; i < optionIds.length; i++) {
      optionsOwned[optionIds[i]] = false;
    }

    reserveAsset[reservesIndex["WETH"]].tokenBalance += amountOfWeth;
    reserveAsset[reservesIndex["RDPX"]].tokenBalance -= rdpxAmount;

    emit LogSettle(optionIds);
  }
```

2. **Availability of upperDepeg and lowerDepeg functions**

It is a key feature of the upper and lower Depeg functions that they are usable by end-users of the protocol, this is specified in both the natspec comment for the `upperDepeg` function on L1047 - “@notice Lets users mint DpxEth at a 1:1 ratio when DpxEth pegs above 1.01 of the ETH token”. It is also described in documentation supplied by Dopex “rDPX v2 for the mentally challenged” ([https://blog.dopex.io/articles/dopex-papers/rdpx-v2-for-the-mentally-challenged](https://blog.dopex.io/articles/dopex-papers/rdpx-v2-for-the-mentally-challenged)) as in the event where dpxETH is above the 1:1 peg to ETH, “Users can mint $dpxETH 1:1 with $ETH. This will allow them to swap the $dpxETH in the pool and profit the difference while returning the pool to a balanced state.” 

It can be observed in the `upperDepeg` function that the `onlyRole` modifier has been set to only allow `DEFAULT_ADMIN_ROLE` to call this function. This implementation completely mismatches the intended implemention described in the documentation and also is not in line with the functionality required by the protocol in the case that a depeg event occurs and users are required to mint and sell dpxETH in the curve pool to rebalance the dpxETH-ETH peg to 1:1.

The `upperDepeg` function also contains the `_isEligibleSender` function modifier, code snippet contained below:

```solidity
 function _isEligibleSender() internal view {
    // the below condition checks whether the caller is a contract or not
    if (msg.sender != tx.origin)
      require(whitelistedContracts[msg.sender], "Contract must be whitelisted");
  }
```

However, it can be observed that this modifier is completely irrelevant due to the `onlyRole` modifier already being set to `DEFAULT_ADMIN_ROLE` . As such, end-users of the protocol would not be able to utilize this function creating an availability risk for a key function of the protocol where users are incentivized to restore the dpxETH-ETH peg through an arbitrage opportunity. Furthermore, even if the onlyRole modifier were not set to `DEFAULT_ADMIN_ROLE` , setting the `_isEligibleSender` to only allow whitelisted contracts to call the function would also limit the scope of availability of this function to a degree which may compromise the intended use-case to allow end-users/market participants to restore the dpxETH-ETH peg.

Similarly, the `lowerDepeg` function is documented in “rDPX v2 for the mentally challenged” ([https://blog.dopex.io/articles/dopex-papers/rdpx-v2-for-the-mentally-challenged](https://blog.dopex.io/articles/dopex-papers/rdpx-v2-for-the-mentally-challenged)) that the intention is to be “called by veDPX holders which will allows it to remove $dpxETH from the Curve pool until the pool is balanced”. Once again the `onlyRole` modifier has been set to `DEFAULT_ADMIN_ROLE` and the irrelevant `_isEligibleSender` is applied also. The impact being end-users or the targeted veDPX holders would not be able to call this function. 

**Discussion with Dopex Project Team / RED-LOTUS Recommendation**

At this point the RED-LOTUS team reached out to the Dopex Development team to understand how they architecturally planned to make the upper and lower depeg (repeg) functions available to end-users and veDPX holders with the the `onlyRole` modifier applied. We recieved the response below.

![https://imgur.com/a/PiDUBnp](https://i.imgur.com/nWX8SfM.png)

Although we understand the communications from the Project Team “that this was the old plan and has since been updated where operations can only be performed by the admin”, it is inadequate from an assurance perspective as the natspec and protocol documentation have not been updated to match the updated code. This is at best confusing for any external assurance or audit personel when reviewing, and at worst could be seen as misleading to end-users of the protocol when understanding the scope and incentives that are available to restore the dpxETH-ETH peg in a depeg scenario. 

In any case, the `_isEligibleSender` function modifier that is included in both upper and lower depeg functions is irrelevant if the planned implementation is to only allow the `DEFAULT_ADMIN_ROLE` to call the function. As such, this unused code that could lead to the function reverting if the admin address has not been whitelisted, should be removed. 

It is our recommendation that the `upperDepeg` and `lowerDepeg` functions are made public to incentivise users with the arbitrage opportunity to restore the dpxETH-ETH peg.

Furthermore, the relevant natspec in L1047 should be updated and the Dopex documentation “rDPX v2 for the mentally challenged” ([https://blog.dopex.io/articles/dopex-papers/rdpx-v2-for-the-mentally-challenged](https://blog.dopex.io/articles/dopex-papers/rdpx-v2-for-the-mentally-challenged)) should also be updated to reflect the current production implementation.

3. **Flash Loan Attack / Token Price Manipulation Vectors**

Malicious use of flashloans is an ever present attack vector and for effective protection, projects must implement robust relevant security controls. It is noted that the `_curveSwap` function in RpdxV2Core.sol may be vulnerable to flash loan attacks. As a consequence, the dpxETH-ETH peg could be compromised.

```solidity
function _curveSwap(
    uint256 _amount,
    bool _ethToDpxEth,
    bool validate,
    uint256 minAmount
  ) internal returns (uint256 amountOut) {
    IStableSwap dpxEthCurvePool = IStableSwap(addresses.dpxEthCurvePool);

    // First compute a reverse swapping of dpxETH to ETH to compute the amount of ETH required
    address coin0 = dpxEthCurvePool.coins(0);
    (uint256 a, uint256 b) = coin0 == weth ? (0, 1) : (1, 0);

    // validate the swap for peg functions
    if (validate) {
      uint256 ethBalance = IStableSwap(addresses.dpxEthCurvePool).balances(a);
      uint256 dpxEthBalance = IStableSwap(addresses.dpxEthCurvePool).balances(
        b
      );
      _ethToDpxEth
        ? _validate(
          ethBalance + _amount <= (ethBalance + dpxEthBalance) / 2,
          14
        )
        : _validate(
          dpxEthBalance + _amount <= (ethBalance + dpxEthBalance) / 2,
          14
        );
    }

    // calculate minimum amount out
    uint256 minOut = _ethToDpxEth
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
  }
```

This vulnerability lies in the `getDpxEthPrice` function. The `getDpxEthPrice` function is a key part of the architecture of the `_curveSwap` implementation to ensure correct amounts of tokens are swapped from the dpxETH curve pool.

Arguably, the most effective security control that can be implemented to protect against flash loan attacks is use of a manipulation resistant oracle. It is noted that the `getDpxEthPrice` is a function emanating from the out of scope dpxETH-ETH Oracle contract for the contest. As such, it cannot be validated that the Oracle is manipulation resistant and not susceptible to flash loan attacks.

However, it is noted that the `_curveSwap` function can only be called by Admin roles and as such the vulnerability is significantly mitigated. Nonetheless, the centralized and permissioned manner in which this business operation is implemented again presents significant risk and to end-users, as this is another method in which a malicious insider could exploit privileged roles to inflate the price of the dpxETH token and sell on the open market.

**3.1 Flash Loan attacks on `Bond` Function**

Upon further investigation, the RED-LOTUS team explored whether the `bond` function could also be susceptible to price manipulation through the malicious use of flashloans.

The following attack chain steps were explored, to determine if the vector was viable to exploit the protocol due to a manipulation resistant oracle currently not being implemented.

1. The attacker first flashloans and sells a huge amount of `WETH` to a rDPX/WETH liquidity pool. The pool now thinks `rDPX` is extremely valuable.
2. The attacker now uses a relatively small amount of `rDPX` to bond using `bond` (code snippet below). The bond cost is calculated with the `calculateBondCost` function (code snippet also below), which also relies on an Oracle contract (rdpxPriceOracle) to retrieve the `rDPX` price. Although it is not stated that this Oracle contract is specifically out of scope, it is not included in the contest repository and as such we are not able to validate if the Oracle is manipulation resistant. If not resistant, the pool would think `rDPX` is very valuable the attacker will be able to bond at a extremely high discount.

```solidity
function bond(
    uint256 _amount,
    uint256 rdpxBondId,
    address _to
  ) public returns (uint256 receiptTokenAmount) {
    _whenNotPaused();
    // Validate amount
    _validate(_amount > 0, 4);

    // Compute the bond cost
    (uint256 rdpxRequired, uint256 wethRequired) = calculateBondCost(
      _amount,
      rdpxBondId
    );

    IERC20WithBurn(weth).safeTransferFrom(
      msg.sender,
      address(this),
      wethRequired
    );

    // update weth reserve
    reserveAsset[reservesIndex["WETH"]].tokenBalance += wethRequired;

    // purchase options
    uint256 premium;
    if (putOptionsRequired) {
      premium = _purchaseOptions(rdpxRequired);
    }

    _transfer(rdpxRequired, wethRequired - premium, _amount, rdpxBondId);

    // Stake the ETH in the ReceiptToken contract
    receiptTokenAmount = _stake(_to, _amount);

    // reLP
    if (isReLPActive) IReLP(addresses.reLPContract).reLP(_amount);

    emit LogBond(rdpxRequired, wethRequired, receiptTokenAmount);
  }
```

1. The attacker can now manipulate the pool in the opposite direction by selling the `dpxETH` they minted on the open market.
2. As `rDPX` is now considered much less valuable than at the point the surplus `dpxETH` was minted it takes a lot more of `rDPX` to bond new `dpxETH`.

For the price of a flashloan and some swap fees, the attacker has now managed to artificially bond/extract a large amount of surplus `dpxETH`. This process can be repeated as long as it is profitable.

Again, we reinforce that this is a hypothetical attack scenario and without examining the production price oracles for dpxETH and rDPX/WETH, we are unable to validate if the protocol is effectively protected.

The RED-LOTUS-TEAM communicated our concerns to the Project Team and it was stated in response that “the oracles are a 30min twap oracle so its not possible to inflate the price in atomic transactions and we would be moving to chainlink once the oracle is live.” 

![https://i.imgur.com/Ge5Xekf.jpeg](https://i.imgur.com/Ge5Xekf.jpeg)

Although this project plan would seem to mitigate the security risk from malicious flash-loans it must be reiterated that this cannot be validated due to the dpxETH-ETH and rDPX/WETH oracles being out of scope for the contest. We feel that as responsible security reviewers, our concerns should be noted and at the minimum a statement from the Project Team must be given that the appropriate protections will be put in place.

Furthermore, the use of a 30 min TWAP oracle is also not without risk and by itself does not mitigate the threat of price manipulation. Referencing academic analysis from "TWAP Oracle Attacks: Easier Done than Said?" written by Mackinga, Nadahalli and Wattenhofer (https://eprint.iacr.org/2022/445.pdf), although using TWAP oracles forces price manipulation and de-manipulation steps into different blocks and flash-loan attacks are no longer an option, manipulation can still occur. 

With the Oracles being out of scope, we will not delve into a deep analysis of the risks associated with TWAP Oracles but strongly suggest that the Project Team reviews relevant security resources on this topic and end-users are made aware that the Oracles have not been reviewed as part of the C4 contest.

4. **Atlantic Perpetual Puts cannot be settled**

The final element of the Peg Stability Module is to allow when rDPX APPs can be exercised, they are exercised to bring back the peg and back the backing reserves more in ETH. This is carried out through calling the `settle` function in `RdpxV2Core.sol` (code snippet contained in first section above). However, it will be demonstrated that due to an underlying issue within `PerpetualAtlanticVaultLP.sol` it will not be possible to settle this option contract and as such the use of `settle` as a component of the Peg Stability Module, is rendered completely ineffective.

The execution flow is as follow:

1- Admin calls `settle(uint256[] memory optionIds)`  inside `RdpxV2Core.sol`

2- `RdpxV2Core.sol` calls `settle(uint256[] memory optionIds)` inside `PerpetualAtlanticVault.sol`

3- `PerpetualAtlanticVault.sol` calls `subtractLoss(uint256 loss)` inside `PerpetualAtlanticVaultLP.sol`.
`subtractLoss` is the vulnerable function (code snippet below).

```solidity
function subtractLoss(uint256 loss) public onlyPerpVault {
    require(
      collateral.balanceOf(address(this)) == _totalCollateral - loss,
      "Not enough collateral was sent out"
    );
    _totalCollateral -= loss;
  }
```

There's a require statement, that enforces that the balance of collateral inside the smart contract, is equal to the one recorded inside the _totalCollateral global variable minus the loss.

Because there's no function or way to sync the balance of ERC20 collateral in the PerpetualAtlanticVaultLP.sol contract with the global variable _totalCollateral,  using the ERC20 functions of the collateral token to directly transfer 1 wei to the PerpetualAtlanticVaultLP.sol will break this assumption and revert in the subsctractLoss(...) function.

As aforementioned, due to this underlying development error within the `settle` function it will not be possible to settle the option and as such, this core element of the Peg Stability Module is rendered ineffective.

5. ********************Conclusion********************

In conclusion, it can be observed that there are serious architectural and security concerns with different elements of the Peg Stability Module. We suggest that the Project Team revisit their decision to permission and restrict the usage of `upperDepeg` and `lowerDepeg` functions to Admin only. It is also critical that the relevant price Oracles are submitted for a thorough security review before going into production. Finally, the issues with the `settle` function for Atlantic Perpetual Puts must be resolved to allow the effective operation of the Peg Stability Module.

# Discussion of Access Control & Centralization Risks

## Introduction  

A number of core and business critical functions are set with the `onlyRole` modifier so that only the `DEFAULT_ADMIN_ROLE` can operate them. It will be discussed that operational security and centralization risks associated with this implementation should be considered. 

Within this section of the Analysis report, several methods that would be available to malicious insiders to 'rug' the rDPX V2 project will also be considered. This is not to state that we believe that the Project Team has malicious intentions but rather to record and communicate concerns that responsible security researchers are obliged to report when reviewing code. We will give detailed descriptions of mitigating controls that can be implemented to give reassurance to end-users of the protocol that their funds are safe.

Furthermore, it will be discussed that due to the lack of an implemented renounce function an availability risk is also presented to the rDPX V2 'module' and wider Dopex protocol. Critical analysis regarding how the OpenZeppelin `AccessControlDefaultAdminRules.sol` has not been inherited and/or implemented to secure to the `DEAFULT_ADMIN_ROLE` will be given.

We reference the finding from the Frax Ether Liquid Staking Contest (https://github.com/code-423n4/2022-09-frax-findings/issues/107) where it was judged that elavated admin privileges without appropriate relevant mitigating controls was deemed to be a High risk.

## Use of OpenZeppelin's `AccessControl.sol`

The `_setupRole` function inherited from in OpenZeppelin's `AccessControl.sol` can be observed within the constructors of `RdpxV2Core.sol`, `RdpxV2Bond.sol`, `UniV2LiquidityAmo.sol` and `UniV3LiquidityAmo.sol` contracts. An example from `RdpxV2Core.sol` is shown in the code snippet below.

```solidity

constructor(address _weth) {

    _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);

    weth = _weth;

```

For information on how the `_setupRole' operates, please see the code snippet from the OZ library.

```solidity

function _setupRole(bytes32 role, address account) internal virtual {

        _grantRole(role, account);

    }

```

It should be noted for analysis purposes that OZ have commented that this function has been deprecated in favor of ‘_grantRole’, code snippet contained below.

```solidity

function _grantRole(bytes32 role, address account) internal virtual {

        if (!hasRole(role, account)) {

            _roles[role].members[account] = true;

            emit RoleGranted(role, account, _msgSender());

        }

    }

```

As aforementioned, there does not appear to be any ownership change function implemented at all in the `RpdxV2Core.sol` / `RpdxV2Bond.sol` / `UniV2LiquidityAmo.sol` / `UniV3LiquidityAmo.sol` contracts. Even if there was such a function, it is advised that best practice is followed to implement a robust two-step process to ensure that the contract is not DoS’d. The scenario in which the relevant contracts would become DoS'd is if/when the externally owned account that has the `DEFAULT_ADMIN_ROLE` becomes compromised it will not be possible to change ownership.

The following ‘rennounceRole’ function from the OZ AccessControl.sol could be utlised.

```solidity

function renounceRole(bytes32 role, address account) public virtual override {

        require(account == _msgSender(), "AccessControl: can only renounce roles for self");

  

        _revokeRole(role, account);

    }

```

However, please note that this a one-step ownership change. It is specifically described in the OpenZeppelin `AccessControl.sol` regarding the DEFAULT_ADMIN_ROLE that: “Extra precautions should be taken to secure accounts that have been granted it. We recommend using {AccessControlDefaultAdminRules} to enforce additional security measures for this role.

It can be observed that the relevant contracts do not inherit from the `AccessControlDefaultAdminRules.sol`, if they did utilize this contract, they would be able to use the more robust ownership change process contained in the code snippet below:  

```solidity

function renounceRole(bytes32 role, address account) public virtual override(AccessControl, IAccessControl) {

        if (role == DEFAULT_ADMIN_ROLE && account == defaultAdmin()) {

            (address newDefaultAdmin, uint48 schedule) = pendingDefaultAdmin();

            require(

                newDefaultAdmin == address(0) && _isScheduleSet(schedule) && _hasSchedulePassed(schedule),

                "AccessControl: only can renounce in two delayed steps"

            );

            delete _pendingDefaultAdminSchedule;

        }

        super.renounceRole(role, account);

    }

```

## Specific Centralized / Business Critical Functions of Concern

One of the functions of specific concern with the `onlyRole` modifier set to `DEFAULT_ADMIN_ROLE` is `emergencyWithdraw` contained within the `RdpxV2Core.sol`, `UniV2LiquidityAmo.sol`, `RpdxDecayingBonds.sol` and `PerpeturalAtlanticVault.sol` contracts, code snippet contained below.

```solidity

function emergencyWithdraw(

    address[] calldata tokens

  ) external onlyRole(DEFAULT_ADMIN_ROLE) {

    _whenPaused();

    IERC20WithBurn token;

  

    for (uint256 i = 0; i < tokens.length; i++) {

      token = IERC20WithBurn(tokens[i]);

      token.safeTransfer(msg.sender, token.balanceOf(address(this)));

    }

  

    emit LogEmergencyWithdraw(msg.sender, tokens);

  }

```

After discussion with the Project Team regarding our concerns over this critical function, which could be seen to act as a 'circuit breaker', the team responded that the Admin role is trusted and that a Multisig would be in place for the `DEFAULT_ADMIN_ROLE`. However, it is recommended that details over how the multisig will be implemented including details of trusted parties and measures for adding and removing members need to be robustly detailed in official documentation. 

Furthermore, the Project Team could further reassure users by implementing an escrow contract that could act as an abritrar over any funds that need to be withdrawn in an emergency scenario. This could be a trusted third party or a timelocked governance-run contract.

It will also be considered in the section below, that another critical function within the codebase actually negates the protections offered by a multi-sig.

## Admin can delegate spending to arbitrary spender who can extract all funds

We further observe that although the Project Team has argued that the multi-sig offers the protections required to mitigate centralization risk, the following `approveContractToSpend` functions (code snippet below) contained in `RdpxV2Core.sol` and `UniV2LiquidityAmo.sol` actually removes this protection.

```solidity

 function approveContractToSpend(
    address _token,
    address _spender,
    uint256 _amount
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_token != address(0), 17);
    _validate(_spender != address(0), 17);
    _validate(_amount > 0, 17);
    IERC20WithBurn(_token).approve(_spender, _amount);
  }

```
We reference the audit finding from the previous C4 Illuminate contest (https://solodit.xyz/issues/m-05-centralisation-risk-admin-can-change-important-variables-to-steal-funds-code4rena-illuminate-illuminate-contest-git) where it was judged that a centralization risk in which 'Admin Can Change Important Variables To Steal Funds' was judged as medium risk.

As a non multi-sig contract can be approved to spend tokens removing the protections offered in the first instance. We recommend that decisions to approve contracts to spend tokens are managed by governance proposals.

## Enhanced Risk Exposure for Users utilizing Liquidation Protection

Users pay a premium to be protected from market volatility through the use of Atlantic Perpetual Puts. However, they have no control over the functions which do this `provideFunding` and `settle` within `RdxpV2Core.sol` (code snippets below).

```solidity

 function provideFunding()
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 fundingAmount)

function settle(
    uint256[] memory optionIds
  )
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 amountOfWeth, uint256 rdpxAmount)

```

This is an example of another centralization risk which cannot be classified as a 'standard' centralization risk as users have paid for service but only the Admin is able to implement its functionality.

If users are paying for this service, they should be able to utilize it, as this is the intended use-case of the functions.

## Conclusion

In addition to the function described above there are wide ranging business critical functions throughout the codebase that are set to `DEFAULT_ADMIN_ROLE`. Although the Project Team have stated that the admin in trusted and a multi-sig in place, as described above there are ways in which these protections can be abused. With current statistics showing that around 11 percent of exploits in DeFi projects are due to 'rug-pulls' (https://rentry.co/audit-leaderboard) it must be clearly stated by the Project Team in their documentation, that the rDPX V2 Product is highly centralized and permissioned so end-users are aware of the risk. Furthermore, explaining the protections that are currently and able to be put in place will reassure end-users and protect the Project Team from liability and reputational risk.

# Market and Protocol Efficacy Risk Scenarios

The RED-LOTUS team note that within the official Dopex documentation, the Project Team emphasize how usage of Dopex Products reduce market risk exposure. However, it must be critically analysed how systemic market risks can affect the rDPX V2 Product. This information is not presented within the available documentation supplied by Dopex for the rDPX V2 Product. Namely, the Product Specification listed in the C4 contest brief ([https://dopex.notion.site/rDPX-V2-RI-b45b5b402af54bcab758d62fb7c69cb4](https://dopex.notion.site/rDPX-V2-RI-b45b5b402af54bcab758d62fb7c69cb4)) and rDPX V2 for The Mentally Challenged Primer (https://blog.dopex.io/articles/dopex-papers/rdpx-v2-for-the-mentally-challenged). The RED-LOTUS team have put together various market risk scenarios that should be considered by end-users. It is strongly suggested that the Project Team consider these scenarios and potential mitigating controls and also consider making them available within public Protocol documentation.
## Extreme Volatility Risk

### Risk Description
The protocol incorporates 25% OTM puts to manage moderate levels of market volatility. However, extreme volatility events that involve rapid and substantial asset price fluctuations pose unique challenges that the protocol may not be adequately equipped to handle.

### Probabilistic Scenario
Extreme market volatility can result in frequent triggering of put options. This high rate of option exercise can drain the WETH reserves in the LP in an accelerated manner, jeopardizing both the protocol's stability and its ability to fulfil redemption requests. In parallel, the oscillatory nature of extreme volatility could also lead to situations where asset prices recover quickly after a steep fall, rendering the triggering of puts as unnecessary and imposing an unwarranted economic burden on the protocol.

### Complex Dynamics Under Extreme Volatility
Extreme volatility introduces nonlinear dynamics into the system's risk profile. For example, frequent put option exercises might necessitate dynamic readjustment of collateral ratios, affecting the overall system's stability.

### Mitigating Controls
Real-time volatility indices or circuit breakers could be employed to temporarily halt the protocol during extreme volatility. This would give time for market participants and the protocol itself to readjust their positions.

## Admin-Driven Fee and Rate Adjustments

### Risk Description
Admins have the authority to modify fees and interest rates within the system. This centralization of decision-making poses significant risk, especially during high-volatility or other exceptional market conditions.

### Probabilistic Scenario
Slow or suboptimal adjustments by the admin can affect the protocol's competitive positioning and financial health. In the worst-case scenario, if the admin input values are not sanity-checked, an erroneous or malicious entry could significantly destabilize the system.

### Autonomy vs. Centralization Trade-offs
Admin-driven rate and fee adjustments introduce a governance risk and potentially dilute the system's decentralized nature, creating a vector for centralized failures.

### Mitigating Controls
Decentralized governance to approve fee and rate adjustments, coupled with sanity checks in the smart contract, could reduce the risks associated with admin control.

## Impermanent Loss from Yield Strategies

### Risk Description
The protocol intends to deploy assets into yield-generating strategies. While this is profitable under stable conditions, it exposes the protocol to impermanent loss, particularly when participating in liquidity provision.

### Probabilistic Scenario
Exposure to yield-generating protocols that are susceptible to impermanent loss could lead to an actual reduction in the assets available for backing, thereby increasing the systemic risk in volatile markets.

### Asset-Strategy Mismatch Risk
If the assets are not matched well with the yield strategies, the impermanent loss could be exacerbated, leading to higher-than-expected reductions in asset values.

### Mitigating Controls
Active management and monitoring of yield strategies, perhaps employing strategies that are less prone to impermanent loss or using insurance coverage for the assets, could mitigate this risk.

## Non-Linear Payoff Risks

### Risk Description
The put options in the system offer a nonlinear payoff profile. This makes the risk modeling far more complex than systems with linear payoffs and could potentially add layers of hidden risks.

### Probabilistic Scenario
During a black swan event where multiple risks converge (extreme volatility, admin errors, mass redemptions), the nonlinear payoffs could interact with these factors in unpredictable ways. This makes the risk assessment more complex and could result in unforeseen vulnerabilities.

### Quantitative Complexity
Non-linear payoffs introduce additional mathematical complexities in risk modeling, making the use of traditional linear risk models insufficient for capturing the nuances of the risk profile.

### Mitigating Controls
Advanced stochastic models that incorporate nonlinearity could be employed for more accurate risk assessments. Combining these models with stress testing under various scenarios could offer more robust risk management.

## Option Settlement Risk

### Risk Description
The function `settle` for option settlement is centralized and can only be invoked by an admin. If the admin fails to perform this operation or chooses to act maliciously, it can pose significant risks to option holders.

### Probabilistic Scenario
In a volatile market, options can act as crucial hedging instruments. Failure to settle options in a timely manner during such volatile times could intensify the market risks for option holders. An extreme case could be a market downturn where a large number of put options are not settled. This could lead to a liquidity crisis for the protocol, causing a cascade of liquidations and potentially inducing systemic risk in associated markets.

### Risk Factors in the Vault Contract
The `settle` function in the vault contract (`IPerpetualAtlanticVault`) performs a series of complex operations, including updating funding rates, burning option tokens, and transferring collateral. Given the code's complexity, there are multiple points where the function could potentially fail, leaving the system in an inconsistent state.

#### Over-reliance on Admin Role
The design decision to require an admin role to invoke the `settle` function can result in points of failure. If the admin keys are compromised or if the admin acts against the best interests of the protocol, this could result in incorrect settlements or even a complete halt in settlements.

### Mitigating Controls
One of the mitigating factors would be to introduce a decentralized mechanism for settling options, potentially by leveraging on-chain governance, thus reducing the reliance on a single admin.

## Liquidity Provisioning Risk

### Risk Description
The protocol allows users to deposit assets into a liquidity pool (LP), and they receive LP tokens (`shares`) in return. Similarly, users can redeem assets and associated `rdpxAmount` from the LP. If the LP runs out of assets (`WETH` in this case), then the option settling function will fail, and users will be unable to redeem their shares.

### Probabilistic Scenario
Consider a market crash scenario where a large number of users want to redeem their shares for `WETH` and `rdpxAmount`. If the LP's reserve runs low, the subsequent users may not be able to redeem their shares. This will instill a lack of trust in the protocol and could lead to further depletion of the LP's reserves as users rush to withdraw their remaining assets.

### Undercapitalization Risk
In the worst-case scenario, the LP could become undercapitalized, leading to a systemic risk for users holding both options and LP tokens. This is because when the LP is undercapitalized, it cannot meet its liabilities, i.e., it cannot execute the settle function or allow users to redeem their shares.

### Mitigating Controls
Some of the mitigating steps could include over-collateralization, risk-adjusted returns, and the creation of an insurance fund to handle adverse market conditions.

## Time-to-Expiry in Perpetual Option

### Risk Description
The concept of a "perpetual option" inherently implies that the option does not have an expiry date. However, the code employs a `timeToExpiry` variable, which is contradictory to the nature of a perpetual financial instrument. The existence of this variable raises questions about the actual financial mechanics the code is trying to implement.

### Probabilistic Scenario
If `timeToExpiry` actually influences the premium or any other financial metric in the perpetual option, this would introduce temporal constraints that deviate from the traditional understanding of a perpetual instrument. This misalignment could lead to misunderstandings among users and potentially flawed risk assessments.

### Semantic and Financial Inconsistencies
The term "perpetual" could be misleading if the instrument incorporates temporal constraints, like `timeToExpiry`, that typically apply to standard, time-bound financial derivatives.

### Mitigating Controls
Clarifying the role of `timeToExpiry` and its impact on the perpetual options within the documentation and the code comments can reduce this risk. If `timeToExpiry` has no impact on the financial metrics of the perpetual option, it should be removed to avoid confusion.

## Checks for Required Collateral

### Risk Description
The vault does employ a check for the required collateral by validating against `perpetualAtlanticVaultLp.totalAvailableCollateral()`. Although it adds a layer of security, it introduces a dependency on the liquidity pool's available collateral.

### Probabilistic Scenario
Should the liquidity pool run low on collateral, the vault may be unable to purchase options, affecting the protocol's risk mitigation mechanisms. Such a scenario could affect the protocol's ability to maintain its peg or other financial metrics.

### Liquidity and Solvency Concerns
The vault's ability to write options is directly tied to the liquidity pool's collateral. Should the pool face a liquidity crisis, it could cascade to the vault's functionality.

### Mitigating Controls
Implementing a dynamic monitoring system to keep track of collateral levels and alert the admin or take automatic actions could provide an additional safety net.

# Technical Architecture Documents and Diagrams

# RdpxV2Core Overview

## Concerns
RdpxV2Core is concerned with:

1. Administration of
     - Bonding parameters
     - Pool Funding/Rebalance parameters
     - Contract Whitelist
     - Oracle Addresses
     - Collateral Addresses
     - Protocol Contract Addresses
     - Spending Permissions
     - Pause/Unpause protocol
2. Issuing of Bonds
      - Bonding to addresses
      - Delegation
      - Redemption
3. Maintaining the peg of dpxETH/ETH
      - Providing Funding to the Perpetuals Vault
      - Adjusting the Peg When dpxETH/ETH is Too High
      - Adjusting the Peg When dpxETH/ETH is Too Low

## Roles
The roles involved are:

- Admin (`DEFAULT_ADMIN_ROLE`)
- User (implicit to `msg.sender`)
# Core Business Logic
The core units of business logic are:

1. Bonding
    - Bond with rDPX
    - Bond with dbrDPX
    - Bond with Delegate
2. Adjust (add to) a delegation
3. Withdraw from a delegation
4. Redeem rewards

What follows are informative diagrams that were created to understand the protocol better.
## Bonding

![https://i.imgur.com/8cZMmty.jpeg](https://i.imgur.com/8cZMmty.jpeg)

3 Ways to bond:
- with actual rDPX: `0` as `bondId`
- with dbrDPX: `bondId` is the id of the NFT
- by splitting ETH or rDPX costs with a delegate

Takeaways:
1. Bonding is about collateral
2. Collateral can come in different forms
3. If you have ETH but no rDPX for basic bonding, you can find someone else who does and split the bond
### Bond with rDPX: regular bonding

![https://i.imgur.com/TofJMx2.jpeg](https://i.imgur.com/TofJMx2.jpeg)

### Delegate Bonding

![https://i.imgur.com/DpSg3BW.jpeg](https://i.imgur.com/DpSg3BW.jpeg)

### Bond with a Delegate

![https://i.imgur.com/DpSg3BW.jpeg](https://i.imgur.com/DpSg3BW.jpeg)

### Bonding with Decaying Bond

![https://i.imgur.com/H4Gj7TL.jpeg](https://i.imgur.com/H4Gj7TL.jpeg)

### Major Calls Graphed
#### Bond with a Delegate

![https://i.imgur.com/TuT99uh.jpeg](https://i.imgur.com/TuT99uh.jpeg)

# Adjust (add to) a Delegation

![https://i.imgur.com/Vfkl9dp.jpeg](https://i.imgur.com/Vfkl9dp.jpeg)

# PerpetualAtlanticVaultLP

## Concerns
PerpetualAtlanticVaultLP is concerned with:

1. Administration of Vault Accounting
    -  Increasing/Decreasing Collateral Values
2. Participating in the Vault With Collateral
    - Deposits and Redemption
## Roles
- Perpetual Vault (`onlyPerpVault` modifier)
- User (implicit to `msg.sender`)
## Notes On Collateral 
The LP vault has a complex accounting of collateral. 

The various collateral represent:
- `ERC20 collateral`: on-chain WETH ERC20 instance
- `uint256 _totalCollateral`: all WETH collateral in the LP vault
- `uint256 _activeCollateral`: all [[#Locking collateral|locked]] WETH collateral
- `uint256 _rdpxCollateral`: all [[#rDPX in PAVLP|rDPX]] collateral in the LP vault
- `uint256 totalVaultCollateral`: sum of WETH and [[#rDPX in PAVLP|rDPX]] (priced in WETH) in the vault. *Used in `convertToShares()`*.

# Core Business Logic
The core units of business logic are:
1. Depositing Collateral
2. Redeeming Collateral
3. Disbursing Shares

What follows are informative diagrams that were created to understand the protocol better.

## Depositing Collateral

![https://i.imgur.com/rOBWe8s.jpeg](https://i.imgur.com/rOBWe8s.jpeg)

## Redeeming Collateral

![https://i.imgur.com/Oy6IHSB.jpeg](https://i.imgur.com/Oy6IHSB.jpeg)


### Time spent:
300 hours