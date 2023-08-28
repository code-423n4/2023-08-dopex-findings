[G-01] Use assembly to check for address(0)
Checking zero address can be improved by replacing the require statement with Assembly.Solidity has a lot of guardrails that can be removed (with care) for optimization purposes, especially for simple functionality like checking if an address is zero.

There are 44 instances of this issue in 5 files:

File: contracts/amo/UniV2LiquidityAmo.sol

84:  _tokenA != address(0)
85:  _tokenB != address(0)
86:  _pair != address(0)
87:  _rdpxV2Core != address(0)
88:  _rdpxOracle != address(0)
89:  _ammFactory != address(0)
90:  _ammRouter != address(0)
131: require(_token != address(0), "reLPContract: token cannot be 0");
132: require(_spender != address(0), "reLPContract: spender cannot be 0");
133: require(_amount > 0, "reLPContract: amount must be greater than 0");
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol

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
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol

File: contracts/perp-vault/PerpetualAtlanticVault.sol

119: _validate(_collateralToken != address(0), 1);
190: _validate(_optionPricing != address(0), 1);
191: _validate(_assetPriceOracle != address(0), 1);
192: _validate(_volatilityOracle != address(0), 1);
193: _validate(_feeDistributor != address(0), 1);
194: _validate(_rdpx != address(0), 1);
195: _validate(_perpetualAtlanticVaultLP != address(0), 1);
196: _validate(_rdpxV2Core != address(0), 1);
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol

File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol

95: _perpetualAtlanticVault != address(0) || _rdpx != address(0),
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol

File: contracts/reLP/ReLPContract.sol

127: _tokenA != address(0) &&
128: _tokenB != address(0) &&
129: _pair != address(0) &&
130: _rdpxV2Core != address(0) &&
131: _tokenAReserve != address(0) &&
132: _amo != address(0) &&
133: _rdpxOracle != address(0) &&
134: _ammFactory != address(0) &&
135: _ammRouter != address(0),
https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol

Test Code
contract GasTest is DSTest {
    Contract0 c0;
    Contract1 c1;
    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();
    }
    function testGas() public {
        c0.solidity_notZero(0xFD2dabe9DFcc4d88a12A9D0D40D834E81217Cccf);
        c1.assembly_notZero(0xFD2dabe9DFcc4d88a12A9D0D40D834E81217Cccf);
    }
}


contract Contract0 {

    error ZeroAddress();

    function solidity_notZero(address toCheck) public pure returns(bool success) {
        if(toCheck == address(0)) revert ZeroAddress();

        return true;
    }
}

contract Contract1{

error ZeroAddress();
function assembly_notZero(address toCheck) public pure returns(bool success) {
    assembly {
        if iszero(toCheck) {
            let ptr := mload(0x40)
            mstore(ptr, 0xd92e233d00000000000000000000000000000000000000000000000000000000) // selector for `ZeroAddress()`
            revert(ptr, 0x4)
        }
    }
    return true;
}
}
Gas Report
src/test/GasTest.t.sol:Contract0 contract					
Deployment Cost	Deployment Size				
55905	311				
Function Name	min	avg	median	max	# calls
solidity_notZero	323	323	323	323	1
src/test/GasTest.t.sol:Contract1 contract					
Deployment Cost	Deployment Size				
50099	281				
Function Name	min	avg	median	max	# calls
assembly_notZero	317	317	317	317	1