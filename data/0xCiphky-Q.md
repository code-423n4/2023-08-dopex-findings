## Restriction for zero approvals can lead to a loss of funds if contract gets compromised

## **Summary:**

The protocol ensures a significant degree of independence between its contracts, further strengthened by the provision of methods to pause a contract and execute emergency withdrawals. Nonetheless, there remains a latent vulnerability. Contracts like **`UniV2LiquidityAMO`** and **`RdpxV2Core`** have the function **`approveContractToSpend`** which, under administrative privileges, can set an approval to any non-zero value. In the unfortunate event of a compromised contract, the majority of the funds can be shielded by resetting the approval but not all as we can’t set approvals to zero.

## **Impact:**

- Severity: Low.  The majority (about 99%) of the funds can be shielded by resetting the approval to the minimal value.
- Likelihood: Low. This vulnerability becomes pertinent only if a contract is compromised.

## **Tools Used:**

- Manual analysis

## **Recommendation:**

Allow the following contracts to reset approvals to zero.