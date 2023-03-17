# Full Report

## High Findings:

### [H-01] First depositor of sGLP token can break share calculations

### Summary

The first depositor into any new Vault can buy a small number of shares and then send a large amount of the underlying asset to the contract which will break the share calculation and make the shares extremely expensive for the subsequent depositors. This is a common attack vector in most of the shares based liquidity pool contract.

### Vulnerability Detail

supply == 0 ? shares

Given the sGLP token vault with DAI as the underlying asset,

Alice (attacker) will deposit initial liquidity of 1 wei of DAI via ERC4626.deposit
Alice receives 1 share of the protocol
Alice proceeds to send a marge large number, ie 10 ** 18 of DAI directly to the sGLP address to artificially inflate the asset balance without minting new shares.
The asset balance is not 10 ** 18 + 1 wei of DAI, the vault share price is not extremely high
Bob, unbeknownst to him, deposits 100 ** 18 of DAI but receives 0 shares
Impact
Subsequent depositors will not be able to get equivalent shares from depositing their underlying asset

### Code Snippet
https://github.com/sherlock-audit/2022-10-rage-trade/blob/main/dn-gmx-vaults/contracts/vaults/DnGmxJuniorVault.sol#L554-L558

### Tool used
Manual Review

### Recommendation
Consider requiring a minimum amount of share tokens to be minted for the first minter or follow Uniswap V2 which mints 10,000 share first to balance liquidity.

https://github.com/Uniswap/v2-core/blob/ee547b17853e71ed4e0101ccfd52e70d5acded58/contracts/UniswapV2Pair.sol#L119-L124

