
# Unused Role

The MANAGER_ROLE is defined but not utilized in the contract.

related code:
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L45

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L127

## Recommended Mitigation Steps

Either use this role or remove it

# lack zero address check for ERC20 tokens

related code:
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L219

## Recommended Mitigation Steps

```Solidity
    function emergencyWithdraw(address[] calldata tokens) external onlyRole(DEFAULT_ADMIN_ROLE) {
        _whenPaused();
        IERC20WithBurn token;

        for (uint256 i = 0; i < tokens.length; i++) {
            // audit: check if the tokens[i] is zero
            require(tokens[i] != address(0), "E1");
            token = IERC20WithBurn(tokens[i]);
            token.safeTransfer(msg.sender, token.balanceOf(address(this)));
        }

        emit EmergencyWithdraw(msg.sender, tokens);
    }
```
