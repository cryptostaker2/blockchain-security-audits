# Code4rena - Y2K Finance Audit

Contest: https://code4rena.com/contests/2022-09-y2k-finance-contest

Full Report: https://github.com/cryptostaker2/blockchain-security-audits/blob/main/code4rena/2022-09-Y2k-Finance/Report.md

Duration: 15 September 2022 - 20 September 2022

## Findings Summary

|  ID  | Title                                                                        | Severity |
| :--: | :--------------------------------------------------------------------------- | :------- |
| G-01 | ++I costs less gas as compared to I++ or I+= 1                               | Gas      |
| G-02 | Increments can be unchecked                                                  | Gas      |
| G-03 | X += Y cost more gas than X = X + Y for state variables                      | Gas      |
| G-04 | Using private rather than public for constants saves gas                     | Gas      |
| G-05 | Increments in for loop can be unchecked (saves 30-40 gas per loop iteration) | Gas      |
| G-06 | Cache Storage values in memory to minimize sloads                            | Gas      |

