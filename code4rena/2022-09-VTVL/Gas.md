## TABLE OF CONTENTS
- [G-01] ++I costs less gas as compared to I++ or I+= 1
- [G-02] Increments can be unchecked
- [G-03] Splitting require() statements that use && saves gas
- [G-04] Using > 0 costs more gas than != 0 when used on a uint in a require() statement
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