# Code4rena - Holograph Audit

Contest: https://code4rena.com/contests/2022-10-holograph-contest

Full Report: https://github.com/cryptostaker2/blockchain-security-audits/blob/main/code4rena/2022-10-Holograph/Report.md

Duration: 19 October 2022 - 26 October 2022

## Findings Summary

| ID   | Title                                                                                                             | Severity |
| :--- | :---------------------------------------------------------------------------------------------------------------- | :------- |
| M-01 | Wrong slashing calculation rewards for operator that did not do his job                                           | Medium   |
| L-01 | Burn function is not implemented in unbondUtilityToken() when operator fails to do his job                        | Low      |
| L-02 | ++I costs less gas as compared to I++ or I+= 1                                                                    | Low      |
| N-01 | Some function parameters should be renamed for clarity                                                            | Info     |
| N-02 | Transferring of ownership should be a two-step process                                                            | Info     |
| N-03 | Best practice is to prevent signature malleability                                                                | Info     |
| G-01 | ++i/i++ should be unchecked when it is not possible to overflow, as is the case when used in for- and while-loops | Gas      |
| G-02 | += costs more gas than = + for state variables                                                                    | Gas      |

