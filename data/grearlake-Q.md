There is no guardian that sum of rdpxBurnPercentage and rdpxFeePercentage is equal to 100%:
RdpxV2Core#setRdpxBurnPercentage and RdpxV2Core#setRdpxFeePercentage are used by admin to set rdpxBurnPercentage and rdpxFeePercentage:

    function setRdpxBurnPercentage(
        uint256 _rdpxBurnPercentage
    ) external onlyRole(DEFAULT_ADMIN_ROLE) {
        _validate(_rdpxBurnPercentage > 0, 3);
        rdpxBurnPercentage = _rdpxBurnPercentage;
        emit LogSetRdpxBurnPercentage(_rdpxBurnPercentage);
    }

    function setRdpxFeePercentage(
        uint256 _rdpxFeePercentage
    ) external onlyRole(DEFAULT_ADMIN_ROLE) {
        _validate(_rdpxFeePercentage > 0, 3);
        rdpxFeePercentage = _rdpxFeePercentage;
        emit LogSetRdpxFeePercentage(_rdpxFeePercentage);
    }

However, It only validate these values is > 0, but not make sure that sum of them is equal to 100%. If sum of them > 100%, RdpxV2Core#_transfer might be fail, and if < 100%, rest of them will be stuck in the contract. 

For mitigration, only set 1 variable, and other one will be calculated based on that variable