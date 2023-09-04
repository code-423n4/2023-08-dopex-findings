# Give the deployer unnecessary privileges

## Links to affected code

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L26](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L26)

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L81](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L81)

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/dpxETH/DpxEthToken.sol#L26](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/dpxETH/DpxEthToken.sol#L26)

## Impact

Too much authority is granted to the deployer.

## Proof of Concept

### RdpxV2Bond

The deployer of RdpxV2Bond is not the RdpxV2Core contract. There is no need to grant `MINTER_ROLE` to contract deployer.

```solidity
constructor() ERC721("rDPX V2 Bond", "rDPXV2Bond") {
  _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
  _setupRole(MINTER_ROLE, msg.sender);
}
```

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L24-L27](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L24-L27)

### **ReLPContract**

The deployer of ReLPContract is not the RdpxV2Core contract. There is no need to grant `RDPXV2CORE_ROLE` to contract deployer.

```solidity
constructor() {
  _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
  _setupRole(RDPXV2CORE_ROLE, msg.sender);
}
```

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L79-L82](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L79-L82)

### **DpxEthToken**

The deployer of DpxEthToken is not the RdpxV2Core contract. There is no need to grant `MINTER_ROLE` to contract deployer.

```solidity
constructor() ERC20("Dopex Synthetic ETH", "dpxETH") {
  _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
  _grantRole(PAUSER_ROLE, msg.sender);
  _grantRole(MINTER_ROLE, msg.sender);
}
```

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/dpxETH/DpxEthToken.sol#L26](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/dpxETH/DpxEthToken.sol#L26)

## Tools Used

Manual Review

## Recommended Mitigation Steps

Do not grant unnecessary roles to the deployer.

# Unnecessary use of `_isEligibleSender`

## Links to affected code

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1085-L1086](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1085-L1086)

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1054-L1055](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1054-L1055)

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L320-L324](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L320-L324)

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L372-L374](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L372-L374)

## Impact

## Proof of Concept

The following functions are functions that can only be called for a specific `msg.sender` by onlyRole. So there is no need to check them twice with `_isEligibleSender`.

### RdpxV2Core

```solidity
function lowerDepeg(
  uint256 _rdpxAmount,
  uint256 _wethAmount,
  uint256 minamountOfWeth,
  uint256 minOut
) external onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256 dpxEthReceived) {
  _isEligibleSender();
  ...
}

function upperDepeg(
  uint256 _amount,
  uint256 minOut
) external onlyRole(DEFAULT_ADMIN_ROLE) returns (uint256 wethReceived) {
  _isEligibleSender();
  ...
}
```

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1085-L1086](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1085-L1086)

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1054-L1055](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1054-L1055)

### **PerpetualAtlanticVault**

```solidity
function settle(
  uint256[] memory optionIds
)
  external
  nonReentrant
  onlyRole(RDPXV2CORE_ROLE)
  returns (uint256 ethAmount, uint256 rdpxAmount)
{
  _whenNotPaused();
  _isEligibleSender();
  ...
}

function payFunding() external onlyRole(RDPXV2CORE_ROLE) returns (uint256) {
  _whenNotPaused();
  _isEligibleSender();
 ...
}
```

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L320-L324](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L320-L324)

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L372-L374](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L372-L374)

## Tools Used

Manual Review

## Recommended Mitigation Steps

Remove `_isEligibleSender` call.

# Remove useless role `MANAGER_ROLE`

## Links to affected code

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L45](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L45)

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L127](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L127)

## Impact

## Proof of Concept

At PerpetualAtlanticVault contract, `MANAGER_ROLE` is defined and initialized at constructor. But There is no usage of `MANAGER_ROLE` at all.

```solidity
bytes32 public constant MANAGER_ROLE = keccak256("MANAGER_ROLE");

constructor(
  string memory _name,
  string memory _symbol,
  address _collateralToken,
  uint256 _gensis
) ERC721(_name, _symbol) {
  ...
  _setupRole(MANAGER_ROLE, msg.sender);
}
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

Remove `MANAGER_ROLE` constant and remove initializing code at constructor 

# Approve for PerpetualAtlanticVaultLP is not needed at PerpetualAtlanticVault

## Links to affected code

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L207-L210](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L207-L210)

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L243-L250](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L243-L250)

## Impact

Unnecessarily approve token usage

## Proof of Concept

The `transferFrom` function is called in the PerpetualAtlanticVaultLP contract when the `deposit` function is called.

Since the `PerpetualAtlanticVaultLP.deposit` function is not called in the PerpetualAtlanticVault contract, there is no need to approve token consumption at `setAddresses` function.

Also, there is no reason for the `PerpetualAtlanticVault.setLpAllowance` function to exist.

## Tools Used

Manual Review

## Recommended Mitigation Steps

Remove `safeApprove` for perpetualAtlanticVaultLP at `setAddresses` and remove `setLpAllowance` function

```diff
function setAddresses(
  address _optionPricing,
  address _assetPriceOracle,
  address _volatilityOracle,
  address _feeDistributor,
  address _rdpx,
  address _perpetualAtlanticVaultLP,
  address _rdpxV2Core
) external onlyRole(DEFAULT_ADMIN_ROLE) {
  _validate(_optionPricing != address(0), 1);
  _validate(_assetPriceOracle != address(0), 1);
  _validate(_volatilityOracle != address(0), 1);
  _validate(_feeDistributor != address(0), 1);
  _validate(_rdpx != address(0), 1);
  _validate(_perpetualAtlanticVaultLP != address(0), 1);
  _validate(_rdpxV2Core != address(0), 1);

  addresses = Addresses({
    optionPricing: _optionPricing,
    assetPriceOracle: _assetPriceOracle,
    volatilityOracle: _volatilityOracle,
    feeDistributor: _feeDistributor,
    rdpx: _rdpx,
    perpetualAtlanticVaultLP: _perpetualAtlanticVaultLP,
    rdpxV2Core: _rdpxV2Core
  });
- collateralToken.safeApprove(
-   addresses.perpetualAtlanticVaultLP,
-   type(uint256).max
- );
  emit AddressesSet(addresses);
}

