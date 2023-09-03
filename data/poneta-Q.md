### Function size is too huge.

Size of function ```rELP``` in ```ReLPContract.sol``` affects the function readability and can introduce errors in case any changes need to be made. https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L202-L307

The same can be seen with the ```purchase``` function in ```PerpetualAtlanticVault.sol```:

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L255-L312

Mitigation: If possible, carefully break the function into smaller functions and add them to the function.

### Confusing variable naming/ incorrect casing.

```mintokenAAmount``` in ```ReLPContract.sol``` should be ```minTokenAAmount``` because the former can be confused with minting operations since the mixedCasing format is not followed as suggested in the solidity documentaion.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L250

Mitigation: Rename mintokenAAmount as minTokenAAmount.

### Events/modifiers/custom errors declared at incorrect locations (recommended code format not followed)

according to the solidity documentation, events/modifiers/custom errors should be declared towards the top of the contract, following the best practices improves readability of the code.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L311

error declared in the middle of the contract:https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L631

modifier declared at the end of the contract:https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L295-L301

error declared at the bottom of the contract:https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1286

Mitigation: Follow the appropriate code recommended for solidity and maintain consistent styling in all the contracts.

### Error message not detailed enough

The error message is not detailed enough to provide information about the error.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L94-L96

Mitigation: make the error messages more descriptive/ use custom error if you intend to keep shorter error messages.

### Unhelpful/inconsistent function specification/ natspec

The natspec is unhelpful because the parameter names are repeated to describe them which do not provide any information about the parameters. Some functions have descriptive natspec while others do not, this inconsistency can affect the readability and understandability of the code.


https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L137-L143

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L490-L493


Mitigation: Provide more descriptive function specification/ natspec to improve readability and understandability of the code.

### Wrong error statement used

the error statement "Not enough available assets to satisfy withdrawal" should be "Not enough available collateral to satisfy withdrawal" because the latter correctly describes the error as opposed to the former and using the former can cause confusion about what the error actually was.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L286-L292

Mitigation: Change the error statement to "Not enough available collateral to satisfy withdrawal".


### Functions that can be converted to modifiers
There are some functions that check for conditions that can be converted into modifiers because they handle the operations that a modiier would generally handle.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L264

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L324

Mitigation: Convert the functions in the above instances into modifiers.



### Missing event parameter
Generally, when transactions are sent, the event emitted also includes the address that the transaction was sent to. However, that is not the case in the following instance: https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L152
Even though the address is trusted, it is suggested to follow the best practices.

Mitigation: Add the appropriate address as a event param.


### Code duplication
the same operations/ calculations carried over and over again can be converted into a function and can be used in all these instances instead od duplicating the code. In this way, if any changes are to be made in the calculations, only the function will have to be changed as opposed to changing every instance individually (which has a high probability of introducing errors).
Instance 1:
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L510-L524
Instance 2:
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L473-L491

### Function allows zero amount
Functions allow zero value to be passed in through the amount parameter, this is because there are no checks to make sure that the amount being sent is above zero.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L598-L615

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L202-L308

Mitigation: Check if the amount passed in is greater than 0 by adding the required checks.

### Any address that will not change after the constructor can be declared as immutable.
addresses that do not change after they get set in the constructor can be declared as immutable.
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L75-L78


### No checks on fee
There are no checks on the fee passed into the function, this can cause it to behave unexpectedly. 

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L94-L114


Mitigation: Consider adding checks on the fee being sent to make sure that it fits between the valid fee range.

### Initialization of slippage fee is higher than what is allowed
The slippage fee is initialized as 0.5 percent, However when functions check the slippage tolerance, it is checked if the slippage tolerance is greater than 0 percent. This can be problematic if the protocol team expects the slippage tolerance to be greater than the initalized tolerance. This can be seen in every contract that deals with slippage tolerance in the scope.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L103

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L100

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L454-L462

Mitigation: If the initialized slippage tolerace should be the minimum, add checks to ensure that the slippage tolerance is greater than that rather than zero.
