# Full Report

## Medium Findings:

### [M-01] PrivatePool.sol#flashLoan() is problematic for ERC20 tokens that don't support zero transfers

### Lines of code
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L623-L654

### Impact
Having no Flash fee will break support for ERC20 tokens that don't support zero transfers

### Proof of Concept
PrivatePool.sol#flashLoan() attempts to transfer funds even if there isn't a fee. If changeFee is not set yet or set to zero, then the flash loan fee will be zero. The function then attempts to transfer 0 value to the contract.

    function flashFee(address, uint256) public view returns (uint256) {
        return changeFee;
    }
        uint256 fee = flashFee(token, tokenId);


        // if base token is ETH then check that caller sent enough for the fee
        if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();


        // transfer the NFT to the borrower
        ERC721(token).safeTransferFrom(address(this), address(receiver), tokenId);


        // call the borrower
        bool success =
            receiver.onFlashLoan(msg.sender, token, tokenId, fee, data) == keccak256("ERC3156FlashBorrower.onFlashLoan");


        // check that flashloan was successful
        if (!success) revert FlashLoanFailed();


        // transfer the NFT from the borrower
        ERC721(token).safeTransferFrom(address(receiver), address(this), tokenId);

//@audit-- If fee is zero, function still attempts to transfer
        // transfer the fee from the borrower
        if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee);


        return success;
    }
If the underlying ERC20 doesn't support transfers with zero value then the flashloan will always revert when the deposit fee is zero. (see https://github.com/d-xo/weird-erc20).

### Tools Used
Manual Review

### Recommended Mitigation Steps
Only transfer if the amount to transfer is not zero:

        uint256 fee = flashFee(token, tokenId);


        // if base token is ETH then check that caller sent enough for the fee
        if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();


        // transfer the NFT to the borrower
        ERC721(token).safeTransferFrom(address(this), address(receiver), tokenId);


        // call the borrower
        bool success =
            receiver.onFlashLoan(msg.sender, token, tokenId, fee, data) == keccak256("ERC3156FlashBorrower.onFlashLoan");


        // check that flashloan was successful
        if (!success) revert FlashLoanFailed();


        // transfer the NFT from the borrower
        ERC721(token).safeTransferFrom(address(receiver), address(this), tokenId);

//@audit-- Transfer the fee only if fee is more than zero

       if(fee > 0){
        // transfer the fee from the borrower
        if (baseToken != address(0)) {ERC20(baseToken).transferFrom(msg.sender, address(this), fee)};
      }

        return success;
    }