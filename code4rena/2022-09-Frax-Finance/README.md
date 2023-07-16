# Code4rena - Frax Ether Liquid Staking Protocol Audit

Contest: https://code4rena.com/contests/2022-09-frax-ether-liquid-staking-contest

Full Report: https://github.com/cryptostaker2/blockchain-security-audits/tree/main/code4rena/2022-09-Frax-Finance/Report.md

Duration: 23 September 2023 - 26 September 2023

## Findings Summary

|  ID  | Title                                                                                 | Severity |
| :--: | :------------------------------------------------------------------------------------ | :------- |
| G-01 | ++I costs less gas as compared to I++ or I+= 1                                        | Gas      |
| G-02 | Increments can be unchecked                                                           | Gas      |
| G-03 | Store array length in for-loops                                                       | Gas      |
| G-04 | Duplicated require() / revert() checks should be refactored to a modifier or function | Gas      |
| G-05 | Use private rather than public for constants, saves gas                               | Gas      |
| G-06 | Using > 0 costs more gas than != 0 when used on a uint in a require() statement       | Gas      |
| G-07 | x += y costs more gas than x = x + y for state variables                              | Gas      |
| G-08 | Updating solidity version to the latest saves gas                                     | Gas      |

