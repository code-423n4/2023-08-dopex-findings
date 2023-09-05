# [L-01] Fix Typos

links to typos:
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L84
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L117

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L273

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L816
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L240
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L117
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L99
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L60
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L52

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L75

# [L-02] Make constants instead of raw values

Replace all occurrences of constant like `1e10` with `ONE_HUNDRED_PERCENT`

List:

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L658
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L658
`1e10` with `ONE_HUNDRED_PERCENT`

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L669
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L673
`1e8` with `ONE_TOKEN`

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L947
`100e8` with `ONE_HUNDRED_PERCENT`

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L949
`1e16` with `MILI_ETHER`

And in all other files the same.

# [L-03] Unify withdraw functions
From now, we have different implementations of `emergencyWithdraw`

```solidity
  function emergencyWithdraw(
    address[] calldata tokens
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    IERC20WithBurn token;

    for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }

    emit LogEmergencyWithdraw(msg.sender, tokens);
  }
```

```solidity
  function emergencyWithdraw(
    address[] calldata tokens
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _whenPaused();
    IERC20WithBurn token;

    for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }

    emit EmergencyWithdraw(msg.sender, tokens);
  }
```

```solidity
  function emergencyWithdraw(
    address[] calldata tokens,
    bool transferNative,
    address payable to,
    uint256 amount,
    uint256 gas
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _whenPaused();
    if (transferNative) {
      (bool success, ) = to.call{ value: amount, gas: gas }("");
      require(success, "RdpxReserve: transfer failed");
    }
    IERC20WithBurn token;

    for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }
  }
```

These functions must be the same.

Make single contract `EmegencyWithdraw.sol` and implement all your contract with inheritance from this contract.

# [L-04] Don't assign role on msg.sender

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L25
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol#L26

`DpxEthToken` minted/burned by Core contract, and here `msg.sender` get `MINTER` and `BURNER` role, 
which must be reassigned latte.
At the first creator can mint multiple tokens to any address before transfer role.
At the second creator can give role to another address.

This is bad solution. Better use approach like in UniswapV3 (I described it in analysis report).


# [L-05] User might have a lot of decaying bonds
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L151-L160

If user has a lot of bonds there might be a problem to receive all list of items.
Due to limit of external nodes like Infura.

Better use pagable pattern -- i.e. request only subpart of bonds.

# [L-06] Unused role
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L45
```solidity
bytes32 public constant MANAGER_ROLE = keccak256("MANAGER_ROLE");
```

This role created, but not assigned.

# [L-07] Copy-paste

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L512
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L516
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L521


This: `(currentFundingRate * (block.timestamp - startTime)) / 1e18` might be assigned to variable.
This improves readability and saves gas.

# [L-08] Unreached code

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L505-L507


`lastUpdateTime == 0` never will be 0 because it's moved anyway in `updateFundingPaymentPointer();`
so rewrite this code to:
`uint256 startTime =  lastUpdateTime;`

# [L-09] wrong function naming
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L240
addAssetTotokenReserves -> addAssetToTokenReserves

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L270
removeAssetFromtokenReserves -> removeAssetFromTokenReserves

# [L-10] Unused array
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L64

amoAddresses only filled, but not used anyway. 

# [L-11] Incorrect error params
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L410

Due to unclear error code and bad design -- developers created a lot of mistakes while create validation code

```solidity
_validate(_amount > 0, 17);
```
error code shouldn't be 17, because: `E17: "Zero address"`

# [L-12] Incorrect comment
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L652

# [L-13] Improve _transfer
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L624-L685

1) Swap if clauses to be consistent with `calculateBondCost`;
2) Use constant `ONE_HUNDRED_PERSENT` instead of 1e10;
3) Move out to variable: `(_rdpxAmount * rdpxBurnPercentage) / 1e10`
4) Use constant `DEFAULT_PRECISION` instead of 1e8.

# [Q-01] Set addresses might fail 
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L181-L212
`// Transfer rDPX and ETH token from user`
You already transferred ETH, so rewrite to:
`// Transfer rDPX token from user`

Current implementation might fail if any address will be changed twice due to `safeApprove` implementation
```solidity
collateralToken.safeApprove(
      addresses.perpetualAtlanticVaultLP,
      type(uint256).max
    );
```

see:
```solidity
    function safeApprove(IERC20 token, address spender, uint256 value) internal {
        // safeApprove should only be called when setting an initial allowance,
        // or when resetting it to zero. To increase and decrease it, use
        // 'safeIncreaseAllowance' and 'safeDecreaseAllowance'
        require(
            (value == 0) || (token.allowance(address(this), spender) == 0),
            "SafeERC20: approve from non-zero to non-zero allowance"
        );
        _callOptionalReturn(token, abi.encodeWithSelector(token.approve.selector, spender, value));
    }
```
This can be handled by:

```solidity
  function setLpAllowance(bool increase) external onlyRole(DEFAULT_ADMIN_ROLE) {
    increase
      ? collateralToken.approve(
        addresses.perpetualAtlanticVaultLP,
        type(uint256).max
      )
      : collateralToken.approve(addresses.perpetualAtlanticVaultLP, 0);
  }

```

But this is not convenient, due to a lot of additional actions. 
Better separate `setAddresses` by multiple atomic setters like:

```solidity
setOptionPricing
setAssetPriceOracle
...
```

# [Q-02] Not needed to handle ATM options

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L332-L333


Current implementation handles ATM options, but your doc told that you don't settle ATM options
https://docs.dopex.io/option-fundamentals/option-settlement

More-over here not any meaning to handle it, because you got same amount of funds but in another tokens.

So, forbid to settle ATM option, and implement option settlement as was described in my previous report.

# [Q-03] Rename variable to symbols
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L70
`string[] public reserveTokens;`

Actually, they are symbols, not tokens.