## [G-01]  - Reduce Storage Reads
**Location:** 
 https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L94C1-L109C4
**Description:** 
The liquidityInPool function reads from storage to get the Uniswap pool's positions mapping. Consider reducing storage reads by caching frequently accessed data when possible.
**Recommended Change:**

function liquidityInPool(
    address _collateral_address,
    int24 _tickLower,
    int24 _tickUpper,
    uint24 _fee
) public view returns (uint128) {
    // Cache the pool address to avoid redundant storage reads
    address poolAddress = univ3_factory.getPool(address(rdpx), _collateral_address, _fee);
    IUniswapV3Pool get_pool = IUniswapV3Pool(poolAddress);

    // goes into the pool's positions mapping, and grabs this address's liquidity
    (uint128 liquidity, , , , ) = get_pool.positions(
        keccak256(abi.encodePacked(address(this), _tickLower, _tickUpper))
    );
    return liquidity;
}


## [G-02]  - Avoid Unnecessary Data Copies
**Location:** 
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L119C1-L133C4
**Description:**
 In the collectFees function, you can avoid creating a memory copy of current_position by directly accessing the positions_array storage. This can reduce gas usage.
**Recommended Change:**


function collectFees() external onlyRole(DEFAULT_ADMIN_ROLE) {
    for (uint i = 0; i < positions_array.length; i++) {
        Position storage current_position = positions_array[i];
        INonfungiblePositionManager.CollectParams
            memory collect_params = INonfungiblePositionManager.CollectParams(
                current_position.token_id,
                rdpxV2Core,
                type(uint128).max,
                type(uint128).max
            );

        // Send to custodian address
        univ3_positions.collect(collect_params);
    }
}


## [G-03]  - Efficient Looping
**Location:** 
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L119C1-L133C4
**Description:**
 When looping through the positions in the collectFees function, you can use a storage pointer to avoid copying the entire positions_array to memory, which can save gas.
**Recommended Change:**


function collectFees() external onlyRole(DEFAULT_ADMIN_ROLE) {
    Position[] storage positions = positions_array;
    for (uint i = 0; i < positions.length; i++) {
        Position storage current_position = positions[i];
        INonfungiblePositionManager.CollectParams
            memory collect_params = INonfungiblePositionManager.CollectParams(
                current_position.token_id,
                rdpxV2Core,
                type(uint128).max,
                type(uint128).max
            );

        // Send to custodian address
        univ3_positions.collect(collect_params);
    }
}


## [G-04]  - Efficient Token Transfers
**Location:** 
_sendTokensToRdpxV2Core function
**Description:** 
When sending tokens to rdpxV2Core, consider batching token transfers if there are multiple tokens to send, reducing the number of external contract calls and saving gas.
**Recommended Change :**


function _sendTokensToRdpxV2Core(address[] memory tokens) internal {
    for (uint256 i = 0; i < tokens.length; i++) {
        address token = tokens[i];
        uint256 tokenBalance = IERC20WithBurn(token).balanceOf(address(this));
        if (tokenBalance > 0) {
            IERC20WithBurn(token).safeTransfer(rdpxV2Core, tokenBalance);
        }
    }
    // Sync token balances once after all transfers
    IRdpxV2Core(rdpxV2Core).sync();
},

## [G-05]  - Avoid Redundant Role Setup
**Location:** https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L24C1-L27C4
**Description:** In the constructor, the DEFAULT_ADMIN_ROLE and MINTER_ROLE roles are both assigned to msg.sender. However, DEFAULT_ADMIN_ROLE is already assigned by OpenZeppelin's AccessControl contract. Avoiding redundant role assignments can save gas.
**code snippet:**


constructor() ERC721("rDPX V2 Bond", "rDPXV2Bond") {
    _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
    _setupRole(MINTER_ROLE, msg.sender);
}

**Recommended Change :**


constructor() ERC721("rDPX V2 Bond", "rDPXV2Bond") {
    _setupRole(MINTER_ROLE, msg.sender);
}
By removing the _setupRole(DEFAULT_ADMIN_ROLE, msg.sender); line,  eliminate the redundant role assignment.

