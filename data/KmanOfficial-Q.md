| ID    | Title |
| -------- | ------- |
| 01 | Using a constant expiration date far in the future in `swap` can eventually lead to non-functioning method.
| 02 | For setter functions involving setting percentages values for the protocol, add a require statement so that the new value doesn't go above 100% (`100e8`)
| 03 | Emit event for state variable/struct updates
| 04 | Function name `decreaseAmount` isn't semantically correct to purpose
| 05 | `setLPAllowance` is missing NATSPEC
| 06 | `@notice` for `emergencyWithdraw` contradicts `to` param
| 07 | `lowerDepeg` has a spelling mistake within the `@notice`
| 07 | Comment referencing override functions should be positioned correctly.

## [01] Using a constant expiration date far in the future in `swap` can eventually lead to non-functioning method.

The Uniswap V3 Swap function that is deployed in the UniV3LiquidityAMO.sol has a deadline set to `2105300114` which is `Wed Sep 17 2036 21:35:14 GMT`. Despite it being far into the future, if the protocol was continued to be used by end users several users later, the functionality here would be broken due to an expired deadline. Smart contracts are immutable and should be functional regardless of the future date.

Consider adding a param to specify a deadline rather than hardcoding in a set value in the future.

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol#L295

## [02] For setter functions involving setting percentages values for the protocol, add a require statement so that the new value doesn't go above 100% (`100e8`)

State variables in the protocol that represents percentages are stored as a precision of `1e8` where `1% = 1e8` as there is various setter functions involving variables representing percentages. Where percentages end at a maximum of 100%, there should be a require statement to ensure the msg.sender sets the percentage within these bounds.

Consider adding a require statement or a protocol `_validate` call/statement that limits percentage values to `<= 100e8`.

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L180-L186
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L193-L199


## [03] Emit event for state variable/struct updates

Where state variables and structs are being updated, it is good practice to emit state events in order to allow for indexing and reference to various important changes within a protocol. Some functions emit events whilst some don't, adding to where necessary allows for code consistency too.

Consider adding state events to functions where values are set, added or removed.
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L378-L381
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L388-L394

## [04] Function name `decreaseAmount` isn't semantically correct to purpose

Function `decreaseAmount` in RdpxDecayingBonds.sol is utilised by `rdpxV2Core` when transferring rdpx or burning the equivalent in rDPX decaying bonds `_transfer`. The function is called to set the value of the bond to the previous amount  minus the rDPX amount being transferred. However, the function `decreaseAmount` itself is coded in a fashion that allows for the value to be set to anything. Either higher or lower than the current value as it overrides the current value of bond rather than solely subtracting from the state amount of the bond.

Consider changing the function name or change the functionality of the function so that it solely deducts from the currently stored amount value of the bond.

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L139-L145

## [05] setLPAllowance` is missing NATSPEC documentation

There is missing NATSPEC comments for `setLPAllowance`

Consider adding comments to describe the functions functionality and it's params.

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L243-L250

## [06] `@notice` for `emergencyWithdraw` contradicts `to` param

The `emergencyWithdraw` `@notice` in `RdpxDecayingBonds.sol` states that the function transfers all funds to the msg.sender of the function. However, the function specifies the sender to state an `to` address in which any native ETH funds would be sent to this address if `transferNative` is set to true whereas all ERC20 tokens stated in the list would be sent to the msg.sender. This contradicts the `@notice` for this function.

Consider rewording the `@notice` and as well consider sending all ERC20 tokens to the specified `to` address param too or sending the native ETH funds to msg.sender and removing the `to` address param.

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L89-L107

## [07] Spelling Mistakes

To improve code readability, fix the spelling mistake in the `@notice` for the lowerDepeg function - 'ot'  to 'to'
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1072-L1107

To improve code readability, fix the spelling mistake in the @notice for `slippageTolerance` variable - 'tolernce' to 'tolerance'
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L99-L100

## [08] Comment referencing override functions should be positioned correctly.

Consider moving the comment `// The following functions are overrides required by Solidity.` from L172 to above L162 `_beforeTokenTransfer` in order to for this comment to support both `_beforeTokenTransfer` and `supportsInterface`

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L172

