### G-1 Double checks
There are 2 instances of double checking in the `RdpxDecayingBonds.mint` function. The `hasRole(MINTER_ROLE, msg.sender)` check is the same as the `onlyRole(MINTER_ROLE)` modifier. The `_whenNotPaused` check is the same as the `_beforeTokenTransfer` function, which is called from the `_mint` function.
```solidity
118  ) external onlyRole(MINTER_ROLE) {
119    _whenNotPaused();
120    require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L118-L120

### G-2 Use unchecked calculation
Consider using unchecked calculation at the `PerpetualAtlanticVaultLP.beforeWithdraw` function due the check at the line L288 (`totalAvailableCollateral() = _totalCollateral - _activeCollateral`).
```solidity
  function beforeWithdraw(uint256 assets, uint256 /*shares*/) internal {
    require(
      assets <= totalAvailableCollateral(),
      "Not enough available assets to satisfy withdrawal"
    );
    _totalCollateral -= assets;
  }
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L286-L292

### G-3 State variables should be immutables
There are 4 variables which can be immutables:
```solidity
69  address public rdpx;

72  address public rdpxV2Core;
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L69
```solidity
46  IPerpetualAtlanticVault public perpetualAtlanticVault;

67  address public rdpx;
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L46


### G-4 State variables should be constants
There are 2 variables which can be constants:
```solidity
104  uint256 public roundingPrecision = 1e6;
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L104
```solidity
104  uint256 public liquiditySlippageTolerance = 5e5; // 0.5%
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L103

### G-5 The public variable can be internal
The `delegates` array can be internal because it has a public getters `getDelegatesLength` and `getDelegatePosition` at the `RdpxV2Core` contract.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L121 

### G-6 Redundant multiplications and divisions
There are redundant multiplications and divisions by the `DEFAULT_PRECISION` constant which have no effect at the `RdpxV2Core` contract:
```solidity
1169      rdpxRequired =
1170        ((RDPX_RATIO_PERCENTAGE - (bondDiscount / 2)) *
1171          _amount *
1172          DEFAULT_PRECISION) /
1173       (DEFAULT_PRECISION * rdpxPrice * 1e2);

1180      rdpxRequired =
1181        (RDPX_RATIO_PERCENTAGE * _amount * DEFAULT_PRECISION) /
1182        (DEFAULT_PRECISION * rdpxPrice * 1e2);
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1169-L1173

### G-7 Redundant calculation
The `PerpetualAtlanticVault.calculateFunding` calculates `timeToExpiry` in a for-loop. But for any conditions the result of the calculation will be exactly `fundingDuration`. This could be a potentially wrong calculations, the [documentation](https://dopex.notion.site/rDPX-V2-RI-b45b5b402af54bcab758d62fb7c69cb4#c13cd86d4f054eec8f8d945596482b51) says `The APP contract follows a epoch system where each epoch is 1 week long. ... The funding is calculated based on the amount of options active for a given strike using BlackScholes with the duration as the start of the epoch until the end of the epoch.` So the result is correct. 
Proof of concept: `uint256 timeToExpiry = nextFundingPaymentTimestamp() - (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration)) = genesis + latestFundingPaymentPointer * fundingDuration - genesis - (latestFundingPaymentPointer - 1) * fundingDuration = latestFundingPaymentPointer * fundingDuration - (latestFundingPaymentPointer - 1) * fundingDuration = latestFundingPaymentPointer * fundingDuration - latestFundingPaymentPointer * fundingDuration + 1 * fundingDuration = fundingDuration`.
This issue is in both the QA and GAS categories.
```solidity
426      uint256 timeToExpiry = nextFundingPaymentTimestamp() -
427        (genesis + ((latestFundingPaymentPointer - 1) * fundingDuration));
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L426-L427


### G-8 Storage variables reading optimization 
There are three [functions](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L190-L214) at the `PerpetualAtlanticVaultLP` contract where a stack variable can be used to reduce storage readings and calculations.
```solidity
function foo(uint256 value) public onlyPerpVault {
  uint256 newVariable = storageVariable + value;
  require(
    collateral.balanceOf(address(this)) >= newVariable,
    "Not enough collateral token was sent"
  );
  storageVariable = newVariable;
}
```
The `strike` variable can be initialized before the `_validate` call at the line L414:
```solidity
413    for (uint256 i = 0; i < strikes.length; i++) {
414      _validate(optionsPerStrike[strikes[i]] > 0, 4);
415      _validate(
416        latestFundingPerStrike[strikes[i]] != latestFundingPaymentPointer,
417        5
418      );
419      uint256 strike = strikes[i];
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L413-L419
The `nextFundingPaymentTimestamp()` call result can be cached before the `if` at the line L464 because the new result is possible only after the `latestFundingPaymentPointer` incrementing.
```solidity
462  function updateFundingPaymentPointer() public {
463    while (block.timestamp >= nextFundingPaymentTimestamp()) {
464      if (lastUpdateTime < nextFundingPaymentTimestamp()) {

468          ? (nextFundingPaymentTimestamp() - fundingDuration)

471        lastUpdateTime = nextFundingPaymentTimestamp();

475          (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /

481            (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /

487          ((currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) /

491      }
492
493      latestFundingPaymentPointer += 1;
494      emit FundingPaymentPointerUpdated(latestFundingPaymentPointer);
495    }
496  }
```
The `_amounts.length` value can be cached before the `_validate` call at the line L827 of the `RdpxV2Core` contract:
```solidity
827    _validate(_amounts.length == _delegateIds.length, 3);

832    uint256[] memory delegateReceiptTokenAmounts = new uint256[](
833      _amounts.length
834    );

836    for (uint256 i = 0; i < _amounts.length; i++) {
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L827-L836


### G-9 Redundant call
There is a redundant call at the `RdpxV2Core._transfer` function. The ignored return at the line L631 is exactly `ownerOf(_bondId)`. It can be cached to avoid another calling to the `IRdpxDecayingBonds(addresses.rdpxDecayingBonds)` contract.
```solidity
631      (, uint256 expiry, uint256 amount) = IRdpxDecayingBonds(
632        addresses.rdpxDecayingBonds
633      ).bonds(_bondId);

638        IRdpxDecayingBonds(addresses.rdpxDecayingBonds).ownerOf(_bondId) ==
639          msg.sender,
640        9
641      );
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L631-L641

