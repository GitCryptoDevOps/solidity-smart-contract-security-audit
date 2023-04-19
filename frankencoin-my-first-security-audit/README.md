# Solidity Smart Contract Security Audit Report

> This is my first Security Audit that I have written during my training.

- Summary
- Scope
- Checked vulnerabilities
- Methodology
- Issues
  - Issue categories
  - Issues found
- Gas optimization
- Summary
- Annexes
  - Lint analysis report (solhint)
  - Running End-to-end tests with gas consumption tracking report (Forge tests)
  - Running functional tests report (Hard Hat tests)
  - Mytril report
  - ERC compliance check report (Slither)
  - Slither report

## Summary

The Frankencoin is a collateralized stablecoin that is intended to track the value of the Swiss Franc. But unlike the minting mechanisms, it uses an auction-based mechanism in order to be independent on external oracles. In the futur, it will support more collaterals but they must respect some characteristics.

This contract is coded mainly with Solidity 0.8.19 and is built, tested, and deployed with Hard Hat and Foundry.

## Scope

The scope of this audit is to analyze and document the Frankencoin smart contract codebase for quality, security, and correctness.

The Audit has been performed between the 15th April, 2023 to 18 April, 2023 based on the Git commit hash 1022cb106919fba963a89205d3b90bf62543f68f.

## Checked vulnerabilities

In this audit, we search vulnerabilities at the Solidity level:
- Call to unknown,
- Gasless send,
- Exception disorders,
- Type casts,
- Reentrance,
- Keeping secrets.

We search also vulnerabilities at the EVM level:
- Immutable bugs,
- Ether lost in transfer,
- Stack size limit.

And we search vulnerabilities at the blockchain level:
- Generating randomness,
- Unpredictable state,
- Time constraints.

## Methodology

The followed methodology is:
- Generate the documentation of the application with Slither.
- Execute the JS tests with Hard Hat.
- Execute the Solidity tests with Foundry and produce the gaz consumption report.
- Evaluate the overall quality of code.
- Check the best practices are used.
- Check the source code for programmatic and stylistic errors with `solhint` (Lint).
- Find vulnerabilities with the Slither security audit tool.
- Check ERC compliance with the Frankencoin token.
- Check the implementation of the ERC-20 token standards.
- Check the efficient use of gas.
- Find vulnerabilities woth the Manticore security audit tool, once deployed.
- Conduct a manual code review.

In the step of analysis of the structure, we have analyzed the structure of the smart contract and the design patterns used in the codebase.

In the step of security static analysis, a security static analysis of the smart contracts has been done to identify vulnerabilities with automated tools.

In the step of manual code review, a code review has been conducted to identify more vulnerabilities and to verify the vulnerabilities found during the security static analysis. The logic has been compared with the business rules described in the gitbook/whitepaper.

In the step of gas consumption, the gas consumption as been checked at the codebase by generating a report. Then we have checked if we can find techniques of optimization to reduce the gas consumption.

The tools used are:
- VSCode,
- Hard Hat,
- Foundry,
- Solhint,
- Mythril,
- Slither.

## Issues

### Issue categories

There are four severity levels:
- high,
- medium,
- low,
- informational.

Each issue has been assigned to a severity level.

The severity levels are:

| Severity-level | Description|
|-|-|
| High | These issues are critical for the functionnality or performance. They can be exploited. We recommend to fix them. |
| Medium | These issues appear because of errors or malfunctioning of the smart contrat. They can bring problems. We recommend to fix them. |
| Low | These issues are warnings that can remain unfixed for now. We recommend to fix them in the future. |
| Informational | These issues are general questions, improvements, or a request for information. |

### Issues found

The number of found issues per severity is:

| Type | High | Medium | Low | Informational |
|-|-|-|-|-|
| Open | 0 | 0 | 4 | 5 |
| Acknowledged | 0 | 0 | 0 | 0 |
| Closed | 0 | 0 | 0 | 0 |

All Foundry tests passed.

Based on tests, the Frankencoin token is ERC compliant.

Here are the recommendantions to mitigate these vulnerabilities.

**1. There are functional tests, not unit tests**

Remediation: Unit tests test individual and isolated functions, integration tests test functions (units) interacting together, and functional tests test features. But the tests in the project don't include unit tests, only functional tests and a kind of end-to-end tests (in the Foundry part). To ensure the functions have no bug, it's recommended to create some basic unit tests, especially at the limits of values.

Severity: Informational

**2. All Hard Hat tests passed but there were many warning during the compilation, only `Unused local variable` messages.**

Remediation: It's recommended to clean All Hard Hat tests passed but there were many warning during the compilation, only `Unused local variable` messages. Removing these unused variables makes more readable the codebase.

Severity: Low

**3. For lint, the only issue found is a line too long (`max-line-length`) for the default value.**

Remediation: This value is based on the default value. Each project can change this value. No action is recommended.

Severity: Informational

**4. As general remark, the security of the smart contract is based on the assumption that the smart contracts of the tokens used as collateral comply with certain requirements.**

Severity: Informational

**5. In the `Position.sol` smart contract, in the `initializeClone()` function, the variable `owner` is shadowed.**

Remediation: Shadowing variables makes reduce the lisibility of the code. Use a suffix for the `owner` argument as for the other arguments would make the code more readable.

Severity: Low

**6. The `mint()`, `burn()`, `burnWithReserve()` and `burnFrom()` function don't trigger any event.**

Remediation: For critical events such as minting or burning, it's a good practice to emit an event for logging. It allows to facilitate investigations and monitoring. It's done for some critical events such as redeem, but not for those.

Severity: Informational

**7. Multiple Solc versions are used.**

Remediation: It's recommended to use a single Solidity version in order to not be exposed to various bugs in the compiler and to improve the readibility for auditing. The severity is informational because the smart contracts with an older version are very well audited smart contracts.

Severity: Informational

**8. In the `Frankencoin` contract, there are too many digits in some constants (`1000000`).**

Remediation: In the `Frankencoin` contract, it's recommended to use a constant `ONE_DEC6` for the value `ONE_DEC18` defined in the `MathUtil` contract, which could be use in the `Frankencoin` contract, as it's done in the `Equity` contract.

Severity: Low

**9. The features tested by each test are not very readable.**

Remediation: In the `test` folder, the tests are not very readable. For example, by adding some comments or annotations, a documentation could be automatically generated for example with the hardhet-dodoc plugin. If some comments would explain the context, the conditions, and the expectation (like with the BDD approach), it would be easier to check each business rule of the smart contract is tested and how many times.

Severity: Low

## Gas optimization

Three improvements in gas consumption were found.

**1 - For constants, `private` variables consumes less gas than `public` variables**

Code: https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L20

Mitigation: Use `uint256 public constant OPENING_FEE = 1_000_000_000_000_000_000_000;`

**2 - For state variables, `x += y` consumes more gas than `x = x + y`**

Instance 1:
- Link: https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L196
- Mitigation: Use `minted = minted + amount;`

Instance 2:
- Link: https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L295
- Mitigation: Use `challengedAmount = challengedAmount + size;`

**3 - Custom errors consume less gas that string error messages**

Three instances have been found:
- Link: https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L109

```
require(_initialCollateral >= _minCollateral, "must start with min col");
```

- Link: https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L116

```
require(zchf.isPosition(position) == address(this), "not our pos");
```

- Link: https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L255

```
require(block.timestamp >= challenge.end, "period has not ended");
```

Mitigation:

Instead of this kind of code:

```
bool check = false;
require(check, "error message");
```

Prefer:

```
error CustomError();

bool check = false;
if (!check) {
  revert CustomError();
}
```

## Summary

In this report, we have considered the security of the Frankencoin smart contract. We performed the audit according the procedure described above.

