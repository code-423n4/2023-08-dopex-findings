Within the constructor function of the contracts **UniV2LiquidityAmo.sol**, **UniV3LiquidityAmo.sol**, **RdpxV2Core.sol**, **RdpxV2Bond.sol**, and **PerpetualAtlanticVault.sol**, there is a use of the **_setupRole function** from the AccessControl contract for initializing the DEFAULT_ADMIN_ROLE roles. However, it should be noted that this approach is no longer recommended in the current version of openzeppelin-contracts-access-AccessContol.sol that is being used, the **_setupRole function** is deprecated. It is advisable to consider utilizing the **_grantRole function**, as demonstrated in the constructor of **DpxEthToken.sol**, for this purpose instead.

**_setupRole function** used in these contracts is now deprecated in the current version of openzeppelin-contracts-access-AccessContol.sol 
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol : Line 80
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol : Line 58
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol : Line 125
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol: Line 25 & 26
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol: Line 61 & 62
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol: Line 126 & 127
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol Line 80 & 81


    constructor() {
        _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
      }


Consider utilizing the **_grantRole function**, as used in the constructor of **DpxEthToken.sol**
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol: Line 24, 25, & 26

    constructor() ERC20("Dopex Synthetic ETH", "dpxETH") {
         _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
         _grantRole(PAUSER_ROLE, msg.sender);
         _grantRole(MINTER_ROLE, msg.sender);
      }
