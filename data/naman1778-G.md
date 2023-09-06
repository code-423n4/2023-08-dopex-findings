## [G-01] Using calldata instead of memory for read-only arguments in external functions saves gas

When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having memory arguments, it’s still valid for implementation contracs to use calldata arguments instead.

If the array is passed to an internal function which passes the array to another internal function where the array is modified and therefore memory is used in the external call, it’s still more gass-efficient to use calldata when the external function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

Note that I’ve also flagged instances where the function is public but can be marked as external since it’s not called by the contract, and cases where a constructor is involved

```
    File: contracts/core/RdpxV2Core.sol	

    240: function addAssetTotokenReserves(
    241:   address _asset,
    242:   string memory _assetSymbol
    243: ) external onlyRole(DEFAULT_ADMIN_ROLE) {

    270: function removeAssetFromtokenReserves(
    271:   string memory _assetSymbol
    272: ) external onlyRole(DEFAULT_ADMIN_ROLE) {

    764: function settle(
    765:   uint256[] memory optionIds
    766: )
    767:   external
    768:   onlyRole(DEFAULT_ADMIN_ROLE)
    769:   returns (uint256 amountOfWeth, uint256 rdpxAmount)
```

    diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
    index e28a65c..fa29597 100644
    --- a/contracts/core/RdpxV2Core.sol
    +++ b/contracts/core/RdpxV2Core.sol
    @@ -239,7 +239,7 @@ contract RdpxV2Core is
        **/
       function addAssetTotokenReserves(
         address _asset,
    -    string memory _assetSymbol
    +    string calldata _assetSymbol
       ) external onlyRole(DEFAULT_ADMIN_ROLE) {
         require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");

    @@ -268,7 +268,7 @@ contract RdpxV2Core is
        * @dev    Can only be called by admin
        **/
       function removeAssetFromtokenReserves(
    -    string memory _assetSymbol
    +    string calldata _assetSymbol
       ) external onlyRole(DEFAULT_ADMIN_ROLE) {
         uint256 index = reservesIndex[_assetSymbol];
         _validate(index != 0, 18);
    @@ -762,7 +762,7 @@ contract RdpxV2Core is
        * @return rdpxAmount the amount of rdpx sent out
        */
       function settle(
    -    uint256[] memory optionIds
    +    uint256[] calldata optionIds
       )
         external
         onlyRole(DEFAULT_ADMIN_ROLE)

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol

```
    File: contracts/perp-vault/PerpetualAtlanticVault.sol	

    315: function settle(
    316:   uint256[] memory optionIds
    317: )
    318:   external

    405: function calculateFunding(
    406:   uint256[] memory strikes
    407: ) external nonReentrant returns (uint256 fundingAmount) {
```

    diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
    index 88ac2b3..29ca49a 100644
    --- a/contracts/perp-vault/PerpetualAtlanticVault.sol
    +++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
    @@ -313,7 +313,7 @@ contract PerpetualAtlanticVault is

       /// @inheritdoc      IPerpetualAtlanticVault
       function settle(
    -    uint256[] memory optionIds
    +    uint256[] calldata optionIds
       )
         external
         nonReentrant
    @@ -403,7 +403,7 @@ contract PerpetualAtlanticVault is
        * @return fundingAmount the funding of options
        **/
       function calculateFunding(
    -    uint256[] memory strikes
    +    uint256[] calldata strikes
       ) external nonReentrant returns (uint256 fundingAmount) {
         _whenNotPaused();
         _isEligibleSender();

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized("Naman");
            c1.optimized("Naman");
        }
    }
    
    contract Contract0 {
    
         function not_optimized(string memory a) public returns(string memory){
             return a;
         }
    }
    
    contract Contract1 {
    
         function optimized(string calldata a) public returns(string calldata){
             return a;
         }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 100747                                    | 535             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 790             | 790 | 790    | 790 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 66917                                     | 366             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 556             | 556 | 556    | 556 | 1       |

## [G-02] Use assembly to write address storage values

When writing value for variables whose type is address, make use of assembly code instead of solidity code.

```
    File: contracts/amo/UniV3LiquidityAmo.sol	

    77: rdpx = _rdpx;

    78: rdpxV2Core = _rdpxV2Core;
```

    diff --git a/contracts/amo/UniV3LiquidityAmo.sol b/contracts/amo/UniV3LiquidityAmo.sol
    index 9129094..64a5e18 100644
    --- a/contracts/amo/UniV3LiquidityAmo.sol
    +++ b/contracts/amo/UniV3LiquidityAmo.sol
    @@ -74,8 +74,12 @@ contract UniV3LiquidityAMO is AccessControl, ERC721Holder {
       /* ========== CONSTRUCTOR ========== */

       constructor(address _rdpx, address _rdpxV2Core) {
    -    rdpx = _rdpx;
    -    rdpxV2Core = _rdpxV2Core;
    +    assembly{
    +       sstore(rdpx.slot,_rdpx)
    +    }
    +    assembly{
    +       sstore(rdpxV2Core.slot,_rdpxV2Core)
    +    }

         _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol

```
    File: contracts/core/RdpxV2Core.sol	

    126: weth = _weth;
```
    diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
    index e28a65c..1957fa5 100644
    --- a/contracts/core/RdpxV2Core.sol
    +++ b/contracts/core/RdpxV2Core.sol
    @@ -123,7 +123,9 @@ contract RdpxV2Core is
       // ================================ CONSTRUCTOR ================================ //
       constructor(address _weth) {
         _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
    -    weth = _weth;
    +    assembly{
    +       sstore(weth.slot, _weth)
    +    }

         // add Zero asset to reserveAsset
         ReserveAsset memory zeroAsset = ReserveAsset({

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol

```
    File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol	

    99: rdpxRdpxV2Core = _rdpxRdpxV2Core;

    101: rdpx = _rdpx;
```

    diff --git a/contracts/perp-vault/PerpetualAtlanticVaultLP.sol b/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
    index 5758161..5ea652d 100644
    --- a/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
    +++ b/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
    @@ -96,9 +96,15 @@ contract PerpetualAtlanticVaultLP is ERC20, IPerpetualAtlanticVaultLP {
           "ZERO_ADDRESS"
         );
         perpetualAtlanticVault = IPerpetualAtlanticVault(_perpetualAtlanticVault);
    -    rdpxRdpxV2Core = _rdpxRdpxV2Core;
    +    assembly{
    +       sstore(rdpxRdpxV2Core.slot, _rdpxRdpxV2Core)
    +    }
    +
         collateralSymbol = _collateralSymbol;
    -    rdpx = _rdpx;
    +    assembly{
    +       sstore(rdpx.slot, _rdpx)
    +    }
    +
         collateral = ERC20(_collateral);

         symbol = string.concat(_collateralSymbol, "-LP");

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;

        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }

        function testGas() public {
            c0.setOwnerAssembly(0xFD2dabe9DFcc4d88a12A9D0D40D834E81217Cccf);
            c1.setOwner(0xFD2dabe9DFcc4d88a12A9D0D40D834E81217Cccf);
        }
    }

    contract Contract0 {

        address owner;
        function setOwnerAssembly(address _owner) public {
            assembly{
                sstore(owner.slot,_owner)
            }
        }

    }

    contract Contract1 {
        address owner;
        function setOwner(address _owner) public {
            owner = _owner;
        }

    }

#### Gas Report

| Contract0 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 35287                                     | 207             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| setOwnerAssembly                          | 22324           | 22324 | 22324  | 22324 | 1       |


| Contract1 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 48499                                     | 273             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| setOwner                                  | 22363           | 22363 | 22363  | 22363 | 1       |

## [G-03] Using XOR (^) and AND (&) bitwise equivalents

Given 4 variables a, b, c and d represented as such:

    0 0 0 0 0 1 1 0 <- a
    0 1 1 0 0 1 1 0 <- b
    0 0 0 0 0 0 0 0 <- c
    1 1 1 1 1 1 1 1 <- d

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

```
    File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol	

    95: _perpetualAtlanticVault != address(0) || _rdpx != address(0),
```

    diff --git a/contracts/perp-vault/PerpetualAtlanticVaultLP.sol b/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
    index 5758161..ed3897b 100644
    --- a/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
    +++ b/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
    @@ -92,7 +92,7 @@ contract PerpetualAtlanticVaultLP is ERC20, IPerpetualAtlanticVaultLP {
         )
       {
         require(
    -      _perpetualAtlanticVault != address(0) || _rdpx != address(0),
    +      (_perpetualAtlanticVault ^ address(0)) | (_rdpx ^ address(0)),
           "ZERO_ADDRESS"
         );
         perpetualAtlanticVault = IPerpetualAtlanticVault(_perpetualAtlanticVault);

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(5,10,16);
            c1.optimized(5,10,16);
        }
    }
    contract Contract0 {
        address a;
        function not_optimized(uint8 x,uint8 y, uint8 z) public returns(bool){
            if(x!=5 || y!=10 || z!=15){
                return true;
            }
        }
    }
    contract Contract1 {
        address a;
        function optimized(uint8 x,uint8 y, uint8 z) public returns(bool){
            if(((x^5) | (y^10) | (z^15)) != 0){
                return true;
            }
        }
    }
    

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 53705                                     | 300             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 622             | 622 | 622    | 622 | 1       |

| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 49899                                     | 280             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 569             | 569 | 569    | 569 | 1       |

## [G-04] Instead of counting down in *for* statements, count up

There are 23 instances of this issue in 4 files

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

```
    File: contracts/amo/UniV2LiquidityAmo.sol	

    147: for (uint256 i = 0; i < tokens.length; i++) {
```

    diff --git a/contracts/amo/UniV2LiquidityAmo.sol b/contracts/amo/UniV2LiquidityAmo.sol
    index ce92d43..378e9dc 100644
    --- a/contracts/amo/UniV2LiquidityAmo.sol
    +++ b/contracts/amo/UniV2LiquidityAmo.sol
    @@ -144,7 +144,7 @@ contract UniV2LiquidityAMO is AccessControl {
       ) external onlyRole(DEFAULT_ADMIN_ROLE) {
         IERC20WithBurn token;

    -    for (uint256 i = 0; i < tokens.length; i++) {
    +    for (uint256 i = tokens.length - 1; i >= 0 ; i++) {
           token = IERC20WithBurn(tokens[i]);
           token.safeTransfer(msg.sender, token.balanceOf(address(this)));
         }

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol

```
    File: contracts/amo/UniV3LiquidityAmo.sol	

    120: for (uint i = 0; i < positions_array.length; i++) {
```

    diff --git a/contracts/amo/UniV3LiquidityAmo.sol b/contracts/amo/UniV3LiquidityAmo.sol
    index 9129094..46ea9f9 100644
    --- a/contracts/amo/UniV3LiquidityAmo.sol
    +++ b/contracts/amo/UniV3LiquidityAmo.sol
    @@ -117,7 +117,7 @@ contract UniV3LiquidityAMO is AccessControl, ERC721Holder {

       // Iterate through all positions and collect fees accumulated
       function collectFees() external onlyRole(DEFAULT_ADMIN_ROLE) {
    -    for (uint i = 0; i < positions_array.length; i++) {
    +    for (uint i = positions_array.length-1; i >= 0; i++) {
           Position memory current_position = positions_array[i];
           INonfungiblePositionManager.CollectParams
             memory collect_params = INonfungiblePositionManager.CollectParams(

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV3LiquidityAmo.sol

```
    File: contracts/core/RdpxV2Core.sol	

    167: for (uint256 i = 0; i < tokens.length; i++) {

    246: for (uint256 i = 1; i < reserveAsset.length; i++) {

    775: for (uint256 i = 0; i < optionIds.length; i++) {

    836: for (uint256 i = 0; i < _amounts.length; i++) {

    996: for (uint256 i = 1; i < reserveAsset.length; i++) {
```

    diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
    index e28a65c..8a414aa 100644
    --- a/contracts/core/RdpxV2Core.sol
    +++ b/contracts/core/RdpxV2Core.sol
    @@ -164,7 +164,7 @@ contract RdpxV2Core is
         _whenPaused();
         IERC20WithBurn token;

    -    for (uint256 i = 0; i < tokens.length; i++) {
    +    for (uint256 i = tokens.length - 1; i >= 0; i++) {
           token = IERC20WithBurn(tokens[i]);
           token.safeTransfer(msg.sender, token.balanceOf(address(this)));
         }
    @@ -243,7 +243,7 @@ contract RdpxV2Core is
       ) external onlyRole(DEFAULT_ADMIN_ROLE) {
         require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");

    -    for (uint256 i = 1; i < reserveAsset.length; i++) {
    +    for (uint256 i = reserveAsset.length - 1; i >= 1; i++) {
           require(
             reserveAsset[i].tokenAddress != _asset,
             "RdpxV2Core: asset already exists"
    @@ -772,7 +772,7 @@ contract RdpxV2Core is
         (amountOfWeth, rdpxAmount) = IPerpetualAtlanticVault(
           addresses.perpetualAtlanticVault
         ).settle(optionIds);
    -    for (uint256 i = 0; i < optionIds.length; i++) {
    +    for (uint256 i = optionIds.length - 1; i >= 0 ; i++) {
           optionsOwned[optionIds[i]] = false;
         }

    @@ -833,7 +833,7 @@ contract RdpxV2Core is
           _amounts.length
         );

    -    for (uint256 i = 0; i < _amounts.length; i++) {
    +    for (uint256 i = _amounts.length - 1; i >= 0; i++) {
           // Validate amount
           _validate(_amounts[i] > 0, 4);

    @@ -993,7 +993,7 @@ contract RdpxV2Core is
        * @notice Syncs asset reserves with contract balances
        **/
       function sync() external {
    -    for (uint256 i = 1; i < reserveAsset.length; i++) {
    +    for (uint256 i = reserveAsset.length - 1; i >= 1; i++) {
           uint256 balance = IERC20WithBurn(reserveAsset[i].tokenAddress).balanceOf(
             address(this)
           );

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol

```
    File: contracts/decaying-bonds/RdpxDecayingBonds.sol	

    103: for (uint256 i = 0; i < tokens.length; i++) {

    156: for (uint256 i; i < ownerTokenCount; i++) {
```

    diff --git a/contracts/decaying-bonds/RdpxDecayingBonds.sol b/contracts/decaying-bonds/RdpxDecayingBonds.sol
    index e89349e..22e142d 100644
    --- a/contracts/decaying-bonds/RdpxDecayingBonds.sol
    +++ b/contracts/decaying-bonds/RdpxDecayingBonds.sol
    @@ -100,7 +100,7 @@ contract RdpxDecayingBonds is
         }
         IERC20WithBurn token;

    -    for (uint256 i = 0; i < tokens.length; i++) {
    +    for (uint256 i = tokens.length - 1; i >=0; i++) {
           token = IERC20WithBurn(tokens[i]);
           token.safeTransfer(msg.sender, token.balanceOf(address(this)));
         }
    @@ -153,7 +153,7 @@ contract RdpxDecayingBonds is
       ) external view returns (uint256[] memory) {
         uint256 ownerTokenCount = balanceOf(_address);
         uint256[] memory tokenIds = new uint256[](ownerTokenCount);
    -    for (uint256 i; i < ownerTokenCount; i++) {
    +    for (uint256 i = ownerTokenCount - 1; i >= 0; i++) {
           tokenIds[i] = tokenOfOwnerByIndex(_address, i);
         }
         return tokenIds;

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol

```
    File: contracts/perp-vault/PerpetualAtlanticVault.sol	

    225: for (uint256 i = 0; i < tokens.length; i++) {

    328: for (uint256 i = 0; i < optionIds.length; i++) {

    413: for (uint256 i = 0; i < strikes.length; i++) {
```

    diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
    index 88ac2b3..a193bde 100644
    --- a/contracts/perp-vault/PerpetualAtlanticVault.sol
    +++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
    @@ -222,7 +222,7 @@ contract PerpetualAtlanticVault is
         _whenPaused();
         IERC20WithBurn token;

    -    for (uint256 i = 0; i < tokens.length; i++) {
    +    for (uint256 i = tokens.length - 1; i >= 0; i++) {
           token = IERC20WithBurn(tokens[i]);
           token.safeTransfer(msg.sender, token.balanceOf(address(this)));
         }
    @@ -325,7 +325,7 @@ contract PerpetualAtlanticVault is

         updateFunding();

    -    for (uint256 i = 0; i < optionIds.length; i++) {
    +    for (uint256 i = optionIds.length - 1; i >= 0; i++) {
           uint256 strike = optionPositions[optionIds[i]].strike;
           uint256 amount = optionPositions[optionIds[i]].amount;

    @@ -410,7 +410,7 @@ contract PerpetualAtlanticVault is

         updateFundingPaymentPointer();

    -    for (uint256 i = 0; i < strikes.length; i++) {
    +    for (uint256 i = strikes.length - 1; i >= 0; i++) {
           _validate(optionsPerStrike[strikes[i]] > 0, 4);
           _validate(
             latestFundingPerStrike[strikes[i]] != latestFundingPaymentPointer,

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }


    contract Contract0 {
        uint256 num = 3;
        function not_optimized() public {
            uint256 _num = num;
            for(uint i=0;i<=9;i++){
                _num = _num +1;
            }
            num = _num;
        }
    }


    contract Contract1 {
        uint256 num = 3;
        function optimized() public {
            uint256 _num = num;
            for(uint i=9;i>=0;i--){
                _num = _num +1;
            }
            num = _num;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| not_optimized                             | 7040            | 7040 | 7040   | 7040 | 1       |

| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| optimized                                 | 3819            | 3819 | 3819   | 3819 | 1       |

## [G-05] Use assembly to check for address(0)

Checking zero address can be improved by replacing the require statement with Assembly.Solidity has a lot of guardrails that can be removed (with care) for optimization purposes, especially for simple functionality like checking if an address is zero.

```
    File: contracts/amo/UniV2LiquidityAmo.sol	

    83: require(
    84:   _tokenA != address(0) &&
    85:     _tokenB != address(0) &&
    86:     _pair != address(0) &&
    87:     _rdpxV2Core != address(0) &&
    88:     _rdpxOracle != address(0) &&
    89:     _ammFactory != address(0) &&
    90:     _ammRouter != address(0),
    91:   "reLPContract: address cannot be 0"
    92: );

    131: require(_token != address(0), "reLPContract: token cannot be 0");

    132: require(_spender != address(0), "reLPContract: spender cannot be 0");
```

    index ce92d43..5091062 100644
    --- a/contracts/amo/UniV2LiquidityAmo.sol
    +++ b/contracts/amo/UniV2LiquidityAmo.sol
    @@ -80,16 +80,11 @@ contract UniV2LiquidityAMO is AccessControl {
         address _ammFactory,
         address _ammRouter
       ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    -    require(
    -      _tokenA != address(0) &&
    -        _tokenB != address(0) &&
    -        _pair != address(0) &&
    -        _rdpxV2Core != address(0) &&
    -        _rdpxOracle != address(0) &&
    -        _ammFactory != address(0) &&
    -        _ammRouter != address(0),
    -      "reLPContract: address cannot be 0"
    -    );
    +    assembly {
    +      if and(iszero(_tokenA),and(iszero(_tokenB),and(iszero(_pair),and(iszero(_rdpxV2Core),and(iszero(_rdpxOracle),and(iszero(_ammFactory),iszero(_ammRouter))))))) {
    +       revert(0x00, 0x20)
    +      }
    +    }
         addresses = Addresses({
           tokenA: _tokenA,
           tokenB: _tokenB,
    @@ -129,8 +124,16 @@ contract UniV2LiquidityAMO is AccessControl {
         uint256 _amount
       ) external onlyRole(DEFAULT_ADMIN_ROLE) {
         require(_token != address(0), "reLPContract: token cannot be 0");
    -    require(_spender != address(0), "reLPContract: spender cannot be 0");
    -    require(_amount > 0, "reLPContract: amount must be greater than 0");
    +    assembly{
    +        if iszero(_token){
    +            revert(0x00,0x20)
    +        }
    +    }
    +    assembly{
    +        if iszero(_spender){
    +            revert(0x00,0x20)
    +        }
    +    }
         IERC20WithBurn(_token).approve(_spender, _amount);
       }

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol

```
    File: contracts/core/RdpxV2Core.sol	

    244: require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");

    316: _validate(_dopexAMMRouter != address(0), 17);

    317: _validate(_dpxEthCurvePool != address(0), 17);

    318: _validate(_rdpxDecayingBonds != address(0), 17);

    319: _validate(_perpetualAtlanticVault != address(0), 17);

    320: _validate(_perpetualAtlanticVaultLP != address(0), 17);

    321: _validate(_rdpxReserve != address(0), 17);

    322: _validate(_rdpxV2ReceiptToken != address(0), 17);

    323: _validate(_feeDistributor != address(0), 17);

    324: _validate(_reLPContract != address(0), 17);

    325: _validate(_receiptTokenBonds != address(0), 17);

    362: _validate(_rdpxPriceOracle != address(0), 17);

    363: _validate(_dpxEthPriceOracle != address(0), 17);

    379: _validate(_addr != address(0), 17);

    408: _validate(_token != address(0), 17);
 
    409: _validate(_spender != address(0), 17);
```

    diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
    index e28a65c..11267c7 100644
    --- a/contracts/core/RdpxV2Core.sol
    +++ b/contracts/core/RdpxV2Core.sol
    @@ -241,9 +241,13 @@ contract RdpxV2Core is
         address _asset,
         string memory _assetSymbol
       ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    -    require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");
    +    assembly{
    +      if iszero(_asset){
    +        revert(0x00,0x20)
    +      }
    +    }

    @@ -313,16 +317,56 @@ contract RdpxV2Core is
         address _reLPContract,
         address _receiptTokenBonds
       ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    -    _validate(_dopexAMMRouter != address(0), 17);
    -    _validate(_dpxEthCurvePool != address(0), 17);
    -    _validate(_rdpxDecayingBonds != address(0), 17);
    -    _validate(_perpetualAtlanticVault != address(0), 17);
    -    _validate(_perpetualAtlanticVaultLP != address(0), 17);
    -    _validate(_rdpxReserve != address(0), 17);
    -    _validate(_rdpxV2ReceiptToken != address(0), 17);
    -    _validate(_feeDistributor != address(0), 17);
    -    _validate(_reLPContract != address(0), 17);
    -    _validate(_receiptTokenBonds != address(0), 17);
    +    assembly{
    +        if iszero(_dopexAMMRouter){
    +            revert(0x00,0x20)
    +        }
    +    }
    +    assembly{
    +        if iszero(_dpxEthCurvePool){
    +            revert(0x00,0x20)
    +        }
    +    }
    +    assembly{
    +        if iszero(_rdpxDecayingBonds){
    +            revert(0x00,0x20)
    +        }
    +    }
    +    assembly{
    +        if iszero(_perpetualAtlanticVault){
    +            revert(0x00,0x20)
    +        }
    +    }
    +    assembly{
    +        if iszero(_perpetualAtlanticVaultLP){
    +            revert(0x00,0x20)
    +        }
    +    }
    +    assembly{
    +        if iszero(_rdpxReserve){
    +            revert(0x00,0x20)
    +        }
    +    }
    +    assembly{
    +        if iszero(_rdpxV2ReceiptToken){
    +            revert(0x00,0x20)
    +        }
    +    }
    +    assembly{
    +        if iszero(_feeDistributor){
    +            revert(0x00,0x20)
    +        }
    +    }
    +    assembly{
    +        if iszero(_reLPContract){
    +            revert(0x00,0x20)
    +        }
    +    }
    +    assembly{
    +        if iszero(_receiptTokenBonds){
    +            revert(0x00,0x20)
    +        }
    +    }

         addresses = Addresses({
           dopexAMMRouter: _dopexAMMRouter,
    @@ -359,8 +403,16 @@ contract RdpxV2Core is
         address _rdpxPriceOracle,
         address _dpxEthPriceOracle
       ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    -    _validate(_rdpxPriceOracle != address(0), 17);
    -    _validate(_dpxEthPriceOracle != address(0), 17);
    +    assembly{
    +        if iszero(_rdpxPriceOracle){
    +            revert(0x00,0x20)
    +        }
    +    }
    +    assembly{
    +        if iszero(_dpxEthPriceOracle){
    +            revert(0x00,0x20)
    +        }
    +    }

         pricingOracleAddresses = PricingOracleAddresses({
           rdpxPriceOracle: _rdpxPriceOracle,
    @@ -376,7 +428,11 @@ contract RdpxV2Core is
        * @param  _addr the address to add to the AMO address array
        */
       function addAMOAddress(address _addr) external onlyRole(DEFAULT_ADMIN_ROLE) {
    -    _validate(_addr != address(0), 17);
    +    assembly{
    +        if iszero(_addr){
    +            revert(0x00,0x20)
    +        }
    +    }
         amoAddresses.push(_addr);
       }

    @@ -405,8 +461,16 @@ contract RdpxV2Core is
         address _spender,
         uint256 _amount
       ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    -    _validate(_token != address(0), 17);
    -    _validate(_spender != address(0), 17);
    +    assembly{
    +        if iszero(_token){
    +            revert(0x00,0x20)
    +        }
    +    }
    +    assembly{
    +        if iszero(_spender){
    +            revert(0x00,0x20)
    +        }
    +    }
         _validate(_amount > 0, 17);
         IERC20WithBurn(_token).approve(_spender, _amount);
       }

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol

```
    File: contracts/perp-vault/PerpetualAtlanticVault.sol	

    119: _validate(_collateralToken != address(0), 1);

    190: _validate(_optionPricing != address(0), 1);

    191: _validate(_assetPriceOracle != address(0), 1);

    192: _validate(_volatilityOracle != address(0), 1);

    193: _validate(_feeDistributor != address(0), 1);

    194: _validate(_rdpx != address(0), 1);

    195: _validate(_perpetualAtlanticVaultLP != address(0), 1);

    196: _validate(_rdpxV2Core != address(0), 1);
```

    diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
    index 88ac2b3..269359b 100644
    --- a/contracts/perp-vault/PerpetualAtlanticVault.sol
    +++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
    @@ -116,8 +116,11 @@ contract PerpetualAtlanticVault is
         address _collateralToken,
         uint256 _gensis
       ) ERC721(_name, _symbol) {
    -    _validate(_collateralToken != address(0), 1);
    -
    +    assembly{
    +        if iszero(_collateralToken){
    +            revert(0x00,0x20)
    +        }
    +    }
         collateralToken = IERC20WithBurn(_collateralToken);
         underlyingSymbol = collateralToken.symbol();
         collateralPrecision = 10 ** collateralToken.decimals();
    @@ -187,14 +190,41 @@ contract PerpetualAtlanticVault is
         address _perpetualAtlanticVaultLP,
         address _rdpxV2Core
       ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    -    _validate(_optionPricing != address(0), 1);
    -    _validate(_assetPriceOracle != address(0), 1);
    -    _validate(_volatilityOracle != address(0), 1);
    -    _validate(_feeDistributor != address(0), 1);
    -    _validate(_rdpx != address(0), 1);
    -    _validate(_perpetualAtlanticVaultLP != address(0), 1);
    -    _validate(_rdpxV2Core != address(0), 1);
    -
    +    assembly{
    +        if iszero(_optionPricing){
    +            revert(0x00,0x20)
    +        }
    +    }
    +    assembly{
    +        if iszero(_assetPriceOracle){
    +            revert(0x00,0x20)
    +        }
    +    }
    +    assembly{
    +        if iszero(_volatilityOracle){
    +            revert(0x00,0x20)
    +        }
    +    }
    +    assembly{
    +        if iszero(_feeDistributor){
    +            revert(0x00,0x20)
    +        }
    +    }
    +    assembly{
    +        if iszero(_rdpx){
    +            revert(0x00,0x20)
    +        }
    +    }
    +    assembly{
    +        if iszero(_perpetualAtlanticVaultLP){
    +            revert(0x00,0x20)
    +        }
    +    }
    +    assembly{
    +        if iszero(_rdpxV2Core){
    +            revert(0x00,0x20)
    +        }
    +    }
         addresses = Addresses({
           optionPricing: _optionPricing,
           assetPriceOracle: _assetPriceOracle,

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol

```
    File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol	

    95: _perpetualAtlanticVault != address(0) || _rdpx != address(0),
```

    diff --git a/contracts/perp-vault/PerpetualAtlanticVaultLP.sol b/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
    index 5758161..4d9ad65 100644
    --- a/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
    +++ b/contracts/perp-vault/PerpetualAtlanticVaultLP.sol
    @@ -91,10 +91,11 @@ contract PerpetualAtlanticVaultLP is ERC20, IPerpetualAtlanticVaultLP {
           ERC20(_collateral).decimals()
         )
       {
    -    require(
    -      _perpetualAtlanticVault != address(0) || _rdpx != address(0),
    -      "ZERO_ADDRESS"
    -    );
    +    assembly{
    +        if or(iszero(_perpetualAtlanticVault),iszero(_rdpx)){
    +            revert(0x00,0x20)
    +        }
    +    }
         perpetualAtlanticVault = IPerpetualAtlanticVault(_perpetualAtlanticVault);
         rdpxRdpxV2Core = _rdpxRdpxV2Core;
         collateralSymbol = _collateralSymbol;

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol

```
    File: contracts/reLP/ReLPContract.sol	

    127: _tokenA != address(0) &&
    128:   _tokenB != address(0) &&
    129:   _pair != address(0) &&
    130:   _rdpxV2Core != address(0) &&
    131:    _tokenAReserve != address(0) &&
    132:    _amo != address(0) &&
    133:    _rdpxOracle != address(0) &&
    134:    _ammFactory != address(0) &&
    135:    _ammRouter != address(0),
```

    diff --git a/contracts/reLP/ReLPContract.sol b/contracts/reLP/ReLPContract.sol
    index bc57fbd..643c01f 100644
    --- a/contracts/reLP/ReLPContract.sol
    +++ b/contracts/reLP/ReLPContract.sol
    @@ -123,18 +123,11 @@ contract ReLPContract is AccessControl {
         address _ammFactory,
         address _ammRouter
       ) external onlyRole(DEFAULT_ADMIN_ROLE) {
    -    require(
    -      _tokenA != address(0) &&
    -        _tokenB != address(0) &&
    -        _pair != address(0) &&
    -        _rdpxV2Core != address(0) &&
    -        _tokenAReserve != address(0) &&
    -        _amo != address(0) &&
    -        _rdpxOracle != address(0) &&
    -        _ammFactory != address(0) &&
    -        _ammRouter != address(0),
    -      "reLPContract: address cannot be 0"
    -    );
    +    assembly {
    +        if and(iszero(_tokenA),and(iszero(_tokenB),and(iszero(_pair),and(iszero(_rdpxV2Core),and(iszero(_tokenAReserve),and(iszero(_amo),and(iszero(_rdpxOracle),and(iszero(_ammFactory),iszero(_ammRouter)))))) {
    +            revert(0x00, 0x20)
    +        }
    +    }
         addresses = Addresses({
           tokenA: _tokenA,
           tokenB: _tokenB,

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(0x0000000000000000000000000000000000000001);
            c1.optimized(0x0000000000000000000000000000000000000001);
        }
    }
    
    contract Contract0 {
    
         function not_optimized(address addr) public pure{
             if(addr == address(0)){
                revert();
             }
         }
    }
    
    contract Contract1 {
    
         function optimized(address addr) public pure{
             assembly {
               if iszero(addr) {
                   revert(0x00, 0x20)
               }
             }
         }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 41893                                     | 240             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 258             | 258 | 258    | 258 | 1       |

| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 37687                                     | 219             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 252             | 252 | 252    | 252 | 1       |

## [G-06] Cache state variables outside of loop to avoid writing storage on every iteration

```
    File: contracts/perp-vault/PerpetualAtlanticVault.sol	

    493: latestFundingPaymentPointer += 1;
```

    diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
    index 88ac2b3..4bfb603 100644
    --- a/contracts/perp-vault/PerpetualAtlanticVault.sol
    +++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
    @@ -460,6 +460,7 @@ contract PerpetualAtlanticVault is

       /// @dev Helper function that updates the latest funding payment pointer based on current timestamp
       function updateFundingPaymentPointer() public {
    +    uint256 _latestFundingPaymentPointer = latestFundingPaymentPointer;
         while (block.timestamp >= nextFundingPaymentTimestamp()) {
           if (lastUpdateTime < nextFundingPaymentTimestamp()) {
             uint256 currentFundingRate = fundingRates[latestFundingPaymentPointer];
    @@ -490,9 +491,10 @@ contract PerpetualAtlanticVault is
             );
           }

    -      latestFundingPaymentPointer += 1;
    +      _latestFundingPaymentPointer += 1;
           emit FundingPaymentPointerUpdated(latestFundingPaymentPointer);
         }
    +  latestFundingPaymentPointer = _latestFundingPaymentPointer;
       }

       /**

https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol