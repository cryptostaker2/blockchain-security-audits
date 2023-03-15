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