## [G-06]  - Avoid Batch Transfer Check
**Location:**https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L45C1-L53C4
**Description:**The _beforeTokenTransfer function includes a batchSize parameter that is not used in the function. we can remove the batchSize parameter to save gas.

 **Code snippet:**


function _beforeTokenTransfer(
    address from,
    address to,
    uint256 tokenId,
    uint256 batchSize
) internal override(ERC721, ERC721Enumerable) {
    _whenNotPaused();
    super._beforeTokenTransfer(from, to, tokenId, batchSize);
}

**Recommended Change :**


function _beforeTokenTransfer(
    address from,
    address to,
    uint256 tokenId
) internal override(ERC721, ERC721Enumerable) {
    _whenNotPaused();
    super._beforeTokenTransfer(from, to, tokenId);
}
By removing the batchSize parameter and its usage,  simplify the function and save gas.

## [G-07] - Revert Instead of Pause Checks
**Location:**https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L45C1-L53C4
**Description:** In the _beforeTokenTransfer function, you are using _whenNotPaused to check if the contract is paused. Instead of pausing the contract, you can directly revert the transfer with a custom error message to save gas.

**Code snippet :**


function _beforeTokenTransfer(
    address from,
    address to,
    uint256 tokenId
) internal override(ERC721, ERC721Enumerable) {
    _whenNotPaused();
    super._beforeTokenTransfer(from, to, tokenId);
}

**Recommended Change :**


function _beforeTokenTransfer(
    address from,
    address to,
    uint256 tokenId
) internal override(ERC721, ERC721Enumerable) {
    require(!paused(), "Transfers are paused");
    super._beforeTokenTransfer(from, to, tokenId);
}
By directly reverting the transfer with a custom error message,  eliminate the need for the _whenNotPaused function and save gas.

## [G-08]  - Avoid Redundant Role Assignment
**Location:** https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L56C1-L64C4
**Description:** In the constructor, both DEFAULT_ADMIN_ROLE and MINTER_ROLE roles are assigned to msg.sender. However, DEFAULT_ADMIN_ROLE is already assigned by OpenZeppelin's AccessControl contract. Avoiding redundant role assignments can save gas.

**Code snippet:**


constructor(
    string memory _name,
    string memory _symbol
) ERC721(_name, _symbol) {
    // Grant the minter role and admin role to deployer
    _setupRole(MINTER_ROLE, msg.sender);
    _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
    _tokenIdCounter.increment();
}

**Recommended change :**


constructor(
    string memory _name,
    string memory _symbol
) ERC721(_name, _symbol) {
    // Grant the minter role to deployer
    _setupRole(MINTER_ROLE, msg.sender);
    _tokenIdCounter.increment();
}
By removing the _setupRole(DEFAULT_ADMIN_ROLE, msg.sender); line,  eliminate the redundant role assignment.







## [G-09]  - Efficient Emergency Withdrawal Loop
**Location:** https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reserve/RdpxReserve.sol#L74C1-L78C4
**Description:** In the emergencyWithdraw function, when transferring multiple ERC20 tokens, you can use a single for loop to iterate through the token addresses and transfer the balances. This can simplify the code and save gas.

**Code SNIPPET :**


for (uint256 i = 0; i < tokens.length; i++) {
    token = IERC20WithBurn(tokens[i]);
    token.safeTransfer(msg.sender, token.balanceOf(address(this)));
}

**Recommended Change:**


for (uint256 i = 0; i < tokens.length; i++) {
    IERC20WithBurn(tokens[i]).safeTransfer(msg.sender, token.balanceOf(address(this)));
}
By eliminating the need for the token variable, simplify the code and make it more gas-efficient.

## [G-10]  - Avoid Redundant Role Assignment
**Location:** Constructor
**Description:** In the constructor, DEFAULT_ADMIN_ROLE, PAUSER_ROLE, and MINTER_ROLE roles are all assigned to msg.sender. However, DEFAULT_ADMIN_ROLE is already assigned by OpenZeppelin's AccessControl contract. Avoiding redundant role assignments can save gas.

