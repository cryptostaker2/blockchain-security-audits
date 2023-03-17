# Code4rena - VTVL Protocol Audit

Contest: https://code4rena.com/contests/2022-09-vtvl-contest

## Findings Summary

|  ID  | Title                                                                           | Severity |
| :--: | :------------------------------------------------------------------------------ | :------: |
| M-01 | Vouchers and vouchees indices become corrupted by UserManager's cancelVouch     |  Medium  |
| L-01 | Admin can revoke admin rights of every other admin                              |   Low    |
| G-01 | ++I costs less gas as compared to I++ or I+= 1                                  |   Gas    |
| G-02 | Increments can be unchecked                                                     |   Gas    |
| G-03 | Splitting require() statements that use && saves gas                            |   Gas    |
| G-04 | Using > 0 costs more gas than != 0 when used on a uint in a require() statement |   Gas    |

# Full Report

## Medium Findings:

### M-01: Vouchers and vouchees indices become corrupted by UserManager's cancelVouch

### Lines of code

[https://github.com/code-423n4/2022-09-vtvl/blob/f68b7f3e61dad0d873b5b5a1e8126b839afeab5f/contracts/token/VariableSupplyERC20Token.sol#L40](https://github.com/code-423n4/2022-09-vtvl/blob/f68b7f3e61dad0d873b5b5a1e8126b839afeab5f/contracts/token/VariableSupplyERC20Token.sol#L40)

### Impact

When creating the contract VariableSupplyERC20Token in [VariableSupplyERC20Token.sol]

(

[https://github.com/code-423n4/2022-09-vtvl/blob/main/contracts/token/VariableSupplyERC20Token.sol#L40](https://github.com/code-423n4/2022-09-vtvl/blob/main/contracts/token/VariableSupplyERC20Token.sol#L40)

), creator is expected to fill in 4 parameters: string memory name*, string memory symbol*, uint256 initialSupply*, uint256 maxSupply*. From the contract, if the creator sets initialSupply\_ to be 0, there will be no premints. If the creator sets maxSupply to be 0, that means there is no max supply.

```

@param name_ - Name of the token

@param symbol_ - Symbol of the token

@param initialSupply_ - How much to immediately mint (saves another transaction). If 0, no mint at the beginning.

@param maxSupply_ - What's the maximum supply. The contract won't allow minting over this amount. Set to 0 for no limit.

mintableSupply = maxSupply_;

```

However, the mintableSupply can exit the maxSupply even when the maxSupply is stated during the creation of the contract.

### Proof of Concept

1. Malicious admin can set maxSupply\_ to 100,000 and create the contract. Maximum supply is now set to 100,000.

2. Malicious admin can then invoke the mint() function to mint 100,000 tokens into his account. Since mintableSupply is more than 0, the if clause will execute and mintableSupply will become 0.

3. When mintableSupply becomes 0, malicious admin can call the mint() contract again. This time, since mintableSupply is not greater than 0, the \_mint() function will execute. Malicious admin can then mint any amount of tokens he wants.

```

function mint(address account, uint256 amount) public onlyAdmin {

require(account != address(0), "INVALID_ADDRESS");

// If we're using maxSupply, we need to make sure we respect it

// mintableSupply = 0 means mint at will

if(mintableSupply > 0) {     -- @audit this check is supposedly meant for the initial "set to 0 for no limit" rule

require(amount <= mintableSupply, "INVALID_AMOUNT");

// We need to reduce the amount only if we're using the limit, if not just leave it be

mintableSupply -= amount; -- @audit when mintableSupply becomes 0, the if block will not execute and the code will proceed to _mint()

}

_mint(account, amount);

}

```

### Tools Used

Manual Auditing

### Recommended Mitigation Steps

Consider setting a soft cap for max supply instead.

## Low Findings

### Lines of code

[https://github.com/code-423n4/2022-09-vtvl/blob/f68b7f3e61dad0d873b5b5a1e8126b839afeab5f/contracts/AccessProtected.sol#L39](https://github.com/code-423n4/2022-09-vtvl/blob/f68b7f3e61dad0d873b5b5a1e8126b839afeab5f/contracts/AccessProtected.sol#L39)

### Impact

Admin can revoke admin rights of every other admin, including himself.

### Proof of Concept

1. Owner A gives admin rights to B,C,D. [Line 39](

[https://github.com/code-423n4/2022-09-vtvl/blob/f68b7f3e61dad0d873b5b5a1e8126b839afeab5f/contracts/AccessProtected.sol#L39](https://github.com/code-423n4/2022-09-vtvl/blob/f68b7f3e61dad0d873b5b5a1e8126b839afeab5f/contracts/AccessProtected.sol#L39)

)

2. Malicious admin D decides to revoke admin rights of A,B,C.

3. Malicious admin D revokes his own rights. No one has admin rights anymore.

4. Functions such as createClaim(), createClaimsBatch(), withdrawAdmin(), revokeClaim(), withdrawOtherToken() in [VTVLVesting.sol](

[https://github.com/code-423n4/2022-09-vtvl/blob/main/contracts/VTVLVesting.sol](https://github.com/code-423n4/2022-09-vtvl/blob/main/contracts/VTVLVesting.sol)

) cannot be called since the modifier onlyAdmin is inherited from AccessProtocol.sol and no one is the admin anymore. This way, the VTVLVesting.sol contract becomes invalidated.

### Tools Used

Manual Review

### Recommended Mitigation Steps

From contest: "Our vesting contract is deliberately designed to allow admin revocation in the circumstances of early employment termination (before the end of vesting period specified)." Since issuing admin rights is already an accepted clause in the contract design, consider adding an additional require statement in setAdmin() [Line 39](

[https://github.com/code-423n4/2022-09-vtvl/blob/f68b7f3e61dad0d873b5b5a1e8126b839afeab5f/contracts/AccessProtected.sol#L39](https://github.com/code-423n4/2022-09-vtvl/blob/f68b7f3e61dad0d873b5b5a1e8126b839afeab5f/contracts/AccessProtected.sol#L39)

) which states that an admin cannot revoke admin rights of himself.

## Gas Findings

### [G-01] ++I costs less gas as compared to I++ or I+= 1

++i costs less gas compared to i++ or i += 1 for unsigned integer, as pre-increment is cheaper (about 5 gas per iteration). This statement is true even with the optimizer enabled.

File: VTVLVesting.sol Line 253

```
for (uint256 i = 0; i < length; i++) {
            _createClaimUnchecked(_recipients[i], _startTimestamps[i], _endTimestamps[i], _cliffReleaseTimestamps[i], _releaseIntervalsSecs[i], _linearVestAmounts[i], _cliffAmounts[i]);
        }
```

can become

```
for (uint256 i = 0; i < length; ++i) {
            _createClaimUnchecked(_recipients[i], _startTimestamps[i], _endTimestamps[i], _cliffReleaseTimestamps[i], _releaseIntervalsSecs[i], _linearVestAmounts[i], _cliffAmounts[i]);
        }
```

### [G-02] No need to explicitly initialize variables with default value

If a variable is not set/initialized, it is assumed to have the default value (0 for uint, false for bool, address(0) for address…). Explicitly initializing it with its default value is an anti-pattern and wastes gas. In the similar code above, the initialization of uint256 i = 0 can be simplified to uint256 i.

File: VTVLVesting.sol Line 253

### [G-03] Splitting require() statements that use && saves gas

Instead of using the && operator in a single require statement to check multiple conditions,using multiple require statements with 1 condition per require statement will save 3 GAS per &&

File: VTVLVesting.sol Line 344

### [G-04] Using > 0 costs more gas than != 0 when used on a uint in a require() statement

!= 0 costs 6 less GAS compared to > 0 for unsigned integers in require statements with the optimizer enabled.

File: FullPremintERC20Token.sol Line 11, Line 27
