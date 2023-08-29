    Any comments for the judge to contextualize your findings
    In this audit, I focuses on math and data structure correctness. 

    1. AddAssetTokenReserves. Does it deal with duplicate symbol well? Does it take care of all data structures correctly?
 
    2. It is important to call updateFunding first before any withdraw and deposit are taken to calculate
       shares/assets correctly.

    3. All amounts are need to be correct in math, such as when RdpxV2Core.withdraw()  is called.

    In essense, RdpxV2core is the entry contract to interact with other contracts. 




    Approach taken in evaluating the codebase

    The following approaches have been takedn in evaluating the codebase: 

     1) The correctness of math and data structure are the key for this audit. 

     2) I have used the Visusal Studio the visualzie the call-graph of the contracts to understand how they interact.

     3) Read all test files and compose my own test files to conduct my own unit test.

     4) create attack flow that might expose the vulneratiblity of the codebase. 



    Architecture recommendations
       1) The delegate machansim does not seem to be necessary. It only complicates the codebase but with no or obvious benefits to the end        users. 

       2. The integation among different contracts are based on roles. it might be better to use one main contract and integrate them together during deployment. Roles can be assigned afterwards and might expose additional vulnerabilities to allow additional contracts/modules to be implanted. This might be dangeous for such open ecosystem.
    
       3. The integration between RdpxV2Core and PerpetualAtlanticVault can be improved. Currently, when RdpxV2Core._purchaseOptions() calls PerpetualAtlanticVault.purchase(amount, to), the recipient ``to`` is actually not used. The minted tokens were sent to addresses.rdpxV2Core instead. The assumption is that ``to``, msg.sender, and addresses.rdpxV2Core will all be equal to each other. This leaves some risk if such assumption is not true one day - for example, a new module with the role of ``RDPXV2CORE_ROLE`` is added 
A better integration would be to eliminate the ``to`` argument, and simply mint tokens for the msg.sender, which in this case, is always RdpxV2Core due to the modifier of the PerpetualAtlanticVault.purchase() function.


    Codebase quality analysis

        Overall, the code is clean is easy to understand. However, math and data strucutres needs a more careful care.


    Centralization risks
        A big concern for the centralization risks that are listed below:

        1) The integration is based on roles, it is better to integrate one time during deployment. Most contracts have their own  DEFAULT_ADMIN_ROLE, which is overly powerful since they can call setAddresses() to change critical contract addresses, which is very risky. 

        2)  RdpxDecayingBonds.emergencyWithdraw() will allow the admin to call an arbitrary contract's code via token.safeTransfer(). Therefore, it is possible for a compromised/malicious admin to call a malicious external contract. One mitigation is only allow the emergencyWithdraw whitelisted tokens. 

        3) The DEFAULT_ADMIN_ROLE can call RdpxV2Core.approveContractToSpend(), which will call token.approve() where ``token`` could be an arbitrary input contract. As a result, if a user of the DEFAULT_ADMIN_ROLE is malicious or compromised, an arbitrary malicious external contract can be invoked.

        4) RdpxV2Core.upperDepeg() allows the user of DEFAULT_ADMIN_ROLE to call and then mint dpxeth to swap for eth for the contract. As a result, if a user of the DEFAULT_ADMIN_ROLE is malicious or compromised, he can steal funds from the contract by 1) call RdpxV2Core.upperDepeg() to mint lots of dpxeth and exchange them for eth, 2) call RdpxV2Core.emergencyWithdraw() to steal the eth funds from the contract.
         

    Mechanism review
     The mechanism for UniV3LiquidityAmo.recoverERC721() is not working. It can lead to lost of funds. See one of the reports for details.


    Systemic risks
        No aware of any.














### Time spent:
50 hours