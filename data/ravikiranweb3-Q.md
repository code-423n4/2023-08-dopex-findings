1) RDPXV2Core::_curveSwap
  IStableSwap dpxEthCurvePool = IStableSwap(addresses.dpxEthCurvePool);

dpxEthCurvePool variable could be used when calling balances of a and b.  


2) PerpetualAtlanticVaultLP::underlyingSymbol
   underlyingSymbol is not initialized in PerpetualAtlanticVaultLP. 
   If this is a redundant variable, remove it from the contract.
