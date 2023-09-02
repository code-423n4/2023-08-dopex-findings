The protocol's scope consist of several contract and as well there are other contracts not in scope, but it was possible to audit the scope without going much into details of the oos part. The documentation is satisfying and contains useful description of the protocol. 

During the audit, I first gave a quick glance to each contract.
During that phase, focused on finding low hanging fruits.
Later, tried to map the whole system together and use a more holistic approach to find possible logic issues.

1. The contracts in scope
The contracts are described below.

- UniV2Liquidity - As per the contest description, this contract encompasses all functions for the Uniswap V2 AMO. When analyzing the contract, it’s also the same conclusion that it is an interface to Uniswap. It has a single owner, or rather an admin inherited from OZ’s AccessControl. The contract is mostly to be used by the Admin, in form of DEFAULT_ADMIN_ROLE, except for view functions and one function sync() which is just used to keep the balance updated.

- UniV3Liquidity - This contract has a similar role as UniV2Liquidity. The main difference is that it serves as an interface to Uniswap V3. Similarly to V2, it is also managed by a DEFAULT_ADMIN_ROLE, assigned upon deployment.

- RdpxV2Core - This contract is a main entry point and contains some key setters which manage the most critical parameters of the system. It allows user interactions with the protocol using functions
function bondWithDelegate(address _to,uint256[] memory _amounts,uint256[] memory _delegateIds,uint256 rdpxBondId) public
function bond(uint256 _amount,uint256 rdpxBondId,address _to) public
function addToDelegate(uint256 _amount,uint256 _fee) external returns
function withdraw(uint256 delegateId) external
function sync() external {
function redeem(uint256 id,address to) external




### Time spent:
45 hours