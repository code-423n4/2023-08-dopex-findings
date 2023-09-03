# Low
## [L-01] Wrong conditional
In [PerpetualAtlanticVaultLP#L95](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L95), the condition shall be an `and/&&` so that we do not let the contract to get deployed with 0-addresses values.

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
In [RdpxV2Core#L374](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L374) and [RdpxV2Core#L384](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L384) should be `an AMO` instead of `a AMO`
