# gas report

*Dr. c mark0v*





#### [G-01] Magic square root gas death


[[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L228]]
```solidity
// File: contracts/reLP/ReLPContract.sol
// Lines: 228-230

228     uint256 baseReLpRatio = (reLPFactor *
229       Math.sqrt(tokenAInfo.tokenAReserve) *
230       1e2) / (Math.sqrt(1e18)); // 1e6 precision
```

should be

```solidity
// File: contracts/reLP/ReLPContract.sol
// Lines: 228-230

228     uint256 baseReLpRatio = (reLPFactor *
229       Math.sqrt(tokenAInfo.tokenAReserve) *
230       ) / 1e7; 
```


The call to the external library to evaluate a constant. also the comment is inaccurate

 $\text{reLPFactor} \in \{1,2,3...10^{8}\}$ This is linear dependence, thus the min is at the bountry $(\sqrt(10^{18}) \cdot 10^{2})/10^{9} = 10^{2}$ which is 2 digits of precision. In reality it is going to be $\text{floor}(\log_{10}(\text{reLPFactor})) + 2 $ digits




[[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1163]]
```solidity
// File: contracts/core/RdpxV2Core.sol
// Lines: 1163-1165

1163       uint256 bondDiscount = (bondDiscountFactor *
1164         Math.sqrt(IRdpxReserve(addresses.rdpxReserve).rdpxReserve()) *
1165         1e2) / (Math.sqrt(1e18)); // 1e8 precision
```

is 


```solidity
     uint256 bondDiscount = (bondDiscountFactor *
        Math.sqrt(IRdpxReserve(addresses.rdpxReserve).rdpxReserve())) / 1e7

```

to save ops as before 



#### [G-02]  



[[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L273]]
```solidity
// File: contracts/reLP/ReLPContract.sol
// Lines: 273-275

273     mintokenAAmount =
274       (((amountB / 2) * tokenAInfo.tokenAPrice) / 1e8) -
275       (((amountB / 2) * tokenAInfo.tokenAPrice * slippageTolerance) / 1e16);
```


```solidity
// File: contracts/reLP/ReLPContract.sol
// Lines: 272-275

272     // calculate min amount of tokenA to be received
273     mintokenAAmount = (amountB*tokenAInfo.tokenAPrice)*(1e8-slippageTolerance))/2e16
```


we know ``1e8 - slippageTolerance`` is positive due to ``1e8`` being our maximum 


#### [G-03] Unnecessary error messages

This is not a user-callable function, therefore it shouldnt need a lengthy error message like this


[[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L240]]
```solidity
// File: contracts/core/RdpxV2Core.sol
// Lines: 240-244

240   function addAssetTotokenReserves(
241     address _asset,
242     string memory _assetSymbol
243   ) external onlyRole(DEFAULT_ADMIN_ROLE) {
244     require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");
```
Another example, 

 
[[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L90]]
```solidity
// File: contracts/reLP/ReLPContract.sol
// Lines: 90-99

90   function setreLpFactor(
91     uint256 _reLPFactor
92   ) external onlyRole(DEFAULT_ADMIN_ROLE) {
93     require(
94       _reLPFactor > 0,
95       "reLPContract: reLP factor must be greater than 0"
96     );
97     reLPFactor = _reLPFactor;
98 
99     emit LogSetReLpFactor(_reLPFactor);
```

Two more in that file.

[[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L171]]
```solidity
// File: contracts/reLP/ReLPContract.sol
// Lines: 171-194

171   function setLiquiditySlippageTolerance(
172     uint256 _liquiditySlippageTolerance
173   ) external onlyRole(DEFAULT_ADMIN_ROLE) {
174     require(
175       _liquiditySlippageTolerance > 0,
176       "reLPContract: liquidity slippage tolerance must be greater than 0"
177     );
178     liquiditySlippageTolerance = _liquiditySlippageTolerance;
179   }
180 
181   /**
182    * @notice sets the slippage tolerance
183    * @dev    Can only be called by admin
184    * @param  _slippageTolerance the slippage tolerance
185    */
186   function setSlippageTolerance(
187     uint256 _slippageTolerance
188   ) external onlyRole(DEFAULT_ADMIN_ROLE) {
189     require(
190       _slippageTolerance > 0,
191       "reLPContract: slippage tolerance must be greater than 0"
192     );
193     slippageTolerance = _slippageTolerance;
194   }
```

Reduce the contract size by storing that string "must be greater than 0" as bytes and combine at runtime using ``encodePacked()``

Or use ordinated error table like in some other contracts here. Or dont use one. This is is an administrative function so not ones ever going to see it. 

#### [G-04]


This can be done in less ops too. I dont have enough time but It needs a massage

[[https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L231]]
```solidity
// File: contracts/reLP/ReLPContract.sol
// Lines: 231-235

231 
232     uint256 tokenAToRemove = ((((_amount * 4) * 1e18) /
233       tokenAInfo.tokenAReserve) *
234       tokenAInfo.tokenALpReserve *
235       baseReLpRatio) / (1e18 * DEFAULT_PRECISION * 1e2);
```

