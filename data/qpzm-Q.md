## 1. In reLP, rDPX to remove calculation is different from the docs.
In Bonding Mechanism > Regular bonding part, it divides by `rdpx_supply`, but it is `tokenAInfo.tokenAReserve` in the `ReLPContract.reLP` code.
`rDPX to remove:  ((amount_lp * 4) / rdpx_supply) * lp_rdpx_reserves * base_relp_percent where base_relp_percent = Math.sqrt(reserves_rdpx) * relp_factor`
Reference
- https://dopex.notion.site/rDPX-V2-RI-b45b5b402af54bcab758d62fb7c69cb4#23161c577fd1449085436e5a601358ae
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L232-L235

## 2. `RdpxV2Core.setBondDiscount` function asserts that `bondingDiscountFactor` must be over zero, but the docs says it may be under zero.
> rDPXLPBondingDiscount is a constant that is set and changeable via governance within predefined parameters. This allows the size and direction of the bonding discount to be adjusted based on market conditions (where BondingDiscount is negative, bonding is disincentivized since it is cheaper to buy $dpxETH from the open market).
https://docs.dopex.io/tokenomics/usdrdpx/bonding#bonding-usdrdpx
 
Reference
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L441-L448
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1163-L1165
- 
## 3. `RdpxV2Core.lowerDepeg` uses `block.timestamp + 10` as deadline in `IUniswapV2Router.swapExactTokensForTokens`.
The transaction can be pending in mempool for a long. Without deadline check, the trade transaction can be executed in a long time after the user submit the transaction, at that time, the trade can be done in a sub-optimal price, which harms user's position.
https://solodit.xyz/issues/h-11-the-deposit-withdraw-trade-transaction-lack-of-expiration-timestamp-check-and-slippage-control-sherlock-ajna-ajna-git

Reference
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1103

## 4. `RdpxV2Core.lowerDepeg` does not approve `addresses.dopexAMMRouter` 
Recommendation: approve `addresses.dopexAMMRouter` in `RdpxV2Core.setAddresses` like weth.
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L343-L348

## 5. `PerpetualAtlanticVaultLP.redeem` revert Arithmetic over/underflow if allowance is not enough.
### Links to affected codes
- https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L155-L157

### POC
Add the code below in `tests/perp-vault.POC.sol` and run `forge test --mt test_redeem`.
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.19;

import { Test } from "forge-std/Test.sol";

import { ERC721Holder } from "@openzeppelin/contracts/token/ERC721/utils/ERC721Holder.sol";
import { Setup } from "./Setup.t.sol";
import { PerpetualAtlanticVault } from "contracts/perp-vault/PerpetualAtlanticVault.sol";
import { console } from "forge-std/console.sol";

contract POC is ERC721Holder, Setup {
    // ================================ HELPERS ================================ //
    function mintWeth(uint256 _amount, address _to) public {
        weth.mint(_to, _amount);
    }

    function mintRdpx(uint256 _amount, address _to) public {
        rdpx.mint(_to, _amount);
    }

    function deposit(uint256 _amount, address _from) public {
        vm.startPrank(_from, _from);
        vaultLp.deposit(_amount, _from);
        vm.stopPrank();
    }

    function purchase(uint256 _amount, address _as) public returns (uint256 id) {
        vm.startPrank(_as, _as);
        (, id) = vault.purchase(_amount, _as);
        vm.stopPrank();
    }

    function setApprovals(address _as) public {
        vm.startPrank(_as, _as);
        rdpx.approve(address(vault), type(uint256).max);
        rdpx.approve(address(vaultLp), type(uint256).max);
        weth.approve(address(vault), type(uint256).max);
        weth.approve(address(vaultLp), type(uint256).max);
        vm.stopPrank();
    }

    // ================================ CORE ================================ //

    function test_redeem() external {
        setApprovals(address(1));
        mintWeth(20 ether, address(1));
        weth.transfer(address(vaultLp), 100);

        deposit(1, address(1));
        vaultLp.redeem(1, address(1), address(1));
    }
}
```

### Recommendation
Check `allowed >= shares` and if not, revert with a message that says the allowance is not enough.
