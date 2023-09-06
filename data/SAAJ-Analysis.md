# Comments for Judge
My findings are mainly related to edge cases of erc20 token which are commonly known and can be manipulated by a malicious attacker causing loss to the protocol and its user.
2 medium findings are related to inadequate checks within the functions that can be manipulated and misuse to cause loss of funds to the user indirectly.
QA and Gas reports are based on improving the working flow of the codebase that will help the protocol in avoiding mishaps that may cause limitation to protocol in providing services to the users effectively and efficiently.

# Approach
The approach i, take for reviewing the codebase is first going through the product specification provide at the notion site.
This helps me in getting clear idea of how different collaterals are added to the protocol in providing liquidity and issuing bond. The bonding mechanism developed to maintain the token dpxETH pegged with the Eth. The understanding of codebase helps me in getting in-depth of looking at areas where the issuance, bonding and swapping of tokens does not working expectedly and able to find multiple findings.

# Architecture Recommendations
The protocol should implement cryptographic process to limit manipulation of any function that is directly related to dealing of funds like liquidation and swapping.

# Codebase quality analysis
Codebase is fork of uniswap but does not make modification related to recent changes that were necessary to have an effective and secure code base with no consideration for avoiding common security pitfalls.

# Centralization risks
The protocol should limit certain administrator roles that may limit the use of protocol by users to a certain area. Centralisation also poses risk to the protocol through manipulation of certain functions like burning of bond amount and shares that may cause loss of funds to users.

# Mechanism review
The mechanism of protocol is linked with other ERC20 tokens providing collateral to the protocol own token that needs to be pegged with Ether. The mechanism is based on providing bond shares that will be the main base for the protocol liquidity.
 
# Systemic risks
The risk is mainly linked to difference in decimals of different erc20 token that may cause loss to either user or protocol if not handled properly.
Risk is also linked to many investors taking out liquidity at the same time that may depegged the protocol token.


### Time spent:
28 hours