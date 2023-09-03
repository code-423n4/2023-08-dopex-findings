# [L-01] Insufficient input validation let's users mint a free bond
If a user calls the `bondWithDelegate` function with empty arrays for both `_amounts` and `_delegateIds`, unintendedly, a Bond with zero amount will be minted to the user.

## PoC
```solidity
contract Exploit is Setup {
    function testBondWithDelegateEmptyArrays() external {
        // create empty arrays to pass as arguments
        uint256[] memory _amounts = new uint256[](0);
        uint256[] memory _delegateIds = new uint256[](0);

        // calls bondWithDelegate
        rdpxV2Core.bondWithDelegate(address(this), _amounts, _delegateIds, 0);

        // assert that the user was able to mint a bond (although the bond has 0 amount).
        assertEq(rdpxV2Bond.balanceOf(address(this)), 1);
    }
} 
```

## Recommended Mitigation
Since the function `bondWithDelegate` already checks the length of `_amounts` and `_delegateIds` arrays are the same, consider adding a check to ensure that one of the arrays has a non-zero length.

```solidity
function bondWithDelegate(
    address _to,
    uint256[] memory _amounts,
    uint256[] memory _delegateIds,
    uint256 rdpxBondId
  ) public returns (uint256 receiptTokenAmount, uint256[] memory) {
    _whenNotPaused();
    // Validate amount
    _validate(_amounts.length == _delegateIds.length, 3);
+   _validate(_amounts.length != 0, 3);

    ...
  }
```