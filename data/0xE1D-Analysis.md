rDPX V2 Protocol Report

Introduction:
rDPX V2 is an enhanced version of the original rDPX token system, introducing a new synthetic coin called dpxETH pegged to ETH. This coin aims to boost ETH yields and act as vital collateral for upcoming Dopex Options Products. The system involves a bonding process, allowing users to mint new dpxETH tokens.

Recommendations for improving the protocol code

1. Fee-on-Transfer:
Description: Transfer functions in certain contracts do not account for potential on-transfer fees which might be deducted. This can lead to accounting discrepancies.
Implications: Malicious actors may exploit this vulnerability to manipulate token balances or induce errors in token accounting.
Recommendation: Implement a mechanism that measures the token balance before and after every transferFrom(). The difference between the two measurements should be treated as the actual transferred amount.
This pattern can be observed in UniV2LiquidityAmo.sol, UniV3LiquidityAmo.sol, PerpetualAtlanticVault.sol

2. Usage of _mint() instead of _safeMint():
Description: The _mint() function is employed directly, risking transfers to contracts that aren't prepared to receive ERC721 tokens.
Implications: If a non-compliant contract or one without IERC721Receiver implementation is the recipient, the tokens could be locked forever.
Recommendation: Replace all occurrences of _mint() with _safeMint() to ensure tokens are only sent to compliant recipients.
This can be seen in RdpxDecayingBonds.sol

3. Unsafe Downcast:
Description: Certain variables are being forcefully downcast, risking truncation of data.
Implications: Data truncation can lead to unpredictable behavior and potential manipulation by adversaries.
Recommendation: Avoid direct downcasting, especially without prior checks. Logic should be redesigned to prevent unexpected data truncation.
It was seen in RdpxV2Core.sol

4. Renouncing Ownership While Paused:
Description: Contract ownership can be renounced while the system is paused.
Implications: This might result in indefinite pausing, making user assets inaccessible.
Recommendation: Implement conditions to prevent ownership renunciation while the contract system is paused.
Despite not being an often behaviour it should be considered in mind: PerpetualAtlanticVault.sol, DpxEthToken.sol, RdpxDecayingBonds.sol,  RdpxV2Bond.sol

5. Missing Null Address Checks:
Description: Some constructors lack checks for the null address when initializing.
Implications: Could lead to unintended behavior or locked assets if null addresses are used in the protocol.
Recommendation: Implement stringent checks to validate against the use of the null address.
Sometimes when ignored can lead to huge issues: PerpetualAtlanticVaultLP.sol, RdpxV2Core.sol, UniV3LiquidityAmo.sol

There are also some tips which can improve the overall gas efficiency of the project:

6. Inefficient require() Statements:
Description: Using multiple conditions with && inside a single require might become less gas efficient over numerous calls.
Implications: Cumulatively, this can lead to unnecessary gas expenditures.
Recommendation: Split the conditions into multiple require statements.
It was observed in: ReLPContract.sol, UniV2LiquidityAmo.sol

7. Revert Functions & Gas Efficiency:
Description: Functions guaranteed to revert can be made payable for better gas efficiency.
Implications: Over time, a non-payable modifier can lead to higher costs for legitimate function callers.
Recommendation: Mark these specific functions as payable.
It was observed in: PerpetualAtlanticVault.sol, DpxEthToken.sol, RdpxDecayingBonds.sol, RdpxV2Core.sol, /RdpxV2Bond.sol, UniV3LiquidityAmo.sol, /UniV2LiquidityAmo.sol


8. Payable Constructors for Gas Efficiency:
Description: Some constructors can be marked as payable.
Implications: This might result in slightly decreased gas costs for deployment.
Recommendation: Where appropriate, mark constructors as payable.
It is considered saved to be updated in those contracts: DpxEthToken.sol, RdpxV2Core.sol, RdpxV2Bond.sol, /UniV2LiquidityAmo.sol

9. State Variable Arithmetic Operations:
Description: Using the += operator for state variables has been identified to be more gas-consuming than = +.
Implications: This can lead to cumulative unnecessary gas expenditures.
Recommendation: Replace the use of += with = + for state variables.
Can be changed inside - RdpxV2Core.sol, UniV2LiquidityAmo.sol


10. For Loops save gas if you change i++ to ++i: 
Description: The use of i++ in for loops, which post-increments the loop counter, is slightly less gas-efficient than pre-incrementing with ++i.
Implications: Over numerous loop iterations, this can lead to a small but unnecessary increase in gas costs.
Recommendation; Change all instances of i++ to ++i in loop constructs to achieve minor gas savings.
Seen in: in PerpetualAtlanticVault.sol, RdpxDecayingBonds.sol, RdpxV2Core.sol, UniV3LiquidityAmo.sol, and UniV2LiquidityAmo.sol.

Vulnerabilities identified within the system emphasize the importance of rigorous testing and review, especially when integrating novel functionalities. While some issues point to potential financial risks, others highlight inefficiencies that can be streamlined for improved gas usage.
Furthermore, the presence of hardcoded addresses, particularly in UniV3LiquidityAmo.sol, underscores the challenges that arise when transitioning contracts between blockchains. Migrations demand meticulous attention to ensure the integrity and functionality of the code.
Addressing these concerns is paramount for the security and efficiency of the Dopex system. By implementing the recommended changes, the platform can better safeguard its users and solidify its reputation as a reliable and optimized protocol.


### Time spent:
72 hours