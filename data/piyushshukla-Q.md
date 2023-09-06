abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()


Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() 
[instead](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739)

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L104C5-L108C22

    // goes into the pool's positions mapping, and grabs this address's liquidity
    (uint128 liquidity, , , , ) = get_pool.positions(
      keccak256(abi.encodePacked(address(this), _tickLower, _tickUpper))
    );
    return liquidity;

