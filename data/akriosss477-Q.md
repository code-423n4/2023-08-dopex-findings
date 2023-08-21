https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L142-L153
Calls inside a loop might lead to a denial-of-service attack

The vulnerability arises because external calls can fail for a variety of reasons, and if one call fails, it will cause the entire transaction to revert. This means that if the contract has a balance in a token that cannot be transferred (for example, because the token contract is paused or has a bug), then it becomes impossible to withdraw any of the other tokens. 

This could potentially lock all funds in the contract, leading to a denial-of-service attack. An attacker could deliberately cause a transfer to fail (for example, by causing the contract to hold a token that they control and can pause), thereby preventing anyone else from withdrawing their funds. 

To mitigate this vulnerability, the contract should catch failures of the `transfer` function and continue with the next iteration of the loop. This can be done using the `try/catch` syntax or the `.call` function in Solidity.
To resolve this issue, you should handle the failure of the `transfer` function and continue with the next iteration of the loop. This can be done using the `try/catch` syntax in Solidity. Here is an example of how you can modify the `emergencyWithdraw` function to handle transfer failures:

```solidity
function emergencyWithdraw(address[] calldata tokens) external onlyRole(DEFAULT_ADMIN_ROLE) {
    IERC20WithBurn token;

    for (uint256 i = 0; i < tokens.length; i++) {
        token = IERC20WithBurn(tokens[i]);
        try token.safeTransfer(msg.sender, token.balanceOf(address(this))) {
            // Emit an event or log the successful transfer
        } catch {
            // Handle the error, log the failure, or emit an event
        }
    }

    emit LogEmergencyWithdraw(msg.sender, tokens);
}
```

In this modified function, if a `transfer` call fails, the contract catches the error and continues with the next iteration of the loop. This ensures that a failure in transferring one token does not prevent the withdrawal of other tokens. 
