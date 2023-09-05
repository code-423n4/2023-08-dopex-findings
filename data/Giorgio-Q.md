## Title
Consider changing variable to Constant

### Details

In the `./PerpetualAtlanticVault.sol` the `roundingPrecision` variable is initialized to 1e6, however this variable could benefit from being set to Constant for clarity since it is never changed. 

#### Affected code

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L103-L104

## Title
Useless Manager Role

### Details

`./PerpetualAtlanticVault.sol` the MANAGER_ROLE is set to msg.sender but never used for throughout the contract, for clarity I would suggest to remove it.

#### Affected code

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L44-L45

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L127-L128

## Title

Redundant Authorization Check in settle and payFunding Functions

### Details

The settle and payFunding functions have a redundant check using _isEligibleSender(), which serves no purpose as the function is already restricted to RDPXV2CORE_ROLE. This could potentially introduce confusion and make the code less maintainable.

#### Affected code

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L315-L324

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L372-L380

## Title

Overcomplicated Calculation for timeToExpiry in Funding Logic

### Details

The calculation for timeToExpiry involves redundant and unnecessarily complicated steps. 

timeToExpiry=nextFundingPaymentTimestamp()−(genesis+((latestFundingPaymentPointer−1)×fundingDuration))

The function nextFundingPaymentTimestamp returns:


`nextFundingPaymentTimestamp()=genesis+(latestFundingPaymentPointer×fundingDuration)`

Substituting this value into the first equation gives:
```
timeToExpiry=(genesis+(latestFundingPaymentPointer×fundingDuration))−(genesis+((latestFundingPaymentPointer−1)×fundingDuration))
```
First, expand and simplify the terms:

```
timeToExpiry =
genesis + latestFundingPaymentPointer × fundingDuration − genesis − latestFundingPaymentPointer
× fundingDuration + fundingDuration

```

The terms involving genesis and latestFundingPaymentPointer * fundingDuration effectively cancel each other out:
```
timeToExpiry = fundingDuration 
```

Hence, the value of timeToExpiry simplifies to just fundingDuration. Given this, the code could be simplified to directly use fundingDuration instead of the complex expression.

#### Affected code

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L426-L427





## Title

Inconsistent Naming Convention for Internal Function beforeWithdraw

### Details

The beforeWithdraw function is marked as internal but does not adhere to the common naming convention of prefixing internal functions with an underscore (_).

#### Affected code

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L286-L292


## Title

Unused parameter

### Details

The function contains an unused parameter uint256 /*shares*/.
Consider removing it.

#### Affected code

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L286-L292 



## Title

Weak Natspec

### Details

Consider further detailing natspecs on key function such as `reLP` to increase the readability, transparency of the code.

#### Affected code

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L198-L201



## Title

Unused burnFrom function 

### Details

Useless burnFrom function better be removed if not used accross the protocol.

#### Affected code

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/dpxETH/DpxEthToken.sol#L47C1-L53C4



## Title

DEFAULT_ADMIN_ROLE initialized but not used anywhere

### Details

The DEFAULT_ADMIN_ROLE isn't being used in the DpxEthToken, thus making its initialization pointless, consider removing it.

#### Affected code

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/dpxETH/DpxEthToken.sol#L23-L24

## Title
Redundant Role Check in mint Function
### Details

The mint function contains a redundant require statement for checking the MINTER_ROLE, even though the onlyRole(MINTER_ROLE) modifier already enforces this. The redundancy could make the code less efficient and more confusing.

#### Affected code
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L118-L120



## Title

Use of Mutable State Variable for Unchangeable Value

### Details

The liquiditySlippageTolerance is set as a public state variable with a default value of 5e5 (representing a 0.5% tolerance). This is intended to define the acceptable level of slippage in liquidity operations. However, the value is set and never changed within the contract, making it effectively a constant, consider making it constant, the protocol will gain in clarity


#### Affected code

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L102-L103



## Title

Reserve Tokens and Assets Mismatch in `removeAssetFromtokenReserves` Function

### Details

A user with DEFAULT_ADMIN_ROLE permissions who wants to correct a wrongly set reserve asset can run into an issue where the reservesIndex, reserveTokens, and reserveAsset mappings and arrays become misaligned. Although this doesn't have a any real damaging impact, it would be better to have both this arrays aligned.

### POC
Proof of Concept
Initial State
reserveAsset = [0x]
reserveTokens = ['0x']
reservesIndex = {'0x': 0}

The Owner adds rdpx token.

reserveAsset = [0x, rdpx]
reserveTokens = ['0x', 'RDPX']
reservesIndex = {'0x': 0, 'RDPX': 1}
The Owner adds weth token.

reserveAsset = [0x, rdpx, weth]
reserveTokens = ['0x', 'RDPX', 'WETH']
reservesIndex = {'0x': 0, 'RDPX': 1, 'WETH': 2}
The Owner removes rdpx (with the wrong address).

reserveAsset = [0x, weth]
reserveTokens = ['0x', 'RDPX'] // Here it pops the last element (WETH)
reservesIndex = {'0x': 0, 'WETH': 1}
The Owner adds rdpx (with the correct address).

reserveAsset = [0x, weth, rdpx]
reserveTokens = ['0x', 'RDPX', 'RDPX'] // Now it has duplicate 'RDPX'
reservesIndex = {'0x': 0, 'WETH': 1, 'RDPX': 2}

