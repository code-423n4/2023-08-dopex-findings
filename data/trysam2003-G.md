As number of assets grows, the gas consumption increases in validating the existence of the asset when "addAssetTotokenReserves" function is called. This is inefficient use of gas and can potentially lead to denial of service when "addAssetTotokenReserves" function is called, due to gas blockage, as the number of assets in the reserve grows considerable 


https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L246-L251

     for (uint256 i = 1; i < reserveAsset.length; i++) {
         require(
             reserveAsset[i].tokenAddress != _asset,
             "RdpxV2Core: asset already exists"
         );

To reduce the gas consumption and avoid the gas bloackage, consider using mapping to track the token contract address, so that there will not be need for looping expression