The audit showed several low and information severity issues. No critical issues related to the code base have been found.

## Annexes

### Lint analysis report (solhint)

```
/Users/test/2023-04-frankencoin/contracts/Equity.sol
  117:2  error  Line length must be no more than 120 but current length is 122  max-line-length
  161:2  error  Line length must be no more than 120 but current length is 140  max-line-length
  268:2  error  Line length must be no more than 120 but current length is 170  max-line-length

/Users/test/2023-04-frankencoin/contracts/ERC20.sol
  53:2  error  Line length must be no more than 120 but current length is 159  max-line-length

/Users/test/2023-04-frankencoin/contracts/ERC20PermitLight.sol
  40:2  error  Line length must be no more than 120 but current length is 131  max-line-length

/Users/test/2023-04-frankencoin/contracts/Frankencoin.sol
   78:2  error  Line length must be no more than 120 but current length is 121  max-line-length
   79:2  error  Line length must be no more than 120 but current length is 121  max-line-length
   80:2  error  Line length must be no more than 120 but current length is 121  max-line-length
   83:2  error  Line length must be no more than 120 but current length is 141  max-line-length
  115:2  error  Line length must be no more than 120 but current length is 126  max-line-length
  169:2  error  Line length must be no more than 120 but current length is 137  max-line-length
  185:2  error  Line length must be no more than 120 but current length is 132  max-line-length
  188:2  error  Line length must be no more than 120 but current length is 145  max-line-length
  190:2  error  Line length must be no more than 120 but current length is 123  max-line-length
  191:2  error  Line length must be no more than 120 but current length is 128  max-line-length
  192:2  error  Line length must be no more than 120 but current length is 127  max-line-length
  201:2  error  Line length must be no more than 120 but current length is 126  max-line-length
  217:2  error  Line length must be no more than 120 but current length is 127  max-line-length
  219:2  error  Line length must be no more than 120 but current length is 126  max-line-length
  220:2  error  Line length must be no more than 120 but current length is 124  max-line-length
  221:2  error  Line length must be no more than 120 but current length is 125  max-line-length
  223:2  error  Line length must be no more than 120 but current length is 135  max-line-length
  232:2  error  Line length must be no more than 120 but current length is 122  max-line-length
  235:2  error  Line length must be no more than 120 but current length is 133  max-line-length
  238:2  error  Line length must be no more than 120 but current length is 134  max-line-length
  243:2  error  Line length must be no more than 120 but current length is 127  max-line-length
  244:2  error  Line length must be no more than 120 but current length is 127  max-line-length
  246:2  error  Line length must be no more than 120 but current length is 127  max-line-length
  248:2  error  Line length must be no more than 120 but current length is 124  max-line-length
  251:2  error  Line length must be no more than 120 but current length is 129  max-line-length
  254:2  error  Line length must be no more than 120 but current length is 147  max-line-length

/Users/test/2023-04-frankencoin/contracts/IPosition.sol
  36:2  error  Line length must be no more than 120 but current length is 143  max-line-length

/Users/test/2023-04-frankencoin/contracts/MintingHub.sol
  121:2  error  Line length must be no more than 120 but current length is 123  max-line-length
  122:2  error  Line length must be no more than 120 but current length is 125  max-line-length
  124:2  error  Line length must be no more than 120 but current length is 140  max-line-length
  140:2  error  Line length must be no more than 120 but current length is 131  max-line-length
  144:2  error  Line length must be no more than 120 but current length is 139  max-line-length
  247:2  error  Line length must be no more than 120 but current length is 137  max-line-length
  250:2  error  Line length must be no more than 120 but current length is 126  max-line-length
  256:2  error  Line length must be no more than 120 but current length is 124  max-line-length
  258:2  error  Line length must be no more than 120 but current length is 129  max-line-length
  260:2  error  Line length must be no more than 120 but current length is 188  max-line-length
  289:2  error  Line length must be no more than 120 but current length is 140  max-line-length
  294:2  error  Line length must be no more than 120 but current length is 129  max-line-length

/Users/test/2023-04-frankencoin/contracts/Position.sol
   21:2  error  Line length must be no more than 120 but current length is 138  max-line-length
   76:2  error  Line length must be no more than 120 but current length is 124  max-line-length
  213:2  error  Line length must be no more than 120 but current length is 129  max-line-length
  219:2  error  Line length must be no more than 120 but current length is 123  max-line-length
  221:2  error  Line length must be no more than 120 but current length is 131  max-line-length
  222:2  error  Line length must be no more than 120 but current length is 130  max-line-length
  223:2  error  Line length must be no more than 120 but current length is 129  max-line-length
  224:2  error  Line length must be no more than 120 but current length is 131  max-line-length
  327:2  error  Line length must be no more than 120 but current length is 121  max-line-length
  329:2  error  Line length must be no more than 120 but current length is 155  max-line-length
  347:2  error  Line length must be no more than 120 but current length is 126  max-line-length

/Users/test/2023-04-frankencoin/contracts/PositionFactory.sol
  36:2  error  Line length must be no more than 120 but current length is 138  max-line-length

/Users/test/2023-04-frankencoin/contracts/test/Math.sol
   51:2  error  Line length must be no more than 120 but current length is 130  max-line-length
   94:2  error  Line length must be no more than 120 but current length is 124  max-line-length
  118:2  error  Line length must be no more than 120 but current length is 123  max-line-length

/Users/test/2023-04-frankencoin/contracts/test/MintingHubTest.sol
  100:2  error  Line length must be no more than 120 but current length is 125  max-line-length
  118:2  error  Line length must be no more than 120 but current length is 123  max-line-length
  195:2  error  Line length must be no more than 120 but current length is 127  max-line-length
  217:2  error  Line length must be no more than 120 but current length is 127  max-line-length
  221:2  error  Line length must be no more than 120 but current length is 133  max-line-length
  261:2  error  Line length must be no more than 120 but current length is 130  max-line-length
  313:2  error  Line length must be no more than 120 but current length is 131  max-line-length
  315:2  error  Line length must be no more than 120 but current length is 173  max-line-length

/Users/test/2023-04-frankencoin/contracts/test/Strings.sol
  66:2  error  Line length must be no more than 120 but current length is 129  max-line-length

✖ 68 problems (68 errors, 0 warnings)
```

## Running end-to-end tests with gas consumption tracking report (Forge tests)

