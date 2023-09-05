## Removing the last element of ```reserveAsset``` via the ```addAssetTotokenReserves()``` function does not properly update the ```reserveIndex``` mapping.

As mentioned in the #dopex-aug21 chat, further assets will be added to this array. ("BTC", "CRV")
The core contract makes much use of reserveIndes["TOKEN_NAME"] as a reliable way to interact with 
the correct element. Normaly removing an element will set the value of reserveIndex["Said symbol"]
to 0. But removing the last element, does not update this, possibly making the contract think,
the element is still there if substituted with another token.

Example: Say we have ```[ZERO, WETH, RDPX, DPXETH, CRV]```.
The indexs:
```
reserveIndex["RDPX"] will be 2
reserveIndex["CRV"] will be 4.
```
Removing RDPX, will update the reserveIndex["RDPX"] to 0.
Removing CRV. The reserveIndex["CRV"] remains 4.
Furthermore having removed CRV & adding BTC to the array will mess with accounting.
New Array having removed CRV, added BTC: 
```
[ZERO, WETH, RDPX, DPXETH, BTC]
```
If any contracts now read from reserveIndex["CRV"], they will be directed to the BTC block,
possibly transfering or burning the wrong token.

POC: Works as child of ```Setup.t.sol```
```
    function testAddReserve() public {
        // Using Setup.t.sol from sponsor as environment.
        // Adding Crv as the last asset:
        rdpxV2Core.addAssetTotokenReserves(
            address(0x82938347),
            "Crv"
        );
        
        (address crvAddress, uint256 crvBal, string memory crvSymb) = rdpxV2Core.reserveAsset(4);
        
        // Removes rdpx from reserveAssets array:
        rdpxV2Core.removeAssetFromtokenReserves(crvSymb);
        // Adds rdpx back as last element:
        rdpxV2Core.addAssetTotokenReserves(
            address(0xb1b1b1bb1),
            "BTC"
        );
        console.log("Ideally; rdpxV2Core.reserveIndex(crvSymbol) = 0 since it was removed.");

        console.log("State of array after removing Crv:");
        for(uint i; i < 5; i++) {
        (address tokenAddress0, uint256 tokenBalance0, string memory tokenSymbol0) = rdpxV2Core.reserveAsset(i);
        console.log(tokenAddress0, tokenBalance0, tokenSymbol0);
        }

        console.log("ReserveIndex of Crv:", rdpxV2Core.reservesIndex(crvSymb));
        console.log("ReserveIndex of BTC:", rdpxV2Core.reservesIndex("BTC"));
    }
```

##Optimization suggestion regarding ```UniV3LiquidityAmo```, ```UniV2LiquidityAmo```:

On the overall usage of uniswap's functions. The ```deadline``` parameters is always hardcoded. This removes granularity to the smart contract. Since the target chain is Arbitrum, there arent any time griefing concerns or unwanted reverts, but it takes away from the whole point of the parameter. Id suggest leaving this parameter open for future choices, to fully make use of the uniswaps service.

#Possible typo:
In ```UniV3LiquidityAmo``` in the ```removeLiquidity()``` function, there is a pointless event emission.
```
emit log(positions_mapping[pos.token_id].token_id);
```
The event reads from a value, which is bound to be 0, since its mapping value is deleted a few lines prior to this line:
```
delete positions_mapping[pos.token_id];
```

