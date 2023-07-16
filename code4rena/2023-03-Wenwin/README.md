# Code4rena - Wenwin Audit

Contest: https://code4rena.com/contests/2023-03-wenwin-contest

Full Report: https://github.com/cryptostaker2/blockchain-security-audits/blob/main/code4rena/2023-03-Wenwin/Report.md

## Findings Summary

| ID   | Title                                                                                        | Severity |
| :--- | :------------------------------------------------------------------------------------------- | :------- |
| M-01 | Owner of winning NFT ticket can frontrun buyer on secondary NFT market when claiming rewards | Medium   |
| L-01 | First draw schedule should not accept a time in the past                                     | Low      |
| L-02 | Avoid Division before Multiplication                                                         | Low      |
| L-03 | Check for address(0) or amount == 0 in the LotterySetupParams constructor                    | Low      |
| L-04 | Unbounded Loop when buying an excessive amount of tickets                                    | Low      |
| L-05 | Use safeMint() instead of mint() for ERC721 tokens                                           | Info     |
| L-06 | Frontend address should check for address(0)                                                 | Info     |
| N-01 | Use require instead of assert                                                                | Info     |
| N-02 | All functions should have NATSPEC comments in the interface file                             | Info     |
| N-03 | Critical functions should emit an event                                                      | Info     |
| N-04 | Increase Code Coverage                                                                       | Info     |
| N-05 | For modern and more readable code; update import usages                                      | Info     |
| N-06 | Lock pragma version                                                                          | Info     |
| N-07 | Standardize pragma version                                                                   | Info     |
| N-08 | Function writing that does not comply with the Solidity Style Guide                          | Info     |

