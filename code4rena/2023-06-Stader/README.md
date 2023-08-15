# Code4rena - Stader Audit

Contest: https://code4rena.com/contests/2023-05-venus-protocol-isolated-pools

Full Report: https://github.com/cryptostaker2/blockchain-security-audits/blob/main/code4rena/2023-06-Stader/Report.md

Duration: 3 June 2023 - 10 June 2023

## Findings Summary

| ID   | Title                                                                                                              | Severity |
| :--- | :----------------------------------------------------------------------------------------------------------------- | :------- |
| M-01 | The winning bid of the auction in Auction.sol can be frontrunned or the last bidder can block-stuff to win the bid | Medium   |
| M-02 | StaderOracle#latestRoundData() may return a stale price                                                            | Medium   |
