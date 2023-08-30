# Low-Criticality Bugs:
## L-1 - Missing Address Check in UniV3LiquidityAmo
In the constructor, there's no check to ensure that rdpx and rdpxV2Core are not the zero address.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L78
Mitigation Step:
Add a require statement to ensure that both rdpx and rdpxV2Core are not the zero address.

## L-2 - Missing WETH Address Check in RdpxV2Core
In the constructor, there's no check to ensure that weth is not the zero address.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L126
Mitigation Step:
Add a require statement to ensure that weth is not the zero address.

## L-3 - Incorrect Address in RdpxDecayingBonds
In the emergencyWithdraw function, tokens are transferred to msg.sender instead of the intended recipient to.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L107
Mitigation Step:
Modify the token.safeTransfer call to use the to address.
## L-4 - Missing Address Checks in PerpetualAtlanticVaultLP
In the constructor, there's no check to ensure that _rdpxRdpxV2Core and collateral are not the zero address.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L95
Mitigation Step:
Add a require statement to ensure that both _rdpxRdpxV2Core and collateral are not the zero address.

## L-5 - Missing Duplicate Address Check in RdpxV2Core
The function allows adding AMO addresses without checking for duplicates. Additionally, the amoAddresses array is not used elsewhere in the contract.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L384
Mitigation Step:
Implement a mechanism to check for duplicate addresses before adding to the amoAddresses array. Consider the necessity of the amoAddresses array if it's not used elsewhere.

# QA Issues:
## Q-1 - Redundant Function Calls in PerpetualAtlanticVault
The function nextFundingPaymentTimestamp() is called multiple times.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L466
Proposed Solution:
Store the result of nextFundingPaymentTimestamp() in a local variable at the start of the function and use this variable throughout.

## Q-2 - Suspicious Minting Behavior in RdpxV2Bond
The contract allows minting even when it's paused.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L41
Proposed Solution:
Inspect the logic and consider adding a require statement to ensure minting can only occur when the contract is not paused.