```
[⠰] Compiling...
[⠢] Compiling 39 files with 0.8.19
[⠔] Solc 0.8.19 finished in 4.89s
Compiler run successful

Running 7 tests for test/GeneralTest.t.sol:GeneralTest
[PASS] test01Equity() (gas: 2126195)
[PASS] test02DenyPosition() (gas: 1877662)
[PASS] test03MintingEarly() (gas: 1862533)
[PASS] test04Mint():(address) (gas: 2125092)
[PASS] test05MintFail() (gas: 1871329)
[PASS] test06Withdraw() (gas: 1943771)
[PASS] test09EndingChallenge() (gas: 4449200)
Test result: ok. 7 passed; 0 failed; finished in 6.64ms
| contracts/Equity.sol:Equity contract |                 |       |        |       |         |
|--------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                      | Deployment Size |       |        |       |         |
| 1503722                              | 7744            |       |        |       |         |
| Function Name                        | min             | avg   | median | max   | # calls |
| checkQualified                       | 10375           | 10375 | 10375  | 10375 | 1       |
| onTokenTransfer                      | 17106           | 57534 | 58348  | 97150 | 6       |
| totalSupply                          | 360             | 760   | 360    | 2360  | 10      |


| contracts/Frankencoin.sol:Frankencoin contract |                 |       |        |        |         |
|------------------------------------------------|-----------------|-------|--------|--------|---------|
| Deployment Cost                                | Deployment Size |       |        |        |         |
| 2960396                                        | 15168           |       |        |        |         |
| Function Name                                  | min             | avg   | median | max    | # calls |
| balanceOf                                      | 629             | 734   | 629    | 2629   | 38      |
| burn                                           | 3108            | 3496  | 3496   | 3884   | 2       |
| burnFrom                                       | 8110            | 8110  | 8110   | 8110   | 2       |
| equity                                         | 686             | 1080  | 759    | 4686   | 24      |
| isPosition                                     | 657             | 657   | 657    | 657    | 11      |
| mint(address,uint256)                          | 3500            | 33100 | 25400  | 49300  | 15      |
| mint(address,uint256,uint32,uint32)            | 7508            | 20048 | 27408  | 31408  | 10      |
| notifyLoss                                     | 3934            | 3934  | 3934   | 3934   | 2       |
| registerPosition                               | 23111           | 24861 | 25111  | 25111  | 8       |
| reserve                                        | 283             | 283   | 283    | 283    | 17      |
| suggestMinter                                  | 28804           | 31804 | 31804  | 34804  | 14      |
| transfer                                       | 3432            | 9845  | 3432   | 20266  | 5       |
| transferAndCall                                | 17196           | 56307 | 50189  | 101538 | 6       |
| transferFrom                                   | 3516            | 16687 | 21836  | 28294  | 12      |


| contracts/MintingHub.sol:MintingHub contract |                 |         |         |         |         |
|----------------------------------------------|-----------------|---------|---------|---------|---------|
| Deployment Cost                              | Deployment Size |         |         |         |         |
| 1833168                                      | 9528            |         |         |         |         |
| Function Name                                | min             | avg     | median  | max     | # calls |
| OPENING_FEE                                  | 252             | 252     | 252     | 252     | 8       |
| bid                                          | 13913           | 41689   | 39670   | 73503   | 4       |
| challenges                                   | 1361            | 1361    | 1361    | 1361    | 6       |
| end                                          | 27756           | 27756   | 27756   | 27756   | 2       |
| isChallengeOpen                              | 694             | 694     | 694     | 694     | 2       |
| launchChallenge                              | 86509           | 107742  | 86509   | 150209  | 3       |
| minBid                                       | 821             | 821     | 821     | 821     | 3       |
| openPosition                                 | 1586826         | 1611176 | 1615226 | 1615226 | 8       |


| contracts/Position.sol:Position contract |                 |       |        |       |         |
|------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                          | Deployment Size |       |        |       |         |
| 1552422                                  | 8719            |       |        |       |         |
| Function Name                            | min             | avg   | median | max   | # calls |
| adjust                                   | 9963            | 16336 | 9963   | 39828 | 12      |
| adjustPrice                              | 5319            | 5319  | 5319   | 5319  | 8       |
| challengePeriod                          | 294             | 294   | 294    | 294   | 3       |
| collateral                               | 260             | 260   | 260    | 260   | 17      |
| cooldown                                 | 352             | 352   | 352    | 352   | 1       |
| deny                                     | 16991           | 16991 | 16991  | 16991 | 1       |
| expiration                               | 296             | 296   | 296    | 296   | 1       |
| getUsableMint                            | 713             | 987   | 987    | 1262  | 16      |
| mint                                     | 593             | 28287 | 34430  | 58330 | 10      |
| minted                                   | 351             | 351   | 351    | 351   | 7       |
| notifyChallengeStarted                   | 699             | 7332  | 699    | 20599 | 3       |
| notifyChallengeSucceeded                 | 8960            | 10080 | 10080  | 11200 | 2       |
| price                                    | 396             | 396   | 396    | 396   | 5       |
| transferOwnership                        | 2394            | 2394  | 2394   | 2394  | 4       |
| tryAvertChallenge                        | 750             | 910   | 750    | 1391  | 4       |
| withdraw                                 | 19616           | 24648 | 24648  | 29681 | 2       |


| contracts/PositionFactory.sol:PositionFactory contract |                 |         |         |         |         |
|--------------------------------------------------------|-----------------|---------|---------|---------|---------|
| Deployment Cost                                        | Deployment Size |         |         |         |         |
| 1839541                                                | 9220            |         |         |         |         |
| Function Name                                          | min             | avg     | median  | max     | # calls |
| createNewPosition                                      | 1587422         | 1587422 | 1587422 | 1587422 | 8       |


| contracts/test/TestToken.sol:TestToken contract |                 |       |        |       |         |
|-------------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                                 | Deployment Size |       |        |       |         |
| 496708                                          | 3277            |       |        |       |         |
| Function Name                                   | min             | avg   | median | max   | # calls |
| allowance                                       | 858             | 858   | 858    | 858   | 15      |
| approve                                         | 22451           | 23876 | 24551  | 24551 | 28      |
| balanceOf                                       | 584             | 584   | 584    | 584   | 99      |
| mint                                            | 22848           | 37039 | 46748  | 46748 | 24      |
| transfer                                        | 2745            | 13295 | 13381  | 23331 | 8       |
| transferFrom                                    | 4823            | 14834 | 22343  | 22343 | 28      |


| test/GeneralTest.t.sol:User contract |                 |         |         |         |         |
|--------------------------------------|-----------------|---------|---------|---------|---------|
| Deployment Cost                      | Deployment Size |         |         |         |         |
| 2049098                              | 10311           |         |         |         |         |
| Function Name                        | min             | avg     | median  | max     | # calls |
| adjustPosition                       | 76810           | 76810   | 76810   | 76810   | 2       |
| avertChallenge                       | 83726           | 83726   | 83726   | 83726   | 1       |
| bid                                  | 18068           | 49828   | 53758   | 77658   | 3       |
| challenge                            | 111129          | 133062  | 111129  | 176929  | 3       |
| deny                                 | 18070           | 18070   | 18070   | 18070   | 1       |
| initiatePosition                     | 1653111         | 1706336 | 1714511 | 1714511 | 8       |
| invest                               | 18509           | 58397   | 51503   | 105180  | 6       |
| mint                                 | 5994            | 38923   | 45054   | 67954   | 10      |
| obtainFrankencoins                   | 48429           | 98589   | 127389  | 129389  | 13      |
| transferOwnership                    | 3259            | 3259    | 3259    | 3259    | 4       |
```

## Running functional tests report (Hard Hat tests)

