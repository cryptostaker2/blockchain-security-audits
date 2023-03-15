# Lines of code

[https://github.com/code-423n4/2022-09-vtvl/blob/f68b7f3e61dad0d873b5b5a1e8126b839afeab5f/contracts/AccessProtected.sol#L39](https://github.com/code-423n4/2022-09-vtvl/blob/f68b7f3e61dad0d873b5b5a1e8126b839afeab5f/contracts/AccessProtected.sol#L39)

# Vulnerability details

## Impact

Admin can revoke admin rights of every other admin, including himself.

## Proof of Concept

1. Owner A gives admin rights to B,C,D. [Line 39](

[https://github.com/code-423n4/2022-09-vtvl/blob/f68b7f3e61dad0d873b5b5a1e8126b839afeab5f/contracts/AccessProtected.sol#L39](https://github.com/code-423n4/2022-09-vtvl/blob/f68b7f3e61dad0d873b5b5a1e8126b839afeab5f/contracts/AccessProtected.sol#L39)

)

2. Malicious admin D decides to revoke admin rights of A,B,C.

3. Malicious admin D revokes his own rights. No one has admin rights anymore.

4. Functions such as createClaim(), createClaimsBatch(), withdrawAdmin(), revokeClaim(), withdrawOtherToken() in [VTVLVesting.sol](

[https://github.com/code-423n4/2022-09-vtvl/blob/main/contracts/VTVLVesting.sol](https://github.com/code-423n4/2022-09-vtvl/blob/main/contracts/VTVLVesting.sol)

) cannot be called since the modifier onlyAdmin is inherited from AccessProtocol.sol and no one is the admin anymore. This way, the VTVLVesting.sol contract becomes invalidated.

## Tools Used

## Recommended Mitigation Steps

From contest: "Our vesting contract is deliberately designed to allow admin revocation in the circumstances of early employment termination (before the end of vesting period specified)." Since issuing admin rights is already an accepted clause in the contract design, consider adding an additional require statement in setAdmin() [Line 39](

[https://github.com/code-423n4/2022-09-vtvl/blob/f68b7f3e61dad0d873b5b5a1e8126b839afeab5f/contracts/AccessProtected.sol#L39](https://github.com/code-423n4/2022-09-vtvl/blob/f68b7f3e61dad0d873b5b5a1e8126b839afeab5f/contracts/AccessProtected.sol#L39)

) which states that an admin cannot revoke admin rights of himself.