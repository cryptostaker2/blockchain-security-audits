# Full Report

## High Findings:

### [H-01] Anyone can call PerpDepository.rebalance()
### Summary
The protocol uses a delta neutral strategy to keep the collateral price in check. If the price of the collateral, eg ETH, falls, the protocol can rebalance its asset amount through a series of withdrawal and deposit because of the protocol's shorting strategy. However, anyone can call the rebalance function. A malicious user can call rebalance on an account many times to waste unnecessary fees.

### Vulnerability Detail
When a user's collateral experience a downfall, the protocol can execute rebalancing to keep the collateral price in check. The function rebalance() calls _rebalanceNegativePnlWithSwap(),
```
function rebalance(
    uint256 amount,
    uint256 amountOutMinimum,
    uint160 sqrtPriceLimitX96,
    uint24 swapPoolFee,
    int8 polarity,
    address account
) external nonReentrant returns (uint256, uint256) {
    if (polarity == -1) {
        return
            _rebalanceNegativePnlWithSwap(
                amount,
                amountOutMinimum,
                sqrtPriceLimitX96,
                swapPoolFee,
                account
            );
    } else if (polarity == 1) {
        // disable rebalancing positive PnL
        revert PositivePnlRebalanceDisabled(msg.sender);
        // return _rebalancePositivePnlWithSwap(amount, amountOutMinimum, sqrtPriceLimitX96, swapPoolFee, account);
    } else {
        revert InvalidRebalance(polarity);
    }
}
```
which executes vault.withdraw and vault.deposit. In between withdraw and deposit, the account has to pay a small amount of fee to execute a swap function.
```
    vault.withdraw(assetToken, baseAmount);
    SwapParams memory params = SwapParams({
        tokenIn: assetToken,
        tokenOut: quoteToken,
        amountIn: baseAmount,
        amountOutMinimum: amountOutMinimum,
        sqrtPriceLimitX96: sqrtPriceLimitX96,
        poolFee: swapPoolFee
    });
    uint256 quoteAmountOut = spotSwapper.swapExactInput(params);
    int256 shortFall = int256(
        quoteAmount.fromDecimalToDecimal(18, ERC20(quoteToken).decimals())
    ) - int256(quoteAmountOut);
    if (shortFall > 0) {
        IERC20(quoteToken).transferFrom(
            account,
            address(this),
            uint256(shortFall)
        );
    } else if (shortFall < 0) {
        // we got excess tokens in the spot swap. Send them to the account paying for rebalance
        IERC20(quoteToken).transfer(
            account,
            _abs(shortFall)
        );
    }
    vault.deposit(quoteToken, quoteAmount);
```
### Impact
A malicious user can execute rebalance() continuously to drain the user funds by having him pay the swap fees over and over again.

### Code Snippet
https://github.com/sherlock-audit/2023-01-uxd/blob/main/contracts/integrations/perp/PerpDepository.sol#L446-L470

https://github.com/sherlock-audit/2023-01-uxd/blob/main/contracts/integrations/perp/PerpDepository.sol#L498-L524

### Tool used
Manual Review

### Recommendation
Introduce a modifier to the rebalance() function so that only verified people can call the function.

## Medium Findings: 

### [M-01] PerpDepository._rebalanceNegativePnlWithSwap uses wrong parameter in quoteAmount.fromDemicalToDecmial()
### Summary
quoteAmount.fromDecimalToDecimal uses wrong parameters.

### Vulnerability Detail
The function fromDecimalToDecimal takes in the decimal of the quoteToken as a first parameter and the number of decimals to convert to as the second parameter. The function is used 5 times in PerpDepository.sol.

Line 386
```
    uint256 normalizedAmount = quoteAmount.fromDecimalToDecimal(
        ERC20(quoteToken).decimals(),
        18
    );
```    
Line 485
```
    uint256 normalizedAmount = amount.fromDecimalToDecimal(
        ERC20(quoteToken).decimals(),
        18
    );
```    
Line 620
```
    uint256 normalizedAmount = amount.fromDecimalToDecimal(
        ERC20(quoteToken).decimals(),
        18
    );
```
Line 770
```
    uint256 quoteTokenBalance = uint256(accountQuoteTokenBalance)
        .fromDecimalToDecimal(ERC20(quoteToken).decimals(), 18);
    (
        ,
```
In line 509, the parameters are switched.
```
        quoteAmount.fromDecimalToDecimal(18, ERC20(quoteToken).decimals())
    ) - int256(quoteAmountOut);
```
### Impact
Decimal precision will be compromised.

### Code Snippet
https://github.com/sherlock-audit/2023-01-uxd/blob/main/contracts/integrations/perp/PerpDepository.sol#L386-L389

