# Code4rena - Inverse Finance Audit

Contest: https://code4rena.com/contests/2022-10-inverse-finance-contest

## Findings Summary

|  ID  | Title                                                               | Severity |
| :--: | :------------------------------------------------------------------ | :------- |
| M-01 | DBR.setReplenishmentPriceBps has no upper bound                     | Medium   |
| M-02 | Chainlink latestAnswer is deprecated, use latestRoundData() instead | Medium   |

# Full Report

## Medium Findings:

## [M-01] DBR.setReplenishmentPriceBps has no upper bound

### Lines of code

https://github.com/code-423n4/2022-10-inverse/blob/3e81f0f5908ea99b36e6ab72f13488bbfe622183/src/DBR.sol#L63

### Impact

DBR.setReplenishmentPriceBps has no upper bound. A malicious operator can set a basis point above 10000 and markets can force borrowers in deficit to incur a huge debt.

### Proof of Concept

replenishmentPriceBps is used in onForceReplenish, and only the market can call onForceReplenish to increase the debt of the borrower and mint them new DBR

```
    function onForceReplenish(address user, uint amount) public {
        require(markets[msg.sender], "Only markets can call onForceReplenish");
        uint deficit = deficitOf(user);
        require(deficit > 0, "No deficit");
        require(deficit >= amount, "Amount > deficit");
        uint replenishmentCost = amount * replenishmentPriceBps / 10000;
        accrueDueTokens(user);
        debts[user] += replenishmentCost;
        _mint(user, amount);
    }
```

If replenishmentPriceBps is 10,000, 1 DBR will be minted and 1 DOLA will be added to the borrower's debt.
If replenishmentPriceBps is 1,000,000, 1 DBR will be minted and 100 DOLA will be added to the borrower's debt, which makes it unfair for the borrower.

### Tools Used

Manual Review

### Recommended Mitigation Steps

Set a limit for setReplenishmentPriceBps

```
    function setReplenishmentPriceBps(uint newReplenishmentPriceBps_) public onlyOperator {
        @--audit no upperbound
        require(newReplenishmentPriceBps_ > 0, "replenishment price must be over 0");
        replenishmentPriceBps = newReplenishmentPriceBps_;
    }
```

similar to setColateralFactorBps and setLiquidationFactorBps in Market.sol

```
    function setReplenishmentPriceBps(uint newReplenishmentPriceBps_) public onlyOperator {
        require(newReplenishmentPriceBps_ > 0 && newReplenishmentPriceBps <= 10000, "replenishment price must be over 0");
        replenishmentPriceBps = newReplenishmentPriceBps_;
    }
```

## [M-02] Chainlink latestAnswer is deprecated, use latestRoundData() instead

### Lines of code

https://github.com/code-423n4/2022-10-inverse/blob/3e81f0f5908ea99b36e6ab72f13488bbfe622183/src/Oracle.sol#L82
https://github.com/code-423n4/2022-10-inverse/blob/3e81f0f5908ea99b36e6ab72f13488bbfe622183/src/Oracle.sol#L116

### Impact

According to Chainlink's documentation, the latestAnswer function is deprecated. This function returns 0 instead of reverting if there is no answer. A best practice is to get the decimals from the oracles instead of hard-coding them in the contract.

### Proof of Concept

https://github.com/code-423n4/2022-10-inverse/blob/3e81f0f5908ea99b36e6ab72f13488bbfe622183/src/Oracle.sol#L82
https://github.com/code-423n4/2022-10-inverse/blob/3e81f0f5908ea99b36e6ab72f13488bbfe622183/src/Oracle.sol#L116

### Tools Used

Manual Review

### Recommended Mitigation Steps

Use V3 interface functions: https://docs.chain.link/docs/price-feeds-api-reference/
