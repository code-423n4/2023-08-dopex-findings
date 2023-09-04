There is no coded poc for my findings but I hope I explained it well.
I first started looking at the core contract and briefly went though all the functions to get a high level understanding of the protocol while also reading the documentations but I have to say documentation lacks proper explanation also there are many confusions because of the documentations like the backing reserves, treasury reserves but the core itself had like backing reserves array.
There are argument names like the bondId in the bond function which took me a long time to understand. It would have been easier if there was a common bond internal function and two different function like bondWithDecayingBond
and bondWithoutDecaying which calls the internal bond function.
There is definitely many centralization risks like the emergencyWithdraw functions. So I hope the users are made aware of this. 
 


### Time spent:
60 hours