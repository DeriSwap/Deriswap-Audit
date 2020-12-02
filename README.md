# Quantstamp Contract Audit

Join telegram channel for audit updates : <br>
https://t.me/DeriswapOfficial
<br>
<br>
Deriswap Security Review<br>
<br>
<br>
1. Summary<br>
This document is a security review of Deriswap.<br>

Timeline: 26/11/20 - 29/11/20<br>
Token Contract : 0xd2750c6f3fadd9135c6bc15d2d0fde3b1dba8c95

Auditors:<br>
Ed Zulkoski, Senior Research Engineer<br>
Shunsuke Tokoshima, Blockchain Researcher<br>
Joseph Xu, Technical R&D Advisor<br>
<br>

2. In scope<br>
Commit: b3672d93<br>
<br>
For GovernorAlpha.sol, Timelock.sol, only diffs were considered as compared to compound-finance commit 857c223.
For deriswapv1/*, we only considered the diff as specified in deriswapv1/README.md, i.e., git diff 4c4bf551417e3df09a25aa0dbb6941cccbbac11a ..
Documentation: Deriswap Medium Article<br>
<br>
3. Findings<br>
<br>
No critical or high risk issues were found. Although we provide some recommendations for fixes below, since the contracts are already deployed, these are primarily for informational purposes. No found issues were critical enough to suggest re-deployment of the existing contracts, however caution should be used when using them.<br>
<br>
3.1. add() does not prevent the same LP token from being added more than once
Severity: Low
Contract(s) affected: Master.sol
Description
On L106 it is noted: "// XXX DO NOT add the same LP token more than once. Rewards will be messed up if you do."

If a token is mistakenly added more than once, it would reset the rewards variables associated with the token (e.g., accDeriPerShare).

Recommendation
This could be prevented by creating a mapping from addresses to booleans, such that LP tokens get mapped to true once they've been added. The function could then have a require-statement preventing the same LP token from being added twice.<br>
<br>
3.2. migrate() is dependent on a currently unspecified migrator contract<br>
Severity: Low<br>
Contract(s) affected: Master.sol<br>
Description<br>
Migrator can be set to any contract, potentially allowing the theft of funds, particularly if the owner's private key is compromised. The migrator can be set at any time (and multiple times) by the owner. In the worst case, this could be set to a contract that transfers all underlying LP tokens to an arbitrary address.
<br>
Note: since the owner is a timelock contract, users would have 2 days to mass exit the pool before the change could occur.<br>
<br>
3.3. massUpdatePools() may run out-of-gas if too many tokens are added<br>
Severity: Low<br>
Contract(s) affected: Master.sol<br>
Description<br>
This may have implications on calling functions add() and set().<br>
<br>
3.4. emergencyWithdraw() does not follow the checks-effects-interactions<br>
Severity: Informational<br>
Contract(s) affected: Master.sol<br>
Description<br>
The values for user.amount and user.rewardDebt are not zeroed until after the external call to pool.lpToken.safeTransfer(). Although the LP tokens are assumed trusted as of now, if a compromised token is added, reentrancy may allow the theft of tokens.<br>
<br>
Recommendation<br>
In general we recommend following the checks-effects-interactions pattern. Zero out the user's state variables prior to the external safeTransfer() call (ensuring the correct amount is still passed in safeTransfer() using a local variable).<br>
<br>
3.5. Missing constructor checks<br>
Severity: Informational<br>
Contract(s) affected: Maker.sol, Swap.sol, Migrator.sol<br>
Description<br>
In the constructors for Maker.sol, Swap.sol, and Migrator.sol, the address parameters are not checked to be non-zero. Similarly, the function Master.setMigrator() does not have input validation.<br>
<br>
Recommendation<br>
In general we recommend ensuring that address arguments are non-zero in constructor and setter functions in order to mitigate deployment issues.<br>
<br>
3.6. Unlocked dependency version<br>
Severity: Informational<br>
File(s) affected: package.json<br>
Description<br>
Some contracts import contracts from @openzeppelin/contracts. The version of @openzeppelin/contracts is not locked in package.json. Unlocked dependency versions may result in compiling the project using versions of dependencies that are different from the ones that were tested or audited.<br>

Recommendation<br>
In package.json, lock dependency versions, e.g., "@openzeppelin/contracts": "3.1.0".<br>

4. Appendix<br>
About Quantstamp<br>
Quantstamp is a Y Combinator-backed company that helps to secure blockchain platforms at scale using computer-aided reasoning tools, with a mission to help boost the adoption of this exponentially growing technology.<br>
<br>
With over 1000 Google scholar citations and numerous published papers, Quantstamp's team has decades of combined experience in formal verification, static analysis, and software verification. Quantstamp has also developed a protocol to help smart contract developers and projects worldwide to perform cost-effective smart contract security scans.<br>
<br>
To date, Quantstamp has protected $5B in digital asset risk from hackers and assisted dozens of blockchain projects globally through its white glove security assessment services. As an evangelist of the blockchain ecosystem, Quantstamp assists core infrastructure projects and leading community initiatives such as the Ethereum Community Fund to expedite the adoption of blockchain technology.<br>
<br>
Quantstamp's collaborations with leading academic institutions such as the National University of Singapore and MIT (Massachusetts Institute of Technology) reflect our commitment to research, development, and enabling world-class blockchain security.<br>
<br>
Timeliness of content<br>
The content contained in the report is current as of the date appearing on the report and is subject to change without notice, unless indicated otherwise by Quantstamp; however, Quantstamp does not guarantee or warrant the accuracy, timeliness, or completeness of any report you access using the internet or other means, and assumes no obligation to update any information following publication. publication.<br>
<br>
Notice of Confidentiality<br>
This report, including the content, data, and underlying methodologies, are subject to the confidentiality and feedback provisions in your agreement with Quantstamp. These materials arenot to be disclosed, extracted, copied, or distributed except to the extent expressly authorized by Quantstamp.<br>
<br>
Links to other websites<br>
You may, through hypertext or other computer links, gain access to web sites operated by persons other than Quantstamp, Inc. (Quantstamp). Such hyperlinks are provided for your reference and convenience only, and are the exclusive responsibility of such web sites' owners. You agree that Quantstamp are not responsible for the content or operation of such web sites, and that Quantstamp shall have no liability to you or any other person or entity for the use of third-party web sites. Except as described below, a hyperlink from this web site to another web site does not imply or mean that Quantstamp endorses the content on that web site or the operator or operations of that site. You are solely responsible for determining the extent to which you may use any content at any other web sites to which you link from the report. Quantstamp assumes no responsibility for the use of third-party software on the website and shall have no liability whatsoever to any person or entity for the accuracy or completeness of any outcome generated by such.<br>
<br>
Disclaimer<br>
This report is based on the scope of materials and documentation provided for a limited review at the time provided. Results may not be complete nor inclusive of all vulnerabilities. The review and this report are provided on an as-is, where-is, and as-available basis. You agree that your access and/or use, including but not limited to any associated services, products, protocols, platforms, content, and materials, will be at your sole risk. Blockchain technology remains under development and is subject to unknown risks and flaws. The review does not extend to the compiler layer, or any other areas beyond the programming language, or other programming aspects that could present security risks. A report does not indicate the endorsement of any particular project or team, nor guarantee its security. No third party should rely on the reports in any way, including for the purpose of making any decisions to buy or sell a product, service or any other asset. To the fullest extent permitted by law, we disclaim all warranties, expressed or implied, in connection with this report, its content, and the related services and products and your use thereof, including, without limitation, the implied warranties of merchantability, fitness for a particular purpose, and non-infringement. We do not warrant, endorse, guarantee, or assume responsibility for any product or service advertised or offered by a third party through the product, any open source or third-party software, code, libraries, materials, or information linked to, called by, referenced by or accessible through the report, its content, and the related services and products, any hyperlinked websites, any websites or mobile applications appearing on any advertising, and we will not be a party to or in any way be responsible for monitoring any transaction between you and any third-party providers of products or services. As with the purchase or use of a product or service through any medium or in any environment, you should use your best judgment and exercise caution where appropriate. FOR AVOIDANCE OF DOUBT, THE REPORT, ITS CONTENT, ACCESS, AND/OR USAGE THEREOF, INCLUDING ANY ASSOCIATED SERVICES OR MATERIALS, SHALL NOT BE CONSIDERED OR RELIED UPON AS ANY FORM OF FINANCIAL, INVESTMENT, TAX, LEGAL, REGULATORY, OR OTHER ADVICE.
