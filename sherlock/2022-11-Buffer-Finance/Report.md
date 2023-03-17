# Full Report

## Medium Findings:

### [M-01] Unsafe usage of ERC20 .transfer

### Summary

Some .transfer checks in the contract are not checked for success. In other words, transfer may fail but function will still continue to work.

### Vulnerability Detail

Some .transfer functions used do not check for success.

### Impact

Fees may not be transferred correctly into and out of the protocol, resulting in protocol / user losses

### Code Snippet

https://github.com/sherlock-audit/2022-11-buffer/blob/main/contracts/contracts/core/BufferBinaryOptions.sol#L141

https://github.com/sherlock-audit/2022-11-buffer/blob/main/contracts/contracts/core/BufferBinaryOptions.sol#L477

https://github.com/sherlock-audit/2022-11-buffer/blob/main/contracts/contracts/core/BufferRouter.sol#L331

https://github.com/sherlock-audit/2022-11-buffer/blob/main/contracts/contracts/core/BufferRouter.sol#L335

https://github.com/sherlock-audit/2022-11-buffer/blob/main/contracts/contracts/core/BufferRouter.sol#L361

https://github.com/sherlock-audit/2022-11-buffer/blob/main/contracts/contracts/core/BufferRouter.sol#L86

### Tool used

Manual Review

### Recommendation

Use OpenZeppellin's safe transfer or check the validity of every transfer such as the one used in BufferBinaryPool.sol

```solidity
    bool success = tokenX.transfer(to, transferTokenXAmount);
    require(success, "Pool: The Payout transfer didn't go through");
```


