# Code4rena - Venus Audit

Contest: https://code4rena.com/contests/2023-05-venus-protocol-isolated-pools

Full Report: https://github.com/cryptostaker2/blockchain-security-audits/blob/main/code4rena/2023-05-Venus/Report.md

Duration: 9 May 2023 - 16 May 2023

## Findings Summary

| ID   | Title                                                                                                        | Severity |
| :--- | :----------------------------------------------------------------------------------------------------------- | :------- |
| M-01 | priceOracle.updatePrice is not called before priceOracle.getUnderlyingPrice in PoolLens#getPoolBadDebt()     | Medium   |
| M-02 | Bad Debt in PoolLens.sol#getPoolBadDebt() is not calculated correctly in USD                                 | Medium   |
| M-03 | Bad debt calculation may be wrong when calling Shortfall#\_startAuction() if VTokens have different decimals | Medium   |
