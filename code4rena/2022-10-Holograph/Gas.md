## Table of Contents
- [G-01] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops
- [G-02] +=  costs more gas than  = +  for state variables
### [G-01] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops
The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop. HolographOperator.sol, HolographRegistry.sol, HolographRegistry.sol

### [G-02] +=  costs more gas than  = +  for state variables
Using the addition operator instead of plus-equals saves 113 gas. HolographERC20.sol