```
Downloading compiler 0.8.13
Warning: This declaration shadows an existing declaration.
   --> contracts/test/MintingHubTest.sol:326:14:
    |
326 |             (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |              ^^^^^^^^^^^^^^^^^^
Note: The shadowed declaration is here:
   --> contracts/test/MintingHubTest.sol:331:10:
    |
331 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |          ^^^^^^^^^^^^^^^^^^


Warning: This declaration shadows an existing declaration.
   --> contracts/test/MintingHubTest.sol:326:34:
    |
326 |             (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                  ^^^^^^^^^^^
Note: The shadowed declaration is here:
   --> contracts/test/MintingHubTest.sol:331:30:
    |
331 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                              ^^^^^^^^^^^


Warning: This declaration shadows an existing declaration.
   --> contracts/test/MintingHubTest.sol:326:47:
    |
326 |             (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                               ^^^^^^^^^^^^
Note: The shadowed declaration is here:
   --> contracts/test/MintingHubTest.sol:331:43:
    |
331 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                           ^^^^^^^^^^^^


Warning: This declaration shadows an existing declaration.
   --> contracts/test/MintingHubTest.sol:326:61:
    |
326 |             (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                                             ^^^^^^^^^
Note: The shadowed declaration is here:
   --> contracts/test/MintingHubTest.sol:331:57:
    |
331 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                                         ^^^^^^^^^


Warning: This declaration shadows an existing declaration.
   --> contracts/test/MintingHubTest.sol:326:72:
    |
326 |             (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                                                        ^^^^^^^^^
Note: The shadowed declaration is here:
   --> contracts/test/MintingHubTest.sol:331:68:
    |
331 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                                                    ^^^^^^^^^


Warning: This declaration shadows an existing declaration.
   --> contracts/test/MintingHubTest.sol:326:83:
    |
326 |             (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                                                                   ^^^^^^^^^^^
Note: The shadowed declaration is here:
   --> contracts/test/MintingHubTest.sol:331:79:
    |
331 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                                                               ^^^^^^^^^^^


Warning: This declaration shadows an existing declaration.
   --> contracts/test/MintingHubTest.sol:331:79:
    |
331 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                                                               ^^^^^^^^^^^
Note: The shadowed declaration is here:
   --> contracts/test/MintingHubTest.sol:336:5:
    |
336 |     function bid(MintingHub hub, uint256 number, uint256 amount) public {
    |     ^ (Relevant source part starts here and spans across multiple lines).


Warning: This declaration shadows an existing declaration.
   --> contracts/test/MintingHubTest.sol:337:79:
    |
337 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(number);
    |                                                                               ^^^^^^^^^^^
Note: The shadowed declaration is here:
   --> contracts/test/MintingHubTest.sol:336:5:
    |
336 |     function bid(MintingHub hub, uint256 number, uint256 amount) public {
    |     ^ (Relevant source part starts here and spans across multiple lines).


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:100:30:
    |
100 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(latestChallenge);
    |                              ^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:100:43:
    |
100 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(latestChallenge);
    |                                           ^^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:100:57:
    |
100 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(latestChallenge);
    |                                                         ^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:100:68:
    |
100 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(latestChallenge);
    |                                                                    ^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:118:11:
    |
118 |          (address challenger1, IPosition p1, uint256 size1, uint256 a1, address b1, uint256 bid1) = hub.challenges(number);
    |           ^^^^^^^^^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:118:32:
    |
118 |          (address challenger1, IPosition p1, uint256 size1, uint256 a1, address b1, uint256 bid1) = hub.challenges(number);
    |                                ^^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:118:61:
    |
118 |          (address challenger1, IPosition p1, uint256 size1, uint256 a1, address b1, uint256 bid1) = hub.challenges(number);
    |                                                             ^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:118:73:
    |
118 |          (address challenger1, IPosition p1, uint256 size1, uint256 a1, address b1, uint256 bid1) = hub.challenges(number);
    |                                                                         ^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:141:57:
    |
141 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(number);
    |                                                         ^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:141:68:
    |
141 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(number);
    |                                                                    ^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:217:10:
    |
217 |         (address challenger, IPosition p, uint256 size, uint256 end, address b, uint256 bid) = hub.challenges(latestChallenge);
    |          ^^^^^^^^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:217:30:
    |
217 |         (address challenger, IPosition p, uint256 size, uint256 end, address b, uint256 bid) = hub.challenges(latestChallenge);
    |                              ^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:217:43:
    |
217 |         (address challenger, IPosition p, uint256 size, uint256 end, address b, uint256 bid) = hub.challenges(latestChallenge);
    |                                           ^^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:217:70:
    |
217 |         (address challenger, IPosition p, uint256 size, uint256 end, address b, uint256 bid) = hub.challenges(latestChallenge);
    |                                                                      ^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:217:81:
    |
217 |         (address challenger, IPosition p, uint256 size, uint256 end, address b, uint256 bid) = hub.challenges(latestChallenge);
    |                                                                                 ^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:221:10:
    |
221 |         (address challenger2, IPosition p2, uint256 size2, uint256 end2, address b2, uint256 bid2) = hub.challenges(latestChallenge);
    |          ^^^^^^^^^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:221:31:
    |
221 |         (address challenger2, IPosition p2, uint256 size2, uint256 end2, address b2, uint256 bid2) = hub.challenges(latestChallenge);
    |                               ^^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:221:45:
    |
221 |         (address challenger2, IPosition p2, uint256 size2, uint256 end2, address b2, uint256 bid2) = hub.challenges(latestChallenge);
    |                                             ^^^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:221:74:
    |
221 |         (address challenger2, IPosition p2, uint256 size2, uint256 end2, address b2, uint256 bid2) = hub.challenges(latestChallenge);
    |                                                                          ^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:221:86:
    |
221 |         (address challenger2, IPosition p2, uint256 size2, uint256 end2, address b2, uint256 bid2) = hub.challenges(latestChallenge);
    |                                                                                      ^^^^^^^^^^^^


Warning: Unused function parameter. Remove or comment out the variable name to silence this warning.
   --> contracts/test/MintingHubTest.sol:272:19:
    |
272 |     function deny(MintingHub hub, address pos) public {
    |                   ^^^^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:326:14:
    |
326 |             (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |              ^^^^^^^^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:326:61:
    |
326 |             (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                                             ^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:326:72:
    |
326 |             (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                                                        ^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:326:83:
    |
326 |             (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                                                                   ^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:331:30:
    |
331 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                              ^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:331:43:
    |
331 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                           ^^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:331:57:
    |
331 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                                         ^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:331:68:
    |
331 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                                                    ^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:331:79:
    |
331 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                                                               ^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:337:10:
    |
337 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(number);
    |          ^^^^^^^^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:337:30:
    |
337 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(number);
    |                              ^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:337:57:
    |
337 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(number);
    |                                                         ^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:337:68:
    |
337 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(number);
    |                                                                    ^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:337:79:
    |
337 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(number);
    |                                                                               ^^^^^^^^^^^


Compiled 21 Solidity files successfully

  Basic Tests
    basic initialization
      ✓ symbol should be ZCHF
    mock bridge
      ✓ create mock token
      ✓ create mock stable coin bridge
      ✓ minting fails if not approved
      ✓ bootstrap suggestMinter
      ✓ minter of XCHF-bridge should receive ZCHF
      ✓ burner of XCHF-bridge should receive XCHF
    exchanges shares & pricing
      ✓ deposit XCHF to reserve pool and receive share tokens
      ✓ cannot redeem shares immediately
      ✓ can redeem shares after *N* blocks
      ✓ redeem 1 share

  Equity Tests
    basic initialization
      ✓ should have symbol ZCHF
      ✓ should have symbol FPS
      ✓ should have the right name
      ✓ should have some coins
    minting shares
      ✓ should create an initial share
      ✓ should create 1000 more shares when adding seven capital
      ✓ should refuse redemption before time passed
      ✓ should allow redemption after time passed
    transfer shares
      ✓ total votes==sum of owner votes
      ✓ total votes correct after transfer
      ✓ total votes correct after mine
      ✓ delegate vote

  Math Tests
    math
      ✓ div
      ✓ mul
      ✓ pow3
      ✓ cubic root

  Plugin Veto Tests
    create secondary bridge plugin
      ✓ create mock DCHF token&bridge
      ✓ Participant suggests minter
      ✓ can't mint before min period
      ✓ deny minter

  Position Tests
    use Minting Hub
      ✓ create position
      ✓ require cooldown
      ✓ get loan after 7 long days
      ✓ clone position
      ✓ correct collateral
      ✓ global mint limit retained
      ✓ correct fees charged
      ✓ clone position with too much mint
      ✓ repay position
    challenge clone
      ✓ send challenge
      ✓ pos owner cannot withdraw during challenge
      ✓ bid on challenged position
      ✓ bid on not existing challenge
      ✓ new bid on top of bid
      ✓ cannot end successful challenge early
      ✓ end successful challenge
    native position test
      ✓ initialize
      ✓ deny position
      ✓ fails when minting too early
      ✓ allows minting after 2 days
      ✓ supports withdrawals
      ✓ fails when someone else mints
      ✓ perform challenge
      ✓ excessive challenge
      ✓ restructuring
      ✓ challenge expired position

·-------------------------------------------------|---------------------------|-------------|-----------------------------·
|              Solc version: 0.8.13               ·  Optimizer enabled: true  ·  Runs: 200  ·  Block limit: 30000000 gas  │
··················································|···························|·············|······························
|  Methods                                                                                                                │
·····················|····························|·············|·············|·············|···············|··············
|  Contract          ·  Method                    ·  Min        ·  Max        ·  Avg        ·  # calls      ·  usd (avg)  │
·····················|····························|·············|·············|·············|···············|··············
|  Equity            ·  delegateVoteTo            ·      45534  ·      45546  ·      45540  ·            2  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  Equity            ·  redeem                    ·          -  ·          -  ·      66954  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  Equity            ·  transfer                  ·          -  ·          -  ·      85662  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  Frankencoin       ·  approve                   ·          -  ·          -  ·      46229  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  Frankencoin       ·  burn                      ·          -  ·          -  ·      33770  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  Frankencoin       ·  denyMinter                ·          -  ·          -  ·      39074  ·            2  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  Frankencoin       ·  mint                      ·          -  ·          -  ·     117324  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  Frankencoin       ·  suggestMinter             ·      56742  ·      77832  ·      62279  ·            8  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  Frankencoin       ·  transfer                  ·      34663  ·      51751  ·      40359  ·            3  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  Frankencoin       ·  transferAndCall           ·      66332  ·     156262  ·     113576  ·            4  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHub        ·  bid                       ·      76367  ·     123570  ·      99969  ·            4  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHub        ·  clonePosition             ·          -  ·          -  ·     275686  ·            2  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHub        ·  end                       ·          -  ·          -  ·     128303  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHub        ·  launchChallenge           ·          -  ·          -  ·     211011  ·            2  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHub        ·  openPosition              ·          -  ·          -  ·    1715989  ·            2  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHubTest    ·  bidNearEndOfChallenge     ·          -  ·          -  ·     143932  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHubTest    ·  challengeExpiredPosition  ·          -  ·          -  ·     273081  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHubTest    ·  endChallenges             ·          -  ·          -  ·     483940  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHubTest    ·  endLastChallenge          ·          -  ·          -  ·     125786  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHubTest    ·  initiateAndDenyPosition   ·          -  ·          -  ·    1810811  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHubTest    ·  initiateEquity            ·          -  ·          -  ·     365975  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHubTest    ·  initiatePosition          ·          -  ·          -  ·    1828022  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHubTest    ·  letAliceMint              ·          -  ·          -  ·     316617  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHubTest    ·  letBobChallenge           ·          -  ·          -  ·     709386  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHubTest    ·  restructure               ·          -  ·          -  ·     183700  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHubTest    ·  testExcessiveChallenge    ·          -  ·          -  ·     292083  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHubTest    ·  testWithdraw              ·          -  ·          -  ·     173247  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  Position          ·  repay                     ·      86980  ·      88824  ·      87902  ·            2  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  Position          ·  withdrawCollateral        ·          -  ·          -  ·      73058  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  StablecoinBridge  ·  burn                      ·          -  ·          -  ·      72553  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  StablecoinBridge  ·  burn                      ·          -  ·          -  ·      55859  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  StablecoinBridge  ·  mint                      ·      76221  ·     110421  ·      98219  ·            6  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  TestToken         ·  approve                   ·      29094  ·      46206  ·      44879  ·           13  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  TestToken         ·  mint                      ·      51294  ·      68394  ·      64118  ·            8  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  TestToken         ·  transfer                  ·          -  ·          -  ·      51749  ·            2  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  TestToken         ·  transferAndCall           ·          -  ·          -  ·      53253  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  Deployments                                    ·                                         ·  % of limit   ·             │
··················································|·············|·············|·············|···············|··············
|  Frankencoin                                    ·          -  ·          -  ·    3251922  ·       10.8 %  ·          -  │
··················································|·············|·············|·············|···············|··············
|  MintingHub                                     ·    2032556  ·    2032568  ·    2032558  ·        6.8 %  ·          -  │
··················································|·············|·············|·············|···············|··············
|  MintingHubTest                                 ·          -  ·          -  ·    8065826  ·       26.9 %  ·          -  │
··················································|·············|·············|·············|···············|··············
|  PositionFactory                                ·          -  ·          -  ·    2022229  ·        6.7 %  ·          -  │
··················································|·············|·············|·············|···············|··············
|  StablecoinBridge                               ·     474824  ·     474836  ·     474832  ·        1.6 %  ·          -  │
··················································|·············|·············|·············|···············|··············
|  TestMathUtil                                   ·          -  ·          -  ·     209258  ·        0.7 %  ·          -  │
··················································|·············|·············|·············|···············|··············
|  TestToken                                      ·     598984  ·     599008  ·     598990  ·          2 %  ·          -  │
·-------------------------------------------------|-------------|-------------|-------------|---------------|-------------·

  57 passing (3s)
```

