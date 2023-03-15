## [L-01] Initializers can be frontrunnable
If the initializer is not executed in the same transaction as the constructor, a malicious user can front-run the initialize() call, forcing the contract to be redeployed.

https://github.com/code-423n4/2023-01-biconomy/blob/53c8c3823175aeb26dee5529eeefa81240a406ba/scw-contracts/contracts/smart-contract-wallet/SmartAccount.sol#L166-L176

## [L-02] Contract should adhere to two-step ownership process
Setting the owner within one function can be dangerous if the address of the owner in the parameter is not set properly. Consider using a two-step process whereby the owner has to acknowledge the change before changing the ownership.

https://github.com/code-423n4/2023-01-biconomy/blob/53c8c3823175aeb26dee5529eeefa81240a406ba/scw-contracts/contracts/smart-contract-wallet/SmartAccount.sol#L109-L125

## [L-03] Unused/Empty receive()/fallback() function
If the intention is for the Ether to be used, the function should call another function, otherwise it should revert (e.g. require(msg.sender == address(weth)). Having no access control on the function means that someone may send Ether to the contract, and have no way to get anything back out, which is a loss of funds

https://github.com/code-423n4/2023-01-biconomy/blob/53c8c3823175aeb26dee5529eeefa81240a406ba/scw-contracts/contracts/smart-contract-wallet/SmartAccount.sol#L550

## [L-04] OpenZeppelin's ECDSA is imported but not utilized
Use OpenZeppelin’s ECDSA contract rather than calling ecrecover() directly

https://github.com/code-423n4/2023-01-biconomy/blob/53c8c3823175aeb26dee5529eeefa81240a406ba/scw-contracts/contracts/smart-contract-wallet/SmartAccount.sol#L347

## [L-05] Import exact functions instead of the whole contract itself
Example:

import {ReentrancyGuardUpgradeable} from "@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";
instead of

import "@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";
Applies to all contracts in Biconomy protocol.

SmartAccount.sol:

import "./libs/LibAddress.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import "@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";
import "./BaseSmartAccount.sol";
import "./common/Singleton.sol";
import "./base/ModuleManager.sol";
import "./base/FallbackManager.sol";
import "./common/SignatureDecoder.sol";
import "./common/SecuredTokenTransfer.sol";
import "./interfaces/ISignatureValidator.sol";
import "./interfaces/IERC165.sol";
import "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";
## [N-01] Non-library/interface files should use fixed compiler versions, not floating ones
eg. 0.8.10 instead of ^0.8.10

https://github.com/code-423n4/2023-01-biconomy/blob/53c8c3823175aeb26dee5529eeefa81240a406ba/scw-contracts/contracts/smart-contract-wallet/paymasters/BasePaymaster.sol#L2

## [N-02] Avoid the use of sensitive terms in favour of neutral ones
Use allowlist rather than whitelist.

https://github.com/code-423n4/2023-01-biconomy/blob/53c8c3823175aeb26dee5529eeefa81240a406ba/scw-contracts/contracts/smart-contract-wallet/base/ModuleManager.sol#L28

## [N-03] Testing coverage is not adequate enough
 What is the overall line coverage percentage provided by your tests?:  41
Protocol should aim to achieve >95% testing coverage to make sure that all functions are in working order.