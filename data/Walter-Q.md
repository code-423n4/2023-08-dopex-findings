## UniV2LiquidityAmo
1) Should add input <= 1e8 check for not setting the new precision more than 100% (1e8)
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L113
2) There's a difference on the output of given tokenA to tokenB output between the two AMO (v2-v3),imo should be present only one of those two,by reducing in this way the opportunity of arbitrage by attackers
after testing i noticed this exchange rates between the two AMO:
////////////////////////
V2: 1e16(rdpx) ==> 11001993998011983981 weth (wei)
V3: 1e16(rdpx) ==> 10999504808180183776 weth (wei)
////////////////////////
V2: 1e18(weth) ==> 49999999999999999999 rdpx (wei)
v3: 1e18(weth) ==> 49999494856333163815 rdpx (wei)
////////////////////////
using the foundry template given by the platform on the repo(Periphery.t.sol)
3) before line 149(https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L149) should be checked that tokens[i]!=LpToken,because there's no update of lpTokenBalance variable