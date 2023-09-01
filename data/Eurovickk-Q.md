https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol

1)For the sake of transparency add events to provide better transparency and allow external monitoring of contract actions.

contract UniV2LiquidityAMO is AccessControl {
    // ... (existing code)

    // Event emitted when contract addresses are set
    event AddressesSet(
        address indexed tokenA,
        address indexed tokenB,
        address indexed pair,
        address rdpxV2Core,
        address rdpxOracle,
        address ammFactory,
        address ammRouter
    );

    // Event emitted when slippage tolerance is updated
    event SlippageToleranceUpdated(uint256 newSlippageTolerance);

    // Event emitted when a contract is approved to spend tokens
    event ContractApproved(address indexed token, address indexed spender, uint256 amount);

    // Event emitted when emergency withdrawal occurs
    event EmergencyWithdrawal(address indexed admin, address[] tokens);

    // Event emitted when tokens are transferred to rdpxV2Core
    event TokensTransferredToRdpxV2Core(address indexed sender, uint256 tokenABalance, uint256 tokenBBalance);

    // Event emitted when liquidity is added
    event LiquidityAdded(
        address indexed admin,
        uint256 tokenAAmount,
        uint256 tokenBAmount,
        uint256 tokenAAmountMin,
        uint256 tokenBAmountMin,
        uint256 tokenAUsed,
        uint256 tokenBUsed,
        uint256 lpReceived
    );

    constructor() {
        // ... (existing code)
    }

    // ... (existing code)

    function setAddresses(
        address _tokenA,
        address _tokenB,
        address _pair,
        address _rdpxV2Core,
        address _rdpxOracle,
        address _ammFactory,
        address _ammRouter
    ) external onlyRole(DEFAULT_ADMIN_ROLE) {
        // ... (existing code)

        emit AddressesSet(_tokenA, _tokenB, _pair, _rdpxV2Core, _rdpxOracle, _ammFactory, _ammRouter);
    }

    function setSlippageTolerance(uint256 _slippageTolerance) external onlyRole(DEFAULT_ADMIN_ROLE) {
        // ... (existing code)

        emit SlippageToleranceUpdated(_slippageTolerance);
    }

    function approveContractToSpend(address _token, address _spender, uint256 _amount) external onlyRole(DEFAULT_ADMIN_ROLE) {
        // ... (existing code)

        emit ContractApproved(_token, _spender, _amount);
    }

    function emergencyWithdraw(address[] calldata tokens) external onlyRole(DEFAULT_ADMIN_ROLE) {
        // ... (existing code)

        emit EmergencyWithdrawal(msg.sender, tokens);
    }

    function _sendTokensToRdpxV2Core() internal {
        // ... (existing code)

        emit TokensTransferredToRdpxV2Core(msg.sender, tokenABalance, tokenBBalance);
    }

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

        emit LiquidityAdded(
            msg.sender,
            tokenAAmount,
            tokenBAmount,
            tokenAAmountMin,
            tokenBAmountMin,
            tokenAUsed,
            tokenBUsed,
            lpReceived
        );
    }

    // ... (existing code)
}
By adding these events to your contract, you're providing a way for external systems and users to listen to and react to important contract actions. 

2)In the swap function, when swapping tokens, there is no check to ensure that the swap resulted in the expected token2AmountOutMin. You might want to add some assertions or error handling here.
.

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol



1)Batch Transfer Logic: In the _beforeTokenTransfer function, you're passing batchSize as a parameter, but it's not clear how this parameter is used within the function. Make sure that the batch transfer logic is correctly implemented.

// ... (other imports and contract definition) ...

contract RdpxV2Bond is
  ERC721,
  ERC721Enumerable,
  Pausable,
  AccessControl,
  ERC721Burnable
{
  // ... (other contract code) ...

  function batchTransfer(
    address[] calldata to,
    uint256[] calldata tokenIds
  ) public {
    require(to.length == tokenIds.length, "Arrays length mismatch");

    uint256 batchSize = to.length;
    for (uint256 i = 0; i < batchSize; i++) {
      _transfer(msg.sender, to[i], tokenIds[i]);
    }
  }

  function _beforeTokenTransfer(
    address from,
    address to,
    uint256 tokenId,
    uint256 batchSize
  ) internal override(ERC721, ERC721Enumerable) {
    _whenNotPaused();
    super._beforeTokenTransfer(from, to, tokenId, batchSize);
  }

  // ... (rest of the contract code) ...
}
In this proof of concept, I've added a new function batchTransfer, which takes two arrays as input: to, which contains the addresses of the recipients, and tokenIds, which contains the token IDs to be transferred. The function iterates through the arrays and transfers each token to its respective recipient using the _transfer function from the ERC721 contract.
The _beforeTokenTransfer function is overridden to call the super._beforeTokenTransfer function with the batchSize parameter. However, keep in mind that the logic within ERC721 and ERC721Enumerable contracts might not expect or handle the batchSize parameter, so you would need to adjust the logic in those contracts accordingly.

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L1


1)Potential Logical Flaw: In the reLP function, the calculation lpToRemove = (tokenAToRemove * totalLpSupply) / tokenAInfo.tokenALpReserve; might lead to division by zero if tokenAInfo.tokenALpReserve is zero. 

function reLP(uint256 _amount) external onlyRole(RDPXV2CORE_ROLE) {
    // ... other code ...

    uint256 lpToRemove = (tokenAToRemove * totalLpSupply) / tokenAInfo.tokenALpReserve;

    // ... other code ...
}
In this code snippet, if tokenAInfo.tokenALpReserve is zero, then the division (tokenAToRemove * totalLpSupply) / tokenAInfo.tokenALpReserve would result in a division by zero error, causing the contract to revert.

Here's a simple proof of concept that demonstrates this scenario:

// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.19;

contract DivisionByZero {
    uint256 public tokenAToRemove = 100;
    uint256 public totalLpSupply = 1000;
    uint256 public tokenALpReserve = 0; // This is the potential issue

    function reLP() external {
        uint256 lpToRemove = (tokenAToRemove * totalLpSupply) / tokenALpReserve;
        // ... other code ...
    }
}

In this example, if you call the reLP function, the division by zero will occur due to tokenALpReserve being zero, causing the transaction to revert.
To prevent this, you should ensure that tokenAInfo.tokenALpReserve is never zero or handle the division by zero scenario gracefully in your contract logic.