The protocol's scope consist of several contract and as well there are other contracts not in scope, but it was possible to audit the scope without going much into details of the oos part. The documentation is satisfying and contains useful description of the protocol. 

During the audit, I first gave a quick glance to each contract.
During that phase, focused on finding low hanging fruits.
Later, tried to map the whole system together and use a more holistic approach to find possible logic issues.

1. The contracts in scope
The contracts are described below.

- UniV2Liquidity - As per the contest description, this contract encompasses all functions for the Uniswap V2 AMO. When analyzing the contract, it’s also the same conclusion that it is an interface to Uniswap. It has a single owner, or rather an admin inherited from OZ’s AccessControl. The contract is mostly to be used by the Admin, in form of DEFAULT_ADMIN_ROLE, except for view functions and one function sync() which is just used to keep the balance updated.

- UniV3Liquidity - This contract has a similar role as UniV2Liquidity. The main difference is that it serves as an interface to Uniswap V3. Similarly to V2, it is also managed by a DEFAULT_ADMIN_ROLE, assigned upon deployment.

- RdpxV2Core - This contract is a main entry point and contains some key setters which manage the most critical parameters of the system. It allows user interactions with the protocol using functions
`bondWithDelegate`, `bond`, `addToDelegate`, `withdraw`, `sync`, `redeem`. All other functions are privileged use only.

- RdpxV2Bond.sol - The Bond ERC721 contract, this is a rather standard ERC721 which is pausable, the deployer has all roles (MINTER and DEFAULT_ADMIN)

- RdpxDecayingBonds - ERC721, similar to above with some additional functions. 

- DpxEthToken - The contract represents the DpxEthToken. It is an ERC20 token with some additional code – instead of Owner it implements Access with ADMIN, PAUSER and MINTER roles and a pause check implemented in a  _beforeTokenTransfer hook.

-  PerpetualAtlanticVault - is used by the Core contract to supply the PUT options.

- PerpetualAtlanticVaultLP - a vault that can be interacted with by users and is also called by the PerpetualAtlanticVault. An ERC4626-like, however not correctly following the standard which I already submitted as a separate issue

- ReLPContract - it is a contract with only one function responsible for performing the RELP strategy, so withdrawing and re-LPing to uniswap.

2. Architecture and codebase
- The main point of user interaction is the Core Contract
- User can also interact with PerpetualAtlanticVaultLP
- Overall the user entries are rather clear
- All other contract are only for privileged use (admin or contracts)

- There are numerous code pieces which could be reused by inheritance, instead of placing it in each contract. For example, the pause logic is declared in numerous places. 
- The overall quality of code is satisfying but could be better. There are some events missing, and some leftovers from development process like `Event log(uint)`. 

3. Centralization risks
- The protocol is very centralized. The DEFAULT_ADMIN_ROLE can set so many parameters it could influence the whole protocol. Even when assuming the best intentions of the team, it should be noted that high degree of centralization may cause the admin role to become a single point of failure, and if its compromised, then the whole protocol could be destroyed and robbed. Having that in mind, a minimal solution is to use a multisig for the privileged roles. Optimal solution would be to minimize the centralization, set the known parameters upfront and one-time, and reduce the available changes to be performed on the protocol to the minimum.

Aside the DEFAULT_ADMIN_ROLE, the contracts use functions similar to initializers, e.g. `setAddresses()` in the Core contract. However, they are multiple use so if a compromise happens, this function could be easily used to destroy the protocol by setting key addresses to malicious ones.

4. Other risks
The protocol interacts with Uniswap. There are several points related to slippage which have been already found in the Bot contest. 





### Time spent:
45 hours