## Mytril report

```
==== Dependence on predictable environment variable ====
SWC ID: 116
Severity: Low
Contract: 0x4125cD1F826099A4DEaD6b7746F7F28B30d8402B
Function name: mint(uint256)
PC address: 1111
Estimated Gas Usage: 2040 - 36982
A control flow decision is made based on The block.timestamp environment variable.
The block.timestamp environment variable is used to determine a control flow decision. Note that the values of variables like coinbase, gaslimit, block number and timestamp are predictable and can be manipulated by a malicious miner. Also keep in mind that attackers know hashes of earlier blocks. Don't use any of those environment variables as sources of randomness and be aware that use of these variables introduces a certain level of trust into miners.
--------------------
Initial State:

Account: [ATTACKER], balance: 0x0, nonce:0, storage:{}
Account: [SOMEGUY], balance: 0x0, nonce:0, storage:{}

Transaction Sequence:

Caller: [SOMEGUY], function: mint(uint256), txdata: 0xa0712d680000000000000000000000000000000000000000000000000000000000000000, decoded_data: (0,), value: 0x0

==== Multiple Calls in a Single Transaction ====
SWC ID: 113
Severity: Low
Contract: 0x4125cD1F826099A4DEaD6b7746F7F28B30d8402B
Function name: mint(uint256)
PC address: 1304
Estimated Gas Usage: 5242 - 109312
Multiple calls are executed in the same transaction.
This call is executed following another call within the same transaction. It is possible that the call never gets executed if a prior call fails permanently. This might be caused intentionally by a malicious callee. If possible, refactor the code such that each transaction only executes one external call or make sure that all callees can be trusted (i.e. they’re part of your own codebase).
--------------------
Initial State:

Account: [ATTACKER], balance: 0x0, nonce:0, storage:{}
Account: [SOMEGUY], balance: 0x0, nonce:0, storage:{}

Transaction Sequence:

Caller: [SOMEGUY], function: mint(uint256), txdata: 0xa0712d680000000000000000000000000000000000000000000000000000000000000000, decoded_data: (0,), value: 0x0

==== Multiple Calls in a Single Transaction ====
SWC ID: 113
Severity: Low
Contract: 0x4125cD1F826099A4DEaD6b7746F7F28B30d8402B
Function name: mint(address,uint256)
PC address: 1304
Estimated Gas Usage: 5454 - 109524
Multiple calls are executed in the same transaction.
This call is executed following another call within the same transaction. It is possible that the call never gets executed if a prior call fails permanently. This might be caused intentionally by a malicious callee. If possible, refactor the code such that each transaction only executes one external call or make sure that all callees can be trusted (i.e. they’re part of your own codebase).
--------------------
Initial State:

Account: [ATTACKER], balance: 0x0, nonce:0, storage:{}
Account: [SOMEGUY], balance: 0x0, nonce:0, storage:{}

Transaction Sequence:

Caller: [SOMEGUY], function: mint(address,uint256), txdata: 0x40c10f190000000000000000000000004125cd1f826099a4dead6b7746f7f28b30d8402b0000000000000000000000000000000000000000000000000000000000000000, decoded_data: ('0x4125cd1f826099a4dead6b7746f7f28b30d8402b', 0), value: 0x0

==== Multiple Calls in a Single Transaction ====
SWC ID: 113
Severity: Low
Contract: 0x4125cD1F826099A4DEaD6b7746F7F28B30d8402B
Function name: mint(address,uint256)
PC address: 1542
Estimated Gas Usage: 5454 - 109524
Multiple calls are executed in the same transaction.
This call is executed following another call within the same transaction. It is possible that the call never gets executed if a prior call fails permanently. This might be caused intentionally by a malicious callee. If possible, refactor the code such that each transaction only executes one external call or make sure that all callees can be trusted (i.e. they’re part of your own codebase).
--------------------
Initial State:

Account: [ATTACKER], balance: 0x0, nonce:0, storage:{}
Account: [SOMEGUY], balance: 0x0, nonce:0, storage:{}

Transaction Sequence:

Caller: [SOMEGUY], function: mint(address,uint256), txdata: 0x40c10f1900000000000000000000000000000000000000000000000000000000000000010000000000000000000000000000000000000000000000000000000000000000, decoded_data: ('0x0000000000000000000000000000000000000001', 0), value: 0x0

==== Multiple Calls in a Single Transaction ====
SWC ID: 113
Severity: Low
Contract: 0x4125cD1F826099A4DEaD6b7746F7F28B30d8402B
Function name: burn(uint256) or collate_propagate_storage(bytes16)
PC address: 1813
Estimated Gas Usage: 3762 - 73080
Multiple calls are executed in the same transaction.
This call is executed following another call within the same transaction. It is possible that the call never gets executed if a prior call fails permanently. This might be caused intentionally by a malicious callee. If possible, refactor the code such that each transaction only executes one external call or make sure that all callees can be trusted (i.e. they’re part of your own codebase).
--------------------
Initial State:

Account: [ATTACKER], balance: 0x0, nonce:0, storage:{}
Account: [SOMEGUY], balance: 0x0, nonce:0, storage:{}

Transaction Sequence:

Caller: [CREATOR], function: burn(uint256), txdata: 0x42966c680000000000000000000000000000000000000000000000000000000000000000, decoded_data: (0,), value: 0x0

==== Multiple Calls in a Single Transaction ====
SWC ID: 113
Severity: Low
Contract: 0x4125cD1F826099A4DEaD6b7746F7F28B30d8402B
Function name: burn(address,uint256)
PC address: 1813
Estimated Gas Usage: 3918 - 73236
Multiple calls are executed in the same transaction.
This call is executed following another call within the same transaction. It is possible that the call never gets executed if a prior call fails permanently. This might be caused intentionally by a malicious callee. If possible, refactor the code such that each transaction only executes one external call or make sure that all callees can be trusted (i.e. they’re part of your own codebase).
--------------------
Initial State:

Account: [ATTACKER], balance: 0x0, nonce:0, storage:{}
Account: [SOMEGUY], balance: 0x0, nonce:0, storage:{}

Transaction Sequence:

Caller: [CREATOR], function: burn(address,uint256), txdata: 0x9dc29fac0000000000000000000000004125cd1f826099a4dead6b7746f7f28b30d8402b0000000000000000000000000000000000000000000000000000000000000000, decoded_data: ('0x4125cd1f826099a4dead6b7746f7f28b30d8402b', 0), value: 0x0

==== Dependence on predictable environment variable ====
SWC ID: 116
Severity: Low
Contract: 0x7a787023f6e18f979b143c79885323a24709b0d8
Function name: mint(address,uint256)
PC address: 2695
Estimated Gas Usage: 2584 - 3059
A control flow decision is made based on The block.timestamp environment variable.
The block.timestamp environment variable is used to determine a control flow decision. Note that the values of variables like coinbase, gaslimit, block number and timestamp are predictable and can be manipulated by a malicious miner. Also keep in mind that attackers know hashes of earlier blocks. Don't use any of those environment variables as sources of randomness and be aware that use of these variables introduces a certain level of trust into miners.
--------------------
Initial State:

Account: [ATTACKER], balance: 0x0, nonce:0, storage:{}
Account: [SOMEGUY], balance: 0x0, nonce:0, storage:{}

Transaction Sequence:

Caller: [ATTACKER], function: mint(uint256), txdata: 0xa0712d680000000000000000000000000000000000000000000000000000000000000000, decoded_data: (0,), value: 0x0

==== Dependence on predictable environment variable ====
SWC ID: 116
Severity: Low
Contract: 0x7a787023f6e18f979b143c79885323a24709b0d8
Function name: mint(address,uint256)
PC address: 2807
Estimated Gas Usage: 2601 - 3076
A control flow decision is made based on The block.timestamp environment variable.
The block.timestamp environment variable is used to determine a control flow decision. Note that the values of variables like coinbase, gaslimit, block number and timestamp are predictable and can be manipulated by a malicious miner. Also keep in mind that attackers know hashes of earlier blocks. Don't use any of those environment variables as sources of randomness and be aware that use of these variables introduces a certain level of trust into miners.
--------------------
Initial State:

Account: [ATTACKER], balance: 0x0, nonce:0, storage:{}
Account: [SOMEGUY], balance: 0x0, nonce:0, storage:{}

Transaction Sequence:

Caller: [CREATOR], function: mint(uint256), txdata: 0xa0712d680000000000000000000000000000000000000000000000000000000000000000, decoded_data: (0,), value: 0x0

==== Dependence on predictable environment variable ====
SWC ID: 116
Severity: Low
Contract: 0x7a787023f6e18f979b143c79885323a24709b0d8
Function name: burn(address,uint256)
PC address: 4214
Estimated Gas Usage: 2539 - 3014
A control flow decision is made based on The block.timestamp environment variable.
The block.timestamp environment variable is used to determine a control flow decision. Note that the values of variables like coinbase, gaslimit, block number and timestamp are predictable and can be manipulated by a malicious miner. Also keep in mind that attackers know hashes of earlier blocks. Don't use any of those environment variables as sources of randomness and be aware that use of these variables introduces a certain level of trust into miners.
--------------------
Initial State:

Account: [ATTACKER], balance: 0x0, nonce:0, storage:{}
Account: [SOMEGUY], balance: 0x0, nonce:0, storage:{}

Transaction Sequence:

Caller: [CREATOR], function: burn(uint256), txdata: 0x42966c686868686868686868686868686868686868686868686868686868686868686868, decoded_data: (47225008943846605192358362513347225163686581981280857490602308771854766598248,), value: 0x0

==== Dependence on predictable environment variable ====
SWC ID: 116
Severity: Low
Contract: 0x7a787023f6e18f979b143c79885323a24709b0d8
Function name: burn(address,uint256)
PC address: 4326
Estimated Gas Usage: 2556 - 3031
A control flow decision is made based on The block.timestamp environment variable.
The block.timestamp environment variable is used to determine a control flow decision. Note that the values of variables like coinbase, gaslimit, block number and timestamp are predictable and can be manipulated by a malicious miner. Also keep in mind that attackers know hashes of earlier blocks. Don't use any of those environment variables as sources of randomness and be aware that use of these variables introduces a certain level of trust into miners.
--------------------
Initial State:

Account: [ATTACKER], balance: 0x0, nonce:0, storage:{}
Account: [SOMEGUY], balance: 0x0, nonce:0, storage:{}

Transaction Sequence:

Caller: [CREATOR], function: burn(uint256), txdata: 0x42966c686868686868686868686868686868686868686868686868686868686868686868, decoded_data: (47225008943846605192358362513347225163686581981280857490602308771854766598248,), value: 0x0

==== Exception State ====
SWC ID: 110
Severity: Medium
Contract: 0xb4272071ecadd69d933adcd19ca99fe80664fc08
Function name: transfer(address,uint256) or many_msg_babbage(bytes1)
PC address: 8862
Estimated Gas Usage: 3901 - 6026
An assertion violation was triggered.
It is possible to trigger an assertion violation. Note that Solidity assert() statements should only be used to check invariants. Review the transaction trace generated for this issue and either make sure your program logic is correct, or use require() instead of assert() if your goal is to constrain user inputs or enforce preconditions. Remember to validate inputs from both callers (for instance, via passed arguments) and callees (for instance, via return values).
--------------------
Initial State:

Account: [ATTACKER], balance: 0x0, nonce:0, storage:{}
Account: [SOMEGUY], balance: 0x0, nonce:0, storage:{}

Transaction Sequence:

Caller: [SOMEGUY], function: burn(uint256), txdata: 0x42966c680101010101010101010101010101010101010101010101010101010101010101, decoded_data: (454086624460063511464984254936031011189294057512315937409637584344757371137,), value: 0x0
```

