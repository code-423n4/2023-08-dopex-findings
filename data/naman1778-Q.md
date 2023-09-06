## [N-01] Lack of address(0) checks in the constructor

Zero-address check should be used in the constructors, to avoid the risk of setting smth as address(0) at deploying time.

    File: contracts/amo/UniV3LiquidityAmo.sol	

    76:  constructor(address _rdpx, address _rdpxV2Core) {

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol

    File: contracts/core/RdpxV2Core.sol	

    124: constructor(address _weth) {

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol

    File: contracts/perp-vault/PerpetualAtlanticVault.sol	

    81: constructor(

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol

## [N-02] According to the syntax rules, use *=> mapping (* instead of *=> mapping(* using spaces as keyword

    File: contracts/perp-vault/PerpetualAtlanticVault.sol	

    69: mapping(uint256 => mapping(uint256 => uint256))

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol*

## [N-03] Contract Owner Has Too Many Privileges

The owner of the contracts has too many privileges relative to standard users. The consequence is disastrous if the contract owner’s private key has been compromised. And, in the event the key was lost or unrecoverable, no implementation upgrades and system parameter updates will ever be possible.

For a project this grand, it increases the likelihood that the owner will be targeted by an attacker, especially given the insufficient protection on sensitive owner private keys. The concentration of privileges creates a single point of failure; and, here are some of the incidents that could possibly transpire:

Transfer ownership and mess up with all the setter functions, hijacking the entire protocol.

Consider:

--splitting privileges (e.g. via the multisig option) to ensure that no one address has excessive ownership of the system,
--clearly documenting the functions and implementations the owner can change,
--documenting the risks associated with privileged users and single points of failure, and
--ensuring that users are aware of all the risks associated with the system.
