# Incomplete Transfer due to Non-Payable Fallback
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L98-L99


Description:
The emergencyWithdraw function, when transferring native currency (like Ether), uses a raw .call method. If the recipient address is a contract without a payable fallback function, the transfer will fail, and the function will revert. This can prevent the emergency withdrawal mechanism from working as intended, especially in scenarios where a timely withdrawal is crucial.


Moreover, if not handled correctly, repeated failed attempts could lead to increased gas costs without successful transfers.


Remediation:


Explicitly Check for Contract Addresses:
Before making the transfer, check if the recipient address is a contract and handle it differently or disallow such transfers.

```
function isContract(address _addr) private view returns (bool) {
    uint32 size;
    assembly {
        size := extcodesize(_addr)
    }
    return (size > 0);
}


function emergencyWithdraw(...) external ... {
    ...
    require(!isContract(to), "Cannot send to contract address");
    ...
}
```
# Use a Safer Transfer Mechanism:
Instead of using a raw .call, consider using a safer method to transfer Ether, such as the OpenZeppelin's Address.sendValue function, which handles the potential pitfalls of transferring Ether.


Provide Clear Documentation:
Clearly document the behavior of the emergencyWithdraw function, highlighting that it might not work when trying to send Ether to contract addresses without a payable fallback.


Fallback Mechanism:
Implement a mechanism where, if the Ether transfer fails, the funds are sent to a predefined safe address or returned back to the contract, ensuring they don't get stuck.


# Lack of check if the to parameter is the zero address in RemoveLiquidity function
.
Impact on User/Protocol Flow: Funds transferred to the zero address are lost forever, leading to irreversible asset loss.
Description of Issue: If transfers to the zero address are permitted, it can lead to unintentional burning of funds.

Accidental Scenario:
An admin, while performing batch transfers, accidentally includes the zero address.
Any value transferred to this address is permanently lost.

Remediation with Code Fixes:
Explanation: Before any transfer operation, validate that the target address is not the zero address. This protects against accidental loss of assets.



https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L189
Addliquidity function 

# Insufficient Validation of Token Amount When Adding Liquidity

Impact on user protocol flow: Without the correct validation, users may accidentally or intentionally add zero liquidity, which can disturb the contract's state, imbalance the liquidity pools, and mislead other users about the actual liquidity status.
Description: Proper checks ensure that only meaningful values are accepted, preventing mistakes and potential exploitation.

Step-by-Step Exploitation:
`Admin accesses the contract management interface.
`Navigates to the "Add Liquidity" function.
`Inputs 0 for amountTokenDesired.
`Executes the transaction.
`Observes unintended behavior or inaccurate liquidity representation.

Result of Exploitation: Creates an opportunity for other users to exploit price slippage, damages trust in the protocol, and can lead to erroneous states.

Remediation with Code Fixes:
Why This Fixes the Issue: By setting up a requirement that the token amount is greater than zero, you enforce the condition that any interaction with the function has a meaningful impact. This way, unintentional or malicious zero-value inputs are rejected upfront.
```
require(amountTokenDesired > 0, "Desired token amount should be greater than zero.");
```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L189

# Insufficient Validation of ETH Amount When Adding Liquidity

Impact on user protocol flow: Entering a zero ETH value can lead to asymmetrical liquidity provision, potentially skewing exchange rates and causing imbalances.

Description: Validating the ETH amount ensures that the liquidity being added corresponds with the intended amounts of both tokens and ETH.

Step-by-Step Exploitation:
`Admin accesses the contract.
`Navigates to "Add Liquidity".
`Provides 0 ETH.
`Executes the transaction.
`Observes asymmetrical liquidity provision.

Result of Exploitation: This could lead to skewed exchange rates, possible arbitrage opportunities, and damage trust.
Remediation with Code Fixes:
Why This Fixes the Issue: By validating that the ETH value is non-zero, you ensure a balanced addition of liquidity. This prevents undesired effects on the liquidity pool's state.
```
require(msg.value > 0, "ETH amount should be greater than zero.");

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L189

# Address Parameter Points to Zero Address

Impact on user protocol flow: Transferring liquidity to the zero address makes those funds irrecoverable, causing direct financial loss.
Description: Address validation ensures that liquidity isn't sent to addresses where it can't be recovered from, ensuring the safety of funds.

Step-by-Step Exploitation:
1 Admin accesses contract.
2 Selects "Add Liquidity".
3 Sets the to parameter as the zero address.
4 Completes the transaction.
5 Liquidity is locked away irreversibly.

Result of Exploitation: Direct financial loss as funds become permanently inaccessible.
Remediation with Code Fixes:
Why This Fixes the Issue: Requiring the recipient address to be non-zero ensures that liquidity is always sent to valid, accessible addresses.
```
require(to != address(0), "'to' address cannot be the zero address.");

