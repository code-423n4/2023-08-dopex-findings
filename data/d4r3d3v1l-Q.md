Vulnerability Details :

## Low Risk Issue 

### Able to mint RdpxV2Bond NFT with 0 amount in bondWithDelegate function.

This `_validate` only checks the length of _amounts and _delegates are equal. 
If we pass a empty arrays , it skips the for loop and calls the `_stake` function which mints the `RdpxV2Bond`

```solidity
     _validate(_amounts.length == _delegateIds.length, 3); 

        uint256 userTotalBondAmount;
        uint256 totalBondAmount;

        uint256[] memory delegateReceiptTokenAmounts = new uint256[](
            _amounts.length
        );

        for (uint256 i = 0; i < _amounts.length; i++) {
```


https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L827

#### Test 
Add this code in `testBondWithDelegate` in Unit test
```
        _amounts = new uint256[](0);
        _delegateIds = new uint256[](0);

        (userAmount, delegateAmounts) = rdpxV2Core.bondWithDelegate(
            address(this),
            _amounts,
            _delegateIds,
            1
        );
```
