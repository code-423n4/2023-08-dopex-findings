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

       2. The integation amont different contracts are based on roles. it might be better to use one main contract and integrate them together during deployment. Roles can be assigned afterwards and might expose additional vulnerabilities to allow additional contracts/modules to be implanted. This might be dangeous for such open ecosystem.

       


    Codebase quality analysis

        Overall, the code is clean is easy to understand. However, math and data strucutres needs a more careful care.


    Centralization risks
        1) The integration is based on roles, it is better to integrate one time during deployment. Most contracts have their own  DEFAULT_ADMIN_ROLE. 

        2) some ``DEFAULT_ADMIN_ROLE`` has too much power. For example, RdpxDecayingBonds.emergencyWithdraw() will allow the admin to call an arbitrary contract's code via token.safeTransfer(). Therefore, it is possible for a compromised/malicious admin to call a malicious external contract. One mitigation is only allow the emergencyWithdraw whitelisted tokens. 


    Mechanism review
     The mechanism for UniV3LiquidityAmo.recoverERC721() is not working. It can lead to lost of funds. See one of the reports for details.


    Systemic risks
        No aware of any.








### Time spent:
50 hours