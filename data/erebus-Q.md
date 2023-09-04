# Low
## [L-01] Wrong conditional
In [PerpetualAtlanticVaultLP#L95](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L95), the condition shall be an `and/&&` so that we do not let the contract to get deployed with 0-addresses values.

## [L-02] Possible failure in `emergencyWithdraw`
NOTE -> I will work with the generic one, just `Ctrl+f` in VSCode.

The function `emergencyWithdraw` does loop through the tokens inside `tokens`.

```
  function emergencyWithdraw(
    address[] calldata tokens
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    IERC20WithBurn token;TODO whenPaused como en el resro

    for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }

    emit LogEmergencyWithdraw(msg.sender, tokens);
  }
```

However, if one of the transfers does revert (e. g. dust attack with a custom token that reverts upon calling `transfer` or the contract being blacklisted for whatever reason), then the funds will remain there until the next call with the updated `tokens` argument, which in a critical situation could mean losing all of them (thieves front-runs the second call by the admin to `emergencyWithdraw`). Consider using a `try`/`catch` approach so that, even if one transfer does revert, at least some funds will be recovered.

## [L-03] Many setters does not have upper/lower caps
NOTE -> although it seems to be duplicated from [bot report L-12](https://github.com/code-423n4/2023-08-dopex/blob/main/bot-report.md#l12-state-variables-not-capped-at-reasonable-values), it does only show one occurrence. I put the others here for completeness (I did read the bot report after writing this one)

Consider putting limits to critical state variables to avoid, for example, paying extremely high fees due to a human error or just the admin being malicious. Occurrences:

- [setSlippageTolerance](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L109C1-L118C1) -> upper limit
- [setRdpxBurnPercentage](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L180C1-L186C4) -> upper limit
- [setRdpxFeePercentage](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L193C1-L199C4) -> upper limit
- [setBondMaturity](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L228C1-L234C4) -> upper limit
- [setBondDiscount](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L441C1-L448C4) -> upper limit
- [setSlippageTolerance](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L455C1-L462C4) and [here](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L186C1-L194C4) -> upper limit
- [updateFundingDuration](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L237C1-L242C1) -> upper and lower limit
- [setreLpFactor](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L90C1-L100C4) -> upper limit
- [setLiquiditySlippageTolerance](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L171C1-L179C4) -> upper limit

## [L-04] Function broken due to hard-coding swap deadline
NOTE -> this submission should be a medium, though, due to the reasons below and there may be more critical situations I did not thought about, but I guess judges will think I am trying to pump the severity, so I put it here

In [UniV3LiquidityAmo::swap](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L295), the deadline for the function `ExactInputSinglePArams` is hard-coded to 2105300114, which is equivalent to Wed Sep 17 2036 21:35:14 GMT+0000, that is, in 13 years. It's bad because 1) blockchain is immutable and this project does not seem to be upgradeable, so once the current Unix timestamp becomes higher such a function will be broken 2) it gives miners and bots a window of oportunities to perform MEV as there is no "real" deadline, so they can store the tx and execute it once it profits them the most or 3) the tx may become idle and be executed at the wrong time (to name some situations). The same applies to [UniV3LiquidityAmo#190](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L190), which sets the deadline to `type(uint256).max` (LMAO), although it does not suffer from 1). Do not hard-code them, do what I say in [[L-05]]()

## [L-05] It's best practice to receive deadlines directly from the front-end instead of relying on `block.timestamp`, which is modifiable by miners
For setting deadlines, it is better to receive the value from the front-end via an argument instead of relying on `block.timestamp`, which may be manipulated by miners to make profits from the caller tx. Consider receiving deadlines via function argument in [UniV2LiquidityAMO#L231](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L231), [UniV2LiquidityAMO#L279](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L279), [UniV2LiquidityAMO#L342](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L342), [UniV3LiquidityAmo#190](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L190), [UniV3LiquidityAmo#250](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L250) and [UniV3LiquidityAmo#295](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L295), which improves UX as the user (or the front-end) knows exactly the deadline, instead of relying on miners who can set a few seconds in the future the current timestamp, which may not be accurate and may harm short-traders.

# Non-critical
## [NC-01] Unlicensed code
For occurrences, see the first line of the files in scope. Anyone can copy your code and make their own project equal to yours without giving you any credits. That means, if for any chance they have more traction, user adoption, fundraising rounds or even better devs, then they can kick you out of the market even when they copied your code and you wouldn't be able to bring them to court due to the fact that your code is unlicensed (AKA free). Just MIT or license it somehow

## [NC-02] Comment does not adhere to implementation
In [RdpxDecayingBonds::emergencyWithdraw](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L89C1-L107C4), the comment for the argument `to` is `The address to transfer the funds to`, so the funds shall go to such address, but there is a `@notice` at the top which says `Transfers all funds to msg.sender`. However, only ETH is sent to the `to` address and the other assets are sent to the caller (AKA the admin).

```
  function emergencyWithdraw(
    address[] calldata tokens,
    bool transferNative,
    address payable to,
    uint256 amount,
    uint256 gas
  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _whenPaused();
    if (transferNative) {
      (bool success, ) = to.call{ value: amount, gas: gas }(""); <=============== HERE funds sent to `to`
      require(success, "RdpxReserve: transfer failed");
    }
    IERC20WithBurn token;

    for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this))); 
    }                         ^============================================== HERE tokens sent the owner with the role DEFAULT_ADMIN_ROLE
  }
```

Consider either sending the tokens to the `to` as the comment suggests and remove the `@notice` or send everything to the caller and remove the `to` parameter, as the comments are contrary to each other

## [NC-03] Comment does not adhere to implementation
In [RdpxDecayingBonds::decreaseAmount](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L139C1-L145C4), the `@notice` and the name of the function suggests that the new amount shall be less than the current one. However, it does not check that, indeed, is less nor it is different, being misleading and doing unnecessary opcodes respectively. Consider changing the function corpse to 

```
_whenNotPaused();
uint256 temp = bonds[bondId].rdpxAmount; // sload
require(amount < temp, "...");
bonds[bondId].rdpxAmount = amount; // sstore
```

or changing the comment and the function name if your intention for the function is to behave like a setter. 

NOTE -> I will put it as gas too

## [NC-04] RdpxDecayingBonds::_tokenIdCounter starts at 1 instead of 0
In the constructor, the contract does increment ˋ_tokenIdCounterˋ by 1, so the call to ˋ_mintTokenˋ.  will mint the bond with index 2 instead of 1. Consider removing the line 63 in the constructor so that the token id start at 1 when calling  ˋ_mintTokenˋ  for the first time.

```
  constructor(
    string memory _name,
    string memory _symbol
  ) ERC721(_name, _symbol) {
    // Grant the minter role and admin role to deployer
    _setupRole(MINTER_ROLE, msg.sender);
    _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
    _tokenIdCounter.increment(); <============================ HERE
  }
```

## [NC-05] Typo
In [RdpxV2Core#L374](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L374) and [RdpxV2Core#L384](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L384) should be `an AMO` instead of `a AMO`. There is another typo [here](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L50), where it says `tolernce` instead of `tolerance`.

## [NC-06] Consistency between functions
The functions `emergencyWithdraw` are supposed to be called when the contract is paused as seen [here](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L222) and [here](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L164). However, in [UniV2LiquidityAmo](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L142C1-L153C4) it does not nor it is `Pausable`. Consider putting at the top `_whenPaused();` and making the contract `Pausable`, as the others. 

## [NC-07] Lack of checks for duplicated `_assetSymbol`
In [RdpxV2core#addAssetTotokenReserves](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L247C1-L251C6) there is a check for the new asset to be indeed new. However, there is no check for `_assetSymbol`. Because it is being done for `_asset`, consider doing the same for `_assetSymbol`.

## [NC-08] Camel case issues
NOTE -> duplicated from [bot report N-55](https://github.com/code-423n4/2023-08-dopex/blob/main/bot-report.md#n55-function-names-should-use-lowercamelcase), I put it here because this is a different occurrence and I did read the report afterwards

The function `RdpxV2core#addAssetTotokenReserves` shall be `RdpxV2core#addAssetToTokenReserves`, with `...token...` being `...Token...`. The same applies to the event `LogAssetAddedTotokenReserves`