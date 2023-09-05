## [G-01] Use calldata instead of memory for function parameters that don’t change 

When you set a data location to 'memory,' it means you're duplicating that information and putting it in the computer's memory. But when you set it as 'calldata,' it remains where it is.
If the value is a large, complex type, using memory may result in extra memory expansion costs.

Total Instances: 5

Estimated Gas Saved: 500

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L316

```solidity
 File: contracts/perp-vault/PerpetualAtlanticVault.sol
function settle(
        uint256[] memory optionIds
    )
        external
        nonReentrant
        onlyRole(RDPXV2CORE_ROLE)
        returns (uint256 ethAmount, uint256 rdpxAmount)
    {
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L406

``` solidity
File: contracts/perp-vault/PerpetualAtlanticVault.sol
function calculateFunding(uint256[] memory strikes) external nonReentrant returns (uint256 fundingAmount) {
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L765C31-L765C31

``` solidity
File: contracts/core/RdpxV2Core.sol
function settle(
    uint256[] memory optionIds
  )
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 amountOfWeth, uint256 rdpxAmount)
  {
```

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L821
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L822

``` solidity
File: contracts/core/RdpxV2Core.sol
function bondWithDelegate(
    address _to,
    uint256[] memory _amounts,
    uint256[] memory _delegateIds,
    uint256 rdpxBondId
  ) public returns (uint256 receiptTokenAmount, uint256[] memory) {
```

