1.

`decreaseAmount` in the RdpxDecayingBonds contract is used when admin plan to decreases the bond amount:

```
  function decreaseAmount(
    uint256 bondId,
    uint256 amount
  ) public onlyRole(RDPXV2CORE_ROLE) {
    _whenNotPaused();
    bonds[bondId].rdpxAmount = amount;
  }
```

However there is no check that amount is lower than previous one and it can be set up to a higher amount undesirably. 

Consider providing a check that the amount is lower than previuos one or rename the functions to, for example, `changeAmount()` if it is planned to change the amount at any direction.

2.

Initially RdpxV2Core contract does not have a `MINTER_ROLE` in the RdpxV2Bond contract. 

To purchase a bond user has to call `bond` function that forward calls to internal functions:

bond() => _stake() => _issueBond() => RdpxV2Bond.mint()

```
  function _issueBond(
    address _to,
    uint256 _amount
  ) internal returns (uint256 bondId) {
    bondId = RdpxV2Bond(addresses.receiptTokenBonds).mint(_to); 
    bonds[bondId] = Bond({
      amount: _amount,
      maturity: block.timestamp + bondMaturity,
      timestamp: block.timestamp
    });
  }
```

The functions will revert because, as I said, RdpxV2Core does not have a minter role. 

Consider adding a role to the contract on a time of setting it up, for example, in the `setAddresses()` function that should be called by admin right after the deployment. 

3. 

tx.origin shoud not be relyed for in the modifier in the ContractWhitelist contract.

There is a modifier:

```
  function _isEligibleSender() internal view {
    // the below condition checks whether the caller is a contract or not
    if (msg.sender != tx.origin)
      require(whitelistedContracts[msg.sender], "Contract must be whitelisted");
  }
```

and it is used in some core protocol functions `RdpxV2Core.upperDepeg()` and `RdpxV2Core.lowerDepeg()` to manage a pool of tokens and peg the protocol token to the WETH. 

In that case if malicious user will manage the admin to call a functions in the `hacker` contract it can bring a lot of problems to the protocol. 