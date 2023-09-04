1. call the ```sync()``` function every time an asset is sent from one contract to the **RdpxV2Core** contract.
   Do this by adding **RdpxV2Core.sync()** in the functions that transfers or pulls assets from the RdpxV2Core contract. For example in the **swap** functions in **UniV2LiquidityAMO** and **UniV3LiquidityAMO** contracts. The sync function can be called anytime but very large amount of swaps can cause calculation issues if no one called the sync function in a very long time but this is very unlikely.

2.