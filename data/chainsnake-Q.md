## Hardcoded and incorrect token symbols may cause missing indication

`reserveAsset` in Core is often accessed by `reservesIndex` indexed by a hardcoded symbol name, like `reserveAsset[reservesIndex["RDPX"]]`.

But for RPDX, its actual name is `rPDX` not `RPDX`. (https://etherscan.io/token/0x0ff5a8451a839f5f0bb3562689d9a44089738d11)

If we access to `reserveAsset` by calling `IERC20().symbol()`, then it won't find the token and may occur unexpected problems.