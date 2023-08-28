1) RDPXV2Core::_curveSwap
  IStableSwap dpxEthCurvePool = IStableSwap(addresses.dpxEthCurvePool);

dpxEthCurvePool variable could be used when calling balances of a and b.  


2) PerpetualAtlanticVaultLP::underlyingSymbol
   underlyingSymbol is not initialized in PerpetualAtlanticVaultLP. 
   If this is a redundant variable, remove it from the contract.


3)  UniV2LiquidityAMO::slippageTolerance
   slippageTolerance is not used in the contract. remove it redundant

4) UniV3LiquidityAMO::swap
   passing hardcoded timestamp for expiry of swap order is unnecessary. It could have be block.timestamp + number of days/hours etc.

5) UniV3LiquidityAMO::removeLiquidity
   positionIndex passed as parameter is not checked to be with in the length of the array. This could 
   lead to out of bounds exception

6) RdpxDecayingBonds::decreaseAmount
   the name of the function is misleading as it does not decrease, but updates the bond amount to new value. Should be renamed as updateAmount