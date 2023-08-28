# Finding 1:-

In 'setAddresses' function, the 'addresses' struct can be assigned in a different way to save gas of both deployment and runtime.

Code Line -> https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L93

**Current Implementation:-**

        addresses = Addresses({
            tokenA: _tokenA,
            tokenB: _tokenB,
            pair: _pair,
            rdpxV2Core: _rdpxV2Core,
            rdpxOracle: _rdpxOracle,
            ammFactory: _ammFactory,
            ammRouter: _ammRouter
        });
 
**Suggested implementation:-**

        addresses.tokenA=_tokenA;
        addresses.tokenB=_tokenB;
        addresses.pair=_pair;
        addresses.rdpxV2Core=_rdpxV2Core;
        addresses.rdpxOracle=_rdpxOracle;
        addresses.ammFactory=_ammFactory;
        addresses.ammRouter=_ammRouter;

########################################################################################################

# Finding 2:-

Use assembly helper function to check for address(0). It saves gas.
You can use your own custom error and appropriate selector hex.
In many places of in the code address(0) checks are done. using this assembly helper function can save significant amount of gas.

**Example:-**
	
        error ZeroAddress();
	function assembly_notZero(address toCheck) public pure returns(bool success) {
  	  assembly {
   	     if iszero(toCheck) {
      	      let ptr := mload(0x40)
      	      mstore(ptr, 0xd92e233d00000000000000000000000000000000000000000000000000000000) 
                              // selector for `ZeroAddress()`.                        
      	      revert(ptr, 0x4)
      	  }
 	   }
   	 return true;
	}

########################################################################################################

# Finding 3:-

Instead of if else, ternary operator can be used to save gas. 
Code Line -> https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L313

Current implementation:-

    if (swapTokenAForTokenB) {
      token1 = addresses.tokenA;
      token2 = addresses.tokenB;
    } else {
      token1 = addresses.tokenB;
      token2 = addresses.tokenA;
    }

Suggested implementation:-

        swapTokenAForTokenB
            ? (token1 = addresses.tokenA, token2 = addresses.tokenB)
            : (token1 = addresses.tokenB, token2 = addresses.tokenA);

