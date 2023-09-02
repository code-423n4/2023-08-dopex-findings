### [L] Lack of Batch Handling Functionality for Large Array Operations in settle and calculateFunding Methods

1. Unavailability of Batch Retrieval for Unsettled Options:

The current smart contract design lacks a convenient way for administrators to retrieve an array of unsettled option IDs. Although the contract includes several mappings related to option IDs, such as **optionsOwned**, **optionsPerStrike**, **totalActiveOptions**, and **optionPositions**, none of these functionalities distinguish between settled and unsettled options. 

    /// @dev Option id => owned or not (boolean)
    mapping(uint256 => bool) public optionsOwned;

    /// @dev amount of options per strike
    mapping(uint256 => uint256) public optionsPerStrike;

    /// @dev the total number of active options
    uint256 public totalActiveOptions;

    /// @dev tokenId => OptionPosition
    mapping(uint256 => OptionPosition) public optionPositions;


This becomes increasingly cumbersome when dealing with a large volume of options whose prices are fluctuating. Without a streamlined method to segregate these, manual settlement becomes impractical.

2. Inadequate Data Filtering for calculateFunding Method:

The calculateFunding function in the Vault contract necessitates an array of strikes for funding calculations. The contract contains mappings like **optionsPerStrike** and **latestFundingPerStrike** which are associated with these strikes.

    /// @dev amount of options per strike
    mapping(uint256 => uint256) public optionsPerStrike;

    /// @dev latest funding update per strike
    mapping(uint256 => uint256) public latestFundingPerStrike;


However, there's no provided functionality to easily filter out strikes that would satisfy the condition optionsPerStrike[strikes[i]] > 0, making it challenging to execute this function effectively.

### [L] Lack of configurability for DEFAULT_PRECISION Constant

The DEFAULT_PRECISION variable is set as a constant with a value of 1e8 and cannot be altered. Meanwhile, the Core contract has functionalities that permit the addition of new reserve tokens and corresponding oracle addresses. The existing documentation does not specify which oracles are intended to be used. It's worth noting that not all Chainlink oracles return data with a 1e8 precision.
Consequently, if there's a potential for switching out tokens and oracle addresses in the future, consider implementing a way to adjust the DEFAULT_PRECISION to align with any such changes.

### [L] Incomplete ERC20 Token Approvals

The codebase exhibits gaps in ERC20 token approvals for both transferFrom and swap operations. These missing approvals result in unexecutable code in certain areas. While I've classified this as a low-severity issue, it's worth noting that most contracts in the protocol do contain a function for administrative override of approvals. However, for best practice, these approvals should be set either in the constructor or the setAddress functions.

Code References:

contracts\amo\UniV2LiquidityAmo.sol

    210: IERC20WithBurn(addresses.tokenA).safeTransferFrom(
      addresses.rdpxV2Core,
      address(this),
      tokenAAmount
    );
    IERC20WithBurn(addresses.tokenB).safeTransferFrom(
      addresses.rdpxV2Core,
      address(this),
      tokenBAmount
    );
    321: IERC20WithBurn(token1).safeTransferFrom( 
      addresses.rdpxV2Core,
      address(this),
      token1Amount
    );

contracts\amo\UniV3LiquidityAmo.sol

    158: IERC20WithBurn(params._tokenA).transferFrom( 
      rdpxV2Core,
      address(this),
      params._amount0Desired
    );
    IERC20WithBurn(params._tokenB).transferFrom(
      rdpxV2Core,
      address(this),
      params._amount1Desired
    );
    283: IERC20WithBurn(_tokenA).transferFrom( 
      rdpxV2Core,
      address(this),
      _amountAtoB
    );

contracts\core\RdpxV2Core.sol

The core contract approves only WETH for curveSwap, swaps can be made from rDPX to WETH, so rDPX should be approved too.

    344: IERC20WithBurn(weth).approve(addresses.dpxEthCurvePool, type(uint256).max);

Similar situation with UnsiwapV2 router. Only WETH was approved, but tokenIn in the swap function is rDPX:

    343: IERC20WithBurn(weth).approve(addresses.dopexAMMRouter, type(uint256).max);
    1101: amountOfWethOut = IUniswapV2Router(addresses.dopexAMMRouter) 
        .swapExactTokensForTokens(
          _rdpxAmount,
          minamountOfWeth,
          path,
          address(this),
          block.timestamp + 10
        )[path.length - 1];

contracts\perp-vault\PerpetualAtlanticVault.sol

    352: IERC20WithBurn(addresses.rdpx).safeTransferFrom( 
      addresses.rdpxV2Core,
      addresses.perpetualAtlanticVaultLP,
      rdpxAmount
    );

contracts\reLP\ReLPContract.sol

     243: IERC20WithBurn(addresses.pair).transferFrom( 
      addresses.amo,
      address(this),
      lpToRemove
    );


### [L] Inconsistent Handling of Emergency Withdrawals

