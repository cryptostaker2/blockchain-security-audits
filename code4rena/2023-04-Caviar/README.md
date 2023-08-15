# Code4rena - Caviar Audit

Contest: https://code4rena.com/contests/2023-04-caviar-private-pools

Full Report: https://github.com/cryptostaker2/blockchain-security-audits/blob/main/code4rena/2023-04-Caviar/Report.md

Duration: 8 April 2023 - 14 April 2023

## Findings Summary

| ID   | Title                                                                                         | Severity |
| :--- | :-------------------------------------------------------------------------------------------- | :------- |
| M-01 | PrivatePool.sol#flashLoan() is problematic for ERC20 tokens that don't support zero transfers | Medium   |
