DOPEX CONTEST - 2023/08/21 - QA REPORT

1.- The Solidity Docs recommend following a specific layout order in the solidity files/contracts to improve the readability of the code.

It’s recommended to follow this order layout in the contracts.

https://docs.soliditylang.org/en/v0.8.21/style-guide.html#order-of-layout


2.- The Counters contract from Openzeppelin is deprecated and it’s not recommended to use deprecated contracts.

The contracts RdpxDecayingBonds and PerpetualAtlanticVault are using the Counters contract from Openzeppelin to handle the tokenIdCounter, but this contract has been deprecated by Openzeppelin, for that, it is recommended to stop using the Counters contract and handle the tokenId using internal variables instead of the deprecated contract.

I share some links about the topic for reference:

https://github.com/OpenZeppelin/openzeppelin-contracts/issues/4233

https://github.com/OpenZeppelin/openzeppelin-contracts/commit/2ee1da12c4498fc130b7770905d0b4c4f91f6a9d