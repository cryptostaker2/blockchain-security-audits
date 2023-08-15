# Code4rena - Asymmetry Audit

Contest: https://code4rena.com/contests/2023-03-asymmetry-contest

Full Report: https://github.com/cryptostaker2/blockchain-security-audits/blob/main/code4rena/2023-03-Asymmetry/Report.md

Duration: 25 March 2023 - 31 March 2023

## Findings Summary

| ID   | Title                                                                                                                     | Severity |
| :--- | :------------------------------------------------------------------------------------------------------------------------ | :------- |
| H-01 | Assuming 1:1 peg between liquid staking derivatives and ETH may skew calculations                                         | High     |
| H-02 | Reth.sol uses slot0 to determine the price of Reth, which makes it easily to manipulate the price                         | High     |
| M-01 | Uniswap ExactInputSingleParams in Reth.sol lacks a deadline parameter                                                     | Medium   |
| M-02 | Frax minting can be paused, which may brick deposits                                                                      | Medium   |
| M-03 | Swapping in Reth.sol may be sub-optimal                                                                                   | Medium   |
| L-01 | Use Ownable2StepUpgradeable instead of OwnableUpgradeable contract                                                        | Low      |
| L-02 | Max Slippage should have a maximum cap to prevent users from losing value when depositing / withdrawing their derivatives | Low      |
| L-03 | Make sure minAmount is lesser than maxAmount and vice versa when changing the values                                      | Low      |
| L-04 | Unbounded loops may caus DoS if there is too many derivatives in the protocol                                             | Low      |
| L-05 | Derivatives cannot be deleted from the contract                                                                           | Low      |
| L-06 | Solidity version used (0.8.13) has known issues                                                                           | Low      |
| L-07 | Hardcoded fee can be refactored                                                                                           | Low      |
| L-08 | Division before multiplication should be avoided                                                                          | Low      |
| N-01 | Non-library/interface files should use fixed compiler versions, not floating ones                                         | Info     |
| N-02 | abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()   | Info     |
| N-03 | For modern and more readable code; update import usages                                                                   | Info     |
| N-04 | Zero values should be checked for best practice                                                                           | Info     |
