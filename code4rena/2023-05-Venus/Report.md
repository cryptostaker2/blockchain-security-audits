# Full Report

## Medium Findings:

### [M-01] priceOracle.updatePrice is not called before priceOracle.getUnderlyingPrice in PoolLens#getPoolBadDebt()

### Lines of code
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Lens/PoolLens.sol#L268

### Impact
Price oracle may return a stale price if oracle.updatePrice is not called before oracle.getUnderlyingPrice.

### Proof of Concept
In RiskFund.sol#_swapAsset, when swapping a single asset to base asset, oracle.updatePrice is called before oracle.getUnderlyingPrice.

    _swapAsset(
        VToken vToken,
...
        ComptrollerViewInterface(comptroller).oracle().updatePrice(address(vToken));

        uint256 underlyingAssetPrice = ComptrollerViewInterface(comptroller).oracle().getUnderlyingPrice(
            address(vToken)
        );
Similarly, in ShortFall.sol#_startAuction(), oracle.updatePrice is called before oracle.getUnderlyingPrice.

    _startAuction(address comptroller) internal {
        PoolRegistryInterface.VenusPool memory pool = PoolRegistry(poolRegistry).getPoolByComptroller(comptroller);
...
        PriceOracle priceOracle = _getPriceOracle(comptroller);
...

            priceOracle.updatePrice(address(vTokens[i]));
            uint256 usdValue = (priceOracle.getUnderlyingPrice(address(vTokens[i])) * marketBadDebt) / 1e18;
However, in PoolLens#getPoolBadDebt(),

    getPoolBadDebt(address comptrollerAddress) external view returns (BadDebtSummary memory) {
...
        PriceOracle priceOracle = comptroller.oracle();
...
```
            badDebt.badDebtUsd =
                VToken(address(markets[i])).badDebt() *
                priceOracle.getUnderlyingPrice(address(markets[i]));
            badDebtSummary.badDebts[i] = badDebt;
            totalBadDebtUsd = totalBadDebtUsd + badDebt.badDebtUsd;
oracle.getUnderlyingPrice is immediately called without calling oracle.updatePrice first.
```
### Tools Used
VSCode

### Recommended Mitigation Steps
Call updatePrice first in getPoolBadDebt as well.

        for (uint256 i; i < markets.length; ++i) {
            BadDebt memory badDebt;
            badDebt.vTokenAddress = address(markets[i]);
+           priceOracle.updatePrice(VToken(address(markets[i])))
            badDebt.badDebtUsd =
                VToken(address(markets[i])).badDebt() *
                priceOracle.getUnderlyingPrice(address(markets[i]));
            badDebtSummary.badDebts[i] = badDebt;
            totalBadDebtUsd = totalBadDebtUsd + badDebt.badDebtUsd;
        }
### Assessed type
Oracle

### [M-02] Bad Debt in PoolLens.sol#getPoolBadDebt() is not calculated correctly in USD

### Lines of code
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Lens/PoolLens.sol#L248

### Vulnerability details
In PoolLens.sol#getPoolBadDebt(), bad debt is calculated as such:

            badDebt.badDebtUsd =
                VToken(address(markets[i])).badDebt() *
                priceOracle.getUnderlyingPrice(address(markets[i]));
            badDebtSummary.badDebts[i] = badDebt;
            totalBadDebtUsd = totalBadDebtUsd + badDebt.badDebtUsd;
In Shortfall.sol#_startAuction(), bad debt is calculated as such:

        uint256[] memory marketsDebt = new uint256[](marketsCount);
        auction.markets = new VToken[](marketsCount);

        for (uint256 i; i < marketsCount; ++i) {
            uint256 marketBadDebt = vTokens[i].badDebt();

            priceOracle.updatePrice(address(vTokens[i]));
            uint256 usdValue = (priceOracle.getUnderlyingPrice(address(vTokens[i])) * marketBadDebt) / 1e18;

            poolBadDebt = poolBadDebt + usdValue;
Focus on the line with the priceOracle.getUnderlyingPrice. In PoolLens.sol#getPoolBadDebt, badDebt in USD is calculated by multiplying the bad debt of the VToken market by the underlying price. However, in Shortfall, badDebt in USD is calculated by the bad debt of the VToken market by the underlying price and divided by 1e18.

The PoolLens#getPoolBadDebt() function forgot to divide the debt in usd by 1e18.

This is what the function is actually counting:

Let's say that the VToken market has a badDebt of 1.3 ETH (1e18 ETH). The pool intends to calculate 1.3 ETH in terms of USD, so it calls the oracle to determine the price of ETH. Let's say the price of ETH is 1500 USD. The total pool debt should be 1.3 * 1500 = 1950 USD. In decimal calculation, the pool debt should be 1.3e18 * 1500e18 (if oracle returns in 18 decimal places) / 1e18 = 1950e18.

### Impact
The badDebt in USD in PoolLens.sol#getPoolBadDebt() will be massively inflated.

### Tools Used
VSCode

### Recommended Mitigation Steps
Normalize the decimals of the bad debt calculation in getPoolBadDebt().

            badDebt.badDebtUsd =
                VToken(address(markets[i])).badDebt() *
+               priceOracle.getUnderlyingPrice(address(markets[i])) / 1e18;
            badDebtSummary.badDebts[i] = badDebt;
            totalBadDebtUsd = totalBadDebtUsd + badDebt.badDebtUsd;

### [M-03] Bad debt calculation may be wrong when calling Shortfall#_startAuction() if VTokens have different decimals

### Lines of code
https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Shortfall/Shortfall.sol#L393


### Impact
BadDebt calculation in each vToken market may affect the accounting of the total bad debt in the pool if each VToken has a different decimal place.

### Proof of Concept
When starting an auction via _startAuction(), the protocol calculates the usd value of the bad debt of all the different markets in the pool.

        for (uint256 i; i < marketsCount; ++i) {
            uint256 marketBadDebt = vTokens[i].badDebt();

            priceOracle.updatePrice(address(vTokens[i]));
            uint256 usdValue = (priceOracle.getUnderlyingPrice(address(vTokens[i])) * marketBadDebt) / 1e18;

            poolBadDebt = poolBadDebt + usdValue;
The usdValue of each market's debt is calculated and divided by 1e18. The function then accumulates all the different debt in various markets into the variable poolBadDebt.

Dividing all the usdValue by 1e18 assumes that all priceOracle of every vToken and all VTokens have the same decimal place. In VToken.sol, each VToken can have a different decimal when initializing. If all the different VToken have different decimals, then dividing by 1e18 will result in accounting error because some VTokens may have more decimals than other VTokens.

VToken.sol

    initialize(
        address underlying_,
        ComptrollerInterface comptroller_,
        InterestRateModel interestRateModel_,
        uint256 initialExchangeRateMantissa_,
        string memory name_,
        string memory symbol_,
        uint8 decimals_,
Assuming that all VToken price oracles return 18 decimals, let's take a look at a hypothetical example. Let's say the vDai market (6 decimals) has a bad debt of 1000e6 vDai, and the vDoge market (18 decimals) has a bad debt of 1000e18 doge. When calculating badDebt in terms of USD, let say vDAI is worth 1 usd and vDoge is worth 0.1 usd

For vDai:

uint256 usdValue = (priceOracle.getUnderlyingPrice(address(vTokens[i])) * marketBadDebt) / 1e18;
usdValue = 1e18 * 1000e6 / 1e18 = 1000e6 of bad debt

For vDoge:

uint256 usdValue = (priceOracle.getUnderlyingPrice(address(vTokens[i])) * marketBadDebt) / 1e18;
usdValue = 0.1e18 * 1000e18 / 1e18 = 100e18 of bad debt (when it should be 100e6, normalized to 6 decimals wrt vDai market).

### Tools Used
VSCode

### Recommended Mitigation Steps
Recommend dividing according to the decimals of each vToken instead

        for (uint256 i; i < marketsCount; ++i) {
            uint256 marketBadDebt = vTokens[i].badDebt();

            priceOracle.updatePrice(address(vTokens[i]));
+            uint256 usdValue = (priceOracle.getUnderlyingPrice(address(vTokens[i])) * marketBadDebt) / 10 ** vTokens[i].decimals();

            poolBadDebt = poolBadDebt + usdValue;