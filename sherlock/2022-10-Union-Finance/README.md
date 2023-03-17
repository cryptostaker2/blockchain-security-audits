# Sherlock - Union Audit

Contest: https://app.sherlock.xyz/audits/contests/11

## Findings Summary

|  ID  | Title                                                                       | Severity |
| :--: | :-------------------------------------------------------------------------- | :------- |
| M-01 | Vouchers and vouchees indices become corrupted by UserManager's cancelVouch | Medium   |

# Full Report

## Medium Findings:

### [M-01] Vouchers that vouches first may not get their stake locked or unlocked sequentially according to updateLocked() if cancelVouch() is called
### Summary
Line 792, UserManager.sol
```

     *  @dev    Locks/Unlocks the borrowers stakers staked amounts in a first in
     *          First out order. Meaning the members that vouched for this borrower
     *          first will be the first members to get their stake locked or unlocked
     *          following a borrow or repayment.
```
When cancelVouch() is called, the ordering of vouchers for a borrower becomes messed up.

### Vulnerability Detail
1. There are 5 vouchers for a borrower
2. Borrower decides to remove the first voucher by calling cancelVouch()
3. Voucher 1 index is replaced by voucher 5 and voucher 5's index is popped off.
4. Now, voucher 5 is the first in line to receive rewards.
## Impact
It will be unfair for early vouchers as they will not get their stake locked or unlock first.

### Code Snippet
https://github.com/sherlock-audit/2022-10-union-finance/blob/main/union-v2-contracts/contracts/user/UserManager.sol#L577-L591

https://github.com/sherlock-audit/2022-10-union-finance/blob/main/union-v2-contracts/contracts/user/UserManager.sol#L800-L809

### Tool used
Manual Review

### Recommendation
Have a mapping for vouchers' order to make sure rewards are received sequentially.