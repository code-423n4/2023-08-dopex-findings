1.Revert on Approval To Zero Address

Some tokens (e.g. OpenZeppelin) will revert if trying to approve the zero address to spend tokens (i.e. a call to approve(address(0), amt)).

Integrators may need to add special cases to handle this logic if working with such a token.

2.Low Decimals

Some tokens have low decimals (e.g. USDC has 6). Even more extreme, some tokens like Gemini USD only have 2 decimals.

This may result in larger than expected precision loss.

3.High Decimals

Some tokens have more than 18 decimals (e.g. YAM-V2 has 24).

This may trigger unexpected reverts due to overflow, posing a liveness risk to the contract.

4.Non string metadata
Some tokens (e.g. MKR) have metadata fields (name / symbol) encoded as bytes32 instead of the string prescribed by the ERC20 specification.

This may cause issues when trying to consume metadata from these tokens.

5.No Revert on Failure
Some tokens do not revert on failure, but instead return false (e.g. ZRX, EURS).

While this is technically compliant with the ERC20 standard, it goes against common solidity coding practices and may be overlooked by developers who forget to wrap their calls to transfer in a require.

6. push 0 problems when version more than 0.8.19

7. Divide by zero should be avoided 

8. latest openzepelin version 

9. All proxy contracts initialized in initialize function

10. initializer could be front run 

11) All hard coded values right ?

12. add blocklist function for NFT 

13) Is timeclock function implemented ?

14) is there any swap function . Slippage protection, deadline, Hardcoded Slippage ?, 

15. Can the 1st deposit raise a problem ?

16) The contract implement a white/blacklist ? or some kind of addresses check ? is blocklist and whitelist tokens checked

17) Solmate ERC20.sageTransferLib do not check the contract existence

18) msg.value not checked can have result in unexpected behaviour

19) Is the function refunds the extra amount paid ? when using msg.value 

Was disableInitializers() called ?

if any contract inheritance has a constructor (erc20, reentrancyGuard, Pausable…) is used : use the upgreadable version for initialize

Signature Malleability : do not use escrevover() but use the openzepplin/ECDSA.sol (The last version should be used here)

Take care if (receiver == caller) can have unexpected behaviour

Hash collisions are possible with abi.encodePacked (here)

Is this possible the oracle returns state data. LatestAnswer() 

Consider implementing two-step procedure for updating protocol addresses

Direct supportsInterface() calls may cause caller to revert