## Mitigation Steps

  function removeAssetFromtokenReserves(
    string memory _assetSymbol
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    uint256 index = reservesIndex[_assetSymbol];
    _validate(index != 0, 18);

    // remove the asset from the mapping
    reservesIndex[_assetSymbol] = 0;

    // add new index for the last element
    reservesIndex[reserveTokens[reserveTokens.length - 1]] = index;

    // update the index of reserveAsset with the last element
    reserveAsset[index] = reserveAsset[reserveAsset.length - 1]; 

    @>reserveTokens[index] = reserveTokens[reserveTokens.length - 1];

    // remove the last element
    reserveAsset.pop();
    reserveTokens.pop();  

    emit LogAssetRemovedFromtokenReserves(_assetSymbol, index);
  }


#### Affected code

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L270-L290



##Title
Missing approval for `PerpetualAtlanticVault.sol` contract to spend rDPX on `RdpxV2Core.sol` behalf

## Impact
The function settle in the contract won't be able to proceed correctly because of missing approval. Until the admin approves the `PerpetualAtlanticVault` contract address to spend rDPX.

## Proof of Concept
```
function settle(
    uint256[] memory optionIds
  )
    external
    nonReentrant
    onlyRole(RDPXV2CORE_ROLE)
    returns (uint256 ethAmount, uint256 rdpxAmount)


  {
    _whenNotPaused();
    _isEligibleSender(); //@audit what is the point of having this if the function is only designed to be called by RDPXV2CORE_ROLE

    updateFunding();

    for (uint256 i = 0; i < optionIds.length; i++) {
      uint256 strike = optionPositions[optionIds[i]].strike;
      uint256 amount = optionPositions[optionIds[i]].amount;

      // check if strike is ITM
      _validate(strike >= getUnderlyingPrice(), 7); 
      ethAmount += (amount * strike) / 1e8;
      rdpxAmount += amount;
      optionsPerStrike[strike] -= amount;
      totalActiveOptions -= amount;

      // Burn option tokens from user
      _burn(optionIds[i]);

      optionPositions[optionIds[i]].strike = 0;
    }
    // Transfer collateral token from perpetual vault to rdpx rdpxV2Core
    collateralToken.safeTransferFrom(
      addresses.perpetualAtlanticVaultLP,
      addresses.rdpxV2Core,
      ethAmount
    );
    
  @> IERC20WithBurn(addresses.rdpx).safeTransferFrom(
      addresses.rdpxV2Core,
      addresses.perpetualAtlanticVaultLP,
      rdpxAmount
    );

    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).subtractLoss(
      ethAmount
    );
    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP)
      .unlockLiquidity(ethAmount);
    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addRdpx(
      rdpxAmount
    );
```
Here should the approval be added.
```
   IERC20WithBurn(weth).approve(
      addresses.perpetualAtlanticVault,
      type(uint256).max
    );
    IERC20WithBurn(weth).approve(addresses.dopexAMMRouter, type(uint256).max);
    IERC20WithBurn(weth).approve(addresses.dpxEthCurvePool, type(uint256).max);
    IERC20WithBurn(weth).approve(
      addresses.rdpxV2ReceiptToken,
      type(uint256).max
    );
    emit LogSetAddresses(addresses);
  }
```
## Tools Used

Manual review

## Recommended Mitigation Steps

Consider directly approving the tokens in the `setAddresses(...)` function. Even though the admin still has the opportunity to approve the contract once deployed, it will unintentionally freeze the protocol until PerpetualAtlanticVault is approved to spend.

#### Affected code

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L764-L783


## Title
Inadequate error number 

### Details

in the core contract, the _curveSwap() function does some preliminary validation before proceeding, if the call fails the error number 14 is reported however the number 14 seems more adequate.

#### Affected code

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L533-L541

## Title
Inacurate comment

### Details

The comment here states that we transfer eth and rdpx from the user however only rdpx is transferred.
```
    IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress)
        .safeTransferFrom(msg.sender, address(this), _rdpxAmount); 
```

#### Affected code

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L653-L654

##Title
The amount of rDPX in reserve can greatly impact the `calculateBondCost`

##Impact

If the amount of rDPX token in reserve increases and the `bondDiscountFactor` isn't updated, and remains at 1e5 just like in the tests, the amount of rdpx needed drops to 50 % and weth drops by 25%, such event can happen when reserves are only contain 2500 rpdx tokens.
Consider adding a way to dynamically adjust the `bondDiscountFactor` so that we the price doesn't drop too much

## POC
Example Scenario for Bond Discount (50 x 1e8)
To calculate how much should be in the reserve for a bond discount of 50 x 1e8, given a bondDiscountFactor of 1e5, we can use the formula:

`bondDiscount = (bondDiscountFactor * sqrt(rdpxReserve) * 1e2) / sqrt(1e18)`

To achieve a bondDiscount of 50 x 1e8, you would rearrange the formula to solve for rdpxReserve:

```
rdpxReserve = ((50 * 1e8 * 1e18) / (1e5 * 1e2))^2
rdpxReserve = ((50 * 1e9) / 1e5)^2
rdpxReserve = (50 x 1e4)^2
rdpxReserve = 2500 x 1e8
rdpxReserve = 2.5 x 1e11
```

Given that the smallest unit for rdpxReserve is 1e8, the amount required in terms of these units would be 2500 units (because 2.5 x 1e11 / 1e8 = 2500).


### Affected code

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1156-L1199


