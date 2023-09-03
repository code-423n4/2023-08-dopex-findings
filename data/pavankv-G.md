## 1 .Gas savings can be achieved by changing the model for assigning value to the structure

By changing the pattern of assigning value to the structure, gas savings of ~130 per instance are achieved. In addition, this use will provide significant savings in distribution costs.


```solidity
  constructor(address _weth) {
    _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
    weth = _weth;

    // add Zero asset to reserveAsset
    ReserveAsset memory zeroAsset = ReserveAsset({ //@audit
      tokenAddress: address(0),
      tokenBalance: 0,
      tokenSymbol: "ZERO"
    });
    reserveAsset.push(zeroAsset);
    putOptionsRequired = true;
  }
```
Look into `//@audit` comment in above code

**After**

```solidity
  constructor(address _weth) {
    _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
    weth = _weth;

    // add Zero asset to reserveAsset
   /* ReserveAsset memory zeroAsset = ReserveAsset({ //@audit
      tokenAddress: address(0),
      tokenBalance: 0,
      tokenSymbol: "ZERO"
    });
    */

    ReserveAsset memory zeroAsset; //@audit changed here

    zeroAsset.tokenAddress = address(0);//@audit changed here
    zeroAsset.tokenBalance = 0;//@audit changed here
    zeroAsset.tokenSymbol="ZERO";//@audit changed here



    reserveAsset.push(zeroAsset);
    putOptionsRequired = true;
  }
```

Please look into `//@audit changed here` comment in above code snippet.

code snippet:-
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L124C1-L136C4


b. 

Here struct assign type can be changed to become gas efficient in nature :

**Before**

```solidity
    ReserveAsset memory asset = ReserveAsset({
      tokenAddress: _asset,
      tokenBalance: 0,
      tokenSymbol: _assetSymbol
    });

```

**After**
```solidity
     ReserveAsset memory asset; //@audit changed here
     asset.tokenAddress = _asset;//@audit changed here
     asset.tokenBalance = 0;//@audit changed here
     asset.tokenSymbol = _assetSymbol;//@audit changed here
```
code snippet:-
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L253C1-L257C8


C. Assigning value to Address struct can changed to save gas . 

**Before**
```solidity
addresses = Addresses({
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

```

**After**

```solidity
      addresses.dopexAMMRouter = _dopexAMMRouter;
      addresses.dpxEthCurvePool = _dpxEthCurvePool;
      addresses.rdpxDecayingBonds = _rdpxDecayingBonds;
      addresses.perpetualAtlanticVault = _perpetualAtlanticVault;
      addresses.rdpxReserve = _rdpxReserve;
      addresses.rdpxV2ReceiptToken = _rdpxV2ReceiptToken;
      addresses.feeDistributor = _feeDistributor;
      addresses.reLPContract = _reLPContract;
      addresses.receiptTokenBonds = _receiptTokenBonds;

```

code snippet:-
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L327C1-L338C8

D. Assinging values to pricingOracleAddresses struct can be changed to save gas .

**Before**

```solidity
    pricingOracleAddresses = PricingOracleAddresses({
      rdpxPriceOracle: _rdpxPriceOracle,
      dpxEthPriceOracle: _dpxEthPriceOracle
    });

```

**After**

```solidity
pricingOracleAddresses.rdpxPriceOracle = _rdpxPriceOracle;
     pricingOracleAddresses.dpxEthPriceOracle = _dpxEthPriceOracle;
```
Above code snippet is gas optimised saves 130gas~


E . Assigning struct value can be changed of BondWithDelegateReturnValue 

**Before**
```solidity
    returnValues = BondWithDelegateReturnValue(
      delegateReceiptTokenAmount,
      bondAmountForUser,
      rdpxRequired,
      wethRequired
    );

```

**After**
```solidity
    returnValues.delegateReceiptToken;
    returnValues.bondAmountForUser;
    returnValues.rdpxRequired;
    returnValues.wethRequired;
```

code snippet:-
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L738C1-L743C7

## 2. Use Do while loop than for loop to save gas :-

For simple for-loop which is not much complex can be done with
do while loop to 5 gas .

**Before**
```solidity
    for (uint256 i = 0; i < optionIds.length; i++) {
      optionsOwned[optionIds[i]] = false;
    }
```

**After**
```solidity
uint i;
do{ optionsOwned[optionIds[i]] = false; ++i;}
while(i < optionIds.length);
```

code snippet:-
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L775C1-L777C6

B . below function's for-loop can be changed into do-while loop 

```solidity
  function sync() external {
    for (uint256 i = 1; i < reserveAsset.length; i++) {
      uint256 balance = IERC20WithBurn(reserveAsset[i].tokenAddress).balanceOf(
        address(this)
      );

      if (weth == reserveAsset[i].tokenAddress) {
        balance = balance - totalWethDelegated;
      }
      reserveAsset[i].tokenBalance = balance;
    }

    emit LogSync();
  }
```

**After**
```solidity
  function sync() external {


    uint i ; //@audit changed here
    do{
            uint256 balance = IERC20WithBurn(reserveAsset[i].tokenAddress).balanceOf(
        address(this)
      );

      if (weth == reserveAsset[i].tokenAddress) {
        balance = balance - totalWethDelegated;
      }
      reserveAsset[i].tokenBalance = balance;
      ++i;

    }while(i < reserveAsset.length); //@audit changed here
    emit LogSync();
  }
```
Look into `//@audit changed here` comment in above function.

Code snippet :-
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L995C1-L1008C4

## 3. Move storage pointer to top of function to avoid off-set calculation :-

In below function [_bondWithDelegate()](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L699) delegate struct is initialised in middle of the function which is not required any validation so it can initialised in top of the function to avoid offset-calculation. 

**Before**
```solidity
  function _bondWithDelegate(
    uint256 _amount,
    uint256 rdpxBondId,
    uint256 delegateId
  ) internal returns (BondWithDelegateReturnValue memory returnValues) {
    // Compute the bond cost
    (uint256 rdpxRequired, uint256 wethRequired) = calculateBondCost(
      _amount,
      rdpxBondId
    );

    // update ETH token reserve
    reserveAsset[reservesIndex["WETH"]].tokenBalance += wethRequired;

    Delegate storage delegate = delegates[delegateId];//@audit
```
Look into `//@audit` comment in above function.

**After**
```solidity
  function _bondWithDelegate(
    uint256 _amount,
    uint256 rdpxBondId,
    uint256 delegateId
  ) internal returns (BondWithDelegateReturnValue memory returnValues) {
        Delegate storage delegate = delegates[delegateId]; //@audit changed here
    // Compute the bond cost
    (uint256 rdpxRequired, uint256 wethRequired) = calculateBondCost(
      _amount,
      rdpxBondId
    );

    // update ETH token reserve
    reserveAsset[reservesIndex["WETH"]].tokenBalance += wethRequired;


```
Look into `//@audit changed here` comment made in above function .


code snippet:-
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L699C1-L715C41



## 4. Use one time memory to store variable instead of two times. 

 



