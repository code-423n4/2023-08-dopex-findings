## For understanding my finding
I have explained my finding very clearly for better understanding you can look for the regular bonding mechanism provided in the protocol specs which will make it crystal clear to understand the finding.
## Approach
1. Understand the high level mechanism of what the protocol intends to do.
2. Understand the various contracts involved and how are they interconnected.
3. Now line by line analysis of the code.
4. Now for finding high/medium findings look for the important functions where transfer of funds is involved as these are the functions which will be utilized the most.
## Architecture recommendations
Might introduce some function to get the total funding for a given epoch
## Codebase quality Analysis
It is written very well and most of the function except the bug that i reported have been written very well.
## Centralization risks
No centralization risks as such but there might be a issue for the users if the discount factor set by the admin is too low nearly negligible which might reduce users interaction with the protocol, there should a minimum discount factor 
## Mechanism Review 
It follows a proper mechanism
## Systemic Risks
Many functionalities depend on the admin and other roles given so if those addresses are lost or not handled properly it could lead to malfunctioning of the protocol.



### Time spent:
20 hours