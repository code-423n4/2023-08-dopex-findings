# [L-01] Checking `MINTER` access control twice in `mint()` function

The `mint()` function performs a double verification of the MINTER access control. This is by through the use of both the `onlyRole(MINTER_ROLE)` modifier and the `require(hasRole(MINTER_ROLE, msg.sender))` statement. Consequently, the message "Caller is not a minter" will never be triggered. 

There is 1 instances of this issue:

File: contracts/decaying-bonds/RdpxDecayingBonds.sol

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L114C3-L125C4

```solidity
  function mint(
    address to,
    uint256 expiry,
    uint256 rdpxAmount
  ) external onlyRole(MINTER_ROLE) {
    _whenNotPaused();
    require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
    uint256 bondId = _mintToken(to);
    bonds[bondId] = Bond(to, expiry, rdpxAmount);


    emit BondMinted(to, bondId, expiry, rdpxAmount);
  }
```

### Tool Used

Manual Review

### Recommendation

The require statement can be removed.

# [L-02] `emergencyWithdraw()` can be reverted if `to` address is a contract that does not implement the `receive()`

The `emergencyWithdraw()` function will be reverted the native coin transfer if the recipient `to` address is a contract that does not implement the `receive()` function. This restriction, in turn, limits the emergency withdrawal of ERC20 coins from the `RdpxDecayingBonds` contract.

There is 1 instance of this issue:

File: contracts/decaying-bonds/RdpxDecayingBonds.sol

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L89C1-L107C4

### Tool Used

Manual Review

### Recommendation

Implement code to check the `to` address is contract address or not

# [L-03] There are several setter functions that do not check if the amount is greater than 100%.

There is 8 instance of this issue:

File: contracts/core/RdpxV2Core.sol 

1. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L180
2. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L193 
3. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L455,
4. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L441

File: contracts/amo/UniV2LiquidityAmo.sol

1. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L109

File: contracts/reLP/ReLPContract.sol

1. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L90C2-L100C4
2. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L171C1-L179C4
3. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L186C1-L194C4

### Impact

Setting values to more than 100% might lead to unintended functionality.

### Tool Used

Manual Review

### Recommendation

Ensure that the values are less than 100%.

# [L-04] Lack of proper event emission at "bond()" and "redeem()" functions.

The "bond()" and "redeem()" functions in the RdpxV2Core contract does not include bondId in the Resolved events.

### Impact

The lack of emission of bondId in the event could affect users with the front-end. Additionally, external observers or off-chain systems may not have access to critical information about bonding and redemption.

There is 2 instance of this issue:

File: contracts/core/RdpxV2Core.sol

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L932

```solidity
emit LogBond(rdpxRequired, wethRequired, receiptTokenAmount);
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.
sol#L1041
```solidity
emit LogRedeem(to, receiptTokenAmount);
```

### Tool Used

Manual Review

### Recommendation

Include bondId in `LogBond` and `LogRedeem` events.

# [L‑05] Missing event for critical parameter change
Events help non-contract tools to track changes. The lack of emission of critical data in the event could affect users with the front-end. Additionally, external observers or off-chain systems may not have access to critical information about contract functions.

There are 16 instances of this issue:

File: contracts/core/RdpxV2Core.sol

1. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L378C1-L381C4
2. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L388C1-L394C4

File: contracts/amo/UniV2LiquidityAmo.sol

1. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L109

File: contracts/decaying-bonds/RdpxDecayingBonds.sol

1. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L139C3-L145C4

File: contracts/amo/UniV3LiquidityAmo.sol

1. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L155C1-L211C4
2. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L213C3-L270C4
3. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L274C1-L308C4
4. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L119C1-L133C4

File: contracts/reLP/ReLPContract.sol

1. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L115C1-L164C4
2. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L171C1-L179C4
3. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L186C3-L194C4

File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol

1. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L180C2-L182C4
2. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L185C3-L187C4
3. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L190C3-L196C4
4. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L199C3-L205C4
5. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L208C1-L214C4

# [L-06] Lack of proper event emission at "reLP" function.

The "reLP()"  function in the ReLPContract contract does not include critical information in the Resolved events.

### Impact

The lack of emission of critical data in the event could affect users with the front-end. Additionally, external observers or off-chain systems may not have access to critical information about "reLP()" function.

There is 1 instance of this issue:

File: contracts/reLP/ReLPContract.sol

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L311



### Tool Used

Manual Review

### Recommendation

Include more critical information like `lp`, `_amount` and `tokenAAmountOut` in the event.

# [N-01] Use more appropriate comments

Use more appropriate comments, otherwise the code make confusions

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L138C1-L138C39

### Recommendation 

Change `/// @param amount amount to decrease` to `/// @param amount decreased bond amount`


# [N-02] Validation is absent in the function that verifies whether it's setting the same value that already exists.

There are 8 instances of this issue:

File: contracts/core/RdpxV2Core.sol : 

1. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L180C3-L186C4
2. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L193C3-L199C4
3. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L228C3-L234C4
4. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L441C3-L448C4

File: contracts/amo/UniV2LiquidityAmo.sol

1. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L109

File: contracts/reLP/ReLPContract.sol

1. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L90C3-L100C4
2. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L171C3-L179C4
3. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L186C3-L194C4

### Recommendation

Check the value with the current value



# [N-03] "amoAddresses" are not used in the contract

`amoAddresses` in `RdpxV2Core` contract is not used in any important functions. Other than adding and removing, `amoAddresses` doesn't have ay significance in the contracts.

File: contracts/core/RdpxV2Core.sol :

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L64

```solidity
    /// @notice Array that contains the addresses of the AMO
    address[] public amoAddresses;
```

```solidity
    function addAMOAddress(address _addr) external onlyRole(DEFAULT_ADMIN_ROLE) {
        _validate(_addr != address(0), 17);
        amoAddresses.push(_addr); 
    }
```

```solidity
    function removeAMOAddress(uint256 _index) external onlyRole(DEFAULT_ADMIN_ROLE) {
        _validate(_index < amoAddresses.length, 18);
        amoAddresses[_index] = amoAddresses[amoAddresses.length - 1];
        amoAddresses.pop(); 
    }
```

# [N-04] There is no need of state variable "lpTokenBalance" and "sync()" in UniV2LiquidityAmo.sol

There is no need of tracking state variable "lpTokenBalance" and calling "sync()" method in UniV2LiquidityAmo.sol. Because, You can always get the value by calling `IERC20WithBurn(addresses.pair).balanceOf(address(this))`

So, Instead of `lpTokenBalance` you can have a view function with the same name `lpTokenBalance` which returns the pair balance.

```solidity
  function lpTokenBalance() public view returns(uint256){
    return IERC20WithBurn(addresses.pair).balanceOf(address(this));
  }
```

