## Table of Contents
- [L-01] Burn function is not implemented in unbondUtilityToken() when operator fails to do his job
- [L-02] _safemint() should be used rather than _mint() whereever possible in ERC721 contracts
- [N-01] Some function parameters should be renamed for clarity
- [N-02] Transferring of ownership should be a two-step process
- [N-03] Best practice is to prevent signature malleability
### [L-01] Burn function is not implemented in unbondUtilityToken() when operator fails to do his job
In unbondUtilityToken, there is no fees when withdrawing the token as stated in the docs

### [L-02] _safemint() should be used rather than _mint() whereever possible in ERC721 contracts
_mint()is discouraged in favor of _safeMint() which ensures that the recipient is either an EOA or implements IERC721Receiver. Both OpenZeppelin and solmate have versions of this function.

### [N-01] Some function parameters should be renamed for clarity
The parameter of _getCurrentBondAmount should be podIndex instead of pod. Same as _getBaseBondAmount

### [N-02] Transferring of ownership should be a two-step process
Owner.sol

### [N-03] Best practice is to prevent signature malleability
Use OpenZeppelin’s ECDSA contract rather than calling ecrecover() directly. HolographFactory.sol