## Impact
The variable 2105300114 in the function swap is used to set the expiration time of the swap. It appears to be a hardcoded Unix timestamp.

When creating a variable for a timestamp, it's best to follow a few best practices:

Use a descriptive name: The variable name should clearly indicate what it represents. In this case, a name like expirationTimestamp would be more descriptive and easy to understand.

Use the appropriate data type: In Solidity, timestamps are represented as uint256. This is because the EVM deals with time in terms of seconds since the Unix epoch.

Avoid hardcoding values: Hardcoding a specific value, like 2105300114, can lead to problems if the timestamp is not updated. It's better to calculate the timestamp dynamically based on the current block time (block.timestamp)

SLOC
`https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L295`

## Mitigation
Since 2105300114 seconds / 86400 seconds/day = 24375 days

`uint256 expirationTimestamp = 24375 days; // Expires in 1 day`

In this example, expirationTimestamp is a uint256 variable that is set to 24375 days. This makes it clear that the variable represents a timestamp and when it will expire.

This timestamp is used in the ISwapRouter.ExactInputSingleParams struct which is part of the Uniswap V3 Periphery library. It sets a deadline for when the swap must be included in a block. If the transaction isn't included in a block before this timestamp, it will fail. This is a common pattern in DeFi to prevent transactions from being manipulated by miners


------------------
## Impact
Wrong initialization in Setup.t.sol test. Developers may think that the test case is valid an initialize in wrong other against documentation. 
:
  /* Inital tokens in the reserve
     index0: ZERO address
     index1: weth
     index2: rdpx
     index3: dpxEth
     index4: crv
  */



# PoC
```
    rdpxV2Core.addAssetTotokenReserves(address(rdpx), "RDPX");
    rdpxV2Core.addAssetTotokenReserves(address(weth), "WETH");
    rdpxV2Core.addAssetTotokenReserves(address(dpxETH), "DPXETH");
```

SLOC
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/tests/rdpxV2-core/Setup.t.sol#L260
```

## Mitigation
Change to WETH first as stated in the RdpxV2 Contract:
```
    rdpxV2Core.addAssetTotokenReserves(address(weth), "WETH");
    rdpxV2Core.addAssetTotokenReserves(address(rdpx), "RDPX");
    rdpxV2Core.addAssetTotokenReserves(address(dpxETH), "DPXETH");
```