**Code snippet:**


constructor() ERC20("Dopex Synthetic ETH", "dpxETH") {
    _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
    _grantRole(PAUSER_ROLE, msg.sender);
    _grantRole(MINTER_ROLE, msg.sender);
}

**Recommended Change:**


constructor() ERC20("Dopex Synthetic ETH", "dpxETH") {
    _grantRole(PAUSER_ROLE, msg.sender);
    _grantRole(MINTER_ROLE, msg.sender);
}
By removing the _grantRole(DEFAULT_ADMIN_ROLE, msg.sender); line,  eliminate the redundant role assignment.

## [G-11] - Unnecessary Super Call in _beforeTokenTransfer

**Location:** https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L45C1-L53C4
*8Description:** The _beforeTokenTransfer function calls super._beforeTokenTransfer after _whenNotPaused, which is redundant since _whenNotPaused already calls super._beforeTokenTransfer. This call to the parent function can be removed to save gas.
**Recommendation:** Remove the call to super._beforeTokenTransfer in the _beforeTokenTransfer function.

function _beforeTokenTransfer(
    address from,
    address to,
    uint256 tokenId,
    uint256 batchSize
) internal override(ERC721, ERC721Enumerable) {
    _whenNotPaused();
    // Remove the following line as it's redundant:
    // super._beforeTokenTransfer(from, to, tokenId, batchSize);
}


  
## [G-12] Use Storage Arrays Instead of Mappings for Sequential Data
**Location:** Several places
**Description:** Mappings are efficient for random access, but they can be gas-costly when you need to iterate over all elements. Consider using storage arrays to store data like fundingPaymentsAccountedFor, latestFundingPerStrike, and other similar data that is iterated over frequently.


**Code snippet:**


mapping(uint256 => uint256) public fundingPaymentsAccountedFor;
mapping(uint256 => uint256) public latestFundingPerStrike;

**Recommended change:**


uint256[] public fundingPaymentsAccountedFor;
uint256[] public latestFundingPerStrike;
By using arrays instead of mappings, we can iterate over the data more efficiently when needed.

## [G-13] Avoid Repeated Type Conversion
**Location:** Several places in the contract
**Description:**  often convert between different numeric types (e.g., uint256 to int256, or uint256 to uint8) multiple times. These type conversions can add up gas costs. Consider avoiding repeated type conversions by using consistent types wherever possible.

**Code snippet:**


uint256 premium = calculatePremium(
  strike,
  amount,
  timeToExpiry,
  getUnderlyingPrice()
);

**Recommended Change:**


int256 strike = int256(optionPositions[optionIds[i]].strike);
int256 amount = int256(optionPositions[optionIds[i]].amount);

By using consistent types, we can avoid the need for repeated type conversions.

## [G-14] Minimize External Contract Calls
**Location:**updateFunding() and other functions that make external contract calls
**Description:** Minimize external contract calls whenever possible, as they can consume a significant amount of gas. Consider consolidating multiple calls into one to reduce gas costs.

**Code snippet:**


collateralToken.safeTransfer(
  addresses.perpetualAtlanticVaultLP,
  (currentFundingRate * (block.timestamp - startTime)) / 1e18
);

**Recommended Change:**


uint256 transferAmount = (currentFundingRate * (block.timestamp - startTime)) / 1e18;
collateralToken.safeTransfer(addresses.perpetualAtlanticVaultLP, transferAmount);

By consolidating the calculation and the transfer into one step, we can reduce the gas cost associated with external contract calls.

## [G-15] Reduce Repeated Storage Access
**Location:** Several places in the contract
**Description:** In some functions,  access the same storage variable multiple times. This can lead to higher gas costs. Consider storing the value in a local variable and reusing it when needed.

**Code snippet :**


