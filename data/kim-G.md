# State variables can cached and worked on in memory before being used to update the state variables.

reserveAsset[reservesIndex["RDPX"]] has been called in the else part of the _tranfer() function four times.
it is more gas efficient to cache the state variable and then operate on the variable in memory as it cost less gas to change variables in memory than in storage. then use the variable in memory to update the state variable 

 
\'\'\'diff.md


624,685c624,654
<   function _transfer(
<     uint256 _rdpxAmount,
<     uint256 _wethAmount,
<     uint256 _bondAmount,
<     uint256 _bondId
<   ) internal {
<     if (_bondId != 0) {
<       (, uint256 expiry, uint256 amount) = IRdpxDecayingBonds(
<         addresses.rdpxDecayingBonds
<       ).bonds(_bondId);
< 
<       _validate(amount >= _rdpxAmount, 1);
<       _validate(expiry >= block.timestamp, 2);
<       _validate(
<         IRdpxDecayingBonds(addresses.rdpxDecayingBonds).ownerOf(_bondId) ==
<           msg.sender,
<         9
<       );
< 
<       IRdpxDecayingBonds(addresses.rdpxDecayingBonds).decreaseAmount(
<         _bondId,
<         amount - _rdpxAmount
<       );
< 
<       IRdpxReserve(addresses.rdpxReserve).withdraw(_rdpxAmount);
< 
<       reserveAsset[reservesIndex["RDPX"]].tokenBalance += _rdpxAmount;
<     } else {
<       // Transfer rDPX and ETH token from user
<       IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress)
<         .safeTransferFrom(msg.sender, address(this), _rdpxAmount);
< 
<       // burn the rdpx
<       IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress).burn(
<         (_rdpxAmount * rdpxBurnPercentage) / 1e10
<       );
< 
<       // transfer the rdpx to the fee distributor
<       IERC20WithBurn(reserveAsset[reservesIndex["RDPX"]].tokenAddress)
<         .safeTransfer(
<           addresses.feeDistributor,
<           (_rdpxAmount * rdpxFeePercentage) / 1e10
<         );
< 
<       // calculate extra rdpx to withdraw to compensate for discount
<       uint256 rdpxAmountInWeth = (_rdpxAmount * getRdpxPrice()) / 1e8;
<       uint256 discountReceivedInWeth = _bondAmount -
<         _wethAmount -
<         rdpxAmountInWeth;
<       uint256 extraRdpxToWithdraw = (discountReceivedInWeth * 1e8) /
<         getRdpxPrice();
< 
<       // withdraw the rdpx
<       IRdpxReserve(addresses.rdpxReserve).withdraw(
<         _rdpxAmount + extraRdpxToWithdraw
<       );
< 
<       reserveAsset[reservesIndex["RDPX"]].tokenBalance +=
<         _rdpxAmount +
<         extraRdpxToWithdraw;
<     }
<   }
---
>    function _transfer(uint256 _rdpxAmount, uint256 _wethAmount, uint256 _bondAmount, uint256 _bondId) internal {
>         if (_bondId != 0) {
>             (, uint256 expiry, uint256 amount) = IRdpxDecayingBonds(addresses.rdpxDecayingBonds).bonds(_bondId);
> 
>             _validate(amount >= _rdpxAmount, 1);
>             _validate(expiry >= block.timestamp, 2);
>             _validate(IRdpxDecayingBonds(addresses.rdpxDecayingBonds).ownerOf(_bondId) == msg.sender, 9);
> 
>             IRdpxDecayingBonds(addresses.rdpxDecayingBonds).decreaseAmount(_bondId, amount - _rdpxAmount);
> 
>             IRdpxReserve(addresses.rdpxReserve).withdraw(_rdpxAmount);
> 
>             reserveAsset[reservesIndex["RDPX"]].tokenBalance += _rdpxAmount;
>         } else {
>             ///@audit gas optimization
>             // Transfer rDPX and ETH token from user
>             ReserveAsset asset = reserveAsset[reservesIndex["RDPX"]];
>             IERC20WithBurn(asset.tokenAddress).safeTransferFrom(msg.sender, address(this), _rdpxAmount);
> 
>             // burn the rdpx
>             IERC20WithBurn(asset.tokenAddress).burn((_rdpxAmount * rdpxBurnPercentage) / 1e10);
> 
>             // transfer the rdpx to the fee distributor
>             IERC20WithBurn(asset.tokenAddress).safeTransfer(
>                 addresses.feeDistributor, (_rdpxAmount * rdpxFeePercentage) / 1e10
>             );
> 
>             // calculate extra rdpx to withdraw to compensate for discount
>             uint256 rdpxAmountInWeth = (_rdpxAmount * getRdpxPrice()) / 1e8;
>             uint256 discountReceivedInWeth = _bondAmount - _wethAmount - rdpxAmountInWeth;
>             uint256 extraRdpxToWithdraw = (discountReceivedInWeth * 1e8) / getRdpxPrice();
686a656,660
>             // withdraw the rdpx
>             IRdpxReserve(addresses.rdpxReserve).withdraw(_rdpxAmount + extraRdpxToWithdraw);
> 
>             reserveAsset[reservesIndex["RDPX"]] = asset;
>         }    }
\'\'\'
## tools used manual review