-function setLpAllowance(bool increase) external onlyRole(DEFAULT_ADMIN_ROLE) {
-  increase
-    ? collateralToken.approve(
-      addresses.perpetualAtlanticVaultLP,
-      type(uint256).max
-    )
-    : collateralToken.approve(addresses.perpetualAtlanticVaultLP, 0);
-}
```

# Delete unused variable

## Links to affected code

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L51](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L51)

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L109-L117](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L109-L117)

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L52](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L52)

## Impact

The variable exists but is not used.

## Proof of Concept

### UniV2LiquidityAmo

UniV2LiquidityAmo has `slippageTolerance` variable and setter function. However, there is no function that actually uses the `slippageTolerance` variable.

```solidity
uint256 public slippageTolerance = 5e5; // 0.5%

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

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L109](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L109)

### PerpetualAtlanticVaultLP

PerpetualAtlanticVaultLP has an `underlyingSymbol` variable. This variable is neither used nor initialized.

```solidity
string public underlyingSymbol;
```

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L52](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L52)

## Tools Used

Manual Review

## Recommended Mitigation Steps

Remove unused variable and setter function.

# ERC721 NFTs will not show up properly on marketplaces like Opensea

## Links to affected code

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L32](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L32)

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L12](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L12)

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L19](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L19)

## Impact

ERC721 NFTs(PerpetualAtlanticVault, RdpxV2Bond, RdpxDecayingBonds) are not properly listed on the NFT Marketplace and cannot be managed by logging into the Marketplace page as an administrator

## Proof of Concept

In order to be listed correctly on platforms such as Opensea, the `tokenURI` and `contractURI` functions must be defined or overridden. Additionally, you can set up the Opensea page if you inherit `Ownable`.

The PerpetualAtlanticVault, RdpxV2Bond, and RdpxDecayingBonds contracts are ERC721 contracts, but they will not be properly listed or managed on the NFT marketplace because they do not follow these.

It would be good to return json from `tokenURI` so that the `optionPositions` information of the PerpetualAtlanticVault contract or the bond information of the RdpxV2Bond and RdpxDecayingBonds contracts can be obtained as metadata.

## Tools Used

Manual Review

## Recommended Mitigation Steps

Define or override `tokenURI` and `contractURI` functions and inherit `Ownable`.

# `decreaseAmount` I**ncorrect naming**

## Links to affected code

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L139-L145](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L139-L145)

## Impact

## Proof of Concept

The `decreaseAmount` function seems to perform an operation to subtract the `amount` just by looking at its name, but in reality it works as a simple setter. Function names and actual behavior do not intuitively match.

```solidity
function decreaseAmount(
  uint256 bondId,
  uint256 amount
) public onlyRole(RDPXV2CORE_ROLE) {
  _whenNotPaused();
  bonds[bondId].rdpxAmount = amount;
}
```

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L139-L145](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L139-L145)

`RdpxV2Core._transfer` that calls `decreaseAmount` is also used in a way that the caller calculates and sets it.

```solidity
IRdpxDecayingBonds(addresses.rdpxDecayingBonds).decreaseAmount(
  _bondId,
  amount - _rdpxAmount
);
```

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L643-L646](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L643-L646)

## Tools Used

Manual Review

## Recommended Mitigation Steps

Change the name of `decreaseAmount` to `setAmount`, or change it to receive `amount` to subtract and perform a subtraction operation at `decreaseAmount` function.

# `emergencyWithdraw` not exist at some contracts

## Links to affected code

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol)

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/dpxETH/DpxEthToken.sol](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/dpxETH/DpxEthToken.sol)

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol)

[https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol)

## Impact

Cannot take out tokens at urgent situation.

## Proof of Concept

The `emergencyWithdraw` function is not required, but it seems that Dopex wants to include `emergencyWithdraw` whenever possible, given that many of Dopex's contracts have an `emergencyWithdraw` function.

If there is no `emergencyWithdraw`, you cannot take out if tokens are sent by mistake or when you need to retrieve tokens urgently due to a hacking accident.

The following are contracts without the `emergencyWithdraw` function or similar function in the scope.

- RdpxV2Bond
- DpxEthToken
- PerpetualAtlanticVaultLP
- ReLPContract

## Tools Used

Manual Review

## Recommended Mitigation Steps

Add `emergencyWithdraw` function.