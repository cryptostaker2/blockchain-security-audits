# Code4rena - Biconomy Audit

Contest: https://code4rena.com/contests/2023-01-biconomy-smart-contract-wallet-contest

Full Report: https://github.com/cryptostaker2/blockchain-security-audits/blob/main/code4rena/2023-01-Biconomy/Report.md

Duration: 5 January 2023 - 10 January 2023

## Findings Summary

|  ID  | Title                                                                                                                                       | Severity |
| :--: | :------------------------------------------------------------------------------------------------------------------------------------------ | :------- |
| M-01 | handleAggregatedOps() does not handle non-atomic transactions which results in whole function revert if one transaction does not go through | Medium   |
| M-02 | EntryPoint address in SmartAccount.sol cannot execute certain functions because of the onlyOwner modifier                                   | Medium   |
| M-03 | Protocol has upgradeable contracts that cannot be upgraded                                                                                  | Medium   |
| L-01 | Initializers can be frontrunnable                                                                                                           | Low      |
| L-02 | Contract should adhere to two-step ownership process                                                                                        | Low      |
| L-03 | Unused/Empty receive()/fallback() function                                                                                                  | Low      |
| L-04 | OpenZeppelin's ECDSA is imported but not utilized                                                                                           | Low      |
| L-05 | Import exact functions instead of the whole contract itself                                                                                 | Low      |
| N-01 | Non-library/interface files should use fixed compiler versions, not floating ones                                                           | Info     |
| N-02 | Avoid the use of sensitive terms in favour of neutral ones                                                                                  | Info     |
| N-03 | Testing coverage is not adequate enough                                                                                                     | Info     |

