## GAS Summary<a name="GAS Summary">

### Gas Optimizations
| |Issue|Contexts|Estimated Gas Saved|
|-|:-|:-|:-:|
| [GAS&#x2011;1](#GAS&#x2011;1) | Activate the optimizer | 1 | - |
| [GAS&#x2011;2](#GAS&#x2011;2) | `address(this)` can be stored in `state variable` that will ultimately cost less, than every time calculating this, specifically when it used multiple times in a `contract` | 1 | - |
| [GAS&#x2011;3](#GAS&#x2011;3) | Use assembly to emit events | 11 | 418 |
| [GAS&#x2011;4](#GAS&#x2011;4) | Avoid emitting event on every iteration | 1 | 375 |
| [GAS&#x2011;5](#GAS&#x2011;5) | Avoid repeating computations | 20 | - |
| [GAS&#x2011;6](#GAS&#x2011;6) | Gas savings can be achieved by changing the model for assigning value to the structure | 7 | 910 |
| [GAS&#x2011;7](#GAS&#x2011;7) | Combine events to save `Glogtopic (375 gas)` for each instance | 2 | 750 |
| [GAS&#x2011;8](#GAS&#x2011;8) | Counting down in `for` statements is more gas efficient | 12 | 3084 |
| [GAS&#x2011;9](#GAS&#x2011;9) | Hash using `assembly` instead of solidity | 2 | 160 |
| [GAS&#x2011;10](#GAS&#x2011;10) | Using `unchecked` blocks to save gas | 3 | 120 |
| [GAS&#x2011;11](#GAS&#x2011;11) | Add `unchecked {}` for subtractions where the operands cannot underflow | 14 | 560 |
| [GAS&#x2011;12](#GAS&#x2011;12) | Use do while loops instead of for loops | 12 | 48 |
| [GAS&#x2011;13](#GAS&#x2011;13) | Using XOR (^) and AND (&) bitwise equivalents | 14 | 182 |

Total: 172 contexts over 13 issues

### Experimental Gas Optimizations

The following gas optimization include snapshots and the code snippet that was used. The issues may also include invalid instances that were detected to either increase gas usage or not provide any improvements.

| |Issue|
|-|:-|
| [EXPGAS&#x2011;1](#EXPGAS&#x2011;1) | Use do while loops instead of for loops |
| [EXPGAS&#x2011;2](#EXPGAS&#x2011;2) | `unchecked {}` can be used on the division of two `uints` in order to save gas |
| [EXPGAS&#x2011;3](#EXPGAS&#x2011;3) | Using `private` rather than `public` for constants, saves gas |


## Gas Optimizations

### <a href="#gas-summary">[GAS&#x2011;1]</a><a name="GAS&#x2011;1"> Activate the optimizer

Before deploying your contract, activate the optimizer when compiling using “solc --optimize --bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ --optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “--optimize-runs” to a high number.

```
module.exports = {
	solidity: {
		version: "0.8.19",
		settings: {
			optimizer: {
				enabled: true,
				runs: 1000,
			},
		},
	},
};
```
Kindly visit the following site for further information:

https://docs.soliditylang.org/en/v0.8.19/using-the-compiler.html#using-the-commandline-compiler

Here is one particular example of instance on opcode comparison that delineates the gas saving mechanism:

```
for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
```

#### <ins>Proof Of Concept</ins>






### <a href="#gas-summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> Consider activating `via-ir` for deploying

The IR-based code generator was introduced with an aim to not only allow code generation to be more transparent and auditable but also to enable more powerful optimization passes that span across functions.

You can enable it on the command line using `--via-ir` or with the option `{"viaIR": true}`.

This will take longer to compile, but you can just simple test it before deploying and if you got a better benchmark then you can add --via-ir to your deploy command

More on: https://docs.soliditylang.org/en/v0.8.17/ir-breaking-changes.html



### <a href="#gas-summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> `address(this)` can be stored in `state variable` that will ultimately cost less, than every time calculating this, specifically when it used multiple times in a `contract`

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: UniV2LiquidityAmo.sol

149: token.safeTransfer(msg.sender, token.balanceOf(address(this)));


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L149

```solidity
File: UniV2LiquidityAmo.sol

162: address(this)


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L162

```solidity
File: UniV2LiquidityAmo.sol

212: address(this),


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L212

```solidity
File: UniV2LiquidityAmo.sol

278: address(this),


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L278

```solidity
File: UniV2LiquidityAmo.sol

323: address(this),


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L323

```solidity
File: UniV2LiquidityAmo.sol

363: lpTokenBalance = IERC20WithBurn(addresses.pair).balanceOf(address(this));


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L363

```solidity
File: UniV3LiquidityAmo.sol

106: keccak256(abi.encodePacked(address(this), _tickLower, _tickUpper))


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L106

```solidity
File: UniV3LiquidityAmo.sol

160: address(this),


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L160

```solidity
File: UniV3LiquidityAmo.sol

285: address(this),


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L285

```solidity
File: UniV3LiquidityAmo.sol

331: address(this),


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L331

```solidity
File: UniV3LiquidityAmo.sol

354: uint256 tokenABalance = IERC20WithBurn(tokenA).balanceOf(address(this));

355: uint256 tokenBBalance = IERC20WithBurn(tokenB).balanceOf(address(this));


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L354-L355

```solidity
File: RdpxV2Core.sol

169: token.safeTransfer(msg.sender, token.balanceOf(address(this)));


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L169

```solidity
File: RdpxV2Core.sol

483: ).purchase(_amount, address(this));


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L483

```solidity
File: RdpxV2Core.sol

573: address(this),


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L573

```solidity
File: RdpxV2Core.sol

654: .safeTransferFrom(msg.sender, address(this), _rdpxAmount);


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L654

```solidity
File: RdpxV2Core.sol

911: address(this),


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L911

```solidity
File: RdpxV2Core.sol

953: IERC20WithBurn(weth).safeTransferFrom(msg.sender, address(this), _amount);


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L953

```solidity
File: RdpxV2Core.sol

998: address(this)


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L998

```solidity
File: RdpxV2Core.sol

1060: address(this),


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L1060

```solidity
File: RdpxV2Core.sol

1102: address(this),


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L1102

```solidity
File: RdpxDecayingBonds.sol

105: token.safeTransfer(msg.sender, token.balanceOf(address(this)));


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L105

```solidity
File: PerpetualAtlanticVault.sol

227: token.safeTransfer(msg.sender, token.balanceOf(address(this)));


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L227

```solidity
File: PerpetualAtlanticVault.sol

289: collateralToken.safeTransferFrom(msg.sender, address(this), premium);


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L289

```solidity
File: PerpetualAtlanticVault.sol

384: address(this),


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L384

```solidity
File: PerpetualAtlanticVaultLP.sol

128: collateral.transferFrom(msg.sender, address(this), assets);


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L128

```solidity
File: PerpetualAtlanticVaultLP.sol

192: collateral.balanceOf(address(this)) >= _totalCollateral + proceeds,


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L192

```solidity
File: PerpetualAtlanticVaultLP.sol

201: collateral.balanceOf(address(this)) == _totalCollateral - loss,


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L201

```solidity
File: PerpetualAtlanticVaultLP.sol

210: IERC20WithBurn(rdpx).balanceOf(address(this)) >= _rdpxCollateral + amount,


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L210

```solidity
File: ReLPContract.sol

245: address(this),

304: IERC20WithBurn(addresses.tokenA).balanceOf(address(this))


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L245

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L304





</details>



### <a href="#gas-summary">[GAS&#x2011;3]</a><a name="GAS&#x2011;3"> Use assembly to emit events

We can use assembly to emit events efficiently by utilizing `scratch space` and the `free memory pointer`. This will allow us to potentially avoid memory expansion costs.
Note: In order to do this optimization safely, we will need to cache and restore the free memory pointer.

For example, for a generic `emit` event for `eventSentAmountExample`:
```solidity
// uint256 id, uint256 value, uint256 amount
emit eventSentAmountExample(id, value, amount);
```

We can use the following assembly emit events:

```solidity
        assembly {
            let memptr := mload(0x40)
            mstore(0x00, calldataload(0x44))
            mstore(0x20, calldataload(0xa4))
            mstore(0x40, amount)
            log1(
                0x00,
                0x60,
                // keccak256("eventSentAmountExample(uint256,uint256,uint256)")
                0xa622cf392588fbf2cd020ff96b2f4ebd9c76d7a4bc7f3e6b2f18012312e76bc3
            )
            mstore(0x40, memptr)
        }
```

#### <ins>Proof Of Concept</ins>

<details>


```solidity
File: UniV2LiquidityAmo.sol

177: emit LogAssetsTransfered(msg.sender, tokenABalance, tokenBBalance);

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L177


```solidity
File: UniV3LiquidityAmo.sol

363: emit LogAssetsTransfered(tokenABalance, tokenBBalance, tokenA, tokenB);

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L363


```solidity
File: RdpxV2Core.sol

932: emit LogBond(rdpxRequired, wethRequired, receiptTokenAmount);

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L932

```solidity
File: RdpxV2Core.sol

966: emit LogAddToDelegate(_amount, _fee, delegates.length - 1);

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L966


```solidity
File: RdpxV2Core.sol

1007: emit LogSync();

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L1007


```solidity
File: RdpxV2Core.sol

1123: emit LogLowerDepeg(_rdpxAmount, _wethAmount, dpxEthReceived);

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L1123

```solidity
File: RdpxDecayingBonds.sol

124: emit BondMinted(to, bondId, expiry, rdpxAmount);

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L124


```solidity
File: PerpetualAtlanticVault.sol

311: emit Purchase(strike, amount, premium, to, msg.sender);

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L311

```solidity
File: PerpetualAtlanticVault.sol

368: emit Settle(ethAmount, rdpxAmount, optionIds);

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L368


```solidity
File: PerpetualAtlanticVaultLP.sol

134: emit Deposit(msg.sender, receiver, assets, shares);

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L134

```solidity
File: PerpetualAtlanticVaultLP.sol

174: emit Withdraw(msg.sender, receiver, owner, assets, shares);

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L174


</details>




### <a href="#gas-summary">[GAS&#x2011;4]</a><a name="GAS&#x2011;4"> Avoid emitting event on every iteration

Expensive operations should always try to be avoided within loops. Such operations include: reading/writing to storage, heavy calculations, external calls, and emitting events. In this instance, an event is being emitted every iteration. Events have a base cost of `Glog (375 gas)` per emit and `Glogdata (8 gas) * number of bytes in event`. We can avoid incurring those costs each iteration by emitting the event outside of the loop.

#### <ins>Proof Of Concept</ins>

```solidity
File: PerpetualAtlanticVault.sol

413: for (uint256 i = 0; i < strikes.length; i++) {
      _validate(optionsPerStrike[strikes[i]] > 0, 4);
      _validate(
        latestFundingPerStrike[strikes[i]] != latestFundingPaymentPointer,
        5
      );
      uint256 strike = strikes[i];

      uint256 amount = optionsPerStrike[strike] -
        fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
          strike
        ];

      uint256 timeToExpiry = nextFundingPaymentTimestamp() -
        (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration));

      uint256 premium = calculatePremium(
        strike,
        amount,
        timeToExpiry,
        getUnderlyingPrice()
      );

      latestFundingPerStrike[strike] = latestFundingPaymentPointer;
      fundingAmount += premium;

      
      fundingPaymentsAccountedFor[latestFundingPaymentPointer] += amount;

      
      fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
        strike
      ] += amount;

      
      
      totalFundingForEpoch[latestFundingPaymentPointer] += premium;

      emit CalculateFunding(
        msg.sender,
        amount,
        strike,
        premium,
        latestFundingPaymentPointer
      );
    }

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L413






### <a href="#gas-summary">[GAS&#x2011;5]</a><a name="GAS&#x2011;5"> Avoid repeating computations

The following instances show only the first instance of the repeated computations.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: RdpxV2Core.sol

280: reserveTokens.length - 1
283: reserveAsset.length - 1

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L280

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L283


```solidity
File: RdpxV2Core.sol

570: _amount / 2
574: _amount / 2

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L570

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L574

```solidity
File: RdpxV2Core.sol

966: delegates.length - 1
967: delegates.length - 1

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L966-L967

```solidity
File: RdpxV2Core.sol

1170: bondDiscount / 2
1176: bondDiscount / 2

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L1170

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L1176


```solidity
File: PerpetualAtlanticVault.sol

512: (currentFundingRate * (block.timestamp - startTime)) / 1e18
516: (currentFundingRate * (block.timestamp - startTime)) / 1e18
521: (currentFundingRate * (block.timestamp - startTime)) / 1e18

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L512

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L516

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L521


```solidity
File: ReLPContract.sol

252: tokenAToRemove * tokenAInfo.tokenAPrice
254: tokenAToRemove * tokenAInfo.tokenAPrice

264: block.timestamp + 10
283: block.timestamp + 10
294: block.timestamp + 10

274: amountB / 2
275: amountB / 2
279: amountB / 2
290: amountB / 2


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L252

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L254

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L264

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L283

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L294

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L274-L275

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L279

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L290

</details>


### <a href="#gas-summary">[GAS&#x2011;6]</a><a name="GAS&#x2011;6"> Gas savings can be achieved by changing the model for assigning value to the structure

Here's an example to illustrate this:
```solidity
	struct MyStruct {
	    uint256 a;
	    uint256 b;
	    uint256 c;
	}

	function assignValuesToStruct(uint256 _a, uint256 _b, uint256 _c) public {
	    MyStruct memory myStruct = MyStruct(_a, _b, _c);
	    // Do something with myStruct
	}
```
In this example, we have a simple MyStruct data structure with three uint256 fields. The assignValuesToStruct function takes three input parameters _a, _b, and _c, and assigns them to the corresponding fields in a new myStruct variable. This is done using the struct constructor syntax, which creates a new instance of the MyStruct struct with the specified field values.

The following approach can be more efficient than assigning values to the struct fields:
```solidity
	function assignValuesToStruct(uint256 _a, uint256 _b, uint256 _c) public {
	    MyStruct memory myStruct;
	    myStruct.a = _a;
	    myStruct.b = _b;
	    myStruct.c = _c;
	    // Do something with myStruct
	}
```
By changing the pattern of assigning value to the structure, gas savings of `~130` per instance are achieved. In addition, this use will provide significant savings in distribution costs.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: UniV2LiquidityAmo.sol

93: addresses = Addresses({
      tokenA: _tokenA,
      tokenB: _tokenB,
      pair: _pair,
      rdpxV2Core: _rdpxV2Core,
      rdpxOracle: _rdpxOracle,
      ammFactory: _ammFactory,
      ammRouter: _ammRouter
    });


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L93

```solidity
File: RdpxV2Core.sol

327: addresses = Addresses({
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

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L327

```solidity
File: RdpxV2Core.sol

365: pricingOracleAddresses = PricingOracleAddresses({
      rdpxPriceOracle: _rdpxPriceOracle,
      dpxEthPriceOracle: _dpxEthPriceOracle
    });


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L365

```solidity
File: RdpxV2Core.sol

500: bonds[bondId] = Bond({
      amount: _amount,
      maturity: block.timestamp + bondMaturity,
      timestamp: block.timestamp
    });


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L500

```solidity
File: PerpetualAtlanticVault.sol

198: addresses = Addresses({
      optionPricing: _optionPricing,
      assetPriceOracle: _assetPriceOracle,
      volatilityOracle: _volatilityOracle,
      feeDistributor: _feeDistributor,
      rdpx: _rdpx,
      perpetualAtlanticVaultLP: _perpetualAtlanticVaultLP,
      rdpxV2Core: _rdpxV2Core
    });


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L198

```solidity
File: PerpetualAtlanticVault.sol

296: optionPositions[tokenId] = OptionPosition({
      strike: strike,
      amount: amount,
      positionId: tokenId
    });


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L296

```solidity
File: ReLPContract.sol

138: addresses = Addresses({
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

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L138



</details>




### <a href="#gas-summary">[GAS&#x2011;7]</a><a name="GAS&#x2011;7"> Combine events to save `Glogtopic (375 gas)` for each instance

We can combine the events into one singular event to save  `Glogtopic (375 gas)` for each saved instances, that would otherwise be paid for the additional events emissions.

#### <ins>Proof Of Concept</ins>


```solidity
File: UniV3LiquidityAmo.sol

268: emit log(positions_array.length);
269: emit log(positions_mapping[pos.token_id].token_id);

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L268-L269



### <a href="#gas-summary">[GAS&#x2011;8]</a><a name="GAS&#x2011;8"> Counting down in `for` statements is more gas efficient

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: UniV2LiquidityAmo.sol

147: for (uint256 i = 0; i < tokens.length; i++) {

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L147

```solidity
File: UniV3LiquidityAmo.sol

120: for (uint i = 0; i < positions_array.length; i++) {

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L120

```solidity
File: RdpxV2Core.sol

167: for (uint256 i = 0; i < tokens.length; i++) {

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L167

```solidity
File: RdpxV2Core.sol

246: for (uint256 i = 1; i < reserveAsset.length; i++) {

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L246

```solidity
File: RdpxV2Core.sol

775: for (uint256 i = 0; i < optionIds.length; i++) {

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L775

```solidity
File: RdpxV2Core.sol

836: for (uint256 i = 0; i < _amounts.length; i++) {

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L836

```solidity
File: RdpxV2Core.sol

996: for (uint256 i = 1; i < reserveAsset.length; i++) {

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L996

```solidity
File: RdpxDecayingBonds.sol

103: for (uint256 i = 0; i < tokens.length; i++) {

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L103

```solidity
File: RdpxDecayingBonds.sol

156: for (uint256 i; i < ownerTokenCount; i++) {

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L156

```solidity
File: PerpetualAtlanticVault.sol

225: for (uint256 i = 0; i < tokens.length; i++) {

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L225

```solidity
File: PerpetualAtlanticVault.sol

328: for (uint256 i = 0; i < optionIds.length; i++) {

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L328

```solidity
File: PerpetualAtlanticVault.sol

413: for (uint256 i = 0; i < strikes.length; i++) {

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L413



</details>


#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.AddNum();
            c1.AddNum();
        }
    }
    
    contract Contract0 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=0;i<=9;i++){
                _num = _num +1;
            }
            num = _num;
        }
    }
    
    contract Contract1 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=9;i>=0;i--){
                _num = _num +1;
            }
            num = _num;
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 7040            | 7040 | 7040   | 7040 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 3819            | 3819 | 3819   | 3819 | 1       |




### <a href="#gas-summary">[GAS&#x2011;9]</a><a name="GAS&#x2011;9"> Hash using `assembly` instead of solidity

Saves about ~80 gas

```solidity
function solidityHash(uint256 a, uint256 b) public view {
    //unoptimized
    keccak256(abi.encodePacked(a, b));
}
```
Gas Test: 313

```solidity
function assemblyHash(uint256 a, uint256 b) public view {
    //optimized
    assembly {
        mstore(0x00, a)
        mstore(0x20, b)
        let hashedVal := keccak256(0x00, 0x40)
    }
} 
```
Gas Test: 231

#### <ins>Proof Of Concept</ins>


```solidity
File: UniV3LiquidityAmo.sol

106: keccak256(abi.encodePacked(address(this), _tickLower, _tickUpper))
    );


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L106

```solidity
File: RdpxV2Core.sol

1141: keccak256(abi.encodePacked(asset.tokenSymbol)) ==
        keccak256(abi.encodePacked(_token)),
      19
    );


```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L1141




### <a href="#gas-summary">[GAS&#x2011;10]</a><a name="GAS&#x2011;10"> Using `unchecked` blocks to save gas

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an `unchecked` block

#### <ins>Proof Of Concept</ins>

```solidity
File: RdpxV2Core.sol

261: reservesIndex[_assetSymbol] = reserveAsset.length - 1;

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L261

```solidity
File: PerpetualAtlanticVaultLP.sol

156: allowance[owner][msg.sender] = allowed - shares;

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L156

```solidity
File: PerpetualAtlanticVaultLP.sol

201: collateral.balanceOf(address(this)) == _totalCollateral - loss,

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L201





### <a href="#gas-summary">[GAS&#x2011;11]</a><a name="GAS&#x2011;11"> Add `unchecked {}` for subtractions where the operands cannot underflow

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: RdpxV2Core.sol

261: reservesIndex[_assetSymbol] = reserveAsset.length - 1;

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L261

```solidity
File: RdpxV2Core.sol

612: amount1 = (amount1 * (100e8 - _delegateFee)) / 1e10;
614: amount2 = _amount - amount1;

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L612

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L614



```solidity
File: RdpxV2Core.sol

966: emit LogAddToDelegate(_amount, _fee, delegates.length - 1);
967: return (delegates.length - 1);

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L966-L967

```solidity
File: RdpxV2Core.sol

1002: balance = balance - totalWethDelegated;

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L1002

```solidity
File: PerpetualAtlanticVault.sol

427: (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration));

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L427

```solidity
File: PerpetualAtlanticVault.sol

512: (currentFundingRate * (block.timestamp - startTime)) / 1e18
521: ((currentFundingRate * (block.timestamp - startTime)) / 1e18),

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L512

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L521



```solidity
File: PerpetualAtlanticVault.sol

559: return strike > price ? ((strike - price) * amount) / 1e8 : 0;

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L559

```solidity
File: PerpetualAtlanticVault.sol

605: (endTime - startTime);
612: ((amount * 1e18) / (endTime - startTime));

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L605

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L612



```solidity
File: PerpetualAtlanticVaultLP.sol

156: allowance[owner][msg.sender] = allowed - shares;

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L156

```solidity
File: PerpetualAtlanticVaultLP.sol

256: return _totalCollateral - _activeCollateral;

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L256



</details>


### <a href="#gas-summary">[GAS&#x2011;12]</a><a name="GAS&#x2011;12"> Use do while loops instead of for loops

A do while loop will cost less gas since the condition is not being checked for the first iteration.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: UniV2LiquidityAmo.sol

142: function emergencyWithdraw

for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L143

```solidity
File: UniV3LiquidityAmo.sol

119: function collectFees

for (uint i = 0; i < positions_array.length; i++) {
      Position memory current_position = positions_array[i];
      INonfungiblePositionManager.CollectParams
        memory collect_params = INonfungiblePositionManager.CollectParams(
          current_position.token_id,
          rdpxV2Core,
          type(uint128).max,
          type(uint128).max
        );

      
      univ3_positions.collect(collect_params);
    }

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L119

```solidity
File: RdpxV2Core.sol

161: function emergencyWithdraw

for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L162

```solidity
File: RdpxV2Core.sol

240: function addAssetTotokenReserves

for (uint256 i = 1; i < reserveAsset.length; i++) {
      require(
        reserveAsset[i].tokenAddress != _asset,
        "RdpxV2Core: asset already exists"
      );
    }

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L241

```solidity
File: RdpxV2Core.sol

764: function settle

for (uint256 i = 0; i < optionIds.length; i++) {
      optionsOwned[optionIds[i]] = false;
    }

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L765

```solidity
File: RdpxV2Core.sol

819: function bondWithDelegate

for (uint256 i = 0; i < _amounts.length; i++) {
      
      _validate(_amounts[i] > 0, 4);

      BondWithDelegateReturnValue
        memory returnValues = BondWithDelegateReturnValue(0, 0, 0, 0);

      returnValues = _bondWithDelegate(
        _amounts[i],
        rdpxBondId,
        _delegateIds[i]
      );

      delegateReceiptTokenAmounts[i] = returnValues.delegateReceiptTokenAmount;

      userTotalBondAmount += returnValues.bondAmountForUser;
      totalBondAmount += _amounts[i];

      
      uint256 premium;
      if (putOptionsRequired) {
        premium = _purchaseOptions(returnValues.rdpxRequired);
      }

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L820

```solidity
File: RdpxV2Core.sol

995: function sync

for (uint256 i = 1; i < reserveAsset.length; i++) {
      uint256 balance = IERC20WithBurn(reserveAsset[i].tokenAddress).balanceOf(
        address(this)
      );

      if (weth == reserveAsset[i].tokenAddress) {
        balance = balance - totalWethDelegated;
      }

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L995

```solidity
File: RdpxDecayingBonds.sol

89: function emergencyWithdraw

for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L90

```solidity
File: RdpxDecayingBonds.sol

151: function getBondsOwned

for (uint256 i; i < ownerTokenCount; i++) {
      tokenIds[i] = tokenOfOwnerByIndex(_address, i);
    }

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L152

```solidity
File: PerpetualAtlanticVault.sol

219: function emergencyWithdraw

for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L220

```solidity
File: PerpetualAtlanticVault.sol

315: function settle

for (uint256 i = 0; i < optionIds.length; i++) {
      uint256 strike = optionPositions[optionIds[i]].strike;
      uint256 amount = optionPositions[optionIds[i]].amount;

      
      _validate(strike >= getUnderlyingPrice(), 7);

      ethAmount += (amount * strike) / 1e8;
      rdpxAmount += amount;
      optionsPerStrike[strike] -= amount;
      totalActiveOptions -= amount;

      
      _burn(optionIds[i]);

      optionPositions[optionIds[i]].strike = 0;
    }

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L316

```solidity
File: PerpetualAtlanticVault.sol

405: function calculateFunding

for (uint256 i = 0; i < strikes.length; i++) {
      _validate(optionsPerStrike[strikes[i]] > 0, 4);
      _validate(
        latestFundingPerStrike[strikes[i]] != latestFundingPaymentPointer,
        5
      );
      uint256 strike = strikes[i];

      uint256 amount = optionsPerStrike[strike] -
        fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
          strike
        ];

      uint256 timeToExpiry = nextFundingPaymentTimestamp() -
        (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration));

      uint256 premium = calculatePremium(
        strike,
        amount,
        timeToExpiry,
        getUnderlyingPrice()
      );

      latestFundingPerStrike[strike] = latestFundingPaymentPointer;
      fundingAmount += premium;

      
      fundingPaymentsAccountedFor[latestFundingPaymentPointer] += amount;

      
      fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
        strike
      ] += amount;

      
      
      totalFundingForEpoch[latestFundingPaymentPointer] += premium;

      emit CalculateFunding(
        msg.sender,
        amount,
        strike,
        premium,
        latestFundingPaymentPointer
      );
    }

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L406



</details>



### <a href="#gas-summary">[GAS&#x2011;13]</a><a name="GAS&#x2011;13"> Using XOR (^) and AND (&) bitwise equivalents

Given 4 variables a, b, c and d represented as such:

    0 0 0 0 0 1 1 0 <- a
    0 1 1 0 0 1 1 0 <- b
    0 0 0 0 0 0 0 0 <- c
    1 1 1 1 1 1 1 1 <- d

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: UniV3LiquidityAmo.sol

199: params._tokenA == address(rdpx) ? params._tokenB : params._tokenA,

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L199

```solidity
File: RdpxV2Core.sol

525: (uint256 a, uint256 b) = coin0 == weth ? (0, 1) : (1, 0);

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L525

```solidity
File: RdpxV2Core.sol

827: _validate(_amounts.length == _delegateIds.length, 3);

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L827

```solidity
File: RdpxV2Core.sol

981: _validate(delegate.owner == msg.sender, 9);

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L981

```solidity
File: RdpxV2Core.sol

1001: if (weth == reserveAsset[i].tokenAddress) {

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L1001

```solidity
File: RdpxV2Core.sol

1026: msg.sender == RdpxV2Bond(addresses.receiptTokenBonds).ownerOf(id),

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L1026

```solidity
File: RdpxV2Core.sol

1162: if (_rdpxBondId == 0) {

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L1162

```solidity
File: PerpetualAtlanticVault.sol

467: uint256 startTime = lastUpdateTime == 0

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L467

```solidity
File: PerpetualAtlanticVault.sol

505: uint256 startTime = lastUpdateTime == 0

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L505

```solidity
File: PerpetualAtlanticVault.sol

578: if (remainder == 0) {

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L578

```solidity
File: PerpetualAtlanticVault.sol

609: if (endTime == startTime) return;

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L609

```solidity
File: PerpetualAtlanticVaultLP.sol

223: (supply == 0)

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L223

```solidity
File: PerpetualAtlanticVaultLP.sol

283: supply == 0 ? assets : assets.mulDivDown(supply, totalVaultCollateral);

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L283

```solidity
File: ReLPContract.sol

224: tokenAInfo.tokenALpReserve = addresses.tokenA == tokenASorted

```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L224



</details>


#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(1,2);
            c1.optimized(1,2);
        }
    }
    
    contract Contract0 {
        function not_optimized(uint8 a,uint8 b) public returns(bool){
            return ((a==1) || (b==1));
        }
    }
    
    contract Contract1 {
        function optimized(uint8 a,uint8 b) public returns(bool){
            return ((a ^ 1) & (b ^ 1)) == 0;
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 46099                                     | 261             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 456             | 456 | 456    | 456 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 42493                                     | 243             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 430             | 430 | 430    | 430 | 1       |



## Experimental Gas Optimizations

### <a href="#gas-summary">[EXPGAS&#x2011;1]</a><a name="EXPGAS&#x2011;1"> Use do while loops instead of for loops

A do while loop will cost less gas since the condition is not being checked for the first iteration.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: UniV3LiquidityAmo.sol

119: function collectFees

for (uint i = 0; i < positions_array.length; i++) {
      Position memory current_position = positions_array[i];
      INonfungiblePositionManager.CollectParams
        memory collect_params = INonfungiblePositionManager.CollectParams(
          current_position.token_id,
          rdpxV2Core,
          type(uint128).max,
          type(uint128).max
        );

      
      univ3_positions.collect(collect_params);
    }

```

Optimize by changing to:

```solidity
uint i = 0;
do {

      Position memory current_position = positions_array[i];
      INonfungiblePositionManager.CollectParams
        memory collect_params = INonfungiblePositionManager.CollectParams(
          current_position.token_id,
          rdpxV2Core,
          type(uint128).max,
          type(uint128).max
        );

      
      univ3_positions.collect(collect_params);
     i++;
} while ( i < positions_array.length);
```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV3LiquidityAmo.sol#L119

```solidity
File: UniV2LiquidityAmo.sol

142: function emergencyWithdraw

for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }

```

Optimize by changing to:

```solidity
uint256 i = 0;
do {

      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
     i++;
} while ( i < tokens.length);
```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L143

```solidity
File: RdpxV2Core.sol

161: function emergencyWithdraw

for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }

```

Optimize by changing to:

```solidity
uint256 i = 0;
do {

      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
     i++;
} while ( i < tokens.length);
```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L162

```solidity
File: RdpxV2Core.sol

240: function addAssetTotokenReserves

for (uint256 i = 1; i < reserveAsset.length; i++) {
      require(
        reserveAsset[i].tokenAddress != _asset,
        "RdpxV2Core: asset already exists"
      );
    }

```

Optimize by changing to:

```solidity
if (reserveAsset.length > 1) {
uint256 i = 1;
do {

      require(
        reserveAsset[i].tokenAddress != _asset,
        "RdpxV2Core: asset already exists"
      );
     i++;
} while ( i < reserveAsset.length);
}
```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L241

```solidity
File: RdpxV2Core.sol

764: function settle

for (uint256 i = 0; i < optionIds.length; i++) {
      optionsOwned[optionIds[i]] = false;
    }

```

Optimize by changing to:

```solidity
uint256 i = 0;
do {

      optionsOwned[optionIds[i]] = false;
     i++;
} while ( i < optionIds.length);
```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L765

```solidity
File: RdpxDecayingBonds.sol

89: function emergencyWithdraw

for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }

```

Optimize by changing to:

```solidity
uint256 i = 0;
do {

      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
     i++;
} while ( i < tokens.length);
```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L90

```solidity
File: RdpxDecayingBonds.sol

151: function getBondsOwned

for (uint256 i; i < ownerTokenCount; i++) {
      tokenIds[i] = tokenOfOwnerByIndex(_address, i);
    }

```

Optimize by changing to:

```solidity
uint256 i;
do {

      tokenIds[i] = tokenOfOwnerByIndex(_address, i);
     i++;
} while ( i < ownerTokenCount);
```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L152

```solidity
File: PerpetualAtlanticVault.sol

219: function emergencyWithdraw

for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }

```

Optimize by changing to:

```solidity
uint256 i = 0;
do {

      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
     i++;
} while ( i < tokens.length);
```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L220

```solidity
File: PerpetualAtlanticVault.sol

315: function settle

for (uint256 i = 0; i < optionIds.length; i++) {
      uint256 strike = optionPositions[optionIds[i]].strike;
      uint256 amount = optionPositions[optionIds[i]].amount;

      
      _validate(strike >= getUnderlyingPrice(), 7);

      ethAmount += (amount * strike) / 1e8;
      rdpxAmount += amount;
      optionsPerStrike[strike] -= amount;
      totalActiveOptions -= amount;

      
      _burn(optionIds[i]);

      optionPositions[optionIds[i]].strike = 0;
    }

```

Optimize by changing to:

```solidity
uint256 i = 0;
do {

      uint256 strike = optionPositions[optionIds[i]].strike;
      uint256 amount = optionPositions[optionIds[i]].amount;

      
      _validate(strike >= getUnderlyingPrice(), 7);

      ethAmount += (amount * strike) / 1e8;
      rdpxAmount += amount;
      optionsPerStrike[strike] -= amount;
      totalActiveOptions -= amount;

      
      _burn(optionIds[i]);

      optionPositions[optionIds[i]].strike = 0;
     i++;
} while ( i < optionIds.length);
```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L316

```solidity
File: PerpetualAtlanticVault.sol

405: function calculateFunding

for (uint256 i = 0; i < strikes.length; i++) {
      _validate(optionsPerStrike[strikes[i]] > 0, 4);
      _validate(
        latestFundingPerStrike[strikes[i]] != latestFundingPaymentPointer,
        5
      );
      uint256 strike = strikes[i];

      uint256 amount = optionsPerStrike[strike] -
        fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
          strike
        ];

      uint256 timeToExpiry = nextFundingPaymentTimestamp() -
        (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration));

      uint256 premium = calculatePremium(
        strike,
        amount,
        timeToExpiry,
        getUnderlyingPrice()
      );

      latestFundingPerStrike[strike] = latestFundingPaymentPointer;
      fundingAmount += premium;

      
      fundingPaymentsAccountedFor[latestFundingPaymentPointer] += amount;

      
      fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
        strike
      ] += amount;

      
      
      totalFundingForEpoch[latestFundingPaymentPointer] += premium;

      emit CalculateFunding(
        msg.sender,
        amount,
        strike,
        premium,
        latestFundingPaymentPointer
      );
    }

```

Optimize by changing to:

```solidity
uint256 i = 0;
do {

      _validate(optionsPerStrike[strikes[i]] > 0, 4);
      _validate(
        latestFundingPerStrike[strikes[i]] != latestFundingPaymentPointer,
        5
      );
      uint256 strike = strikes[i];

      uint256 amount = optionsPerStrike[strike] -
        fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
          strike
        ];

      uint256 timeToExpiry = nextFundingPaymentTimestamp() -
        (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration));

      uint256 premium = calculatePremium(
        strike,
        amount,
        timeToExpiry,
        getUnderlyingPrice()
      );

      latestFundingPerStrike[strike] = latestFundingPaymentPointer;
      fundingAmount += premium;

      
      fundingPaymentsAccountedFor[latestFundingPaymentPointer] += amount;

      
      fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
        strike
      ] += amount;

      
      
      totalFundingForEpoch[latestFundingPaymentPointer] += premium;

      emit CalculateFunding(
        msg.sender,
        amount,
        strike,
        premium,
        latestFundingPaymentPointer
      );
     i++;
} while ( i < strikes.length);
```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L406

#### <ins>Forge test</ins>

Running forge test snapshot comparison report:

Compiling 13 files with 0.8.19
Solc 0.8.19 finished in 8.42s
Compiler run $\textcolor{green}{\textsf{successful!}}$

Running 1 test for tests/RdpxDecayingBondsTest.t.sol:RdpxDecayingBondsTest
$\textcolor{green}{\textsf{[PASS]}}$ testMint() (gas: 205469)

Test result: $\textcolor{green}{\textsf{ok}}$. $\textcolor{green}{\textsf{1}}$ passed; $\textcolor{red}{\textsf{0}}$ failed; $\textcolor{yellow}{\textsf{0}}$ skipped; finished in 637.67µs

Running 7 tests for tests/perp-vault/Admin.t.sol:Admin
$\textcolor{green}{\textsf{[PASS]}}$ testContractWhitelist() (gas: 1635945)

$\textcolor{green}{\textsf{[PASS]}}$ testEmergencyWithdraw() (gas: 118505)

$\textcolor{green}{\textsf{[PASS]}}$ testEmergencyWithdrawNonNative() (gas: 106904)

$\textcolor{green}{\textsf{[PASS]}}$ testPause() (gas: 39407)

$\textcolor{green}{\textsf{[PASS]}}$ testSetAddresses() (gas: 77431)

$\textcolor{green}{\textsf{[PASS]}}$ testUnpause() (gas: 26164)

$\textcolor{green}{\textsf{[PASS]}}$ testUpdateFundingDuration() (gas: 49303)

Test result: $\textcolor{green}{\textsf{ok}}$. $\textcolor{green}{\textsf{7}}$ passed; $\textcolor{red}{\textsf{0}}$ failed; $\textcolor{yellow}{\textsf{0}}$ skipped; finished in 1.00s

Running 1 test for tests/perp-vault/Integration.t.sol:Integration
$\textcolor{green}{\textsf{[PASS]}}$ testIntegration() (gas: 1812321)

Test result: $\textcolor{green}{\textsf{ok}}$. $\textcolor{green}{\textsf{1}}$ passed; $\textcolor{red}{\textsf{0}}$ failed; $\textcolor{yellow}{\textsf{0}}$ skipped; finished in 1.01s

Running 15 tests for tests/perp-vault/Unit.t.sol:Unit
$\textcolor{green}{\textsf{[PASS]}}$ testCalculatePremium() (gas: 6017)

$\textcolor{green}{\textsf{[PASS]}}$ testDeposit() (gas: 222488)

$\textcolor{green}{\textsf{[PASS]}}$ testDepositZero() (gas: 28008)

$\textcolor{green}{\textsf{[PASS]}}$ testFunding() (gas: 676857)

$\textcolor{green}{\textsf{[PASS]}}$ testFundingAccruedForOneOption() (gas: 710239)

$\textcolor{green}{\textsf{[PASS]}}$ testGetVolatility() (gas: 10922)

$\textcolor{green}{\textsf{[PASS]}}$ testMockups() (gas: 1230469)

$\textcolor{green}{\textsf{[PASS]}}$ testPurchase() (gas: 960192)

$\textcolor{green}{\textsf{[PASS]}}$ testRedeem() (gas: 907966)

$\textcolor{green}{\textsf{[PASS]}}$ testRedeemOnBehalfOf() (gas: 907071)

$\textcolor{green}{\textsf{[PASS]}}$ testRoundUp() (gas: 9256)

$\textcolor{green}{\textsf{[PASS]}}$ testSettle() (gas: 749099)

$\textcolor{green}{\textsf{[PASS]}}$ testSupportsInterface() (gas: 6079)

$\textcolor{green}{\textsf{[PASS]}}$ testUnusedOverrides() (gas: 29971)

$\textcolor{green}{\textsf{[PASS]}}$ testUpdateFundingPaymentPointer() (gas: 630209)

Test result: $\textcolor{green}{\textsf{ok}}$. $\textcolor{green}{\textsf{15}}$ passed; $\textcolor{red}{\textsf{0}}$ failed; $\textcolor{yellow}{\textsf{0}}$ skipped; finished in 1.01s

Running 1 test for tests/rdpxV2-core/Integration.t.sol:Integration
$\textcolor{green}{\textsf{[PASS]}}$ testIntegration() (gas: 5136984)

Test result: $\textcolor{green}{\textsf{ok}}$. $\textcolor{green}{\textsf{1}}$ passed; $\textcolor{red}{\textsf{0}}$ failed; $\textcolor{yellow}{\textsf{0}}$ skipped; finished in 1.02s

Running 3 tests for tests/rdpxV2-core/Periphery.t.sol:Periphery
$\textcolor{green}{\textsf{[PASS]}}$ testReLpContract() (gas: 4024129)

$\textcolor{green}{\textsf{[PASS]}}$ testUniV3Amo() (gas: 8367681)

$\textcolor{green}{\textsf{[PASS]}}$ testV2Amo() (gas: 2374010)

Test result: $\textcolor{green}{\textsf{ok}}$. $\textcolor{green}{\textsf{3}}$ passed; $\textcolor{red}{\textsf{0}}$ failed; $\textcolor{yellow}{\textsf{0}}$ skipped; finished in 1.03s

Running 1 test for tests/rdpxV2-core/Admin.t.sol:Admin
$\textcolor{green}{\textsf{[PASS]}}$ testAdminFunctions() (gas: 332450)

Test result: $\textcolor{green}{\textsf{ok}}$. $\textcolor{green}{\textsf{1}}$ passed; $\textcolor{red}{\textsf{0}}$ failed; $\textcolor{yellow}{\textsf{0}}$ skipped; finished in 1.03s

Running 11 tests for tests/rdpxV2-core/Unit.t.sol:Unit
$\textcolor{green}{\textsf{[PASS]}}$ testAddToDelegate() (gas: 342041)

$\textcolor{green}{\textsf{[PASS]}}$ testBond() (gas: 3651136)

$\textcolor{green}{\textsf{[PASS]}}$ testBondWithDelegate() (gas: 3296269)

$\textcolor{green}{\textsf{[PASS]}}$ testBondWithDelegateMintDecayRiptide() (gas: 1281985)

$\textcolor{green}{\textsf{[PASS]}}$ testBondWithoutOptions() (gas: 3907728)

$\textcolor{green}{\textsf{[PASS]}}$ testPayFunding() (gas: 2385788)

$\textcolor{green}{\textsf{[PASS]}}$ testRedeem() (gas: 958120)

$\textcolor{green}{\textsf{[PASS]}}$ testSettle() (gas: 2363059)

$\textcolor{green}{\textsf{[PASS]}}$ testUpperDepeg() (gas: 325486)

$\textcolor{green}{\textsf{[PASS]}}$ testWithdraw() (gas: 1413418)

$\textcolor{green}{\textsf{[PASS]}}$ testlowerDepeg() (gas: 622344)

Test result: $\textcolor{green}{\textsf{ok}}$. $\textcolor{green}{\textsf{11}}$ passed; $\textcolor{red}{\textsf{0}}$ failed; $\textcolor{yellow}{\textsf{0}}$ skipped; finished in 1.03s
 
Ran 8 test suites: $\textcolor{green}{\textsf{40}}$ tests passed, $\textcolor{red}{\textsf{0}}$ failed, $\textcolor{yellow}{\textsf{0}}$ skipped (40 total tests)
testMint() (gas: 0 (0.000%))
 
testContractWhitelist() (gas: 0 (0.000%))
 
testPause() (gas: 0 (0.000%))
 
testSetAddresses() (gas: 0 (0.000%))
 
testUnpause() (gas: 0 (0.000%))
 
testUpdateFundingDuration() (gas: 0 (0.000%))
 
testCalculatePremium() (gas: 0 (0.000%))
 
testDeposit() (gas: 0 (0.000%))
 
testDepositZero() (gas: 0 (0.000%))
 
testFunding() (gas: 0 (0.000%))
 
testFundingAccruedForOneOption() (gas: 0 (0.000%))
 
testGetVolatility() (gas: 0 (0.000%))
 
testMockups() (gas: 0 (0.000%))
 
testPurchase() (gas: 0 (0.000%))
 
testRoundUp() (gas: 0 (0.000%))
 
testSupportsInterface() (gas: 0 (0.000%))
 
testUnusedOverrides() (gas: 0 (0.000%))
 
testUpdateFundingPaymentPointer() (gas: 0 (0.000%))
 
testAdminFunctions() (gas: 0 (0.000%))
 
testUniV3Amo() (gas: 0 (0.000%))
 
testAddToDelegate() (gas: 0 (0.000%))
 
testBond() (gas: 0 (0.000%))
 
testBondWithDelegate() (gas: 0 (0.000%))
 
testBondWithDelegateMintDecayRiptide() (gas: 0 (0.000%))
 
testBondWithoutOptions() (gas: 0 (0.000%))
 
testPayFunding() (gas: 0 (0.000%))
 
testUpperDepeg() (gas: 0 (0.000%))
 
testWithdraw() (gas: 0 (0.000%))
 
testlowerDepeg() (gas: 0 (0.000%))
 
testReLpContract() (gas: $\textcolor{green}{\textsf{-1207}}$ ($\textcolor{green}{\textsf{-0.030%}}$))
 
testEmergencyWithdraw() (gas: $\textcolor{green}{\textsf{-44}}$ ($\textcolor{green}{\textsf{-0.037%}}$))
 
testEmergencyWithdrawNonNative() (gas: $\textcolor{green}{\textsf{-44}}$ ($\textcolor{green}{\textsf{-0.041%}}$))
 
testV2Amo() (gas: $\textcolor{green}{\textsf{-1207}}$ ($\textcolor{green}{\textsf{-0.051%}}$))
 
Overall gas change: $\textcolor{green}{\textsf{-2502}}$ ($\textcolor{green}{\textsf{-0.006%}}$)


</details>


### <a href="#gas-summary">[EXPGAS&#x2011;2]</a><a name="EXPGAS&#x2011;2"> `unchecked {}` can be used on the division of two `uints` in order to save gas

Make such found divisions are unchecked when ensured it is safe to do so.

#### <ins>Proof Of Concept</ins>


```solidity
File: RdpxV2Core.sol

605: uint256 rdpxRequiredInWeth = (_rdpxRequired * getRdpxPrice()) / 1e8;
```

Optimize by changing to:

```solidity
uint256 rdpxRequiredInWeth;
unchecked {
  rdpxRequiredInWeth = (_rdpxRequired * getRdpxPrice()) / 1e8;
}
```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L605

```solidity
File: RdpxV2Core.sol

669: uint256 rdpxAmountInWeth = (_rdpxAmount * getRdpxPrice()) / 1e8;
```

Optimize by changing to:

```solidity
uint256 rdpxAmountInWeth;
unchecked {
  rdpxAmountInWeth = (_rdpxAmount * getRdpxPrice()) / 1e8;
}
```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L669

```solidity
File: PerpetualAtlanticVault.sol

270: uint256 strike = roundUp(currentPrice - (currentPrice / 4));
```

Optimize by changing to:

```solidity
uint256 strike;
unchecked {
  strike = roundUp(currentPrice - (currentPrice / 4));
}
```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L270

```solidity
File: PerpetualAtlanticVault.sol

276: uint256 requiredCollateral = (amount * strike) / 1e8;
```

Optimize by changing to:

```solidity
uint256 requiredCollateral;
unchecked {
  requiredCollateral = (amount * strike) / 1e8;
}
```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L276

#### <ins>Forge test</ins>

Running forge test snapshot comparison report:

Compiling 10 files with 0.8.19
Solc 0.8.19 finished in 8.18s
Compiler run $\textcolor{green}{\textsf{successful!}}$

Running 1 test for tests/RdpxDecayingBondsTest.t.sol:RdpxDecayingBondsTest
$\textcolor{green}{\textsf{[PASS]}}$ testMint() (gas: 205469)

Test result: $\textcolor{green}{\textsf{ok}}$. $\textcolor{green}{\textsf{1}}$ passed; $\textcolor{red}{\textsf{0}}$ failed; $\textcolor{yellow}{\textsf{0}}$ skipped; finished in 1.30ms

Running 1 test for tests/perp-vault/Integration.t.sol:Integration
$\textcolor{green}{\textsf{[PASS]}}$ testIntegration() (gas: 1811916)

Test result: $\textcolor{green}{\textsf{ok}}$. $\textcolor{green}{\textsf{1}}$ passed; $\textcolor{red}{\textsf{0}}$ failed; $\textcolor{yellow}{\textsf{0}}$ skipped; finished in 978.87ms

Running 7 tests for tests/perp-vault/Admin.t.sol:Admin
$\textcolor{green}{\textsf{[PASS]}}$ testContractWhitelist() (gas: 1635945)

$\textcolor{green}{\textsf{[PASS]}}$ testEmergencyWithdraw() (gas: 118549)

$\textcolor{green}{\textsf{[PASS]}}$ testEmergencyWithdrawNonNative() (gas: 106948)

$\textcolor{green}{\textsf{[PASS]}}$ testPause() (gas: 39407)

$\textcolor{green}{\textsf{[PASS]}}$ testSetAddresses() (gas: 77431)

$\textcolor{green}{\textsf{[PASS]}}$ testUnpause() (gas: 26164)

$\textcolor{green}{\textsf{[PASS]}}$ testUpdateFundingDuration() (gas: 49303)

Test result: $\textcolor{green}{\textsf{ok}}$. $\textcolor{green}{\textsf{7}}$ passed; $\textcolor{red}{\textsf{0}}$ failed; $\textcolor{yellow}{\textsf{0}}$ skipped; finished in 984.88ms

Running 15 tests for tests/perp-vault/Unit.t.sol:Unit
$\textcolor{green}{\textsf{[PASS]}}$ testCalculatePremium() (gas: 6017)

$\textcolor{green}{\textsf{[PASS]}}$ testDeposit() (gas: 222488)

$\textcolor{green}{\textsf{[PASS]}}$ testDepositZero() (gas: 28008)

$\textcolor{green}{\textsf{[PASS]}}$ testFunding() (gas: 676604)

$\textcolor{green}{\textsf{[PASS]}}$ testFundingAccruedForOneOption() (gas: 709986)

$\textcolor{green}{\textsf{[PASS]}}$ testGetVolatility() (gas: 10922)

$\textcolor{green}{\textsf{[PASS]}}$ testMockups() (gas: 1229963)

$\textcolor{green}{\textsf{[PASS]}}$ testPurchase() (gas: 959180)

$\textcolor{green}{\textsf{[PASS]}}$ testRedeem() (gas: 907764)

$\textcolor{green}{\textsf{[PASS]}}$ testRedeemOnBehalfOf() (gas: 906868)

$\textcolor{green}{\textsf{[PASS]}}$ testRoundUp() (gas: 9256)

$\textcolor{green}{\textsf{[PASS]}}$ testSettle() (gas: 748846)

$\textcolor{green}{\textsf{[PASS]}}$ testSupportsInterface() (gas: 6079)

$\textcolor{green}{\textsf{[PASS]}}$ testUnusedOverrides() (gas: 29971)

$\textcolor{green}{\textsf{[PASS]}}$ testUpdateFundingPaymentPointer() (gas: 629956)

Test result: $\textcolor{green}{\textsf{ok}}$. $\textcolor{green}{\textsf{15}}$ passed; $\textcolor{red}{\textsf{0}}$ failed; $\textcolor{yellow}{\textsf{0}}$ skipped; finished in 985.22ms

Running 1 test for tests/rdpxV2-core/Admin.t.sol:Admin
$\textcolor{green}{\textsf{[PASS]}}$ testAdminFunctions() (gas: 332450)

Test result: $\textcolor{green}{\textsf{ok}}$. $\textcolor{green}{\textsf{1}}$ passed; $\textcolor{red}{\textsf{0}}$ failed; $\textcolor{yellow}{\textsf{0}}$ skipped; finished in 991.22ms

Running 1 test for tests/rdpxV2-core/Integration.t.sol:Integration
$\textcolor{green}{\textsf{[PASS]}}$ testIntegration() (gas: 5133864)

Test result: $\textcolor{green}{\textsf{ok}}$. $\textcolor{green}{\textsf{1}}$ passed; $\textcolor{red}{\textsf{0}}$ failed; $\textcolor{yellow}{\textsf{0}}$ skipped; finished in 996.59ms

Running 11 tests for tests/rdpxV2-core/Unit.t.sol:Unit
$\textcolor{green}{\textsf{[PASS]}}$ testAddToDelegate() (gas: 342041)

$\textcolor{green}{\textsf{[PASS]}}$ testBond() (gas: 3649244)

$\textcolor{green}{\textsf{[PASS]}}$ testBondWithDelegate() (gas: 3294289)

$\textcolor{green}{\textsf{[PASS]}}$ testBondWithDelegateMintDecayRiptide() (gas: 1281611)

$\textcolor{green}{\textsf{[PASS]}}$ testBondWithoutOptions() (gas: 3905715)

$\textcolor{green}{\textsf{[PASS]}}$ testPayFunding() (gas: 2384666)

$\textcolor{green}{\textsf{[PASS]}}$ testRedeem() (gas: 957746)

$\textcolor{green}{\textsf{[PASS]}}$ testSettle() (gas: 2361955)

$\textcolor{green}{\textsf{[PASS]}}$ testUpperDepeg() (gas: 325486)

$\textcolor{green}{\textsf{[PASS]}}$ testWithdraw() (gas: 1412923)

$\textcolor{green}{\textsf{[PASS]}}$ testlowerDepeg() (gas: 622344)

Test result: $\textcolor{green}{\textsf{ok}}$. $\textcolor{green}{\textsf{11}}$ passed; $\textcolor{red}{\textsf{0}}$ failed; $\textcolor{yellow}{\textsf{0}}$ skipped; finished in 1.00s

Running 3 tests for tests/rdpxV2-core/Periphery.t.sol:Periphery
$\textcolor{green}{\textsf{[PASS]}}$ testReLpContract() (gas: 4024962)

$\textcolor{green}{\textsf{[PASS]}}$ testUniV3Amo() (gas: 8367681)

$\textcolor{green}{\textsf{[PASS]}}$ testV2Amo() (gas: 2375217)

Test result: $\textcolor{green}{\textsf{ok}}$. $\textcolor{green}{\textsf{3}}$ passed; $\textcolor{red}{\textsf{0}}$ failed; $\textcolor{yellow}{\textsf{0}}$ skipped; finished in 1.00s
 
Ran 8 test suites: $\textcolor{green}{\textsf{40}}$ tests passed, $\textcolor{red}{\textsf{0}}$ failed, $\textcolor{yellow}{\textsf{0}}$ skipped (40 total tests)
testMint() (gas: 0 (0.000%))
 
testContractWhitelist() (gas: 0 (0.000%))
 
testEmergencyWithdraw() (gas: 0 (0.000%))
 
testEmergencyWithdrawNonNative() (gas: 0 (0.000%))
 
testPause() (gas: 0 (0.000%))
 
testSetAddresses() (gas: 0 (0.000%))
 
testUnpause() (gas: 0 (0.000%))
 
testUpdateFundingDuration() (gas: 0 (0.000%))
 
testCalculatePremium() (gas: 0 (0.000%))
 
testDeposit() (gas: 0 (0.000%))
 
testDepositZero() (gas: 0 (0.000%))
 
testGetVolatility() (gas: 0 (0.000%))
 
testRoundUp() (gas: 0 (0.000%))
 
testSupportsInterface() (gas: 0 (0.000%))
 
testUnusedOverrides() (gas: 0 (0.000%))
 
testAdminFunctions() (gas: 0 (0.000%))
 
testUniV3Amo() (gas: 0 (0.000%))
 
testV2Amo() (gas: 0 (0.000%))
 
testAddToDelegate() (gas: 0 (0.000%))
 
testUpperDepeg() (gas: 0 (0.000%))
 
testlowerDepeg() (gas: 0 (0.000%))
 
testReLpContract() (gas: $\textcolor{green}{\textsf{-374}}$ ($\textcolor{green}{\textsf{-0.009%}}$))
 
testBondWithDelegateMintDecayRiptide() (gas: $\textcolor{green}{\textsf{-374}}$ ($\textcolor{green}{\textsf{-0.029%}}$))
 
testWithdraw() (gas: $\textcolor{green}{\textsf{-495}}$ ($\textcolor{green}{\textsf{-0.035%}}$))
 
testFundingAccruedForOneOption() (gas: $\textcolor{green}{\textsf{-253}}$ ($\textcolor{green}{\textsf{-0.036%}}$))
 
testFunding() (gas: $\textcolor{green}{\textsf{-253}}$ ($\textcolor{green}{\textsf{-0.037%}}$))
 
testUpdateFundingPaymentPointer() (gas: $\textcolor{green}{\textsf{-253}}$ ($\textcolor{green}{\textsf{-0.040%}}$))
 
testMockups() (gas: $\textcolor{green}{\textsf{-506}}$ ($\textcolor{green}{\textsf{-0.041%}}$))
 
testPayFunding() (gas: $\textcolor{green}{\textsf{-1122}}$ ($\textcolor{green}{\textsf{-0.047%}}$))
 
testBondWithoutOptions() (gas: $\textcolor{green}{\textsf{-2013}}$ ($\textcolor{green}{\textsf{-0.052%}}$))
 
testBond() (gas: $\textcolor{green}{\textsf{-1892}}$ ($\textcolor{green}{\textsf{-0.052%}}$))
 
testBondWithDelegate() (gas: $\textcolor{green}{\textsf{-1980}}$ ($\textcolor{green}{\textsf{-0.060%}}$))
 
testPurchase() (gas: $\textcolor{green}{\textsf{-1012}}$ ($\textcolor{green}{\textsf{-0.105%}}$))
 
Overall gas change: $\textcolor{green}{\textsf{-10527}}$ ($\textcolor{green}{\textsf{-0.027%}}$)



### <a href="#gas-summary">[EXPGAS&#x2011;3]</a><a name="EXPGAS&#x2011;3"> Using `private` rather than `public` for constants, saves gas

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: UniV2LiquidityAmo.sol

48: uint256 public constant DEFAULT_PRECISION = 1e8;

```

Optimize by changing to:

```solidity
uint256 private constant DEFAULT_PRECISION = 1e8;
```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/amo/UniV2LiquidityAmo.sol#L48

```solidity
File: RdpxV2Core.sol

85: uint256 public constant DEFAULT_PRECISION = 1e8;

```

Optimize by changing to:

```solidity
uint256 private constant DEFAULT_PRECISION = 1e8;
```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L85

```solidity
File: RdpxV2Core.sol

88: uint256 public constant RDPX_RATIO_PERCENTAGE = 25 * DEFAULT_PRECISION;

```

Optimize by changing to:

```solidity
uint256 private constant RDPX_RATIO_PERCENTAGE = 25 * DEFAULT_PRECISION;
```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L88

```solidity
File: RdpxV2Core.sol

91: uint256 public constant ETH_RATIO_PERCENTAGE = 75 * DEFAULT_PRECISION;

```

Optimize by changing to:

```solidity
uint256 private constant ETH_RATIO_PERCENTAGE = 75 * DEFAULT_PRECISION;
```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/core/RdpxV2Core.sol#L91

```solidity
File: ReLPContract.sol

67: uint256 public constant DEFAULT_PRECISION = 1e8;

```

Optimize by changing to:

```solidity
uint256 private constant DEFAULT_PRECISION = 1e8;
```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/reLP/ReLPContract.sol#L67

```solidity
File: DpxEthToken.sol

19: bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");

```

Optimize by changing to:

```solidity
bytes32 private constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/dpxETH/DpxEthToken.sol#L19

```solidity
File: PerpetualAtlanticVault.sol

45: bytes32 public constant MANAGER_ROLE = keccak256("MANAGER_ROLE");

```

Optimize by changing to:

```solidity
bytes32 private constant MANAGER_ROLE = keccak256("MANAGER_ROLE");
```

https://github.com/code-423n4/2023-08-dopex/tree/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L45

#### <ins>Forge test</ins>

Running forge test snapshot comparison report:

Compiling 13 files with 0.8.19
Solc 0.8.19 finished in 8.38s
Compiler run $\textcolor{green}{\textsf{successful!}}$

Running 1 test for tests/RdpxDecayingBondsTest.t.sol:RdpxDecayingBondsTest
$\textcolor{green}{\textsf{[PASS]}}$ testMint() (gas: 205469)

Test result: $\textcolor{green}{\textsf{ok}}$. $\textcolor{green}{\textsf{1}}$ passed; $\textcolor{red}{\textsf{0}}$ failed; $\textcolor{yellow}{\textsf{0}}$ skipped; finished in 667.67µs

Running 7 tests for tests/perp-vault/Admin.t.sol:Admin
$\textcolor{green}{\textsf{[PASS]}}$ testContractWhitelist() (gas: 1635945)

$\textcolor{green}{\textsf{[PASS]}}$ testEmergencyWithdraw() (gas: 118549)

$\textcolor{green}{\textsf{[PASS]}}$ testEmergencyWithdrawNonNative() (gas: 106948)

$\textcolor{green}{\textsf{[PASS]}}$ testPause() (gas: 39407)

$\textcolor{green}{\textsf{[PASS]}}$ testSetAddresses() (gas: 77431)

$\textcolor{green}{\textsf{[PASS]}}$ testUnpause() (gas: 26164)

$\textcolor{green}{\textsf{[PASS]}}$ testUpdateFundingDuration() (gas: 49303)

Test result: $\textcolor{green}{\textsf{ok}}$. $\textcolor{green}{\textsf{7}}$ passed; $\textcolor{red}{\textsf{0}}$ failed; $\textcolor{yellow}{\textsf{0}}$ skipped; finished in 976.84ms

Running 1 test for tests/perp-vault/Integration.t.sol:Integration
$\textcolor{green}{\textsf{[PASS]}}$ testIntegration() (gas: 1812321)

Test result: $\textcolor{green}{\textsf{ok}}$. $\textcolor{green}{\textsf{1}}$ passed; $\textcolor{red}{\textsf{0}}$ failed; $\textcolor{yellow}{\textsf{0}}$ skipped; finished in 980.25ms

Running 15 tests for tests/perp-vault/Unit.t.sol:Unit
$\textcolor{green}{\textsf{[PASS]}}$ testCalculatePremium() (gas: 6017)

$\textcolor{green}{\textsf{[PASS]}}$ testDeposit() (gas: 222488)

$\textcolor{green}{\textsf{[PASS]}}$ testDepositZero() (gas: 28008)

$\textcolor{green}{\textsf{[PASS]}}$ testFunding() (gas: 676857)

$\textcolor{green}{\textsf{[PASS]}}$ testFundingAccruedForOneOption() (gas: 710239)

$\textcolor{green}{\textsf{[PASS]}}$ testGetVolatility() (gas: 10922)

$\textcolor{green}{\textsf{[PASS]}}$ testMockups() (gas: 1230469)

$\textcolor{green}{\textsf{[PASS]}}$ testPurchase() (gas: 960170)

$\textcolor{green}{\textsf{[PASS]}}$ testRedeem() (gas: 907966)

$\textcolor{green}{\textsf{[PASS]}}$ testRedeemOnBehalfOf() (gas: 907071)

$\textcolor{green}{\textsf{[PASS]}}$ testRoundUp() (gas: 9256)

$\textcolor{green}{\textsf{[PASS]}}$ testSettle() (gas: 749099)

$\textcolor{green}{\textsf{[PASS]}}$ testSupportsInterface() (gas: 6079)

$\textcolor{green}{\textsf{[PASS]}}$ testUnusedOverrides() (gas: 29971)

$\textcolor{green}{\textsf{[PASS]}}$ testUpdateFundingPaymentPointer() (gas: 630209)

Test result: $\textcolor{green}{\textsf{ok}}$. $\textcolor{green}{\textsf{15}}$ passed; $\textcolor{red}{\textsf{0}}$ failed; $\textcolor{yellow}{\textsf{0}}$ skipped; finished in 986.37ms

Running 1 test for tests/rdpxV2-core/Admin.t.sol:Admin
$\textcolor{green}{\textsf{[PASS]}}$ testAdminFunctions() (gas: 332603)

Test result: $\textcolor{green}{\textsf{ok}}$. $\textcolor{green}{\textsf{1}}$ passed; $\textcolor{red}{\textsf{0}}$ failed; $\textcolor{yellow}{\textsf{0}}$ skipped; finished in 988.74ms

Running 11 tests for tests/rdpxV2-core/Unit.t.sol:Unit
$\textcolor{green}{\textsf{[PASS]}}$ testAddToDelegate() (gas: 341975)

$\textcolor{green}{\textsf{[PASS]}}$ testBond() (gas: 3651358)

$\textcolor{green}{\textsf{[PASS]}}$ testBondWithDelegate() (gas: 3295851)

$\textcolor{green}{\textsf{[PASS]}}$ testBondWithDelegateMintDecayRiptide() (gas: 1281941)

$\textcolor{green}{\textsf{[PASS]}}$ testBondWithoutOptions() (gas: 3907992)

$\textcolor{green}{\textsf{[PASS]}}$ testPayFunding() (gas: 2385803)

$\textcolor{green}{\textsf{[PASS]}}$ testRedeem() (gas: 957898)

$\textcolor{green}{\textsf{[PASS]}}$ testSettle() (gas: 2362916)

$\textcolor{green}{\textsf{[PASS]}}$ testUpperDepeg() (gas: 325353)

$\textcolor{green}{\textsf{[PASS]}}$ testWithdraw() (gas: 1413330)

$\textcolor{green}{\textsf{[PASS]}}$ testlowerDepeg() (gas: 621951)

Test result: $\textcolor{green}{\textsf{ok}}$. $\textcolor{green}{\textsf{11}}$ passed; $\textcolor{red}{\textsf{0}}$ failed; $\textcolor{yellow}{\textsf{0}}$ skipped; finished in 998.75ms

Running 1 test for tests/rdpxV2-core/Integration.t.sol:Integration
$\textcolor{green}{\textsf{[PASS]}}$ testIntegration() (gas: 5137193)

Test result: $\textcolor{green}{\textsf{ok}}$. $\textcolor{green}{\textsf{1}}$ passed; $\textcolor{red}{\textsf{0}}$ failed; $\textcolor{yellow}{\textsf{0}}$ skipped; finished in 999.70ms

Running 3 tests for tests/rdpxV2-core/Periphery.t.sol:Periphery
$\textcolor{green}{\textsf{[PASS]}}$ testReLpContract() (gas: 4020763)

$\textcolor{green}{\textsf{[PASS]}}$ testUniV3Amo() (gas: 8367609)

$\textcolor{green}{\textsf{[PASS]}}$ testV2Amo() (gas: 2370757)

Test result: $\textcolor{green}{\textsf{ok}}$. $\textcolor{green}{\textsf{3}}$ passed; $\textcolor{red}{\textsf{0}}$ failed; $\textcolor{yellow}{\textsf{0}}$ skipped; finished in 1.00s
 
Ran 8 test suites: $\textcolor{green}{\textsf{40}}$ tests passed, $\textcolor{red}{\textsf{0}}$ failed, $\textcolor{yellow}{\textsf{0}}$ skipped (40 total tests)
testMint() (gas: 0 (0.000%))
 
testContractWhitelist() (gas: 0 (0.000%))
 
testEmergencyWithdraw() (gas: 0 (0.000%))
 
testEmergencyWithdrawNonNative() (gas: 0 (0.000%))
 
testPause() (gas: 0 (0.000%))
 
testSetAddresses() (gas: 0 (0.000%))
 
testUnpause() (gas: 0 (0.000%))
 
testUpdateFundingDuration() (gas: 0 (0.000%))
 
testCalculatePremium() (gas: 0 (0.000%))
 
testDeposit() (gas: 0 (0.000%))
 
testDepositZero() (gas: 0 (0.000%))
 
testFunding() (gas: 0 (0.000%))
 
testFundingAccruedForOneOption() (gas: 0 (0.000%))
 
testGetVolatility() (gas: 0 (0.000%))
 
testMockups() (gas: 0 (0.000%))
 
testRoundUp() (gas: 0 (0.000%))
 
testSupportsInterface() (gas: 0 (0.000%))
 
testUnusedOverrides() (gas: 0 (0.000%))
 
testUpdateFundingPaymentPointer() (gas: 0 (0.000%))
 
testPayFunding() (gas: $\textcolor{red}{\textsf{15}}$ ($\textcolor{red}{\textsf{0.001%}}$))
 
testUniV3Amo() (gas: $\textcolor{green}{\textsf{-72}}$ ($\textcolor{green}{\textsf{-0.001%}}$))
 
testPurchase() (gas: $\textcolor{green}{\textsf{-22}}$ ($\textcolor{green}{\textsf{-0.002%}}$))
 
testBondWithDelegateMintDecayRiptide() (gas: $\textcolor{green}{\textsf{-44}}$ ($\textcolor{green}{\textsf{-0.003%}}$))
 
testBond() (gas: $\textcolor{red}{\textsf{222}}$ ($\textcolor{red}{\textsf{0.006%}}$))
 
testWithdraw() (gas: $\textcolor{green}{\textsf{-88}}$ ($\textcolor{green}{\textsf{-0.006%}}$))
 
testBondWithoutOptions() (gas: $\textcolor{red}{\textsf{264}}$ ($\textcolor{red}{\textsf{0.007%}}$))
 
testBondWithDelegate() (gas: $\textcolor{green}{\textsf{-418}}$ ($\textcolor{green}{\textsf{-0.013%}}$))
 
testAddToDelegate() (gas: $\textcolor{green}{\textsf{-66}}$ ($\textcolor{green}{\textsf{-0.019%}}$))
 
testUpperDepeg() (gas: $\textcolor{green}{\textsf{-133}}$ ($\textcolor{green}{\textsf{-0.041%}}$))
 
testAdminFunctions() (gas: $\textcolor{red}{\textsf{153}}$ ($\textcolor{red}{\textsf{0.046%}}$))
 
testlowerDepeg() (gas: $\textcolor{green}{\textsf{-393}}$ ($\textcolor{green}{\textsf{-0.063%}}$))
 
testReLpContract() (gas: $\textcolor{green}{\textsf{-4573}}$ ($\textcolor{green}{\textsf{-0.114%}}$))
 
testV2Amo() (gas: $\textcolor{green}{\textsf{-4460}}$ ($\textcolor{green}{\textsf{-0.188%}}$))
 
Overall gas change: $\textcolor{green}{\textsf{-9615}}$ ($\textcolor{green}{\textsf{-0.025%}}$)


</details>

#### <ins>Invalid instances</ins>

Reports that contain these instances should be invalidated as they either break the foundry test runs. increase the overall gas usage or 0 gas optimzation:

```solidity
File: PerpetualAtlanticVault.sol

Error: Error (9582): Member "RDPXV2CORE_ROLE" not found or not visible after argument-dependent lookup in contract PerpetualAtlanticVault.

bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
```

```solidity
File: DpxEthToken.sol

Error: Error (9582): Member "MINTER_ROLE" not found or not visible after argument-dependent lookup in contract DpxEthToken.

bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
```

```solidity
File: RdpxV2Bond.sol

Error: Error (9582): Member "MINTER_ROLE" not found or not visible after argument-dependent lookup in contract RdpxV2Bond.

bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
```

```solidity
File: DpxEthToken.sol

Error: Error (9582): Member "BURNER_ROLE" not found or not visible after argument-dependent lookup in contract DpxEthToken.

bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
```

```solidity
File: RdpxDecayingBonds.sol

Error: Error (9582): Member "RDPXV2CORE_ROLE" not found or not visible after argument-dependent lookup in contract RdpxDecayingBonds.

bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
```

```solidity
File: ReLPContract.sol

Error: Error (9582): Member "RDPXV2CORE_ROLE" not found or not visible after argument-dependent lookup in contract ReLPContract.

bytes32 public constant RDPXV2CORE_ROLE = keccak256("RDPXV2CORE_ROLE");
```

```solidity
File: RdpxDecayingBonds.sol

Error: Error (9582): Member "MINTER_ROLE" not found or not visible after argument-dependent lookup in contract RdpxDecayingBonds.

bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
```