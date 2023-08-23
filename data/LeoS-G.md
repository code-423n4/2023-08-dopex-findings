# Summary
|        | Issue | Instances | Gas Saved |
|--------|-------|-----------|-----------|
|[G-01]|Use `calldata` instead of `memory`|7|-15 946|
|[G-02]| Addition to  automated findings [G-07]|1|-210|
|[G-03]|Parts of the solmate library are available more efficiently|1|-104|
|[G-04]|Increase the number of optimiser runs|-|-133 044|



The gas saved column simply adds up the evolution in the snapshot, using the method described in the next section.

# Benchmark
A benchmark is performed on each optimization, using the tests snapshot provided by foundry. This snapshot is based on tests and therefore does not take into account all functions, this loss is accepted. But it is also subject to intrinsic variance. This means that some tests are not relevant to the comparison because they vary too much and are thus not taken into account. These are listed below:
>testSettle
>testIntegration
>testRedeem



# [G-01] Use `calldata` instead of `memory`
Using `calldata` instead of `memory` for function parameters can save gas if the argument is only read in the function. Only instances compilable with a simple type swap are considered valid.

*7 instances*

- [RdpxV2Core.sol#L242](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L242)
- [RdpxV2Core.sol#L271](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L271)
- [RdpxV2Core.sol#L821](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L821)
- [RdpxV2Core.sol#L822](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L822)
- [RdpxV2Core.sol#L1136](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1136)
- [PerpetualAtlanticVault.sol#L316](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L316)
- [PerpetualAtlanticVault.sol#L406](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L406)

Applying this optimisation, those changes appear in the snapshot:

```
testAdminFunctions() (gas: -12 (-0.004%))
testEmergencyWithdrawNonNative() (gas: 32 (0.030%))
testFundingAccruedForOneOption() (gas: -215 (-0.030%))
testRedeemOnBehalfOf() (gas: -396 (-0.044%))
testBondWithDelegateMintDecayRiptide() (gas: -580 (-0.045%))
testWithdraw() (gas: -685 (-0.048%))
testBondWithoutOptions() (gas: -1940 (-0.050%))
testMockups() (gas: -645 (-0.052%))
testPayFunding() (gas: -1264 (-0.053%))
testBond() (gas: -1940 (-0.053%))
testEmergencyWithdraw() (gas: 63 (0.053%))
testBondWithDelegate() (gas: -5066 (-0.154%))
testUpperDepeg() (gas: -970 (-0.298%))
testlowerDepeg() (gas: -2328 (-0.374%))
Overall gas change: -15946 (-0.040%)
```

# [G-02] Correction/Addition to  automated findings [G-07]
The optimization proposed in the automatic report ([G-07] Using `storage` instead of `memory` for structs/arrays saves gas) can be complemented.

## Additional instance

*1 instances*

- [RdpxV2Core.sol#L1272](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1272)

Applying this optimisation, those changes appear in the snapshot:
```
testAddToDelegate() (gas: -210 (-0.061%))
Overall gas change: -210 (-0.001%)
```

# [G-03] Parts of the solmate library are available more efficiently

*1 instance*

- [PerpetualAtlanticVaultLP.sol#L5](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L5)

For some `solmate` functions, more efficient versions are available, such as `ERC20 approve` ([Availible here](https://github.com/Vectorized/solady/blob/main/src/tokens/ERC20.sol#L146-L159)). It is therefore possible to import it to get the most out of it. (a simple copy and paste with no further changes is sufficient).

The security of these other implementations is debatable, howevert
 the `solady` library has a good reputation and no bad history.

Applying this optimisation, those changes appear in the snapshot:
```
testRedeemOnBehalfOf() (gas: -104 (-0.011%))
Overall gas change: -104 (-0.000%)
```
(Long-term impact probably underestimated)

# [G-04] Increase the number of optimiser runs

Here, the number of optimizer runs is set by default to 200. By greatly increasing this number, you'll greatly improve code optimization (more than any of the improvements suggested here). On the negative side, this increases the cost of deployment considerably. That's why a better number is recommended here, but it can be widely discussed by the deployer.

Increasing the number of runs to 1,000,000 (the size limit is still respected) improves test performance enormously, but makes some tests worse. These concern parts of the code that are not intended to be used many times and are therefore amortized over the long term.

Applying this optimisation (100,000,000 runs) those changes appear in the snapshot:
```
testRoundUp() (gas: -13 (-0.140%))
testMint() (gas: -513 (-0.250%))
testBondWithDelegateMintDecayRiptide() (gas: -3697 (-0.288%))
testFundingAccruedForOneOption() (gas: -2634 (-0.371%))
testFunding() (gas: -2515 (-0.372%))
testDepositZero() (gas: -106 (-0.378%))
testWithdraw() (gas: -5464 (-0.387%))
testAddToDelegate() (gas: -1393 (-0.407%))
testMockups() (gas: -5437 (-0.442%))
testUpdateFundingPaymentPointer() (gas: -2906 (-0.461%))
testPayFunding() (gas: -11484 (-0.481%))
testPurchase() (gas: -4746 (-0.494%))
testBondWithDelegate() (gas: -16509 (-0.501%))
testRedeemOnBehalfOf() (gas: -4628 (-0.510%))
testPause() (gas: -204 (-0.518%))
testGetVolatility() (gas: -58 (-0.531%))
testBond() (gas: -20556 (-0.563%))
testUpperDepeg() (gas: -1856 (-0.570%))
testBondWithoutOptions() (gas: -22775 (-0.583%))
testDeposit() (gas: -1365 (-0.614%))
testUnusedOverrides() (gas: -184 (-0.614%))
testCalculatePremium() (gas: -40 (-0.665%))
testlowerDepeg() (gas: -4310 (-0.693%))
testEmergencyWithdrawNonNative() (gas: -780 (-0.729%))
testSetAddresses() (gas: -591 (-0.763%))
testUnpause() (gas: -226 (-0.864%))
testEmergencyWithdraw() (gas: -1068 (-0.901%))
testSupportsInterface() (gas: -124 (-2.040%))
testAdminFunctions() (gas: -14380 (-4.325%))
testUpdateFundingDuration() (gas: -2481 (-5.032%))
testUniV3Amo() (gas: 604537 (7.225%))
testReLpContract() (gas: 462141 (11.481%))
testV2Amo() (gas: 470270 (19.799%))
testContractWhitelist() (gas: 402236 (24.587%))
Overall gas change: 1806141 (4.514%)
```
The overall change doesn't look good, but if the no-relevant in the long term tests are removed, the result is : `-133043 gas change`
