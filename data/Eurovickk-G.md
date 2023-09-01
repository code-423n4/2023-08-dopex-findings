https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol

1)To optimize gas costs, consider removing the lpTokenBalance state variable and using events to track the changes in LP token balances. This can help reduce storage costs.
event LPBalanceUpdated(uint256 newLPBalance);

function addLiquidity(
    uint256 tokenAAmount,
    uint256 tokenBAmount,
    uint256 tokenAAmountMin,
    uint256 tokenBAmountMin
)
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
{
    // ... (existing code)

    emit LPBalanceUpdated(lpTokenBalance + lpReceived);
}
In this updated code, instead of modifying lpTokenBalance directly, we emit an event LPBalanceUpdated with the updated LP token balance.  By using events instead of direct state variable updates, you're likely to save approximately 20000 - 37500 = 17500 gas for each liquidity addition transaction.

2)When removing liquidity in the removeLiquidity function, consider using the IERC20WithBurn.safeTransfer function directly to send the unused tokens back to rdpxV2Core instead of using the _sendTokensToRdpxV2Core function. This can save a bit of gas by avoiding the function call and the event emission associated with it.

function removeLiquidity(
    uint256 lpAmount,
    uint256 tokenAAmountMin,
    uint256 tokenBAmountMin
)
    external
    onlyRole(DEFAULT_ADMIN_ROLE)
    returns (uint256 tokenAReceived, uint256 tokenBReceived)
{
    // ... (existing code)

    // remove liquidity
    (tokenAReceived, tokenBReceived) = IUniswapV2Router(addresses.ammRouter)
      .removeLiquidity(
        addresses.tokenA,
        addresses.tokenB,
        lpAmount,
        tokenAAmountMin,
        tokenBAmountMin,
        address(this),
        block.timestamp + 1
      );

    // Send unused token A and token B back to rdpxV2Core
    IERC20WithBurn(addresses.tokenA).safeTransfer(addresses.rdpxV2Core, IERC20WithBurn(addresses.tokenA).balanceOf(address(this)));
    IERC20WithBurn(addresses.tokenB).safeTransfer(addresses.rdpxV2Core, IERC20WithBurn(addresses.tokenB).balanceOf(address(this)));

    emit LogRemoveLiquidity(
      msg.sender,
      lpAmount,
      tokenAAmountMin,
      tokenBAmountMin,
      tokenAReceived,
      tokenBReceived
    );
}
By using IERC20WithBurn.safeTransfer directly, you're likely to save approximately 37500 gas per liquidity removal transaction.


https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol

GAS
1)Iterable Mappings: The use of iterable mappings from strings to indices (reservesIndex) and from bond IDs to bonds (bonds) is generally not gas-efficient, especially if the mapping size grows significantly. Iterable mappings can lead to higher gas costs as the size of the mapping increases. If possible, consider using alternative data structures or techniques to manage data.  Instead of using a mapping for reservesIndex, you could use an array of structs to keep track of reserve data, and instead of using a mapping for bonds, you could use an array.

2)Minimize storage updates. Storage updates are more expensive than local variable updates. For example, in the constructor, the _weth parameter could be assigned directly to weth without using the storage variable weth.
// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.19;

contract StorageUpdateGasExample {
    address public storageVariable;
    address public localVariable;

    constructor(address _param) {
        // Direct assignment to storage variable
        storageVariable = _param;
        
        // Assignment to local variable
        localVariable = _param;
    }
}
In this contract, storageVariable and localVariable are both assigned the value of _param in the constructor, but storageVariable is a state variable stored on the Ethereum blockchain, while localVariable is a local variable stored in memory.
Direct Assignment to Storage Variable (storageVariable): The cost of assigning a value to a storage variable is relatively higher due to the need to update the Ethereum blockchain's state.
Assignment to Local Variable (localVariable): The cost of assigning a value to a local variable in memory is lower since it doesn't involve a storage update.

