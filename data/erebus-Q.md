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