```
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L189

# Failure to Validate Transaction Deadline
Impact on user protocol flow: Expired transactions, if executed, can lead to undesired state changes based on outdated market conditions, leading to potential financial discrepancies.
Description: Deadlines ensure that transactions are executed within a relevant time frame, preventing outdated actions from taking place.

Step-by-Step Exploitation:
1 admin accesses contract.
2 Selects "Add Liquidity".
3 Sets a past timestamp for deadline.
4 Waits past the deadline and attempts to execute the transaction.
5 If successful, outdated action disrupts contract state.

Result of Exploitation: Financial discrepancies, trust degradation, and potential arbitrage opportunities based on old market conditions.
Remediation with Code Fixes:
Why This Fixes the Issue: By validating that the deadline timestamp is in the future relative to the current block time, you ensure only timely and relevant transactions are processed.
```
require(deadline > block.timestamp, "Transaction has expired.");
```



#Can be immutable-Gas optimization 

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L29-L41



## UniV3LiquidityAmo
# Gas op - not using libraries
2023-08-dopex/contracts/amo/UniV3LiquidityAmo.sol#13-15
Remove Unused Imports:

If you're not using certain imports, remove them. For instance, TickMath and LiquidityAmounts from Uniswap V3 libraries seem to be imported but not used.


#Remove safeMath
Has compiler version 0.8 and up so it doesn’t need to have this library because of the automatic protection of under/underflow.


#Should be declared as immutable
IUniswapV3Factory public immutable univ3_factory;
INonfungiblePositionManager public immutable univ3_positions;
ISwapRouter public immutable univ3_router;
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L35-L37

  address public immutable rdpx;
  address public immutable  rdpxV2Core;


https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L69-L72

#Allowance Front-running
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L139

Summary: A potential vulnerability has been identified related to the ERC20 race condition in the context of approvals. This vulnerability could be exploited by a malicious spender who might front-run a transaction when an administrator attempts to decrease an allowance. The attacker could rapidly use the initial allowance and subsequently use the newly adjusted allowance once the administrator's transaction is confirmed. This could result in the malicious spender utilizing more funds tha

Proposed Solution: Introduce a bifurcated procedure for modifying allowances:

1. Initially set the allowance to zero.
2. Subsequently, adjust the allowance to the intended value.

Here is the revised function:

```
function modifyAllowance(
    address _recipient,
    address _token,
    uint256 _value
) public onlyRole(DEFAULT_ADMIN_ROLE) {
    uint256 existingAllowance = IERC20WithBurn(_token).allowance(address(this), _recipient);

    if (existingAllowance > 0) {
        IERC20WithBurn(_token).approve(_recipient, 0); // Step 1: Reset to 0
    }
    IERC20WithBurn(_token).approve(_recipient, _value); // Step 2: Set to the intended value
}
```
This function first checks if the existing allowance is greater than zero. If it is, the function resets the allowance to zero. Finally, it sets the allowance to the desired value.


Needs additional checks
Human Error:
Issue: Even if only the admin can call this function, there's always the possibility of human error. The admin might:

Approve a larger amount than intended.
Approve a malicious or incorrect contract.
Forget to first set the allowance to 0 before reducing it, leading to potential front-running.

Solution: Implement safeguards or checks. For instance, the function could include checks against a list of known, safe addresses or have a maximum approval limit. Additionally, always using a pattern that first sets the allowance to 0 before setting a new value can prevent mistakes.



# No Event Emission on rdpxAmount Decrease
Location: decreaseAmount function

Description: The decreaseAmount function doesn't emit an event when the rdpxAmount is decreased, complicating contract state tracking.
Fix:
```
function adminTransfer(address to, uint256 amount) public onlyOwner {
    require(to != address(0), "Cannot transfer to zero address");
    // Remaining logic for the transfer
}
```