The emergencyWithdraw function in UniV2LiquidityAMO doesn't include a whenPaused modifier to check whether the protocol is paused, unlike similar functions in other contracts. This inconsistency can disrupt the protocol's operations during emergencies.

Code References:

    142: function emergencyWithdraw(
    address[] calldata tokens
      ) external onlyRole(DEFAULT_ADMIN_ROLE) { 
     IERC20WithBurn token;

    for (uint256 i = 0; i < tokens.length; i++) {
      token = IERC20WithBurn(tokens[i]);
      token.safeTransfer(msg.sender, token.balanceOf(address(this)));
    }

    emit LogEmergencyWithdraw(msg.sender, tokens);
  }


### [L] Weak Validation in Setting Burn and Fee Percentages

The Core contract allows setting rdpxBurnPercentage and rdpxFeePercentage without checking if their sum equals 100%. Stronger validation should be implemented to prevent potential issues.

Suggested Code Pattern:

    function setRdpxBurnAndFeePercentage(uint256 _rdpxBurnPercentage, uint256 _rdpxFeePercentage) external onlyRole(DEFAULT_ADMIN_ROLE) {
    require(_rdpxBurnPercentage + _rdpxFeePercentage == 100 * DEFAULT_PRECISION, "Sum must be 100%");
    rdpxBurnPercentage = _rdpxBurnPercentage;
    rdpxFeePercentage = _rdpxFeePercentage;
    emit LogSetRdpxBurnAndFeePercentage(_rdpxBurnPercentage, _rdpxFeePercentage);
}


### [L] Role Assignment During Contract Deployment

For contracts that include minting functionality (RdpxV2Bond, RdpxDecayingBonds, DpxEthToken), the MINTER_ROLE is initially set only to the deployer (msg.sender). But in the process of work, these functions are supposed to be called by different contracts, so the minter role should be granted to the appropriate contracts at the very beginning of the protocol’s work to prevent errors.



### [L] Missing whenNotPaused Modifiers for Mint Functions in RdpxV2Bond and DpxEthToken contracts 

Certain contracts lack whenNotPaused modifiers for minting tokens. Given that minting is a sensitive operation, these modifiers should be included for better security.

### [QA] User-Friendly Delegate ID Retrieval

After delegating WETH to the Core contract, users are assigned a delegateID that is not easily retrievable later. Adding a mapping and a function to obtain this ID would improve the user experience.

Suggested Code Pattern:

    mapping(address => uint256[]) public addressTodelegateIDs;

    function getUserIds(address user) public view returns (uint256[] memory) {
    return addressTodelegateIDs[user];
}


### [QA] PnL Calculation Shortcomings

The calculatePnl function could be improved to include both profits and losses and to consider premiums in its calculations.

Suggested Code Pattern:

    function calculatePnl(
        uint256 price,
        uint256 strike,
        uint256 amount,
        uint256 premium
    ) public pure returns (int256) {
        int256 pnl;
        
        // Calculate PnL when strike > price
        if(strike > price) {
            pnl = int256(((strike - price) * amount) / 1e8);
        } else {
            // Calculate losses when strike <= price
            pnl = -int256(premium); // You lose the premium
        }
        
        // Deduct the premium cost from the PnL
        pnl -= int256(premium);
        
        return pnl;
    }


### [QA] User-Centric Withdrawal Function

The current withdraw function does not allow for partial withdrawals, limiting its user-friendliness. The function should be updated to accommodate partial withdrawals.

    function withdraw(
    uint256 delegateId
    ) external returns (uint256 amountWithdrawn) { 
    _whenNotPaused(); //@audit let people withdraw
    _validate(delegateId < delegates.length, 14);
    Delegate storage delegate = delegates[delegateId];
    _validate(delegate.owner == msg.sender, 9);

    amountWithdrawn = delegate.amount - delegate.activeCollateral;
    _validate(amountWithdrawn > 0, 15);
    delegate.amount = delegate.activeCollateral; //@Mid forgot to sub total delegate

    IERC20WithBurn(weth).safeTransfer(msg.sender, amountWithdrawn);

    emit LogDelegateWithdraw(delegateId, amountWithdrawn);
  }

 If a user wants to withdraw part of delegated WETH he can’t do it because withdraw function lets him withdraw only the amount delegated minus the amount of active collateral. 


### [QA] Settle function from the Vault contract doesn’t check if the option is ITM.

In the world of options trading, ITM (In-The-Money) and ATM (At-The-Money) are terms used to describe the relationship between the option's strike price and the current market price of the underlying asset. Here's how they differ:

ITM (In-The-Money):
 
Put Option: A put option is "in-the-money" when the current market price of the underlying asset is lower than the option's strike price.

Market Price < Strike Price

ATM (At-The-Money):

Both Call and Put: An option is considered "at-the-money" when the current market price of the underlying asset is approximately equal to the option's strike price.

Market Price ≈ Strike Price

As we see from the comments: 

    331: // check if strike is ITM 
      _validate(strike >= getUnderlyingPrice(), 7);

So equation should be changed to >

      _validate(strike > getUnderlyingPrice(), 7);


