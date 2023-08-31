## QA Report (low / non-critical)

#### [L-1] There are no restrictions how big value can be:

- RdpxV2Core:
    - `_validate(_rdpxBurnPercentage > 0, 3)`
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L180-L186
    - `_validate(_rdpxFeePercentage > 0, 3)`
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L193-L199
    - `_validate(_bondMaturity > 0, 3)`
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L228-L234
    - `_validate(_bondDiscountFactor > 0, 3)`
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L441-L448
    - `_validate(_slippageTolerance > 0, 3)`
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L455-L462

- PerpetualAtlanticVault:
    - https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L237-L241

- UniV2LiquidityAmo:
    - `require(_slippageTolerance > 0, "reLPContract: slippage tolerance must be greater than 0")`
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L109-L117

- ReLPContract:
    - `require(_reLPFactor > 0, "reLPContract: reLP factor must be greater than 0")`
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L90-L100
    - `require(_slippageTolerance > 0, "reLPContract: slippage tolerance must be greater than 0")` 
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L186-L194
    - `require(_liquiditySlippageTolerance > 0, "reLPContract: liquidity slippage tolerance must be greater than 0")`
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L171-L179

It is important to have predefined limits. Setting one of these variables with too big value could have potential consequences like reverts on overflow / underflow or division by zero.

#### [L-2] Hard-coded slippage in RdpxV2Core and ReLPContrct. 

`slippageTolerance` is implemented as a state variable. Having predefined value for slippage may result in unoptimal swaps. Depending on the present condition of the pool inadequate slippage value may result in loss of tokens. Make the max slippage parameterized in the function `_curveSwap` in RdpxV2Core and `ReLP` in ReLPContract. 

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L100
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L76

#### [L-3] BondDiscountFactor is not initialized in RdpxV2Core so it's value is equal to zero.
Uninitialized state variables which are used in calculation may negatively affect the result. Make sure to initialize all important state variables in constructor.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L109

#### [L-4] Wrong assumption in `_curveSwap` when the `coin0 != weth`.
When `coin0 != weth` then using this line of code `(uint256 a, uint256 b) = coin0 == weth ? (0, 1) : (1, 0);` variables are initialized as:
`uint256 a = 1` and `uint256 b = 0`
Later in `if(validate)` the values of `ethBalance` and `dpxEthBalance` will be set the other way around which may cause wrong assumptions later in that function or revert the whole transaction in the `if` statement.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L524-L542

#### [N-1] Allow redeeming at maturity in RdpxV2Core.
`_validate(block.timestamp > bonds[id].maturity, 7);`
Change `>` to `>=` to allow users to redeem bond when it reaches maturity timestamp.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1023

#### [N-2] Comment might confuse users.
It is written that fee needs to be greater than 1% but in reality i can also be 1% so change comment to `greater or equal 1%`.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L950

Only Rdpx is transfered not Rdpx and WETH. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L652

#### [N-3] `strike` and `timeToExpiry` are calculated even when there are not used.
Put it into `if(putOptionsRequired)` statement to improve code clarity.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1189-L1198

#### [N-4] Open to do in PerpetualAtlanticVault.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L84-L87

#### [N-5] `BURNER_ROLE` is not set up in constructor in DpxEthToken.
Same thing with `RDPXV2CORE_ROLE` in RdpxDecayingBonds
In ReLP the `RDPXV2CORE_ROLE` is set to msg.sender. Msg.sender is also set as `DEFAULT_ADMIN_ROLE`. I belive `RDPXV2CORE_ROLE` should be set to RdpxV2Core address.

#### [N-6] Hard-coded addresses in UniV3LiquidityAmo.sol.
It is better to implement state variables which are initialized in constructor. Also create setter functions to update those addresses in case of emergency at the current address. Using hard-coded addresses could be also a problem when trying to deploy to different chains where these addresses might differ.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L82-L88

#### [N-7] Removing asset from reserve does not transfer tokens from contract.
`removeAssetFromtokenReserves` does not transfer tokens after removing it from the reserves. To transfer them protocol would have to `pause` the contract and call `emergencyWithdraw` or use `approveContractToSpend` and transferFrom RdpxV2Core to desired address. Impement safeTransfer function when removing token from reserve.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L270-L290
 










