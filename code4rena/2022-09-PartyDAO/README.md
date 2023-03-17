# Code4rena - PartyDAO Protocol Audit

Contest: https://code4rena.com/contests/2022-09-partydao-contest

## Findings Summary

|  ID  | Title                                                         | Severity |
| :--: | :------------------------------------------------------------ | :------: |
| G-01 | No need to explicitly initialize variables with default value |   Gas    |
| G-02 | Using abi.encode() is less efficient than abi.encodePacked()  |   Gas    |

# Full Report

## Gas Findings 

### [G-01] No need to explicitly initialize variables with default value
If a variable is not set/initialized, it is assumed to have the default value (0 for uint, false for bool, address(0) for address…). Explicitly initializing it with its default value is an anti-pattern and wastes gas. In the similar code above, the initialization of uint256 i = 0 can be simplified to uint256 i

File: ListOnOpenseaProposal.sol Line 291 ArbitraryCallsProposal.sol Line 52 Line 61 Line 78

 for (uint256 i = 0; i < fees.length; ++i) {
                cons = orderParams.consideration[1 + i];
                cons.itemType = IOpenseaExchange.ItemType.NATIVE;
                cons.token = address(0);
                cons.identifierOrCriteria = 0;
                cons.startAmount = cons.endAmount = fees[i];
                cons.recipient = feeRecipients[i];
            }
can become

 for (uint i; i < fees.length; ++i) {
                cons = orderParams.consideration[1 + i];
                cons.itemType = IOpenseaExchange.ItemType.NATIVE;
                cons.token = address(0);
                cons.identifierOrCriteria = 0;
                cons.startAmount = cons.endAmount = fees[i];
                cons.recipient = feeRecipients[i];
            }
### [G-02] Using abi.encode() is less efficient than abi.encodePacked()

https://ethereum.stackexchange.com/questions/119583/when-to-use-abi-encode-abi-encodepacked-or-abi-encodewithsignature-in-solidity

File: ListOnZoraProposal.sol Line 115 File: ListOnOpenseaProposal.sol Line 164, Line 219