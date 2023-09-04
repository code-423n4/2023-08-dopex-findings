# Check for Zero Balance
## Impact
It is advisable to verify the zero balance prior to initiating a transfer to prevent unnecessary gas consumption and actions, adhering to best practices.
## Proof of Concept
The `balance of` the contract is not checked before attempting to transfer tokens to avoid zero balance transfer
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L169
## Tools Used
Manual Review
## Recommended Mitigation Steps
Initiate a preliminary balance verification using the 'token.balanceOf' function. If the balance is greater than zero, proceed with the transfer initiation.


# Unbounded Loop for token transfer
## Impact
Having an unbounded length of tokens could lead to some tokens not getting sent, as there is no check for successful transfer 
## Proof of Concept
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L167
## Tools Used
Manual Review
## Recommended Mitigation Steps
Initiate a preliminary balance verification using the 'token.balanceOf' function. If the balance is greater than zero, proceed with the transfer initiation.


