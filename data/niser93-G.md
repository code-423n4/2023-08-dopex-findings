## Summary
        

### Gas Optimizations
|       |Issue  |Instances|Total Gas Saved|
|:-----:|:-----|:-------:|:---------------:|
|[[GO-01](#go-01-using-a-positive-conditional-flow-to-save-a-not-opcode)]|Using a positive conditional flow to save a NOT opcode|1|3|
|[[GO-02](#go-02-it-costs-more-gas-to-initialize-non-constantnon-immutable-state-variables-to-zero-than-to-let-the-default-of-zero-be-applied)]|It costs more gas to initialize non-`constant`/non-`immutable` state variables to zero than to let the default of zero be applied|1|2900|
|[[GO-03](#go-03-using-storage-instead-of-memory-for-structsarrays-saves-gas)]|Using `storage` instead of `memory` for structs/arrays saves gas|3|16800|
|[[GO-04](#go-04-state-variables-should-be-cached-in-stack-variables-rather-than-re-reading-them-from-storage)]|State variables should be cached in stack variables rather than re-reading them from storage|140|16878|
|[[GO-05](#go-05-multiple-accesses-of-a-mapping-should-use-a-local-variable-cache)]|Multiple accesses of a mapping should use a local variable cache|14|1134|
|[[GO-06](#go-06-not-using-the-named-return-variables-anywhere-in-the-function-is-confusing)]|Not using the named return variables anywhere in the function is confusing|1|_|
|[[GO-07](#go-07-add-unchecked--for-subtractions-where-the-operands-cannot-underflow-because-of-a-previous-require-or-if-statement)]|Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if`-statement|6|_|
|[[GO-08](#go-08-state-variable-read-in-a-loop)]|State variable read in a loop|28|_|
|[[GO-09](#go-09-add-unchecked-blocks-for-divisions-where-the-operands-cannot-overflow)]|Add `unchecked` blocks for divisions where the operands cannot overflow|39|6201|
|[[GO-10](#go-10-simple-checks-for-zero-can-be-done-using-assembly-to-save-gas)]|Simple checks for zero can be done using assembly to save gas|56|_|
|[[GO-11](#go-11-state-variables-only-set-in-the-constructor-should-be-declared-immutable)]|State variables only set in the constructor should be declared `immutable`|26|520000|
|[[GO-12](#go-12-avoid-using-state-variable-in-emit)]|Avoid using state variable in emit|4|80000|
|[[GO-13](#go-13-avoid-transferring-0-amount)]|Avoid transferring 0 amount|16|320000|
|[[GO-14](#go-14-using-do-while-loop-instead-of-for-loop-or-while-loop)]|Using do-while loop instead of for loop or while loop|12|_|
|[[GO-15](#go-15-use-constants-instead-of-typeuintxmax)]|Use constants instead of type(uintx).max|17|_|

Total: 444 instances over 15 issues with **963916 gas** saved


Gas totals are estimates based on data from the Ethereum Yellowpaper. The estimates use the lower bounds of ranges and count two iterations of each `for`-loop. All values above are runtime, not deployment, values; deployment values are listed in the individual issue descriptions. The table above as well as its gas numbers do not include any of the excluded findings.
### Gas Optimizations
### [GO-01] Using a positive conditional flow to save a NOT opcode
In order to save some gas (NOT opcode costing 3 gas), switch to a positive statement



```diff
- if(!condition){
-     action1();
- }else{
-     action2();
- }
+ if(condition){
+     action2();
+ }else{
+     action1();
+ }
```            

*Estimated saving:* 3 gas



*There is 1 instance of this issue:*               

```
File: RdpxV2Core.sol

  
  630      if (_bondId != 0) {
  631        (, uint256 expiry, uint256 amount) = IRdpxDecayingBonds(
  632          addresses.rdpxDecayingBonds
  633        ).bonds(_bondId);
  634  
  635        _validate(amount >= _rdpxAmount, 1);
  636        _validate(expiry >= block.timestamp, 2);
  637        _validate(
  638          IRdpxDecayingBonds(addresses.rdpxDecayingBonds).ownerOf(_bondId) ==
  639            msg.sender,
  640          9
  641        );
  642  
  643        IRdpxDecayingBonds(addresses.rdpxDecayingBonds).decreaseAmount(
  644          _bondId,
  645          amount - _rdpxAmount
  646        );
  647  
  648        IRdpxReserve(addresses.rdpxReserve).withdraw(_rdpxAmount);
  649  
  650        reserveAsset[reservesIndex["RDPX"]].tokenBalance += _rdpxAmount;
  651      } else {
  652        // Transfer rDPX and ETH token from user
  653        IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress)
  654          .safeTransferFrom(msg.sender, address(this), _rdpxAmount);
  655  
  656        // burn the rdpx
  657        IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress).burn(
  658          (_rdpxAmount * rdpxBurnPercentage) / 1e10
  659        );
  660  
  661        // transfer the rdpx to the fee distributor
  662        IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress)
  663          .safeTransfer(
  664            addresses.feeDistributor,
  665            (_rdpxAmount * rdpxFeePercentage) / 1e10
  666          );
  667  
  668        // calculate extra rdpx to withdraw to compensate for discount
  669        uint256 rdpxAmountInWeth = (_rdpxAmount * getRdpxPrice()) / 1e8;
  670        uint256 discountReceivedInWeth = _bondAmount -
  671          _wethAmount -
  672          rdpxAmountInWeth;
  673        uint256 extraRdpxToWithdraw = (discountReceivedInWeth * 1e8) /
  674          getRdpxPrice();
  675  
  676        // withdraw the rdpx
  677        IRdpxReserve(addresses.rdpxReserve).withdraw(
  678          _rdpxAmount + extraRdpxToWithdraw
  679        );
  680  
  681        reserveAsset[reservesIndex["RDPX"]].tokenBalance +=
  682          _rdpxAmount +
  683          extraRdpxToWithdraw;
  684      }

```
[[630-684](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L630-L684)]

<details>
<summary>Mitigation proposals</summary>

```diff
File: RdpxV2Core.sol

-   630      if (_bondId != 0) {
-   631        (, uint256 expiry, uint256 amount) = IRdpxDecayingBonds(
-   632          addresses.rdpxDecayingBonds
-   633        ).bonds(_bondId);
-   634  
-   635        _validate(amount >= _rdpxAmount, 1);
-   636        _validate(expiry >= block.timestamp, 2);
-   637        _validate(
-   638          IRdpxDecayingBonds(addresses.rdpxDecayingBonds).ownerOf(_bondId) ==
-   639            msg.sender,
-   640          9
-   641        );
-   642  
-   643        IRdpxDecayingBonds(addresses.rdpxDecayingBonds).decreaseAmount(
-   644          _bondId,
-   645          amount - _rdpxAmount
-   646        );
-   647  
-   648        IRdpxReserve(addresses.rdpxReserve).withdraw(_rdpxAmount);
-   649  
-   650        reserveAsset[reservesIndex["RDPX"]].tokenBalance += _rdpxAmount;
-   651      } else {
-   652        // Transfer rDPX and ETH token from user
-   653        IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress)
-   654          .safeTransferFrom(msg.sender, address(this), _rdpxAmount);
-   655  
-   656        // burn the rdpx
-   657        IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress).burn(
-   658          (_rdpxAmount * rdpxBurnPercentage) / 1e10
-   659        );
-   660  
-   661        // transfer the rdpx to the fee distributor
-   662        IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress)
-   663          .safeTransfer(
-   664            addresses.feeDistributor,
-   665            (_rdpxAmount * rdpxFeePercentage) / 1e10
-   666          );
-   667  
-   668        // calculate extra rdpx to withdraw to compensate for discount
-   669        uint256 rdpxAmountInWeth = (_rdpxAmount * getRdpxPrice()) / 1e8;
-   670        uint256 discountReceivedInWeth = _bondAmount -
-   671          _wethAmount -
-   672          rdpxAmountInWeth;
-   673        uint256 extraRdpxToWithdraw = (discountReceivedInWeth * 1e8) /
-   674          getRdpxPrice();
-   675  
-   676        // withdraw the rdpx
-   677        IRdpxReserve(addresses.rdpxReserve).withdraw(
-   678          _rdpxAmount + extraRdpxToWithdraw
-   679        );
-   680  
-   681        reserveAsset[reservesIndex["RDPX"]].tokenBalance +=
-   682          _rdpxAmount +
-   683          extraRdpxToWithdraw;
-   684      }


+   630         if (_bondId == 0) {
+   631           // Transfer rDPX and ETH token from user
+   632           IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress)
+   633             .safeTransferFrom(msg.sender, address(this), _rdpxAmount);
+   634     
+   635           // burn the rdpx
+   636           IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress).burn(
+   637             (_rdpxAmount * rdpxBurnPercentage) / 1e10
+   638           );
+   639     
+   640           // transfer the rdpx to the fee distributor
+   641           IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress)
+   642             .safeTransfer(
+   643               addresses.feeDistributor,
+   644               (_rdpxAmount * rdpxFeePercentage) / 1e10
+   645             );
+   646     
+   647           // calculate extra rdpx to withdraw to compensate for discount
+   648           uint256 rdpxAmountInWeth = (_rdpxAmount * getRdpxPrice()) / 1e8;
+   649           uint256 discountReceivedInWeth = _bondAmount -
+   650             _wethAmount -
+   651             rdpxAmountInWeth;
+   652           uint256 extraRdpxToWithdraw = (discountReceivedInWeth * 1e8) /
+   653             getRdpxPrice();
+   654     
+   655           // withdraw the rdpx
+   656           IRdpxReserve(addresses.rdpxReserve).withdraw(
+   657             _rdpxAmount + extraRdpxToWithdraw
+   658           );
+   659     
+   660           reserveAsset[reservesIndex["RDPX"]].tokenBalance +=
+   661             _rdpxAmount +
+   662             extraRdpxToWithdraw;
+   663         } else {
+   664           (, uint256 expiry, uint256 amount) = IRdpxDecayingBonds(
+   665             addresses.rdpxDecayingBonds
+   666           ).bonds(_bondId);
+   667     
+   668           _validate(amount >= _rdpxAmount, 1);
+   669           _validate(expiry >= block.timestamp, 2);
+   670           _validate(
+   671             IRdpxDecayingBonds(addresses.rdpxDecayingBonds).ownerOf(_bondId) ==
+   672               msg.sender,
+   673             9
+   674           );
+   675     
+   676           IRdpxDecayingBonds(addresses.rdpxDecayingBonds).decreaseAmount(
+   677             _bondId,
+   678             amount - _rdpxAmount
+   679           );
+   680     
+   681           IRdpxReserve(addresses.rdpxReserve).withdraw(_rdpxAmount);
+   682     
+   683           reserveAsset[reservesIndex["RDPX"]].tokenBalance += _rdpxAmount;
+   684         }

```


</details>

### [GO-02] It costs more gas to initialize non-`constant`/non-`immutable` state variables to zero than to let the default of zero be applied
Not overwriting the default for storage variables avoids a Gsreset ([**2900 gas**](https://gist.github.com/IllIllI000/85b09af2291f626095eb11403ddc74f1)) during deployment


*Estimated saving:* Gsreset [**2900 gas**]



*There is 1 instance of this issue:*               

```
File: PerpetualAtlanticVault.sol

  
  89     uint256 public latestFundingPaymentPointer = 0;

```
[[89](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L89)]
### [GO-03] Using `storage` instead of `memory` for structs/arrays saves gas
When fetching data from a storage location, assigning the data to a `memory` variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (**2100 gas**) for *each* field of the struct/array. If the fields are read from the new memory variable, they incur an additional `MLOAD` rather than a cheap stack read. Instead of declearing the variable with the `memory` keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires `memory`, or if the array/struct is being read from another `memory` array/struct

We condider that structs/arrays have 2 fields


*Estimated saving:* 2100 for each field of struct/array



*There are 4 instances of this issue:*           

```
File: RdpxV2Core.sol

  
  1138     ReserveAsset memory asset = reserveAsset[reservesIndex[_token]];

  
  1272     Delegate memory delegatePosition = delegates[_delegateId];

```
[[1138](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1138), [1272](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1272)]

```
File: UniV3LiquidityAmo.sol

  
  121        Position memory current_position = positions_array[i];

  
  218      Position memory pos = positions_array[positionIndex];

```
[[121](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L121), [218](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L218)]
### [GO-04] State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (**100 gas**) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.


*Estimated saving:* 97



*There are 140 instances of this issue:*              
                
<details>
<summary>Expand finding</summary>

```
File: PerpetualAtlanticVault.sol



  @audit: addresses is called 1+2 times inside function setAddresses() (line 198 and 208, 211)
  
  198      addresses = Addresses({
  
  208        addresses.perpetualAtlanticVaultLP,
  
  211      emit AddressesSet(addresses);


  @audit: latestFundingPaymentPointer is called 1+1 times inside function purchase() (line 303 and 307)
  
  303      fundingPaymentsAccountedFor[latestFundingPaymentPointer] += amount;
  
  307      fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][


  @audit: addresses is called 1+7 times inside function settle() (line 348 and 349, 353, 354, 355, 359, 362, 364)
  
  348        addresses.perpetualAtlanticVaultLP,
  
  349        addresses.rdpxV2Core,
  
  353      IERC20WithBurn(addresses.rdpx).safeTransferFrom(
  
  354        addresses.rdpxV2Core,
  
  355        addresses.perpetualAtlanticVaultLP,
  
  359      IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).subtractLoss(
  
  362      IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP)
  
  364      IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addRdpx(


  @audit: latestFundingPaymentPointer is called 1+5 times inside function payFunding() (line 378 and 385, 387, 391, 392, 395)
  
  378          fundingPaymentsAccountedFor[latestFundingPaymentPointer],
  
  385        totalFundingForEpoch[latestFundingPaymentPointer]
  
  387      _updateFundingRate(totalFundingForEpoch[latestFundingPaymentPointer]);
  
  391        totalFundingForEpoch[latestFundingPaymentPointer],
  
  392        latestFundingPaymentPointer
  
  395      return (totalFundingForEpoch[latestFundingPaymentPointer]);


  @audit: latestFundingPaymentPointer is called 1+7 times inside function calculateFunding() (line 416 and 422, 427, 436, 440, 443, 449, 456)
  
  416          latestFundingPerStrike[strikes[i]] != latestFundingPaymentPointer,
  
  422          fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
  
  427          (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration));
  
  436        latestFundingPerStrike[strike] = latestFundingPaymentPointer;
  
  440        fundingPaymentsAccountedFor[latestFundingPaymentPointer] += amount;
  
  443        fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
  
  449        totalFundingForEpoch[latestFundingPaymentPointer] += premium;
  
  456          latestFundingPaymentPointer


  @audit: lastUpdateTime is called 1+3 times inside function updateFundingPaymentPointer() (line 464 and 467, 469, 471)
  
  464        if (lastUpdateTime < nextFundingPaymentTimestamp()) {
  
  467          uint256 startTime = lastUpdateTime == 0
  
  469            : lastUpdateTime;
  
  471          lastUpdateTime = nextFundingPaymentTimestamp();


  @audit: latestFundingPaymentPointer is called 1+3 times inside function updateFundingPaymentPointer() (line 465 and 489, 493, 494)
  
  465          uint256 currentFundingRate = fundingRates[latestFundingPaymentPointer];
  
  489            latestFundingPaymentPointer
  
  493        latestFundingPaymentPointer += 1;
  
  494        emit FundingPaymentPointerUpdated(latestFundingPaymentPointer);


  @audit: addresses is called 1+1 times inside function updateFundingPaymentPointer() (line 474 and 479)
  
  474            addresses.perpetualAtlanticVaultLP,
  
  479          IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP)


  @audit: latestFundingPaymentPointer is called 1+1 times inside function updateFunding() (line 504 and 522)
  
  504      uint256 currentFundingRate = fundingRates[latestFundingPaymentPointer];
  
  522        latestFundingPaymentPointer


  @audit: lastUpdateTime is called 1+2 times inside function updateFunding() (line 505 and 507, 508)
  
  505      uint256 startTime = lastUpdateTime == 0
  
  507        : lastUpdateTime;
  
  508      lastUpdateTime = block.timestamp;


  @audit: addresses is called 1+1 times inside function updateFunding() (line 511 and 515)
  
  511        addresses.perpetualAtlanticVaultLP,
  
  515      IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addProceeds(


  @audit: roundingPrecision is called 1+1 times inside function roundUp() (line 577 and 581)
  
  577      uint256 remainder = _strike % roundingPrecision;
  
  581        return _strike - remainder + roundingPrecision;


  @audit: _tokenIdCounter is called 1+1 times inside function _mintOptionToken() (line 589 and 590)
  
  589      tokenId = _tokenIdCounter.current();
  
  590      _tokenIdCounter.increment();


  @audit: latestFundingPaymentPointer is called 1+3 times inside function _updateFundingRate() (line 595 and 603, 610, 611)
  
  595      if (fundingRates[latestFundingPaymentPointer] == 0) {
  
  603        fundingRates[latestFundingPaymentPointer] =
  
  610        fundingRates[latestFundingPaymentPointer] =
  
  611          fundingRates[latestFundingPaymentPointer] +


  @audit: lastUpdateTime is called 1+2 times inside function _updateFundingRate() (line 597 and 598, 607)
  
  597        if (lastUpdateTime > nextFundingPaymentTimestamp() - fundingDuration) {
  
  598          startTime = lastUpdateTime;
  
  607        uint256 startTime = lastUpdateTime;


  @audit: fundingDuration is called 1+1 times inside function _updateFundingRate() (line 597 and 600)
  
  597        if (lastUpdateTime > nextFundingPaymentTimestamp() - fundingDuration) {
  
  600          startTime = nextFundingPaymentTimestamp() - fundingDuration;

```
[[211](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L211), [307](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L307), [364](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L364), [395](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L395), [456](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L456), [471](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L471), [494](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L494), [479](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L479), [522](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L522), [508](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L508), [515](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L515), [581](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L581), [590](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L590), [611](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L611), [607](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L607), [600](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L600)]

```
File: PerpetualAtlanticVaultLP.sol


  @audit: rdpx is called 1+1 times inside function constructor() (line 101 and 107)
  
  101      rdpx = _rdpx;
  
  107      ERC20(rdpx).approve(_perpetualAtlanticVault, type(uint256).max);


  @audit: collateral is called 1+1 times inside function constructor() (line 102 and 106)
  
  102      collateral = ERC20(_collateral);
  
  106      collateral.approve(_perpetualAtlanticVault, type(uint256).max);


  @audit: _totalCollateral is called 1+1 times inside function addProceeds() (line 192 and 195)
  
  192        collateral.balanceOf(address(this)) >= _totalCollateral + proceeds,
  
  195      _totalCollateral += proceeds;


  @audit: _totalCollateral is called 1+1 times inside function subtractLoss() (line 201 and 204)
  
  201        collateral.balanceOf(address(this)) == _totalCollateral - loss,
  
  204      _totalCollateral -= loss;


  @audit: _rdpxCollateral is called 1+1 times inside function addRdpx() (line 210 and 213)
  
  210        IERC20WithBurn(rdpx).balanceOf(address(this)) >= _rdpxCollateral + amount,
  
  213      _rdpxCollateral += amount;

```
[[107](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L107), [106](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L106), [195](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L195), [204](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L204), [213](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L213)]

```
File: RdpxDecayingBonds.sol


  @audit: _tokenIdCounter is called 1+1 times inside function _mintToken() (line 130 and 131)
  
  130      tokenId = _tokenIdCounter.current();
  
  131      _tokenIdCounter.increment();

```
[[131](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L131)]

```
File: RdpxV2Bond.sol


  @audit: _tokenIdCounter is called 1+1 times inside function mint() (line 40 and 41)
  
  40       tokenId = _tokenIdCounter.current();
  
  41       _tokenIdCounter.increment();

```
[[41](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol#L41)]

```
File: RdpxV2Core.sol


  @audit: reserveAsset is called 1+3 times inside function addAssetTotokenReserves() (line 246 and 248, 258, 261)
  
  246      for (uint256 i = 1; i < reserveAsset.length; i++) {
  
  248          reserveAsset[i].tokenAddress != _asset,
  
  258      reserveAsset.push(asset);
  
  261      reservesIndex[_assetSymbol] = reserveAsset.length - 1;


  @audit: reserveTokens is called 1+2 times inside function removeAssetFromtokenReserves() (line 280 and 280, 287)
  
  280      reservesIndex[reserveTokens[reserveTokens.length - 1]] = index;
  
  280      reservesIndex[reserveTokens[reserveTokens.length - 1]] = index;
  
  287      reserveTokens.pop();


  @audit: reserveAsset is called 1+3 times inside function removeAssetFromtokenReserves() (line 283 and 283, 283, 286)
  
  283      reserveAsset[index] = reserveAsset[reserveAsset.length - 1];
  
  283      reserveAsset[index] = reserveAsset[reserveAsset.length - 1];
  
  283      reserveAsset[index] = reserveAsset[reserveAsset.length - 1];
  
  286      reserveAsset.pop();




  @audit: amoAddresses is called 1+4 times inside function removeAMOAddress() (line 391 and 392, 392, 392, 393)
  
  391      _validate(_index < amoAddresses.length, 18);
  
  392      amoAddresses[_index] = amoAddresses[amoAddresses.length - 1];
  
  392      amoAddresses[_index] = amoAddresses[amoAddresses.length - 1];
  
  392      amoAddresses[_index] = amoAddresses[amoAddresses.length - 1];
  
  393      amoAddresses.pop();


  @audit: addresses is called 1+2 times inside function _curveSwap() (line 521 and 529, 530)
  
  521      IStableSwap dpxEthCurvePool = IStableSwap(addresses.dpxEthCurvePool);
  
  529        uint256 ethBalance = IStableSwap(addresses.dpxEthCurvePool).balances(a);
  
  530        uint256 dpxEthBalance = IStableSwap(addresses.dpxEthCurvePool).balances(



  @audit: reserveAsset is called 1+1 times inside function _stake() (line 570 and 572)
  
  570      reserveAsset[reservesIndex["WETH"]].tokenBalance -= _amount / 2;
  
  572      IDpxEthToken(reserveAsset[reservesIndex["DPXETH"]].tokenAddress).mint(


  @audit: addresses is called 1+5 times inside function _transfer() (line 632 and 638, 643, 648, 664, 677)
  
  632          addresses.rdpxDecayingBonds
  
  638          IRdpxDecayingBonds(addresses.rdpxDecayingBonds).ownerOf(_bondId) ==
  
  643        IRdpxDecayingBonds(addresses.rdpxDecayingBonds).decreaseAmount(
  
  648        IRdpxReserve(addresses.rdpxReserve).withdraw(_rdpxAmount);
  
  664            addresses.feeDistributor,
  
  677        IRdpxReserve(addresses.rdpxReserve).withdraw(


  @audit: reserveAsset is called 1+4 times inside function _transfer() (line 650 and 653, 657, 662, 681)
  
  650        reserveAsset[reservesIndex["RDPX"]].tokenBalance += _rdpxAmount;
  
  653        IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress)
  
  657        IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress).burn(
  
  662        IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress)
  
  681        reserveAsset[reservesIndex["RDPX"]].tokenBalance +=


  @audit: reserveAsset is called 1+1 times inside function settle() (line 779 and 780)
  
  779      reserveAsset[reservesIndex["WETH"]].tokenBalance += amountOfWeth;
  
  780      reserveAsset[reservesIndex["RDPX"]].tokenBalance -= rdpxAmount;


  @audit: addresses is called 1+1 times inside function provideFunding() (line 796 and 800)
  
  796      uint256 pointer = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
  
  800      fundingAmount = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)


  @audit: delegates is called 1+2 times inside function addToDelegate() (line 961 and 966, 967)
  
  961      delegates.push(delegatePosition);
  
  966      emit LogAddToDelegate(_amount, _fee, delegates.length - 1);
  
  967      return (delegates.length - 1);


  @audit: delegates is called 1+1 times inside function withdraw() (line 979 and 980)
  
  979      _validate(delegateId < delegates.length, 14);
  
  980      Delegate storage delegate = delegates[delegateId];


  @audit: reserveAsset is called 1+3 times inside function sync() (line 996 and 997, 1001, 1004)
  
  996      for (uint256 i = 1; i < reserveAsset.length; i++) {
  
  997        uint256 balance = IERC20WithBurn(reserveAsset[i].tokenAddress).balanceOf(
  
  1001       if (weth == reserveAsset[i].tokenAddress) {
  
  1004       reserveAsset[i].tokenBalance = balance;


  @audit: addresses is called 1+2 times inside function redeem() (line 1026 and 1032, 1036)
  
  1026       msg.sender == RdpxV2Bond(addresses.receiptTokenBonds).ownerOf(id),
  
  1032     RdpxV2Bond(addresses.receiptTokenBonds).burn(id);
  
  1036     IERC20WithBurn(addresses.rdpxV2ReceiptToken).safeTransfer(


  @audit: reserveAsset is called 1+1 times inside function upperDepeg() (line 1059 and 1067)
  
  1059     IDpxEthToken(reserveAsset[reservesIndex["DPXETH"]].tokenAddress).mint(
  
  1067     reserveAsset[reservesIndex["WETH"]].tokenBalance += wethReceived;


  @audit: reserveAsset is called 1+3 times inside function lowerDepeg() (line 1094 and 1106, 1110, 1119)
  
  1094       path[0] = reserveAsset[reservesIndex["RDPX"]].tokenAddress;
  
  1106       reserveAsset[reservesIndex["RDPX"]].tokenBalance -= _rdpxAmount;
  
  1110     reserveAsset[reservesIndex["WETH"]].tokenBalance -= _wethAmount;
  
  1119     IDpxEthToken(reserveAsset[reservesIndex["DPXETH"]].tokenAddress).burn(


  @audit: addresses is called 1+3 times inside function calculateBondCost() (line 1164 and 1189, 1193, 1196)
  
  1164         Math.sqrt(IRdpxReserve(addresses.rdpxReserve).rdpxReserve()) *
  
  1189     uint256 strike = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
  
  1193       addresses.perpetualAtlanticVault
  
  1196       wethRequired += IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)

```
[[261](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L261), [287](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L287), [286](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L286), [349](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L349), [393](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L393), [530](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L530), [572](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L572), [677](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L677), [681](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L681), [780](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L780), [800](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L800), [967](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L967), [980](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L980), [1004](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1004), [1036](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1036), [1067](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1067), [1119](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1119), [1196](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1196)]

```
File: ReLPContract.sol


  @audit: addresses is called 1+6 times inside function setAddresses() (line 138 and 150, 151, 155, 156, 160, 161)
  
  138      addresses = Addresses({
  
  150      IERC20WithBurn(addresses.pair).safeApprove(
  
  151        addresses.ammRouter,
  
  155      IERC20WithBurn(addresses.tokenA).safeApprove(
  
  156        addresses.ammRouter,
  
  160      IERC20WithBurn(addresses.tokenB).safeApprove(
  
  161        addresses.ammRouter,


  @audit: addresses is called 1+24 times inside function reLP() (line 205 and 206, 209, 217, 221, 224, 237, 243, 244, 257, 258, 259, 269, 270, 277, 286, 287, 288, 298, 298, 299, 302, 303, 304, 306)
  
  205        addresses.tokenA,
  
  206        addresses.tokenB
  
  209        addresses.ammFactory,
  
  217      tokenAInfo.tokenAReserve = IRdpxReserve(addresses.tokenAReserve)
  
  221      tokenAInfo.tokenAPrice = IRdpxEthOracle(addresses.rdpxOracle)
  
  224      tokenAInfo.tokenALpReserve = addresses.tokenA == tokenASorted
  
  237      uint256 totalLpSupply = IUniswapV2Pair(addresses.pair).totalSupply();
  
  243      IERC20WithBurn(addresses.pair).transferFrom(
  
  244        addresses.amo,
  
  257      (, uint256 amountB) = IUniswapV2Router(addresses.ammRouter).removeLiquidity(
  
  258        addresses.tokenA,
  
  259        addresses.tokenB,
  
  269      path[0] = addresses.tokenB;
  
  270      path[1] = addresses.tokenA;
  
  277      uint256 tokenAAmountOut = IUniswapV2Router(addresses.ammRouter)
  
  286      (, , uint256 lp) = IUniswapV2Router(addresses.ammRouter).addLiquidity(
  
  287        addresses.tokenA,
  
  288        addresses.tokenB,
  
  298      IERC20WithBurn(addresses.pair).safeTransfer(addresses.amo, lp);
  
  298      IERC20WithBurn(addresses.pair).safeTransfer(addresses.amo, lp);
  
  299      IUniV2LiquidityAmo(addresses.amo).sync();
  
  302      IERC20WithBurn(addresses.tokenA).safeTransfer(
  
  303        addresses.rdpxV2Core,
  
  304        IERC20WithBurn(addresses.tokenA).balanceOf(address(this))
  
  306      IRdpxV2Core(addresses.rdpxV2Core).sync();


  @audit: liquiditySlippageTolerance is called 1+1 times inside function reLP() (line 251 and 254)
  
  251        ((tokenAToRemove * liquiditySlippageTolerance) / 1e8);
  
  254        ((tokenAToRemove * tokenAInfo.tokenAPrice) * liquiditySlippageTolerance) /

```
[[161](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L161), [306](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L306), [254](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L254)]

```
File: UniV2LiquidityAmo.sol


  @audit: addresses is called 1+5 times inside function _sendTokensToRdpxV2Core() (line 161 and 164, 168, 169, 172, 173)
  
  161      uint256 tokenABalance = IERC20WithBurn(addresses.tokenA).balanceOf(
  
  164      uint256 tokenBBalance = IERC20WithBurn(addresses.tokenB).balanceOf(
  
  168      IERC20WithBurn(addresses.tokenA).safeTransfer(
  
  169        addresses.rdpxV2Core,
  
  172      IERC20WithBurn(addresses.tokenB).safeTransfer(
  
  173        addresses.rdpxV2Core,


  @audit: addresses is called 1+10 times inside function addLiquidity() (line 200 and 201, 204, 205, 210, 211, 215, 216, 222, 224, 225)
  
  200      IERC20WithBurn(addresses.tokenA).safeApprove(
  
  201        addresses.ammRouter,
  
  204      IERC20WithBurn(addresses.tokenB).safeApprove(
  
  205        addresses.ammRouter,
  
  210      IERC20WithBurn(addresses.tokenA).safeTransferFrom(
  
  211        addresses.rdpxV2Core,
  
  215      IERC20WithBurn(addresses.tokenB).safeTransferFrom(
  
  216        addresses.rdpxV2Core,
  
  222      (tokenAUsed, tokenBUsed, lpReceived) = IUniswapV2Router(addresses.ammRouter)
  
  224          addresses.tokenA,
  
  225          addresses.tokenB,


  @audit: addresses is called 1+4 times inside function removeLiquidity() (line 268 and 268, 271, 273, 274)
  
  268      IERC20WithBurn(addresses.pair).safeApprove(addresses.ammRouter, lpAmount);
  
  268      IERC20WithBurn(addresses.pair).safeApprove(addresses.ammRouter, lpAmount);
  
  271      (tokenAReceived, tokenBReceived) = IUniswapV2Router(addresses.ammRouter)
  
  273          addresses.tokenA,
  
  274          addresses.tokenB,


  @audit: addresses is called 1+6 times inside function swap() (line 314 and 315, 317, 318, 322, 328, 336)
  
  314        token1 = addresses.tokenA;
  
  315        token2 = addresses.tokenB;
  
  317        token1 = addresses.tokenB;
  
  318        token2 = addresses.tokenA;
  
  322        addresses.rdpxV2Core,
  
  328      IERC20WithBurn(token1).safeApprove(addresses.ammRouter, token1Amount);
  
  336      token2Amount = IUniswapV2Router(addresses.ammRouter)

```
[[173](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L173), [225](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L225), [274](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L274), [336](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L336)]

```
File: UniV3LiquidityAmo.sol


  @audit: positions_array is called 1+1 times inside function collectFees() (line 120 and 121)
  
  120      for (uint i = 0; i < positions_array.length; i++) {
  
  121        Position memory current_position = positions_array[i];


  @audit: rdpxV2Core is called 1+1 times inside function addLiquidity() (line 159 and 164)
  
  159        rdpxV2Core,
  
  164        rdpxV2Core,



  @audit: positions_array is called 1+5 times inside function removeLiquidity() (line 218 and 259, 259, 260, 262, 268)
  
  218      Position memory pos = positions_array[positionIndex];
  
  259      positions_array[positionIndex] = positions_array[
  
  259      positions_array[positionIndex] = positions_array[
  
  260        positions_array.length - 1
  
  262      positions_array.pop();
  
  268      emit log(positions_array.length);

```
[[121](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L121), [164](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L164), [268](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L268)]

            
</details>

### [GO-05] Multiple accesses of a mapping should use a local variable cache
The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local `storage` or `calldata` variable when the value is accessed [multiple times](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0), saves **~42 gas per access** due to not having to recalculate the key's keccak256 hash (Gkeccak256 - **30 gas**) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into memory/calldata


*Estimated saving:* 42 gas per access



*There are 14 instances of this issue:*              
                
<details>
<summary>Expand finding</summary>

```
File: PerpetualAtlanticVault.sol


  @audit: totalFundingForEpoch is called 1+3 times inside function payFunding() (lines 385 and 387, 391, 395)
  
  385        totalFundingForEpoch[latestFundingPaymentPointer]
  
  387      _updateFundingRate(totalFundingForEpoch[latestFundingPaymentPointer]);
  
  391        totalFundingForEpoch[latestFundingPaymentPointer],
  
  395      return (totalFundingForEpoch[latestFundingPaymentPointer]);


  @audit: optionsPerStrike is called 1+1 times inside function calculateFunding() (lines 414 and 421)
  
  414        _validate(optionsPerStrike[strikes[i]] > 0, 4);
  
  421        uint256 amount = optionsPerStrike[strike] -


  @audit: latestFundingPerStrike is called 1+1 times inside function calculateFunding() (lines 416 and 436)
  
  416          latestFundingPerStrike[strikes[i]] != latestFundingPaymentPointer,
  
  436        latestFundingPerStrike[strike] = latestFundingPaymentPointer;


  @audit: fundingPaymentsAccountedForPerStrike is called 1+1 times inside function calculateFunding() (lines 422 and 443)
  
  422          fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
  
  443        fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][


  @audit: fundingRates is called 1+3 times inside function _updateFundingRate() (lines 595 and 603, 610, 611)
  
  595      if (fundingRates[latestFundingPaymentPointer] == 0) {
  
  603        fundingRates[latestFundingPaymentPointer] =
  
  610        fundingRates[latestFundingPaymentPointer] =
  
  611          fundingRates[latestFundingPaymentPointer] +

```
[[343](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L343), [395](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L395), [421](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L421), [436](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L436), [443](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L443), [611](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L611)]

```
File: RdpxV2Core.sol


  @audit: reservesIndex is called 1+2 times inside function removeAssetFromtokenReserves() (lines 273 and 277, 280)
  
  273      uint256 index = reservesIndex[_assetSymbol];
  
  277      reservesIndex[_assetSymbol] = 0;
  
  280      reservesIndex[reserveTokens[reserveTokens.length - 1]] = index;


  @audit: reservesIndex is called 1+1 times inside function _stake() (lines 570 and 572)
  
  570      reserveAsset[reservesIndex["WETH"]].tokenBalance -= _amount / 2;
  
  572      IDpxEthToken(reserveAsset[reservesIndex["DPXETH"]].tokenAddress).mint(



  @audit: fundingPaidFor is called 1+1 times inside function provideFunding() (lines 798 and 805)
  
  798      _validate(fundingPaidFor[pointer] == false, 16);
  
  805      fundingPaidFor[pointer] = true;


```
[[280](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L280), [572](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L572), [805](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L805)]

```
File: UniV3LiquidityAmo.sol


  @audit: positions_mapping is called 1+1 times inside function removeLiquidity() (lines 263 and 269)
  
  263      delete positions_mapping[pos.token_id];
  
  269      emit log(positions_mapping[pos.token_id].token_id);

```
[[269](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L269)]

            
</details>

### [GO-06] Not using the named return variables anywhere in the function is confusing
Consider changing the variable to be an unnamed one, since the variable is never assigned, nor is it returned by name. If the optimizer is not turned on, leaving the code as it is will also waste gas for the stack variable.


*Estimated saving:* Not definied



*There are 1 instances of this issue:*              
                
<details>
<summary>Expand finding</summary>

```
File: PerpetualAtlanticVault.sol

  @audit: Return variables not used anywhere inside roundUp(): strike
  576    function roundUp(uint256 _strike) public view returns (uint256 strike) {

```
[[576](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L576)]


            
</details>

### [GO-07] Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if`-statement
`require(a <= b); x = b - a` => `require(a <= b); unchecked { x = b - a }`


*Estimated saving:* Not definied



*There are 6 instances of this issue:*              
                
<details>
<summary>Expand finding</summary>

```
File: PerpetualAtlanticVault.sol

  
  427          (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration));

  
  559      return strike > price ? ((strike - price) * amount) / 1e8 : 0;

  
  605          (endTime - startTime);

  
  612          ((amount * 1e18) / (endTime - startTime));

```
[[427](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L427), [559](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L559), [605](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L605), [612](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L612)]

```
File: PerpetualAtlanticVaultLP.sol

  
  201        collateral.balanceOf(address(this)) == _totalCollateral - loss,

```
[[201](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L201)]

```
File: RdpxV2Core.sol

  
  645          amount - _rdpxAmount

```
[[645](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L645)]

            
</details>

### [GO-08] State variable read in a loop
The state variable should be cached in a local variable rather than reading it on every iteration of the for-loop, which will replace each Gwarmaccess (**100 gas**) with a much cheaper stack read.


*Estimated saving:* Not definied



*There are 21 instances of this issue:*              
                
<details>
<summary>Expand finding</summary>

```
File: PerpetualAtlanticVault.sol

  
  329        uint256 strike = optionPositions[optionIds[i]].strike;

  
  330        uint256 amount = optionPositions[optionIds[i]].amount;

  
  337        optionsPerStrike[strike] -= amount;

  
  338        totalActiveOptions -= amount;

  
  343        optionPositions[optionIds[i]].strike = 0;

  
  414        _validate(optionsPerStrike[strikes[i]] > 0, 4);

  
  421        uint256 amount = optionsPerStrike[strike] -

  
  422          fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][

  
  422          fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][

  
  436        latestFundingPerStrike[strike] = latestFundingPaymentPointer;

  
  436        latestFundingPerStrike[strike] = latestFundingPaymentPointer;

  
  440        fundingPaymentsAccountedFor[latestFundingPaymentPointer] += amount;

  
  440        fundingPaymentsAccountedFor[latestFundingPaymentPointer] += amount;

  
  443        fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][

  
  443        fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][

  
  449        totalFundingForEpoch[latestFundingPaymentPointer] += premium;

  
  449        totalFundingForEpoch[latestFundingPaymentPointer] += premium;


```
[[329](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L329), [330](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L330), [337](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L337), [338](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L338), [343](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L343), [414](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L414), [421](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L421), [422](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L422), [422](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L422), [436](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L436), [436](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L436), [440](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L440), [440](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L440), [443](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L443), [443](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L443), [449](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L449), [449](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L449)]

```
File: RdpxV2Core.sol

  
  996      for (uint256 i = 1; i < reserveAsset.length; i++) {

  
  997        uint256 balance = IERC20WithBurn(reserveAsset[i].tokenAddress).balanceOf(

  
  1004       reserveAsset[i].tokenBalance = balance;

```
[[996](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L996), [997](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L997), [1004](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1004)]

```
File: UniV3LiquidityAmo.sol


  
  131        univ3_positions.collect(collect_params);

```
[[131](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L131)]

            
</details>

### [GO-09] Add `unchecked` blocks for divisions where the operands cannot overflow
`uint` divisions can't overflow, while `int` divisions can overflow only in [one specific case](https://docs.soliditylang.org/en/latest/types.html#division).

Consider adding an `unchecked` block to have some [gas savings](https://gist.github.com/DadeKuma/3bc597338ae774b8b3bd43280d55271f).


*Estimated saving:* 159 gas (see [this](https://gist.github.com/DadeKuma/3bc597338ae774b8b3bd43280d55271f))



*There are 39 instances of this issue:*              
                
<details>
<summary>Expand finding</summary>

```
File: PerpetualAtlanticVault.sol

  
  270      uint256 strike = roundUp(currentPrice - (currentPrice / 4)); // 25% below the current price

  
  276      uint256 requiredCollateral = (amount * strike) / 1e8;

  
  335        ethAmount += (amount * strike) / 1e8;

  
  475            (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
  476              1e18

  
  481              (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
  482                1e18

  
  487            ((currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /
  488              1e18),

  
  512        (currentFundingRate * (block.timestamp - startTime)) / 1e18

  
  516        (currentFundingRate * (block.timestamp - startTime)) / 1e18

  
  521        ((currentFundingRate * (block.timestamp - startTime)) / 1e18),

  
  545      premium = ((IOptionPricing(addresses.optionPricing).getOptionPrice(
  546        _strike,
  547        _price > 0 ? _price : getUnderlyingPrice(),
  548        getVolatility(_strike),
  549        timeToExpiry
  550      ) * _amount) / 1e8);

  
  559      return strike > price ? ((strike - price) * amount) / 1e8 : 0;


```
[[270](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L270), [276](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L276), [335](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L335), [475-476](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L475-L476), [481-482](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L481-L482), [487-488](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L487-L488), [512](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L512), [516](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L516), [521](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L521), [545-550](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L545-L550), [559](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L559)]

```
File: PerpetualAtlanticVaultLP.sol

  
  281        ((_rdpxCollateral * rdpxPriceInAlphaToken) / 1e8);

```
[[281](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L281)]

```
File: RdpxV2Core.sol

  
  535            ethBalance + _amount <= (ethBalance + dpxEthBalance) / 2,

  
  539            dpxEthBalance + _amount <= (ethBalance + dpxEthBalance) / 2,

  
  546        ? (((_amount * getDpxEthPrice()) / 1e8) -

  
  547          (((_amount * getDpxEthPrice()) * slippageTolerance) / 1e16))

  
  548        : (((_amount * getEthPrice()) / 1e8) -

  
  549          (((_amount * getEthPrice()) * slippageTolerance) / 1e16));

  
  570      reserveAsset[reservesIndex["WETH"]].tokenBalance -= _amount / 2;

  
  574        _amount / 2

  
  579        .deposit(_amount / 2);

  
  605      uint256 rdpxRequiredInWeth = (_rdpxRequired * getRdpxPrice()) / 1e8;

  

  
  612      amount1 = (amount1 * (100e8 - _delegateFee)) / 1e10;

  
  658          (_rdpxAmount * rdpxBurnPercentage) / 1e10

  
  665            (_rdpxAmount * rdpxFeePercentage) / 1e10

  
  669        uint256 rdpxAmountInWeth = (_rdpxAmount * getRdpxPrice()) / 1e8;

  

  
  1190       .roundUp(rdpxPrice - (rdpxPrice / 4)); // 25% below the current price

```
[[535](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L535), [539](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L539), [546](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L546), [547](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L547), [548](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L548), [549](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L549), [570](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L570), [574](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L574), [579](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L579), [605](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L605), [612](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L612), [658](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L658), [665](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L665), [669](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L669),   [1190](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1190)]

```
File: ReLPContract.sol

  
  251        ((tokenAToRemove * liquiditySlippageTolerance) / 1e8);

  
  252      uint256 mintokenBAmount = ((tokenAToRemove * tokenAInfo.tokenAPrice) /
  253        1e8) -

  
  254        ((tokenAToRemove * tokenAInfo.tokenAPrice) * liquiditySlippageTolerance) /
  255        1e16;

  
  274        (((amountB / 2) * tokenAInfo.tokenAPrice) / 1e8) -

  
  274        (((amountB / 2) * tokenAInfo.tokenAPrice) / 1e8) -

  
  275        (((amountB / 2) * tokenAInfo.tokenAPrice * slippageTolerance) / 1e16);

  
  275        (((amountB / 2) * tokenAInfo.tokenAPrice * slippageTolerance) / 1e16);

  
  279          amountB / 2,

  
  290        amountB / 2,

```
[[251](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L251), [252-253](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L252-L253), [254-255](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L254-L255), [274](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L274), [274](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L274), [275](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L275), [275](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L275), [279](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L279), [290](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L290)]

```
File: UniV2LiquidityAmo.sol

  
  373      return (lpTokenBalance * getLpPrice()) / 1e8;

```
[[373](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L373)]

            
</details>

### [GO-10] Simple checks for zero can be done using assembly to save gas



*Estimated saving:* Not definied



*There are 56 instances of this issue:*              
                
<details>
<summary>Expand finding</summary>

```
File: PerpetualAtlanticVault.sol

  
  119      _validate(_collateralToken != address(0), 1);

  
  190      _validate(_optionPricing != address(0), 1);

  
  191      _validate(_assetPriceOracle != address(0), 1);

  
  192      _validate(_volatilityOracle != address(0), 1);

  
  193      _validate(_feeDistributor != address(0), 1);

  
  194      _validate(_rdpx != address(0), 1);

  
  195      _validate(_perpetualAtlanticVaultLP != address(0), 1);

  
  196      _validate(_rdpxV2Core != address(0), 1);

  
  467          uint256 startTime = lastUpdateTime == 0

  
  505      uint256 startTime = lastUpdateTime == 0

  
  578      if (remainder == 0) {

  
  595      if (fundingRates[latestFundingPaymentPointer] == 0) {

```
[[119](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L119), [190](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L190), [191](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L191), [192](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L192), [193](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L193), [194](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L194), [195](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L195), [196](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L196), [467](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L467), [505](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L505), [578](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L578), [595](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L595)]

```
File: PerpetualAtlanticVaultLP.sol

  
  95         _perpetualAtlanticVault != address(0) || _rdpx != address(0),

  
  95         _perpetualAtlanticVault != address(0) || _rdpx != address(0),

  
  123      require((shares = previewDeposit(assets)) != 0, "ZERO_SHARES");

  
  162      require(assets != 0, "ZERO_ASSETS");

  
  223        (supply == 0)

  
  283        supply == 0 ? assets : assets.mulDivDown(supply, totalVaultCollateral);

```
[[95](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L95), [95](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L95), [123](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L123), [162](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L162), [223](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L223), [283](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L283)]

```
File: RdpxV2Core.sol

  
  244      require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");

  
  274      _validate(index != 0, 18);

  
  316      _validate(_dopexAMMRouter != address(0), 17);

  
  317      _validate(_dpxEthCurvePool != address(0), 17);

  
  318      _validate(_rdpxDecayingBonds != address(0), 17);

  
  319      _validate(_perpetualAtlanticVault != address(0), 17);

  
  320      _validate(_perpetualAtlanticVaultLP != address(0), 17);

  
  321      _validate(_rdpxReserve != address(0), 17);

  
  322      _validate(_rdpxV2ReceiptToken != address(0), 17);

  
  323      _validate(_feeDistributor != address(0), 17);

  
  324      _validate(_reLPContract != address(0), 17);

  
  325      _validate(_receiptTokenBonds != address(0), 17);

  
  362      _validate(_rdpxPriceOracle != address(0), 17);

  
  363      _validate(_dpxEthPriceOracle != address(0), 17);

  
  379      _validate(_addr != address(0), 17);

  
  408      _validate(_token != address(0), 17);

  
  409      _validate(_spender != address(0), 17);

  
  630      if (_bondId != 0) {

  
  1091     if (_rdpxAmount != 0) {

  
  1162     if (_rdpxBondId == 0) {

```
[[244](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L244), [274](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L274), [316](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L316), [317](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L317), [318](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L318), [319](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L319), [320](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L320), [321](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L321), [322](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L322), [323](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L323), [324](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L324), [325](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L325), [362](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L362), [363](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L363), [379](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L379), [408](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L408), [409](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L409), [630](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L630), [1091](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1091), [1162](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1162)]

```
File: ReLPContract.sol

  
  127        _tokenA != address(0) &&

  
  128          _tokenB != address(0) &&

  
  129          _pair != address(0) &&

  
  130          _rdpxV2Core != address(0) &&

  
  131          _tokenAReserve != address(0) &&

  
  132          _amo != address(0) &&

  
  133          _rdpxOracle != address(0) &&

  
  134          _ammFactory != address(0) &&

  
  135          _ammRouter != address(0),

```
[[127](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L127), [128](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L128), [129](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L129), [130](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L130), [131](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L131), [132](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L132), [133](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L133), [134](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L134), [135](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L135)]

```
File: UniV2LiquidityAmo.sol

  
  84         _tokenA != address(0) &&

  
  85           _tokenB != address(0) &&

  
  86           _pair != address(0) &&

  
  87           _rdpxV2Core != address(0) &&

  
  88           _rdpxOracle != address(0) &&

  
  89           _ammFactory != address(0) &&

  
  90           _ammRouter != address(0),

  
  131      require(_token != address(0), "reLPContract: token cannot be 0");

  
  132      require(_spender != address(0), "reLPContract: spender cannot be 0");

```
[[84](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L84), [85](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L85), [86](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L86), [87](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L87), [88](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L88), [89](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L89), [90](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L90), [131](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L131), [132](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L132)]

            
</details>

### [GO-11] State variables only set in the constructor should be declared `immutable`
Accessing state variables within a function involves an SLOAD operation, but `immutable` variables can be accessed directly without the need of it, thus reducing gas costs. As these state variables are assigned only in the constructor, consider declaring them `immutable`.


*Estimated saving:* 20000 gas of SLOAD operation



*There are 26 instances of this issue:*              
                
<details>
<summary>Expand finding</summary>

```
File: PerpetualAtlanticVault.sol

  
  51     string public underlyingSymbol;

  
  57     IERC20WithBurn public collateralToken;

  
  60     uint256 public collateralPrecision;

  
  66     mapping(uint256 => uint256) public fundingPaymentsAccountedFor;

  
  69     mapping(uint256 => mapping(uint256 => uint256))
  70       public fundingPaymentsAccountedForPerStrike;

  
  73     mapping(uint256 => uint256) public totalFundingForEpoch;

  
  76     mapping(uint256 => uint256) public optionsPerStrike;

  
  92     uint256 public totalActiveOptions;

  
  95     uint256 public genesis;

  
  104    uint256 public roundingPrecision = 1e6;

```
[[51](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L51), [57](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L57), [60](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L60), [66](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L66), [69-70](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L69-L70), [73](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L73), [76](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L76), [92](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L92), [95](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L95), [104](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L104)]

```
File: PerpetualAtlanticVaultLP.sol

  
  46     IPerpetualAtlanticVault public perpetualAtlanticVault;

  
  49     ERC20 public collateral;

  
  52     string public underlyingSymbol;

  
  55     string public collateralSymbol;

  
  58     uint256 private _totalCollateral;

  
  61     uint256 private _activeCollateral;

  
  64     uint256 private _rdpxCollateral;

  
  67     address public rdpx;

  
  70     address public rdpxRdpxV2Core;

```
[[46](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L46), [49](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L49), [52](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L52), [55](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L55), [58](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L58), [61](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L61), [64](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L64), [67](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L67), [70](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L70)]

```
File: RdpxV2Core.sol

  
  103    uint256 public liquiditySlippageTolerance = 5e5; // 0.5%

  
  121    Delegate[] public delegates;

```
[[103](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L103), [121](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L121)]

```
File: UniV3LiquidityAmo.sol

  
  35     IUniswapV3Factory public univ3_factory;

  
  36     INonfungiblePositionManager public univ3_positions;

  
  37     ISwapRouter public univ3_router;

  
  69     address public rdpx;

  
  72     address public rdpxV2Core;

```
[[35](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L35), [36](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L36), [37](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L37), [69](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L69), [72](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L72)]

            
</details>

### [GO-12] Avoid using state variable in emit
In scenarios where you have a memory, calldata variable or parameter with the same value as the state variable you should use the memory, calldata variable or parameter in the emit statement rather than state variable.


*Estimated saving:* 20000 gas of SLOAD operation



*There are 4 instances of this issue:*           

```
File: PerpetualAtlanticVault.sol

  
  211      emit AddressesSet(addresses);

  
  451        emit CalculateFunding(
  452          msg.sender,
  453          amount,
  454          strike,
  455          premium,
  456          latestFundingPaymentPointer
  457        );

```
[[211](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L211), [451-457](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L451-L457)]

```
File: RdpxV2Core.sol

  
  349      emit LogSetAddresses(addresses);

  
  370      emit LogSetPricingOracleAddresses(pricingOracleAddresses);

```
[[349](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L349), [370](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L370)]
### [GO-13] Avoid transferring 0 amount
Consider to implement 0 amount check in order to avoid to perform useless transfer operation


*Estimated saving:* 20000 gas of SLOAD operation



*There are 16 instances of this issue:*              
                
<details>
<summary>Expand finding</summary>

```
File: PerpetualAtlanticVault.sol

  
  289      collateralToken.safeTransferFrom(msg.sender, address(this), premium);

  
  347      collateralToken.safeTransferFrom(
  348        addresses.perpetualAtlanticVaultLP,
  349        addresses.rdpxV2Core,
  350        ethAmount
  351      );

  
  353      IERC20WithBurn(addresses.rdpx).safeTransferFrom(
  354        addresses.rdpxV2Core,
  355        addresses.perpetualAtlanticVaultLP,
  356        rdpxAmount
  357      );

```
[[289](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L289), [347-351](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L347-L351), [353-357](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L353-L357)]

```
File: PerpetualAtlanticVaultLP.sol

  
  172      IERC20WithBurn(rdpx).safeTransfer(receiver, rdpxAmount);

```
[[172](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L172)]

```
File: RdpxV2Core.sol

  
  909      IERC20WithBurn(weth).safeTransferFrom(
  910        msg.sender,
  911        address(this),
  912        wethRequired
  913      );

  
  1036     IERC20WithBurn(addresses.rdpxV2ReceiptToken).safeTransfer(
  1037       to,
  1038       receiptTokenAmount
  1039     );

```
[[909-913](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L909-L913), [1036-1039](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1036-L1039)]

```
File: ReLPContract.sol

  
  298      IERC20WithBurn(addresses.pair).safeTransfer(addresses.amo, lp);

```
[[298](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L298)]

```
File: UniV2LiquidityAmo.sol

  
  168      IERC20WithBurn(addresses.tokenA).safeTransfer(
  169        addresses.rdpxV2Core,
  170        tokenABalance
  171      );

  
  172      IERC20WithBurn(addresses.tokenB).safeTransfer(
  173        addresses.rdpxV2Core,
  174        tokenBBalance
  175      );

  
  210      IERC20WithBurn(addresses.tokenA).safeTransferFrom(
  211        addresses.rdpxV2Core,
  212        address(this),
  213        tokenAAmount
  214      );

  
  215      IERC20WithBurn(addresses.tokenB).safeTransferFrom(
  216        addresses.rdpxV2Core,
  217        address(this),
  218        tokenBAmount
  219      );

  
  321      IERC20WithBurn(token1).safeTransferFrom(
  322        addresses.rdpxV2Core,
  323        address(this),
  324        token1Amount
  325      );

```
[[168-171](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L168-L171), [172-175](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L172-L175), [210-214](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L210-L214), [215-219](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L215-L219), [321-325](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L321-L325)]

```
File: UniV3LiquidityAmo.sol

  
  319      TransferHelper.safeTransfer(tokenAddress, rdpxV2Core, tokenAmount);

  
  330      INonfungiblePositionManager(tokenAddress).safeTransferFrom(
  331        address(this),
  332        rdpxV2Core,
  333        token_id
  334      );

  
  357      IERC20WithBurn(tokenA).safeTransfer(rdpxV2Core, tokenABalance);

  
  358      IERC20WithBurn(tokenB).safeTransfer(rdpxV2Core, tokenBBalance);

```
[[319](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L319), [330-334](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L330-L334), [357](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L357), [358](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L358)]

            
</details>

### [GO-14] Using do-while loop instead of for loop or while loop
Using do-while loop could save gas, if you're sure that *code inside loop will run at least once*. In this way, it could save at least one condition check gas cost.

 See [10 Simple Ways to Save Gas with Solidity - 2.](https://medium.com/coinmonks/10-simple-ways-to-save-gas-with-solidity-f74cd2a59089)


*Estimated saving:* At least one loop condition check



*There are 12 instances of this issue:*              
                
<details>
<summary>Expand finding</summary>

```
File: PerpetualAtlanticVault.sol

  
  225      for (uint256 i = 0; i < tokens.length; i++) {
  226        token = IERC20WithBurn(tokens[i]);
  227        token.safeTransfer(msg.sender, token.balanceOf(address(this)));
  228      }

  
  328      for (uint256 i = 0; i < optionIds.length; i++) {
  329        uint256 strike = optionPositions[optionIds[i]].strike;
  330        uint256 amount = optionPositions[optionIds[i]].amount;
  331  
  332        // check if strike is ITM
  333        _validate(strike >= getUnderlyingPrice(), 7);
  334  
  335        ethAmount += (amount * strike) / 1e8;
  336        rdpxAmount += amount;
  337        optionsPerStrike[strike] -= amount;
  338        totalActiveOptions -= amount;
  339  
  340        // Burn option tokens from user
  341        _burn(optionIds[i]);
  342  
  343        optionPositions[optionIds[i]].strike = 0;
  344      }

  
  413      for (uint256 i = 0; i < strikes.length; i++) {
  414        _validate(optionsPerStrike[strikes[i]] > 0, 4);
  415        _validate(
  416          latestFundingPerStrike[strikes[i]] != latestFundingPaymentPointer,
  417          5
  418        );
  419        uint256 strike = strikes[i];
  420  
  421        uint256 amount = optionsPerStrike[strike] -
  422          fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
  423            strike
  424          ];
  425  
  426        uint256 timeToExpiry = nextFundingPaymentTimestamp() -
  427          (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration));
  428  
  429        uint256 premium = calculatePremium(
  430          strike,
  431          amount,
  432          timeToExpiry,
  433          getUnderlyingPrice()
  434        );
  435  
  436        latestFundingPerStrike[strike] = latestFundingPaymentPointer;
  437        fundingAmount += premium;
  438  
  439        // Record number of options that funding payments were accounted for, for this epoch
  440        fundingPaymentsAccountedFor[latestFundingPaymentPointer] += amount;
  441  
  442        // record the number of options funding has been accounted for the epoch and strike
  443        fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
  444          strike
  445        ] += amount;
  446  
  447        // Record total funding for this epoch
  448        // This does not need to be done in purchase() since it's already accounted for using `addProceeds()`
  449        totalFundingForEpoch[latestFundingPaymentPointer] += premium;
  450  
  451        emit CalculateFunding(
  452          msg.sender,
  453          amount,
  454          strike,
  455          premium,
  456          latestFundingPaymentPointer
  457        );
  458      }

```
[[225-228](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L225-L228), [328-344](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L328-L344), [413-458](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L413-L458)]

```
File: RdpxDecayingBonds.sol

  
  103      for (uint256 i = 0; i < tokens.length; i++) {
  104        token = IERC20WithBurn(tokens[i]);
  105        token.safeTransfer(msg.sender, token.balanceOf(address(this)));
  106      }

  
  156      for (uint256 i; i < ownerTokenCount; i++) {
  157        tokenIds[i] = tokenOfOwnerByIndex(_address, i);
  158      }

```
[[103-106](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L103-L106), [156-158](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L156-L158)]

```
File: RdpxV2Core.sol

  
  167      for (uint256 i = 0; i < tokens.length; i++) {
  168        token = IERC20WithBurn(tokens[i]);
  169        token.safeTransfer(msg.sender, token.balanceOf(address(this)));
  170      }

  
  246      for (uint256 i = 1; i < reserveAsset.length; i++) {
  247        require(
  248          reserveAsset[i].tokenAddress != _asset,
  249          "RdpxV2Core: asset already exists"
  250        );
  251      }

  
  775      for (uint256 i = 0; i < optionIds.length; i++) {
  776        optionsOwned[optionIds[i]] = false;
  777      }

  
  836      for (uint256 i = 0; i < _amounts.length; i++) {
  837        // Validate amount
  838        _validate(_amounts[i] > 0, 4);
  839  
  840        BondWithDelegateReturnValue
  841          memory returnValues = BondWithDelegateReturnValue(0, 0, 0, 0);
  842  
  843        returnValues = _bondWithDelegate(
  844          _amounts[i],
  845          rdpxBondId,
  846          _delegateIds[i]
  847        );
  848  
  849        delegateReceiptTokenAmounts[i] = returnValues.delegateReceiptTokenAmount;
  850  
  851        userTotalBondAmount += returnValues.bondAmountForUser;
  852        totalBondAmount += _amounts[i];
  853  
  854        // purchase options
  855        uint256 premium;
  856        if (putOptionsRequired) {
  857          premium = _purchaseOptions(returnValues.rdpxRequired);
  858        }
  859  
  860        // transfer the required rdpx
  861        _transfer(
  862          returnValues.rdpxRequired,
  863          returnValues.wethRequired - premium,
  864          _amounts[i],
  865          rdpxBondId
  866        );
  867      }

  
  996      for (uint256 i = 1; i < reserveAsset.length; i++) {
  997        uint256 balance = IERC20WithBurn(reserveAsset[i].tokenAddress).balanceOf(
  998          address(this)
  999        );
  1000 
  1001       if (weth == reserveAsset[i].tokenAddress) {
  1002         balance = balance - totalWethDelegated;
  1003       }
  1004       reserveAsset[i].tokenBalance = balance;
  1005     }

```
[[167-170](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L167-L170), [246-251](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L246-L251), [775-777](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L775-L777), [836-867](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L836-L867), [996-1005](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L996-L1005)]

```
File: UniV2LiquidityAmo.sol

  
  147      for (uint256 i = 0; i < tokens.length; i++) {
  148        token = IERC20WithBurn(tokens[i]);
  149        token.safeTransfer(msg.sender, token.balanceOf(address(this)));
  150      }

```
[[147-150](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L147-L150)]

```
File: UniV3LiquidityAmo.sol

  
  120      for (uint i = 0; i < positions_array.length; i++) {
  121        Position memory current_position = positions_array[i];
  122        INonfungiblePositionManager.CollectParams
  123          memory collect_params = INonfungiblePositionManager.CollectParams(
  124            current_position.token_id,
  125            rdpxV2Core,
  126            type(uint128).max,
  127            type(uint128).max
  128          );
  129  
  130        // Send to custodian address
  131        univ3_positions.collect(collect_params);
  132      }

```
[[120-132](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L120-L132)]

            
</details>

### [GO-15] Use constants instead of type(uintx).max
type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time


*Estimated saving:* Not definied



*There are 17 instances of this issue:*              
                
<details>
<summary>Expand finding</summary>

```
File: PerpetualAtlanticVault.sol

  
  209        type(uint256).max

  
  247          type(uint256).max

```
[[209](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L209), [247](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L247)]

```
File: PerpetualAtlanticVaultLP.sol

  
  106      collateral.approve(_perpetualAtlanticVault, type(uint256).max);

  
  107      ERC20(rdpx).approve(_perpetualAtlanticVault, type(uint256).max);

  
  155        if (allowed != type(uint256).max) {

```
[[106](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L106), [107](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L107), [155](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L155)]

```
File: RdpxV2Core.sol

  
  341        type(uint256).max

  
  343      IERC20WithBurn(weth).approve(addresses.dopexAMMRouter, type(uint256).max);

  
  344      IERC20WithBurn(weth).approve(addresses.dpxEthCurvePool, type(uint256).max);

  
  347        type(uint256).max

```
[[341](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L341), [343](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L343), [344](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L344), [347](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L347)]

```
File: ReLPContract.sol

  
  152        type(uint256).max

  
  157        type(uint256).max

  
  162        type(uint256).max

```
[[152](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L152), [157](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L157), [162](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L162)]

```
File: UniV3LiquidityAmo.sol

  
  126            type(uint128).max,

  
  127            type(uint128).max

  
  190          type(uint256).max

  
  223          type(uint128).max,

  
  224          type(uint128).max

```
[[126](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L126), [127](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L127), [190](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L190), [223](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L223), [224](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L224)]

            
</details>

