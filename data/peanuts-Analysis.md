### 1. Any comments for the judge to contextualize your findings

- Findings are largely focused on RdpxV2Core, UniV2LiquidityAmo and reLPContract

### 2. Approach taken in evaluating the codebase

- I focused on the e2e scenario from bonding -> purchase -> stake -> reLP -> bond issuing
- Also focused on the monetary flow of the codebase. For eg, calling bond() with 1 as _amount() leads to user paying 12.25 rDPX and 0.8025 ETH. Of the 12.25 rDPX, half gets burned, half goes to the veDPX holders. The 12.25 rDPX is then reimbursed by the RdpxReserve + incentive. Of the 0.8025 ETH spent, 0.5 ETH goes to the receiptToken contract to mint receipt tokens, 0.06125 goes to the perpetualAtlanticVault to be paid as premium and 0.245 goes to the core contract. At the end, the core contract has 12.75 rDPX and 0.245 ETH. 
- Examined the adding / removing of liquidity of UniswapV2 and the swaps.

### 3. Mechanism review

1. The distribution of rDPX to veDPX holders is not clear, as the contract merely transfers half of the rDPX to the feeDistributor address, which is set as address(100) in the test suite

> 50% of the rDPX provided for bonding is burnt from the Treasury Reserve and another 50% is sent as emissions to veDPX holders. These percentages are variable and can be controlled by governance.

```
      // transfer the rdpx to the fee distributor
      IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress)
        .safeTransfer(
          addresses.feeDistributor,
          (_rdpxAmount * rdpxFeePercentage) / 1e10
        );
```

### 4. Centralization risks

1. The whole protocol relies on a sole admin account, which can be pretty dangerous if the owner turns malicious

```
onlyRole(DEFAULT_ADMIN_ROLE)
```

2. The protocol uses a pausing system for most of its contract, and only the admin can toggle the pause.
```
contract RdpxV2Core is
  IRdpxV2Core,
  AccessControl,
  ContractWhitelist,
  ERC721Holder,
->  Pausable
```

### 5. Systemic risks

1. The admin can manipulate the core reserve through upperDepeg() or lowerDepeg(). There are no restrictions for the usage of rDPX or WETH in the reserve contract. Admin can call lowerDepeg() with all of the rDPX and WETH in the reserves and swap them to dpxETH. Likewise, admin can mint an exorbitant amount of dpxETH by calling upperDepeg() and manipulating the dpxETH-ETH peg.

I'd reckon there should be a limit to how much _amount() the function can use, but then again the admin can just repeatedly call the function to produce the same malicious effect. 

2. The protocol relies on its reserve for the whole process to work. For example, when a user wants to call bond(), the rDPX reserve must have some rDPX inside to compensate for the burn and the transfer to veDPX emission holders. Similarly, for lowerDepeg(), the core contract must have some WETH or rDPX in order to bring back the peg, otherwise the function wouldn't work.

3. There is a fixed timestamp used in UniV3LiquidityAmo.sol in the swap function which may not be ideal because this number will expire in 13 years. Better to use a user input timestamp.

```
    ISwapRouter.ExactInputSingleParams memory swap_params = ISwapRouter
      .ExactInputSingleParams(
        _tokenA,
        _tokenB,
        _fee_tier,
        address(this),
->      2105300114, // Expiration: a long time from now
        _amountAtoB,
        _amountOutMinimum,
        _sqrtPriceLimitX96
      );
```

4. Usage of safeApprove() is deprecated. Better to use safeIncreaseAllowance() or safeDecreaseAllowance()

### Time spent:
18 hours