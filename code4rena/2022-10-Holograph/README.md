# Code4rena - Holograph Audit

Contest: https://code4rena.com/contests/2022-10-holograph-contest

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

# Full Report

## Medium Findings:

### [M-01] Wrong slashing calculation rewards for operator that did not do his job

### Lines of code

https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/HolographOperator.sol#L374-L382

### Impact

Wrong slashing calculation may create unfair punishment for operators that accidentally forgot to execute their job.

### Proof of Concept

Docs: If an operator acts maliciously, a percentage of their bonded HLG will get slashed. Misbehavior includes (i) downtime, (ii) double-signing transactions, and (iii) abusing transaction speeds. 50% of the slashed HLG will be rewarded to the next operator to execute the transaction, and the remaining 50% will be burned or returned to the Treasury.

The docs also include a guide for the number of slashes and the percentage of bond slashed. However, in the contract, there is no slashing of percentage fees. Rather, the whole \_getBaseBondAmount() fee is slashed from the job.operator instead.

```
        uint256 amount = _getBaseBondAmount(pod);
        /**
         * @dev select operator that failed to do the job, is slashed the pod base fee
         */
        _bondedAmounts[job.operator] -= amount;
        /**
         * @dev the slashed amount is sent to current operator
         */
        _bondedAmounts[msg.sender] += amount;
```

Documentation states that only a portion should be slashed and the number of slashes should be noted down.

### Tools Used

Manual Review

### Recommended Mitigation Steps

Implement the correct percentage of slashing and include a mapping to note down the number of slashes that an operator has

## Low & Informational Findings:

### [L-01] Burn function is not implemented in unbondUtilityToken() when operator fails to do his job

In unbondUtilityToken, there is no fees when withdrawing the token as stated in the docs

### [L-02] \_safemint() should be used rather than \_mint() whereever possible in ERC721 contracts

\_mint()is discouraged in favor of \_safeMint() which ensures that the recipient is either an EOA or implements IERC721Receiver. Both OpenZeppelin and solmate have versions of this function.

### [N-01] Some function parameters should be renamed for clarity

The parameter of \_getCurrentBondAmount should be podIndex instead of pod. Same as \_getBaseBondAmount

### [N-02] Transferring of ownership should be a two-step process

Owner.sol

### [N-03] Best practice is to prevent signature malleability

Use OpenZeppelin’s ECDSA contract rather than calling ecrecover() directly. HolographFactory.sol

## Gas Findings:

### [G-01] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop. HolographOperator.sol, HolographRegistry.sol, HolographRegistry.sol

### [G-02] += costs more gas than = + for state variables

Using the addition operator instead of plus-equals saves 113 gas. HolographERC20.sol
