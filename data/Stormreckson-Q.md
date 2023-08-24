https://github.com/code-423n4/2023-08-dopex/blob/b174dcd7b68a5372d7b9a97c9dd50895e742689c/contracts/core/RdpxV2Core.sol#L894-L899


In the `bond` function,the `_to` parameter represents the recipient of the bond receipt, it wouldbe rational for `_to` to be `msg.sender`.
Setting `_to` as `msg.sender` would ensure that the bond receipt is sent directly to the caller of the function who is making the bond with the delegates. 

1- Setting `_to` as msg.sender eliminates the need for the caller to provide an additional address parameter, simplifying the function call.

2- By using msg.sender as the recipient, it avoids an additional storage write operation to store the recipient's address, resulting in potential gas savings.

3-  Gas Efficiency: By using msg.sender as the recipient, it avoids an additional storage write operation to store the recipient's address, resulting in potential gas savings.
In order to make the functionality more intuitive and align it with the caller's intent, I propose updating the `bondWithDelegate` function to set the `_to` parameter as `msg.sender`. By doing so, the caller of the function will automatically become the recipient of the bond receipt.

Modified Function:


     function bondWithDelegate(
         uint256[] memory _amounts,
         uint256[] memory _delegateIds,
         uint256 rdpxBondId
      ) public returns (uint256 
     receiptTokenAmount, uint256[]                   
     memory delegateReceiptTokenAmounts)

The change only adjusts the default behavior so that the caller becomes the recipient of the bond receipt. It improves usability and saves users from the need to explicitly provide their own address as the receipt recipient.
