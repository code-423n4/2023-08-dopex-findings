**contracts/amo/UniV2LiquidityAmo.sol**
- L12/14/48 - There are multiple imports and creations of variables in storage that are never used, thus reducing overall code understanding.


**contracts/perp-vault/PerpetualAtlanticVault.sol**
- L127 - The MANAGER_ROLE role is created, but it is never used throughout the project, therefore it is not necessary to define this role, since it can cause confusion in the code.

- L658/659/660/661/662/663/664 - Error codes that are defined in comments should be written in natspec or in the code itself.


**contracts/core/RdpxV2Core.sol**
- L126 - The variable in storage, weth, only has a setting in the constructor, therefore it could be defined as an immutable variable. Therefore, in the constructor you should check that the address input of the constructor is != 0x.

- L1141/1142 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123 ,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.

- L1289-1308 - Error codes that are defined in comments should be written in natspec or in the code itself.


**contracts/perp-vault/PerpetualAtlanticVaultLP.sol**
- L128/130/132 - Code does not follow the best practice of check-effects-interaction (add new case)


**contracts/reLP/ReLPContract.sol**
- L90 - The function setreLpFactor does not fully respect the camelCase, the correct name of the function should be setReLpFactor


**contracts/amo/UniV3LiquidityAmo.sol**
- L22 - An abstract contract is created, OracleLike, which is not used in the whole project, therefore it should be eliminated. This generates an extra gas expense and confusion in the understanding of the code.

- L35/36/37/41/42/46/63/66 - CamelCase is mixed with SnakeCase, this does not meet any standard.

- L77/78 - The variables in storage, rdpx and rdpxV2Core, only have constructor settings, therefore they could be defined as immutable variables. Therefore, in the constructor you should check that the two address inputs of the constructor are != 0x.

- L106 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.


**contracts/decaying-bonds/RdpxDecayingBonds.sol**
- L34/139 - The BURNER_ROLE role is used, but the function decreaseAmount is never set, therefore, it would have a DoS, as default value.
Until the grantRole() function is executed and the value is set.

- L56/70/76 - There is a lot of duplicate logic in multiple contracts (RdpxV2Core, PerpetualAtlanticVault and RdpxReserve), this could be unified in an abstract contract. Mainly the logic of the pause(), unpause() and the setting in the constructor
from the _setupRole of MINTER_ROLE, DEFAULT_ADMIN_ROLE and also _tokenIdCounter.increment().


**contracts/dpxETH/DpxEthToken.sol**
- L21/43/50 - The BURNER_ROLE role is used, but it is never set therefore the burn and burnFrom functions would have a DoS, as default value.
Until the grantRole() function is executed.
