# Code4rena - Debt DAO Audit

Contest: https://code4rena.com/contests/2022-11-debt-dao-contest

Full Report: https://github.com/cryptostaker2/blockchain-security-audits/blob/main/code4rena/2022-11-Debt-DAO/Report.md

Duration: 4 November 2022 - 11 November 2022

## Findings Summary

| ID   | Title                                                                                                             | Severity |
| :--- | :---------------------------------------------------------------------------------------------------------------- | :------- |
| M-01 | LineLib.sol uses payable().transfer, which may lead to denial of service                                          | Medium   |
| L-01 | Best Practices to prevent Re-Entrancy                                                                             | Low      |
| N-01 | Avoid using sensitive terms                                                                                       | Info     |
| N-02 | Constants should be defined rather than using magic numbers                                                       | Info     |
| N-03 | if( should be if ( to match other lines in the file                                                               | Info     |
| N-04 | Typos                                                                                                             | Info     |
| N-05 | Unused/Empty receive()/fallback() function                                                                        | Info     |
| G-01 | ++i/i++ should be unchecked when it is not possible to overflow, as is the case when used in for- and while-loops | Gas      |
| G-02 | Use a more recent version of solidity                                                                             | Gas      |
| G-03 | Using storage instead of memory for structs/arrays saves gas                                                      | Gas      |
| G-04 | Redundant Operation                                                                                               | Gas      |