## ERC compliance check report (Slither)

```
# Check Frankencoin

## Check functions
[✓] totalSupply() is present
	[✓] totalSupply() -> (uint256) (correct return type)
	[✓] totalSupply() is view
[✓] balanceOf(address) is present
	[✓] balanceOf(address) -> (uint256) (correct return type)
	[✓] balanceOf(address) is view
[✓] transfer(address,uint256) is present
	[✓] transfer(address,uint256) -> (bool) (correct return type)
	[✓] Transfer(address,address,uint256) is emitted
[✓] transferFrom(address,address,uint256) is present
	[✓] transferFrom(address,address,uint256) -> (bool) (correct return type)
	[✓] Transfer(address,address,uint256) is emitted
[✓] approve(address,uint256) is present
	[✓] approve(address,uint256) -> (bool) (correct return type)
	[✓] Approval(address,address,uint256) is emitted
[✓] allowance(address,address) is present
	[✓] allowance(address,address) -> (uint256) (correct return type)
	[✓] allowance(address,address) is view
[✓] name() is present
	[✓] name() -> (string) (correct return type)
	[✓] name() is view
[✓] symbol() is present
	[✓] symbol() -> (string) (correct return type)
	[✓] symbol() is view
[✓] decimals() is present
	[✓] decimals() -> (uint8) (correct return type)
	[✓] decimals() is view

## Check events
[✓] Transfer(address,address,uint256) is present
	[✓] parameter 0 is indexed
	[✓] parameter 1 is indexed
[✓] Approval(address,address,uint256) is present
	[✓] parameter 0 is indexed
	[✓] parameter 1 is indexed


	[ ] Frankencoin is not protected for the ERC20 approval race condition
```