https://github.com/sherlock-audit/2023-01-uxd/blob/main/contracts/integrations/perp/PerpDepository.sol#L509-L510

### Tool used
Manual Review

### Recommendation
Switch the parameters in line 509.

### [M-02] PerpDepository.getDebtValue() is not used or checked when redeeming collateral for UXD
### Summary
PerpDepository.getDebtValue() is written but not used or checked. A user might have bad debt but still be able to withdraw collateral.

### Vulnerability Detail
In PerpDepositry.sol(), there exist a function called getDebtValue(). getDebtValue gets the debt of the account, and the debt is calculated as such:
```
Debt: balance + unrealized PnL - Pending fee - pending funding payments
```
PerpDepository.getDebtValue()
```
function getDebtValue(address account) external view returns (uint256) {
    IAccountBalance perpAccountBalance = IAccountBalance(
        clearingHouse.getAccountBalance()
    );
    IExchange perpExchange = IExchange(clearingHouse.getExchange());
    int256 accountQuoteTokenBalance = vault.getBalance(account);
    if (accountQuoteTokenBalance < 0) {
        revert InvalidQuoteTokenBalance(accountQuoteTokenBalance);
    }
    int256 fundingPayment = perpExchange.getAllPendingFundingPayment(
        account
    );
    uint256 quoteTokenBalance = uint256(accountQuoteTokenBalance)
        .fromDecimalToDecimal(ERC20(quoteToken).decimals(), 18);
    (
        ,
        int256 perpUnrealizedPnl,
        uint256 perpPendingFee
    ) = perpAccountBalance.getPnlAndPendingFee(account);
    int256 debt = int256(quoteTokenBalance) +
        perpUnrealizedPnl -
        int256(perpPendingFee) -
        fundingPayment;
    return (debt > 0) ? 0 : _abs(debt);
}
```
A user might have incurred some debt through fees or raking up a negative PnL. In the current instance of the protocol, the debtValue is not checked. A user can redeem his collateral using UXD while leaving bad debt behind.

### Impact
Debt is not checked for 0 value. A user can leave bad debt when exiting from the protocol.

### Code Snippet
https://github.com/sherlock-audit/2023-01-uxd/blob/main/contracts/integrations/perp/PerpDepository.sol#L758

### Tool used
Manual Review

### Recommendation
Use getDebtValue() in the redeem function. Check that debt = 0 for the account before executing redeem(). If there is any debt accrued, make sure the user pays it first before redeeming collateral for UXD.

### [M-03] Clearing house fees are stored in PerpDepository() but not paid
### Summary
Fees are calculated in PerpDepository() but not paid.

## Vulnerability Detail
When exchanging collateral for UXD through Perpetual Protocol, PerpDepository._placePerpOrder() is called. In _placePerpOrder, feeAmount is counted for the account and totalFeesPaid is tallied but the value is not subtracted from the amount.

This is the feeAmount in _placePerpOrder(), calculated by calling _calculatePerpOrderFeeAmount and passing in the quoteAmount. totalFeesPaid is the sum of all feeAmount.
```
    uint256 feeAmount = _calculatePerpOrderFeeAmount(quoteAmount);
    totalFeesPaid += feeAmount;
```    
In the comments, amount should be the collateral amount - fees. However, the amount passed in is the base amount and not after fees deduction.
```
            amount: amount, // collateral amount - fees
```
The amount passed into the params struct should be after fees deduction.

### Impact
Clearing house fees are not paid.

### Code Snippet
https://github.com/sherlock-audit/2023-01-uxd/blob/main/contracts/integrations/perp/PerpDepository.sol#L352-L371

### Tool used
Manual Review

### Recommendation
Calculate the feesAmount first and subtract from the amount before passing the value into the params struct.

_placePerpOrder()
```
+        uint256 feeAmount = _calculatePerpOrderFeeAmount(quoteAmount);
         IClearingHouse.OpenPositionParams memory params = IClearingHouse
             .OpenPositionParams({
                 baseToken: market,
                 isBaseToQuote: isShort, // true for short
                 isExactInput: amountIsInput, // we specify exact input amount
-                amount: amount, // collateral amount - fees
+                amount: amount - feeAmount, // collateral amount - fees
                 oppositeAmountBound: upperBound, // output upper bound
                 // solhint-disable-next-line not-rely-on-time
                 deadline: block.timestamp,
                 sqrtPriceLimitX96: sqrtPriceLimit, // max slippage
                 referralCode: 0x0
             });


         (uint256 baseAmount, uint256 quoteAmount) = clearingHouse.openPosition(
             params
         );
-        uint256 feeAmount = _calculatePerpOrderFeeAmount(quoteAmount);
         totalFeesPaid += feeAmount;
```