if (totalActiveOptions == fundingPaymentsAccountedFor[latestFundingPaymentPointer]) {

**Recommended Change:**


uint256 activeOptions = totalActiveOptions;
if (activeOptions == fundingPaymentsAccountedFor[latestFundingPaymentPointer]) {

By storing the value of totalActiveOptions in a local variable, we can reduce gas costs associated with repeated storage access.,

## [G-16] - Optimize ERC20 Transfer
**Location:** Inside the deposit and redeem functions.

**Description:** Using transferFrom and transfer functions for ERC20 token transfers can be gas-expensive, especially if they involve external calls. Consider using the latest OpenZeppelin's SafeERC20 functions for transferring tokens, which are optimized for gas efficiency.

**Code snippet :**


collateral.transferFrom(msg.sender, address(this), assets);
collateral.transfer(receiver, assets);


**Recommended Change:**


collateral.safeTransferFrom(msg.sender, address(this), assets);
collateral.safeTransfer(receiver, assets);




## [G-17] - Gas-Efficient Math Operations
**Location:** Inside the _convertToAssets and convertToShares functions.

**Description:** Gas-efficient math operations are crucial. Use the built-in mul, div, and add operations instead of custom functions when working with integers. Also, be cautious with division, as it can be gas-intensive.

**Code snippet:**


shares.mulDivDown(totalCollateral(), supply)

**Recommended Change:**


assets * totalCollateral() / supply



## [G-18] - Avoid Redundant Checks
**Location: **Inside various functions.

**Description:** Avoid redundant checks or conditions that are already checked by the EVM. For example, checking if supply is zero before performing a division is not needed since the EVM handles division by zero gracefully.

**Code snippet:**


supply == 0 ? assets : assets.mulDivDown(supply, totalVaultCollateral);

**Recommended Change:**


assets * totalVaultCollateral / supply,

## [G-19] - Use SafeMath for Precise Arithmetic
**Location:** Throughout the contract.

**Description:** Use SafeMath for all arithmetic operations involving uint256 to prevent overflow or underflow. SafeMath helps ensure that mathematical operations do not result in unexpected behavior and additional gas costs.

**Code snipet:**


reLPFactor * Math.sqrt(tokenAInfo.tokenAReserve) * 1e2 / Math.sqrt(1e18)

**Recommended Change:**


reLPFactor.mul(Math.sqrt(tokenAInfo.tokenAReserve)).mul(1e2).div(Math.sqrt(1e18))




## [G-20] - Use Structs for Multiple Variables
*8Location:** Inside the reLP function.

**Description:** Instead of declaring multiple variables of the same type, consider using a struct to group related variables together. This can improve code readability and potentially reduce gas usage.

**Code snippet :**


uint256 mintokenAAmount = ...
uint256 mintokenBAmount = ...

**Recommended Change:**


struct MintAmounts {
    uint256 mintokenAAmount;
    uint256 mintokenBAmount;
}

MintAmounts memory mintAmounts = MintAmounts(...);


## [G-21] - Cache Repeated Function Calls
**Location:** Inside the reLP function.

**Description:** If the results of function calls, such as IERC20WithBurn(addresses.tokenA).balanceOf(address(this)), are reused multiple times, it's more efficient to cache the result in a variable to avoid redundant external calls.

**Code snippet:**


IERC20WithBurn(addresses.tokenA).balanceOf(address(this))

**Recommended Change:**


uint256 tokenABalance = IERC20WithBurn(addresses.tokenA).balanceOf(address(this));
// Use tokenABalance as needed

## [G-22] - Avoid Overly Complex Expressions
**Location: **Inside the reLP function.

**Description:** Avoid overly complex expressions in a single line, as they can make the code harder to read and may lead to unintended issues. Break down complex logic into multiple steps for better readability and maintainability.

**Code snippet:**


(((amountB / 2) * tokenAInfo.tokenAPrice) / 1e8) - (((amountB / 2) * tokenAInfo.tokenAPrice) * slippageTolerance) / 1e16

**Recommended Change:**


uint256 amountBHalf = amountB / 2;
uint256 tokenAAmount = (amountBHalf * tokenAInfo.tokenAPrice) / 1e8;
uint256 slippageAdjustedAmount = (tokenAAmount * slippageTolerance) / 1e16;
uint256 mintokenAAmount = tokenAAmount - slippageAdjustedAmount;