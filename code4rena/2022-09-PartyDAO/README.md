# Code4rena - PartyDAO Protocol Audit

Contest: https://code4rena.com/contests/2022-09-partydao-contest

Full Report: https://github.com/cryptostaker2/blockchain-security-audits/blob/main/code4rena/2022-09-PartyDAO/Report.md

Duration: 13 September 2022 - 20 September 2022

## Findings Summary

|  ID  | Title                                                         | Severity |
| :--: | :------------------------------------------------------------ | :------- |
| G-01 | No need to explicitly initialize variables with default value | Gas      |
| G-02 | Using abi.encode() is less efficient than abi.encodePacked()  | Gas      |

