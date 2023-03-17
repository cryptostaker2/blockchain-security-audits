# Full Report

## High Findings:

### [H-01] Burn amount that is refunded to users is not updated in CashManager.completeRedemption()

### Lines of code
https://github.com/code-423n4/2023-01-ondo/blob/f3426e5b6b4561e09460b2e6471eb694efdd6c70/contracts/cash/CashManager.sol#L707-L727
https://github.com/code-423n4/2023-01-ondo/blob/f3426e5b6b4561e09460b2e6471eb694efdd6c70/contracts/cash/CashManager.sol#L755-L757

### Impact
Users will get less USDC compensation than intended when swapping CASH if completeRedemptions() is called more than once should there be any refunds.

### Proof of Concept
The function completeRedemptions() in CashManager.sol is called by the admin to ensure that users can get back USDC by exchanging their CASH. In the rare occasion that CASH needs to be refunded to a user, a refund function _processRefund() is executed.

In _processRefund(), users that called requestRedemption() to burn their CASH tokens is refunded back the full amount.
```
  function _processRefund(
    address[] calldata refundees,
    uint256 epochToService
  ) private returns (uint256 totalCashAmountRefunded) {
    uint256 size = refundees.length;
    for (uint256 i = 0; i < size; ++i) {
      address refundee = refundees[i];
      uint256 cashAmountBurned = redemptionInfoPerEpoch[epochToService]
        .addressToBurnAmt[refundee];
      redemptionInfoPerEpoch[epochToService].addressToBurnAmt[refundee] = 0;
      cash.mint(refundee, cashAmountBurned);
      totalCashAmountRefunded += cashAmountBurned;
      emit RefundIssued(refundee, cashAmountBurned, epochToService);
    }
    return totalCashAmountRefunded;
  }
```
The total refunded amount from all refundees is calculated in completeRedemptions() and subtracted from quantityBurned.
```
    uint256 refundedAmt = _processRefund(refundees, epochToService);
    uint256 quantityBurned = redemptionInfoPerEpoch[epochToService]
      .totalBurned - refundedAmt;
```
The variable quantityBurned is then passed on as a parameter to _processRedemption to calculate the collateralAmount due for each person.
```
    _processRedemption(redeemers, amountToDist, quantityBurned, epochToService);
```
The logic flows like this:

CompleteRedemption is called by the admin. The function has to check how much CASH token is burned so it will know how much USDC to compensate for the burn.
Before checking the burned amount, it checks whether any user needs a refund. If so, the total burned is reduced because CASH is being refunded to the refundees.
The function then calculates how much USDC each person gets from the amount of CASH they burned.
For example, 1 CASH token equate to 1USDC token.
In the particular epoch, 5000 CASH tokens are burned. This means that 5000 USDC tokens should be paid out.
Some users need a refund for reasons known to the admin (legal reasons). 1000 CASH tokens are refunded in total.
Now, instead of 5000 CASH tokens burned, 4000 tokens are burned instead because 1000 CASH tokens are refunded.
This means 4000 USDC should be paid out instead of 5000.
The problem lies when the function completeRedemptions() is called multiple times because of gas limit (according to the conversation with the developer, ideally completeRedemptions() is called once per Epoch, but can be called many times if there is gas limit). If the first call is for refundees and some redeemers, the refunded amount will not be saved in redemptionInfoPerEpoch[epochToService].totalBurned. The next few calls will affect the redeemers and they will have a lower compensation because refunded amount is not reflected in the totalBurned amount. Their share of USDC to CASH will be lesser as quantity burned acts as the denominator (the larger the denominator, the lower the collateralAmountDue).
```
      uint256 collateralAmountDue = (amountToDist * cashAmountReturned) /
        quantityBurned;
```
### Tools Used
Manual Review

### Recommended Mitigation Steps
Make sure the totalBurned amount is updated.
```
Line 719

    // Calculate the total quantity of shares tokens burned w/n an epoch
    uint256 refundedAmt = _processRefund(refundees, epochToService);
    uint256 quantityBurned = redemptionInfoPerEpoch[epochToService]
      .totalBurned - refundedAmt;
+   redemptionInfoPerEpoch[epochToService].totalBurned = quantityBurned
    uint256 amountToDist = collateralAmountToDist - fees;
```

## Low and Non-Critical Findings: 

## [L-01] nonReentrant modifier should always come first do prevent reentrancy from other modifiers
https://github.com/code-423n4/2023-01-ondo/blob/f3426e5b6b4561e09460b2e6471eb694efdd6c70/contracts/cash/CashManager.sol#L201 https://github.com/code-423n4/2023-01-ondo/blob/f3426e5b6b4561e09460b2e6471eb694efdd6c70/contracts/cash/CashManager.sol#L244 https://github.com/code-423n4/2023-01-ondo/blob/f3426e5b6b4561e09460b2e6471eb694efdd6c70/contracts/cash/CashManager.sol#L668 https://github.com/code-423n4/2023-01-ondo/blob/f3426e5b6b4561e09460b2e6471eb694efdd6c70/contracts/cash/CashManager.sol#L955

## [L-02] Missing 0 check in Mint and Redeem functions in CashManager.sol
Make sure that protocol cannot set the mint limit as 0

https://github.com/code-423n4/2023-01-ondo/blob/f3426e5b6b4561e09460b2e6471eb694efdd6c70/contracts/cash/CashManager.sol#L596 https://github.com/code-423n4/2023-01-ondo/blob/f3426e5b6b4561e09460b2e6471eb694efdd6c70/contracts/cash/CashManager.sol#L609

## [L-03] Event should emit critical parameters
Event should emit critical parameters, especially lastSetMintExchangeRate and newRate. Also, Each event should use three indexed fields if there are three or more fields.

https://github.com/code-423n4/2023-01-ondo/blob/f3426e5b6b4561e09460b2e6471eb694efdd6c70/contracts/cash/interfaces/ICashManager.sol#L153-L155

https://github.com/code-423n4/2023-01-ondo/blob/f3426e5b6b4561e09460b2e6471eb694efdd6c70/contracts/cash/interfaces/ICashManager.sol#L176-L178

## [L-04] Initializers are front-runnable
Initializers in the protocol are front-runnable because there is no admin privilege. Anyone can front run and set and the protocol has to re-initialize the contract, which waste gas and time

https://github.com/code-423n4/2023-01-ondo/blob/f3426e5b6b4561e09460b2e6471eb694efdd6c70/contracts/cash/token/CashKYCSender.sol#L51 https://github.com/code-423n4/2023-01-ondo/blob/f3426e5b6b4561e09460b2e6471eb694efdd6c70/contracts/lending/tokens/cToken/CErc20.sol#L40

## [N-01] Numeric values having to do with time should use time units for readability
https://github.com/code-423n4/2023-01-ondo/blob/f3426e5b6b4561e09460b2e6471eb694efdd6c70/contracts/lending/OndoPriceOracleV2.sol#L77

uint256 public maxChainlinkOracleTimeDelay = 90000; // 25 hours

can be written as maxChainlinkOracleTimeDelay = 25 hours;

## [N-02] Avoid using sensitive terms
Use alternative variants, e.g. allowlist/denylist instead of whitelist/blacklist

https://github.com/code-423n4/2023-01-ondo/blob/f3426e5b6b4561e09460b2e6471eb694efdd6c70/contracts/lending/compound/governance/GovernorBravoDelegate.sol#L121