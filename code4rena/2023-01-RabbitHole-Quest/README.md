# Code4rena - RabbitHole Quest Protocol Audit

Contest: https://code4rena.com/contests/2023-01-rabbithole-quest-protocol-contest

Full Report: https://github.com/cryptostaker2/blockchain-security-audits/blob/main/code4rena/2023-01-RabbitHole-Quest/Report.md

Duration: 26 January 2023 - 31 January 2023

## Findings Summary

| ID   | Title                                                                                                                             | Severity |
| :--- | :-------------------------------------------------------------------------------------------------------------------------------- | :------- |
| H-01 | Erc20Quest#withdrawFee() can be called multiple times                                                                             | High     |
| M-01 | QuestFactory#mintReceipt() does not adhere to endTime\_                                                                           | Medium   |
| M-02 | Successful participants who minted a ticket but didn't claim yet cannot do so if Erc1155Quest#withdrawRemainingTokens() is called | Medium   |
| M-03 | Erc20Quest#withdrawRemainingTokens() may count remaining tokens incorrectly if withdrawFee() is called first.                     | Medium   |
| L-01 | Use safetransferownership instead of transferownership function.                                                                  | Low      |
| L-02 | Sanity checks in QuestFactory.sol                                                                                                 | Low      |
| L-03 | Confusing pausing function application                                                                                            | Low      |
| N-01 | Non-library/interface files should use fixed compiler versions, not floating ones                                                 | Info     |
| N-02 | Use bytes.concat() instead of abi.encodePacked()                                                                                  | Info     |
| N-03 | Update Import Usages for more readable and modern code                                                                            | Info     |
| N-04 | Confusing variable name                                                                                                           | Info     |
| N-05 | Improve test coverage                                                                                                             | Info     |
| N-06 | Function writing does not comply with the solidity style guide                                                                    | Info     |

