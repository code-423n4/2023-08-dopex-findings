  **DOPEX CONTEST - 2023/08/21 - Gas Optimizations.**

1.- To save gas you can cache some math calculations in a variable, so the compiler just does the math once and then you reuse that variable in the different places where it’s needed.

For example in the function: _stake  - RdpxV2Core SC

```
function _stake(address _to, uint256 _amount) internal returns (uint256 receiptTokenAmount) {
Uint256 midAmount = _amount / 2;
        reserveAsset[reservesIndex["WETH"]].tokenBalance -= midAmount;

        IDpxEthToken(reserveAsset[reservesIndex["DPXETH"]].tokenAddress).mint(address(this), midAmount);

        // deposit into the rdpxV2ReceiptToken contract
        receiptTokenAmount = IRdpxV2ReceiptToken(addresses.rdpxV2ReceiptToken).deposit(midAmount);

        // mint receipt token bonds
        _issueBond(_to, receiptTokenAmount);
    }
```
In function: calculateBondCost - RdpxV2Core SC
```
function calculateBondCost(uint256 _amount, uint256 _rdpxBondId)
        public
        view
        returns (uint256 rdpxRequired, uint256 wethRequired)
    {
     
    ** Code omitted *
     
uint256 bondDiscountHalf = bondDiscount / 2;
Uint256 divisor1 = (DEFAULT_PRECISION * rdpxPrice * 1e2);
Uint256 divisor2 = DEFAULT_PRECISION * 1e2;


            rdpxRequired = ((RDPX_RATIO_PERCENTAGE - (bondDiscountHalf)) * _amount * DEFAULT_PRECISION)
                / divisor1;

            wethRequired = ((ETH_RATIO_PERCENTAGE - (bondDiscountHalf)) * _amount) /divisor2;
        } else {
            // if rdpx decaying bonds are being used there is no discount
            rdpxRequired = (RDPX_RATIO_PERCENTAGE * _amount * DEFAULT_PRECISION) / divisor1;

            wethRequired = (ETH_RATIO_PERCENTAGE * _amount) / divisor2;
        }

      ** Code omitted **
    }
```

In function updateFundingPaymentPointer -  PerpetualAtlanticVault
```
function updateFundingPaymentPointer() public {
        while (block.timestamp >= nextFundingPaymentTimestamp()) {
            *
	uint256 amount = (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) / 1e18;
                collateralToken.safeTransfer(
                    addresses.perpetualAtlanticVaultLP, amount
                  
                );

                IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addProceeds( amount);

              *

    }

```
In function updateFunding -  PerpetualAtlanticVault
```
function updateFunding() public {
        ***
	uint256 amount = (currentFundingRate * (block.timestamp - startTime)) / 1e18;

        collateralToken.safeTransfer(
            addresses.perpetualAtlanticVaultLP, amount);

        IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addProceeds(amount);

        emit FundingPaid(
            msg.sender, ((currentFundingRate * (block.timestamp - startTime)) / 1e18), latestFundingPaymentPointer
        );
    }

```
2.- In for loops you can use the following syntaxis to save gas:
```
for (uint256 i = 0; i < _amounts.length; ) {
	*** Do your process ***
// At the end of the for loop
unchecked{++i;}
}
```

3.- in contract RdpxDecayingBonds - mint function,  the require is not necessary due to the function already using the modifier onlyRole.
```
 function mint(address to, uint256 expiry, uint256 rdpxAmount) external onlyRole(MINTER_ROLE) {
       *
        require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter”); // you can delete this
       *
    }
```

4.- The contract DpxEthToken Inherit from ERC20Burnable which is inherited form ERC20 so you don’t need to add ERC20 again to the contract.

Change this:
```
contract DpxEthToken is ERC20, ERC20Burnable, Pausable, AccessControl, IDpxEthToken 
```
To this:
```
contract DpxEthToken is ERC20Burnable, Pausable, AccessControl, IDpxEthToken 
```
The same for the PerpetualAtlanticVault contract the ERC721 is already inherited for ERC721Enumerable, so there’s no need to inherit again.

Change this:
```
contract PerpetualAtlanticVault is
    IPerpetualAtlanticVault,
    ReentrancyGuard,
    Pausable,
    ERC721,
    ERC721Enumerable,
    ERC721Burnable,
    AccessControl,
    ContractWhitelist {}
```
To this:
```
contract PerpetualAtlanticVault is
    IPerpetualAtlanticVault,
    ReentrancyGuard,
    Pausable,
    ERC721Enumerable,
    ERC721Burnable,
    AccessControl,
    ContractWhitelist {}
```