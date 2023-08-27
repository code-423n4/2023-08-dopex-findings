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