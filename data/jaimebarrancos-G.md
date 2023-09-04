Cache \_amounts.length

827 - \_validate(\_amounts.length == \_delegateIds.length, 3);
833  - \_amounts.length
836 - for (uint256 i = 0; i < \_amounts.length; i++) {

Use ++i instead of i++

167 - for (uint256 i = 0; i < tokens.length; i++) {
246 -     for (uint256 i = 1; i < reserveAsset.length; i++) {
775 -     for (uint256 i = 0; i < optionIds.length; i++) {
836 - for (uint256 i = 0; i < \_amounts.length; i++) {
996 - for (uint256 i = 1; i < reserveAsset.length; i++) {

Calculate strike and timeToExpiry only if putOptionsRequired is true instead of always, since it's only necessary if the if statement is true (put calculations inside if statement).

```solidity
1189 - uint256 strike = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
1190 -   .roundUp(rdpxPrice - (rdpxPrice / 4)); // 25% below the current price
1191 - 
1192		- uint256 timeToExpiry = IPerpetualAtlanticVault(
1193 -   addresses.perpetualAtlanticVault
1194 - ).nextFundingPaymentTimestamp() - block.timestamp;
1195 - if (putOptionsRequired) {
1196 -   wethRequired += IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
1197 - 	.calculatePremium(strike, rdpxRequired, timeToExpiry, 0);
1198 - }
```
