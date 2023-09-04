## `approveContractToSpend` has a subject gas saving
If the owner unlikely  passes in `_token=address(0)`  the interface will not be able to call that function it will revert.
So you can remove that check and save gas 