## Slither report

```
Equity.redeem(address,uint256) (../../share/contracts/Equity.sol#275-282) ignores return value by zchf.transfer(target,proceeds) (../../share/contracts/Equity.sol#279)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#unchecked-transfer

Frankencoin.calculateAssignedReserve(uint256,uint32) (../../share/contracts/Frankencoin.sol#204-213) performs a multiplication on the result of a division:
        -theoreticalReserve = _reservePPM * mintedAmount / 1000000 (../../share/contracts/Frankencoin.sol#205)
        -theoreticalReserve * currentReserve / minterReserve() (../../share/contracts/Frankencoin.sol#209)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#divide-before-multiply

Frankencoin.mint(address,uint256,uint32,uint32) (../../share/contracts/Frankencoin.sol#165-170) should emit an event for: 
        - minterReserveE6 += _amount * _reservePPM (../../share/contracts/Frankencoin.sol#169) 
Frankencoin.burn(uint256,uint32) (../../share/contracts/Frankencoin.sol#194-197) should emit an event for: 
        - minterReserveE6 -= amount * reservePPM (../../share/contracts/Frankencoin.sol#196) 
Frankencoin.burnFrom(address,uint256,uint32) (../../share/contracts/Frankencoin.sol#223-229) should emit an event for: 
        - minterReserveE6 -= targetTotalBurnAmount * _reservePPM (../../share/contracts/Frankencoin.sol#227) 
Frankencoin.burnWithReserve(uint256,uint32) (../../share/contracts/Frankencoin.sol#251-257) should emit an event for: 
        - minterReserveE6 -= freedAmount * _reservePPM (../../share/contracts/Frankencoin.sol#253) 
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#missing-events-arithmetic

Reentrancy in Equity.redeem(address,uint256) (../../share/contracts/Equity.sol#275-282):
        External calls:
        - zchf.transfer(target,proceeds) (../../share/contracts/Equity.sol#279)
        Event emitted after the call(s):
        - Trade(msg.sender,- int256(shares),proceeds,price()) (../../share/contracts/Equity.sol#280)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#reentrancy-vulnerabilities-3

ERC20PermitLight.permit(address,address,uint256,uint256,uint8,bytes32,bytes32) (../../share/contracts/ERC20PermitLight.sol#21-59) uses timestamp for comparisons
        Dangerous comparisons:
        - require(bool,string)(deadline >= block.timestamp,PERMIT_DEADLINE_EXPIRED) (../../share/contracts/ERC20PermitLight.sol#30)
Frankencoin.allowanceInternal(address,address) (../../share/contracts/Frankencoin.sol#102-111) uses timestamp for comparisons
        Dangerous comparisons:
        - isMinter(spender) || isMinter(isPosition(spender)) (../../share/contracts/Frankencoin.sol#106)
Frankencoin.denyMinter(address,address[],string) (../../share/contracts/Frankencoin.sol#152-157) uses timestamp for comparisons
        Dangerous comparisons:
        - block.timestamp > minters[_minter] (../../share/contracts/Frankencoin.sol#153)
Frankencoin.isMinter(address) (../../share/contracts/Frankencoin.sol#293-295) uses timestamp for comparisons
        Dangerous comparisons:
        - minters[_minter] != 0 && block.timestamp >= minters[_minter] (../../share/contracts/Frankencoin.sol#294)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#block-timestamp

Different versions of Solidity are used:
        - Version used: ['>=0.8.0<0.9.0', '^0.8.0']
        - ^0.8.0 (../../share/contracts/ERC20.sol#12)
        - ^0.8.0 (../../share/contracts/ERC20PermitLight.sol#5)
        - >=0.8.0<0.9.0 (../../share/contracts/Equity.sol#4)
        - ^0.8.0 (../../share/contracts/Frankencoin.sol#2)
        - ^0.8.0 (../../share/contracts/IERC20.sol#7)
        - ^0.8.0 (../../share/contracts/IERC677Receiver.sol#2)
        - ^0.8.0 (../../share/contracts/IFrankencoin.sol#2)
        - ^0.8.0 (../../share/contracts/IReserve.sol#2)
        - >=0.8.0<0.9.0 (../../share/contracts/MathUtil.sol#3)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#different-pragma-directives-are-used

Equity.adjustTotalVotes(address,uint256,uint256) (../../share/contracts/Equity.sol#144-148) has costly operations inside a loop:
        - totalVotesAtAnchor = uint192(totalVotes() - roundingLoss - lostVotes) (../../share/contracts/Equity.sol#146)
Equity.adjustTotalVotes(address,uint256,uint256) (../../share/contracts/Equity.sol#144-148) has costly operations inside a loop:
        - totalVotesAnchorTime = anchorTime() (../../share/contracts/Equity.sol#147)
ERC20._burn(address,uint256) (../../share/contracts/ERC20.sol#200-206) has costly operations inside a loop:
        - _totalSupply -= amount (../../share/contracts/ERC20.sol#203)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#costly-operations-inside-a-loop

Pragma version^0.8.0 (../../share/contracts/ERC20.sol#12) allows old versions
Pragma version^0.8.0 (../../share/contracts/ERC20PermitLight.sol#5) allows old versions
Pragma version>=0.8.0<0.9.0 (../../share/contracts/Equity.sol#4) is too complex
Pragma version^0.8.0 (../../share/contracts/Frankencoin.sol#2) allows old versions
Pragma version^0.8.0 (../../share/contracts/IERC20.sol#7) allows old versions
Pragma version^0.8.0 (../../share/contracts/IERC677Receiver.sol#2) allows old versions
Pragma version^0.8.0 (../../share/contracts/IFrankencoin.sol#2) allows old versions
Pragma version^0.8.0 (../../share/contracts/IReserve.sol#2) allows old versions
Pragma version>=0.8.0<0.9.0 (../../share/contracts/MathUtil.sol#3) is too complex
solc-0.8.9 is not recommended for deployment
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#incorrect-versions-of-solidity

Equity (../../share/contracts/Equity.sol#20-319) should inherit from IERC677Receiver (../../share/contracts/IERC677Receiver.sol#4-9)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#missing-inheritance

Function ERC20PermitLight.DOMAIN_SEPARATOR() (../../share/contracts/ERC20PermitLight.sol#61-71) is not in mixedCase
Parameter Frankencoin.suggestMinter(address,uint256,uint256,string)._minter (../../share/contracts/Frankencoin.sol#83) is not in mixedCase
Parameter Frankencoin.suggestMinter(address,uint256,uint256,string)._applicationPeriod (../../share/contracts/Frankencoin.sol#83) is not in mixedCase
Parameter Frankencoin.suggestMinter(address,uint256,uint256,string)._applicationFee (../../share/contracts/Frankencoin.sol#83) is not in mixedCase
Parameter Frankencoin.suggestMinter(address,uint256,uint256,string)._message (../../share/contracts/Frankencoin.sol#83) is not in mixedCase
Parameter Frankencoin.registerPosition(address)._position (../../share/contracts/Frankencoin.sol#125) is not in mixedCase
Parameter Frankencoin.denyMinter(address,address[],string)._minter (../../share/contracts/Frankencoin.sol#152) is not in mixedCase
Parameter Frankencoin.denyMinter(address,address[],string)._helpers (../../share/contracts/Frankencoin.sol#152) is not in mixedCase
Parameter Frankencoin.denyMinter(address,address[],string)._message (../../share/contracts/Frankencoin.sol#152) is not in mixedCase
Parameter Frankencoin.mint(address,uint256,uint32,uint32)._target (../../share/contracts/Frankencoin.sol#165) is not in mixedCase
Parameter Frankencoin.mint(address,uint256,uint32,uint32)._amount (../../share/contracts/Frankencoin.sol#165) is not in mixedCase
Parameter Frankencoin.mint(address,uint256,uint32,uint32)._reservePPM (../../share/contracts/Frankencoin.sol#165) is not in mixedCase
Parameter Frankencoin.mint(address,uint256,uint32,uint32)._feesPPM (../../share/contracts/Frankencoin.sol#165) is not in mixedCase
Parameter Frankencoin.mint(address,uint256)._target (../../share/contracts/Frankencoin.sol#172) is not in mixedCase
Parameter Frankencoin.mint(address,uint256)._amount (../../share/contracts/Frankencoin.sol#172) is not in mixedCase
Parameter Frankencoin.burn(uint256)._amount (../../share/contracts/Frankencoin.sol#179) is not in mixedCase
Parameter Frankencoin.calculateAssignedReserve(uint256,uint32)._reservePPM (../../share/contracts/Frankencoin.sol#204) is not in mixedCase
Parameter Frankencoin.burnFrom(address,uint256,uint32)._reservePPM (../../share/contracts/Frankencoin.sol#223) is not in mixedCase
Parameter Frankencoin.burnWithReserve(uint256,uint32)._amountExcludingReserve (../../share/contracts/Frankencoin.sol#251) is not in mixedCase
Parameter Frankencoin.burnWithReserve(uint256,uint32)._reservePPM (../../share/contracts/Frankencoin.sol#251) is not in mixedCase
Parameter Frankencoin.burn(address,uint256)._owner (../../share/contracts/Frankencoin.sol#262) is not in mixedCase
Parameter Frankencoin.burn(address,uint256)._amount (../../share/contracts/Frankencoin.sol#262) is not in mixedCase
Parameter Frankencoin.notifyLoss(uint256)._amount (../../share/contracts/Frankencoin.sol#280) is not in mixedCase
Parameter Frankencoin.isMinter(address)._minter (../../share/contracts/Frankencoin.sol#293) is not in mixedCase
Parameter Frankencoin.isPosition(address)._position (../../share/contracts/Frankencoin.sol#300) is not in mixedCase
Variable Frankencoin.MIN_APPLICATION_PERIOD (../../share/contracts/Frankencoin.sol#26) is not in mixedCase
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#conformance-to-solidity-naming-conventions

Variable IFrankencoin.burnWithReserve(uint256,uint32).amountExcludingReserve (../../share/contracts/IFrankencoin.sol#36) is too similar to IFrankencoin.burn(uint256,uint32).amountIncludingReserve (../../share/contracts/IFrankencoin.sol#32)
Variable Frankencoin.calculateFreedAmount(uint256,uint32).amountExcludingReserve (../../share/contracts/Frankencoin.sol#235) is too similar to IFrankencoin.burn(uint256,uint32).amountIncludingReserve (../../share/contracts/IFrankencoin.sol#32)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#variable-names-are-too-similar

Equity.slitherConstructorConstantVariables() (../../share/contracts/Equity.sol#20-319) uses literals with too many digits:
        - THRESH_DEC18 = 10000000000000000 (../../share/contracts/MathUtil.sol#11)
Frankencoin.minterReserve() (../../share/contracts/Frankencoin.sol#117-119) uses literals with too many digits:
        - minterReserveE6 / 1000000 (../../share/contracts/Frankencoin.sol#118)
Frankencoin.calculateAssignedReserve(uint256,uint32) (../../share/contracts/Frankencoin.sol#204-213) uses literals with too many digits:
        - theoreticalReserve = _reservePPM * mintedAmount / 1000000 (../../share/contracts/Frankencoin.sol#205)
Frankencoin.calculateFreedAmount(uint256,uint32) (../../share/contracts/Frankencoin.sol#235-240) uses literals with too many digits:
        - 1000000 * amountExcludingReserve / (1000000 - adjustedReservePPM) (../../share/contracts/Frankencoin.sol#239)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#too-many-digits

permit(address,address,uint256,uint256,uint8,bytes32,bytes32) should be declared external:
        - ERC20PermitLight.permit(address,address,uint256,uint256,uint8,bytes32,bytes32) (../../share/contracts/ERC20PermitLight.sol#21-59)
calculateShares(uint256) should be declared external:
        - Equity.calculateShares(uint256) (../../share/contracts/Equity.sol#262-264)
redeem(address,uint256) should be declared external:
        - Equity.redeem(address,uint256) (../../share/contracts/Equity.sol#275-282)
restructureCapTable(address[],address[]) should be declared external:
        - Equity.restructureCapTable(address[],address[]) (../../share/contracts/Equity.sol#309-316)
equity() should be declared external:
        - Frankencoin.equity() (../../share/contracts/Frankencoin.sol#138-146)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#public-function-that-could-be-declared-external
/share/contracts/Frankencoin.sol analyzed (9 contracts with 78 detectors), 63 result(s) found
```
