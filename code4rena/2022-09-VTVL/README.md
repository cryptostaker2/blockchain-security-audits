# Code4rena - VTVL Protocol Audit

Contest: https://code4rena.com/contests/2022-09-vtvl-contest

Full Report: https://github.com/cryptostaker2/blockchain-security-audits/tree/main/code4rena/2022-09-VTVL/Report.md

Duration: 21 September 2022 - 24 September 2022

## Findings Summary

|  ID  | Title                                                                           | Severity |
| :--: | :------------------------------------------------------------------------------ | :------- |
| M-01 | Vouchers and vouchees indices become corrupted by UserManager's cancelVouch     | Medium   |
| L-01 | Admin can revoke admin rights of every other admin                              | Low      |
| G-01 | ++I costs less gas as compared to I++ or I+= 1                                  | Gas      |
| G-02 | Increments can be unchecked                                                     | Gas      |
| G-03 | Splitting require() statements that use && saves gas                            | Gas      |
| G-04 | Using > 0 costs more gas than != 0 when used on a uint in a require() statement | Gas      |