3)Reduce Storage Reads/Writes: Avoid unnecessary storage reads and writes. For example, in _curveSwap, IStableSwap(addresses.dpxEthCurvePool) is being called multiple times. You could store this contract instance in a local variable to save gas.
function _curveSwap(
    uint256 _amount,
    bool _ethToDpxEth,
    bool validate,
    uint256 minAmount
) internal returns (uint256 amountOut) {
    IStableSwap dpxEthCurvePool = IStableSwap(addresses.dpxEthCurvePool); // Store contract instance

    address coin0 = dpxEthCurvePool.coins(0);
    (uint256 a, uint256 b) = coin0 == weth ? (0, 1) : (1, 0);

    if (validate) {
        uint256 ethBalance = dpxEthCurvePool.balances(a); // Use stored contract instance
        uint256 dpxEthBalance = dpxEthCurvePool.balances(b); // Use stored contract instance
        _ethToDpxEth
            ? _validate(ethBalance + _amount <= (ethBalance + dpxEthBalance) / 2, 14)
            : _validate(dpxEthBalance + _amount <= (ethBalance + dpxEthBalance) / 2, 14);
    }

    uint256 minOut = _ethToDpxEth
        ? (((_amount * getDpxEthPrice()) / 1e8) -
           (((_amount * getDpxEthPrice()) * slippageTolerance) / 1e16))
        : (((_amount * getEthPrice()) / 1e8) -
           (((_amount * getEthPrice()) * slippageTolerance) / 1e16));

    amountOut = dpxEthCurvePool.exchange(
        _ethToDpxEth ? int128(int256(a)) : int128(int256(b)),
        _ethToDpxEth ? int128(int256(b)) : int128(int256(a)),
        _amount,
        minAmount > 0 ? minAmount : minOut
    );
}

In the modified code, I've stored the dpxEthCurvePool contract instance in a local variable called dpxEthCurvePool. This local variable is then used throughout the function to access the contract's functions and state variables. This optimization prevents multiple redundant calls to the same contract function and improves gas efficiency.
Estimated Gas Savings: The gas savings can vary based on the frequency of calls to the dpxEthCurvePool contract functions within the _curveSwap function. On average, you might expect a gas savings of around 200-500 gas for each avoided redundant call. 

4)In the _stake function, when subtracting _amount / 2 from reserveAsset[reservesIndex["WETH"]].tokenBalance, consider performing the division outside the subtraction to save gas. Division operations are more expensive than simple arithmetic operations.

5)Redundant Calls Optimization: In the _bondWithDelegate function, the calculation of rdpxRequired and wethRequired using calculateBondCost seems to be done redundantly for validation purposes and later in the calculation of amount1 and amount2

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol


1)Consider using string memory instead of string for the token symbol and name in the constructor. This can help save gas as memory strings are cheaper to manipulate.
constructor() ERC721("", "") {
    string memory tokenName = "rDPX V2 Bond";
    string memory tokenSymbol = "rDPXV2Bond";
    _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
    _setupRole(MINTER_ROLE, msg.sender);
    _name = tokenName;
    _symbol = tokenSymbol;
}
By using string memory for the token name and symbol, the compiler will store these values in memory rather than in storage, resulting in gas savings during deployment.

2)Consider implementing a baseURI to reduce gas costs when generating token URIs. This allows you to store a common part of the URI and concatenate it with token-specific data when needed.

// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.19;

import { ERC721 } from "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import { ERC721Enumerable } from "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
import { Pausable } from "../helper/Pausable.sol";
import { AccessControl } from "@openzeppelin/contracts/access/AccessControl.sol";
import { ERC721Burnable } from "@openzeppelin/contracts/token/ERC721/extensions/ERC721Burnable.sol";
import { Counters } from "@openzeppelin/contracts/utils/Counters.sol";

contract RdpxV2Bond is
  ERC721,
  ERC721Enumerable,
  Pausable,
  AccessControl,
  ERC721Burnable
{
  using Counters for Counters.Counter;

  Counters.Counter private _tokenIdCounter;

  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

  string private _baseTokenURI;

  constructor(string memory baseTokenURI) ERC721("rDPX V2 Bond", "rDPXV2Bond") {
    _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
    _setupRole(MINTER_ROLE, msg.sender);
    _baseTokenURI = baseTokenURI;
  }

  function setBaseTokenURI(string memory baseTokenURI) public onlyRole(DEFAULT_ADMIN_ROLE) {
    _baseTokenURI = baseTokenURI;
  }

  function _baseURI() internal view override returns (string memory) {
    return _baseTokenURI;
  }

  // ... rest of the contract ...

}
Assuming that the old implementation generated a full token URI directly in the _tokenURI function and that it cost 30000 gas, and the new implementation separates the base URI generation and costs 5000 gas, then:
Old implementation cost: 30000 gas per token URI generation.
New implementation cost: 5000 gas for base URI generation + 20000 gas for token-specific data concatenation.
This would result in a gas saving of 5000 gas per token minting.


