# Code4rena - Ondo Finance Audit

Contest: https://code4rena.com/contests/2023-01-ondo-finance-contest

Full Report: https://github.com/cryptostaker2/blockchain-security-audits/blob/main/code4rena/2023-01-Ondo-Finance/Report.md

Duration: 12 January 2023 - 18 January 2023

## Findings Summary

| ID   | Title                                                                                     | Severity |
| :--- | :---------------------------------------------------------------------------------------- | :------- |
| H-01 | Burn amount that is refunded to users is not updated in CashManager.completeRedemption()  | High     |
| L-01 | nonReentrant modifier should always come first do prevent reentrancy from other modifiers | Low      |
| L-02 | Missing 0 check in Mint and Redeem functions in CashManager.sol                           | Low      |
| L-03 | Event should emit critical parameters                                                     | Low      |
| L-04 | Initializers are front-runnable                                                           | Low      |
| N-01 | Numeric values having to do with time should use time units for readability               | Info     |
| N-02 | Avoid using sensitive terms                                                               | Info     |

