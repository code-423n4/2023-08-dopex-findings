# Summary

# [L-01] `bond` and `bondWithDelegate` do not validate the `_to` address

## Description
In `RdpxV2Core`, `bondWithDelegate()` does not validate the `_to` address:

[core/RdpxV2Core.sol:819](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L819)
```
819:   function bondWithDelegate(
820:     address _to,
821:     uint256[] memory _amounts,
822:     uint256[] memory _delegateIds,
823:     uint256 rdpxBondId
824:   ) public returns (uint256 receiptTokenAmount, uint256[] memory) {
```

The same for `bond`():

[core/RdpxV2Core.sol:894](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L894)
```solidity
894:   function bond(
895:     uint256 _amount,
896:     uint256 rdpxBondId,
897:     address _to
898:   ) public returns (uint256 receiptTokenAmount) {
```

This holds across all the subsequent internal calls, too.
## Recommendation
Validate that `_to` is not `address(0)` to prevent accidental loss of user funds in the event that a user does not provide a valid `_to` address.

# [L-02] `addAssetTotokenReserves` does not validate `_assetSymbol`

## Description
In `RdpxV2Core`, `addAssetTotokenReserves` does not validate `_assetSymbol`:

[core/RdpxV2Core.sol:240](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L240)
```solidity
240:   function addAssetTotokenReserves(
241:     address _asset,
242:     string memory _assetSymbol
243:   ) external onlyRole(DEFAULT_ADMIN_ROLE) {
```

If there is a bug in the deployment script, multiple assets could be added without a symbol or the same symbol, overwriting each other in the `reservesIndex` map:

[core/RdpxV2Core.sol:253](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L253)
```solidity
253:     ReserveAsset memory asset = ReserveAsset({
254:       tokenAddress: _asset,
255:       tokenBalance: 0,
256:       tokenSymbol: _assetSymbol
257:     });
258:     reserveAsset.push(asset);
259:     reserveTokens.push(_assetSymbol);
260: 
261:     reservesIndex[_assetSymbol] = reserveAsset.length - 1;
```

Since the corresponding `removeAssetFromtokenReserves` uses the `_assetSymbol_` to specify which asset to remove, these misconfigured assets will be stuck:

[core/RdpxV2Core.sol:270](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L270)
```
270:   function removeAssetFromtokenReserves(
271:     string memory _assetSymbol
272:   ) external onlyRole(DEFAULT_ADMIN_ROLE) {
273:     uint256 index = reservesIndex[_assetSymbol];
274:     _validate(index != 0, 18);
275: 
276:     // remove the asset from the mapping
277:     reservesIndex[_assetSymbol] = 0;
278: 
279:     // add new index for the last element
280:     reservesIndex[reserveTokens[reserveTokens.length - 1]] = index;
281: 
282:     // update the index of reserveAsset with the last element
283:     reserveAsset[index] = reserveAsset[reserveAsset.length - 1];
```

## Recommendation
Validate whether \_assetSymbol and/or appears in the reserves already.


# [L-03] Lack of fuzz testing is error prone

## Description
All the tests have hard-coded parameters. This is error prone and unlikely to explore all possible edge cases.

## Recommendation
Introduce fuzz testing, especially to cover the following scenarios:
- Bonding with fuzzed combinations of rdpx and weth
- Bonding multiple times with fuzzed decaying bond
- Minting a decaying bond and redeeming at fuzzed intervals
- Adding to delegate with fuzzed fees, then bonding with delegates


# [L-04] Lack of visualizable test output is error prone

## Description
Assessing the performance of rDPX V2 using purely numerical test outputs is very challenging.

Given that the core product involves financial engineering of various asset flows, an improvement would be to produce visualizations of these asset flows vs. parameters.

## Recommendation
Implement a `forge script` to run rDPX V2 through various representative uses cases and scenarios. Output curve point data into a format that can be used to graph the curves. Produce these graphs as part of the build to visualize test output for additional implementation confidence.  Alternatively, animated graphs services such as Desmos may be used.