# System overview
The **rDPXV2** is a system where users can bond using the **bond** function in the **rDPXV2Core** contract.
This **bond** is an NFT which gets minted to the address the user specified when calling the bond function.

* **There are actually three ways in which users can bond** :
        1. *By using their **weth** and **rDPX** tokens*.
        2. *By using their **weth** and **decaying bonds**(only if they have it)*.
               - **decaying bonds** ? : It is an NFT(ERC721) token which has an expiry date and stores a virtual amount of rDPX tokens. This token is minted to users as rebates for losses incurred on Dopex Option Products. 
        3. *By using their **rdpx** and the **weth** delegated by someone else* .

* By delegating their **weth** and specifying how much fees to charge for it when someone uses it* a user also gets minted a bond.
        - while this is also technically one way to mint a bond but is a bit different from the other three ways. In this the the delegatee(the user who delegates their weth) only gets minted a bond when someone uses his **weth** (i.e. the 3rd way of bonding). The fees to charge for is also taken into account in this process. 

**delegate** ? : A user can delegate their **weth** and charge fees for it.
**delegatee** ? : The user who delegates his **weth** .

### A more in depth explaination of what the bond actually is
The **bond** is actually an NFT(ERC721) token which represents some amount of **receiptTokens** which in turn represents dpxETH/ETH on the Curve LP.

* The bond has some values associated with it : 
      1. The amount : This is the amount of **receiptToken** and not necessarily the same for every bond it can be different according to how much **weth** and **rDPX** tokens is used while bonding.
      2. The maturity : This maturity time is the time in which the users should wait before they can redeem their bonds. 
      3. The timestamp: The time in which the bond was minted.
      4. The bondId : The unique id of the bond.
  
* The users can redeem their bonds for the **receiptTokens** and then stake it to earn yields.
* Users have to be aware that the maturity time can highly be manipulated be the admin of the protocol. 
This takes us to the centralization risks section below:

# Centralization risks
This system or protocol is highly dependent on the admin and there are many contracts where the admin could rug pull users. 
Also there are function where the admin could approve any address to spend the tokens held by the contracts. While this is needed in order to approve tokens to the other contracts in the system the approve can be done directly with the required amounts inside function where it is only needed and remove the independent function where the admin could approve any address to spend the tokens. 
OR atleast users should be made known about this risks.

# Recommendations
Since the system is very complex the only recommendation could give is to apply different function in the core contract for bonding:
Instead of using only the **bond** function directly for both users who want to bond with their delegate bonds and the users who want to bond with their **weth** and **rDPX** you could make the **bond** function internal and use two different functions. 




### Time spent:
60 hours