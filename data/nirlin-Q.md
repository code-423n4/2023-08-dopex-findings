
## LOW - 1  - Set some upper and lower limits on important setter function

Many of the setter function for fees, slippage and discounts are left uncapped, some have lower limit have zero but no upper ones can be set to 100 or more too, set some upper and lower limit validation for them. Such function are

In rdpxv2core.sol

```solidity
  function setBondDiscount(
    uint256 _bondDiscountFactor
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_bondDiscountFactor > 0, 3);
    bondDiscountFactor = _bondDiscountFactor;

    emit LogSetBondDiscountFactor(_bondDiscountFactor);
  }
  
  
   function setSlippageTolerance(
    uint256 _slippageTolerance
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _validate(_slippageTolerance > 0, 3);
    slippageTolerance = _slippageTolerance;

    emit LogSetSlippageTolerance(_slippageTolerance);
  }
```
In uniV2LiquidityAMO.sol

```solidity
  function setSlippageTolerance(
    uint256 _slippageTolerance
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(
      _slippageTolerance > 0,
      "reLPContract: slippage tolerance must be greater than 0"
    );
    slippageTolerance = _slippageTolerance;
  }
```
In ReLPContract.sol
```solidity
  function setSlippageTolerance(
    uint256 _slippageTolerance
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(
      _slippageTolerance > 0,
      "reLPContract: slippage tolerance must be greater than 0"
    );
    slippageTolerance = _slippageTolerance;
  }

 function setLiquiditySlippageTolerance(
    uint256 _liquiditySlippageTolerance
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(
      _liquiditySlippageTolerance > 0,
      "reLPContract: liquidity slippage tolerance must be greater than 0"
    );
    liquiditySlippageTolerance = _liquiditySlippageTolerance;
  }

  /**
   * @notice sets the slippage tolerance
   * @dev    Can only be called by admin
   * @param  _slippageTolerance the slippage tolerance
   */
  function setSlippageTolerance(
    uint256 _slippageTolerance
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(
      _slippageTolerance > 0,
      "reLPContract: slippage tolerance must be greater than 0"
    );
    slippageTolerance = _slippageTolerance;
  }
 
```
## LOW - 2  - In `setisReLP()` revert if passed value is equal to already set value
 `setisReLP()` in core v2 contract takes the bool to set to, if the passed bool is already set value simply revert:
 
 https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L206-L209
 
 ```solidity
  function setIsreLP(bool _isReLPActive) external onlyRole(DEFAULT_ADMIN_ROLE) {
    isReLPActive = _isReLPActive;
    emit LogSetIsReLPActive(_isReLPActive);
  }
  ```
  
## LOW - 3  - Unbounded arrays will be dossed in `addAssetTotokenReserves`
In this function reserve asset array is traversed to check for dups, which can lead to out of gas error if too many assets are added. Instead the enumerable mapping by openzeppelin can be used.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L240-L251

  function addAssetTotokenReserves(
    address _asset,
    string memory _assetSymbol
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");

    for (uint256 i = 1; i < reserveAsset.length; i++) {
      require(
        reserveAsset[i].tokenAddress != _asset,
        "RdpxV2Core: asset already exists"
      );
    }
    
## LOW - 4  - missing recovery functions in relp

Every important contract in scope have the emergency withdraw functoin but the relp contract is missing it. Also dev was not sure too why they missed it .

Consider adding the emergency withdraw to the relp

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol

## Low - 5  - onlyEoa modifier may not work in future
the whitelisting contract used checks if the caller is contract or not using the only EoA approach of comparing the tx.origin and msg.sender but it may not work 

For onlyEOAEx, tx.origin is used to ensure that the caller is from an EOA and not a smart contract.
```solidity
  function _isEligibleSender() internal view {
    // the below condition checks whether the caller is a contract or not
    if (msg.sender != tx.origin)
      require(whitelistedContracts[msg.sender], "Contract must be whitelisted");
  }
```

However, according to [EIP 3074](https://eips.ethereum.org/EIPS/eip-3074#abstract),

This EIP introduces two EVM instructions AUTH and AUTHCALL. The first sets a context variable authorized based on an ECDSA signature. The second sends a call as the authorized account. This essentially delegates control of the externally owned account (EOA) to a smart contract.

Therefore, using tx.origin to ensure msg.sender is an EOA will not hold true in the event EIP 3074 goes through. such making all the _isEligibleContract() calls useless. So better approach is to focus on design of protcol, such that even if caller is not contract it should not cause any issue or not be exposed to anything.

IMO this can be upgraded to medium if judge finds it necessary but for now adding it into QA.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/helper/ContractWhitelist.sol#L35-L39


## NC- 1  - In `withdraw()` function in core contract change the amount withdrawn variable name
    
The name of variable suggests as the amount that have been withdrawn but in actuality it is the amount that is availible to withdraw, instead name it amountAvailibleForWithdraw or something along those lines;

```solidity
  function withdraw(
    uint256 delegateId
  ) external returns (uint256 amountWithdrawn) {
    _whenNotPaused();
    _validate(delegateId < delegates.length, 14);
    Delegate storage delegate = delegates[delegateId];
    _validate(delegate.owner == msg.sender, 9);

    // lets say the delegate.amount = 100 and delegate.activeCollateral = 90
    // withdraw availible = 100 - 90 = 10
    amountWithdrawn = delegate.amount - delegate.activeCollateral;
    _validate(amountWithdrawn > 0, 15);
    // amount = 90
    delegate.amount = delegate.activeCollateral;

    IERC20WithBurn(weth).safeTransfer(msg.sender, amountWithdrawn);

    emit LogDelegateWithdraw(delegateId, amountWithdrawn);
  }
  ```
  
  https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L975-L990
  
## NC- 2  -_setupRole has been deprecated for _grantRole in the openzepplein access control but are being still used
_setup role function have been deprecated in the access code library function but is still being used by the protocol instead of _grantRole, instead use the _grantRole

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L24C1-L27C4

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L79-L82

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L57C1-L59C4

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L126C1-L127C42

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L81

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L61C1-L62C48

## NC-3- Remove the comment about erc777 as it is not supported by protocol any more
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L127
```solidity
  function deposit(
    uint256 assets,
    address receiver
  ) public virtual returns (uint256 shares) {
    // Check for rounding error since we round down in previewDeposit.
    require((shares = previewDeposit(assets)) != 0, "ZERO_SHARES");

    perpetualAtlanticVault.updateFunding();

    // Need to transfer before minting or ERC777s could reenter.
    collateral.transferFrom(msg.sender, address(this), assets);

    _mint(receiver, shares);

    _totalCollateral += assets;

    emit Deposit(msg.sender, receiver, assets, shares);
  }
  
```


## NC-4- Remove the Slippage tolerance from the v2 amo as it is not being used but is only set in the setter function or either use it.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L51

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L109-L117

## NC-5- wrong comment in v2 amo, it should have been token1 instaed of token a in swap function
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L320

