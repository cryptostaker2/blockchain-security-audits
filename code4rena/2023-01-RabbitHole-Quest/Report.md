# Full Report

## High Findings:

### [H-01] Erc20Quest#withdrawFee() can be called multiple times

### Lines of code

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L102-L104

### Impact

Quest deployer will lose funds / users cannot collect their funds with their receipt.

### Proof of Concept

The withdrawFee() function in Erc20Quest.sol only checks whether the caller is the owner and whether the block.timestamp is greater than the endTime.

function withdrawFee() public onlyAdminWithdrawAfterEnd {
IERC20(rewardToken).safeTransfer(protocolFeeRecipient, protocolFee());
}
The quest owner may accidentally call withdrawFee multiple times. If that happens, funds will go to the protocolFeeRecipient, and successful quest participants will not be able to retrieve their reward because the contract balance will be lesser than intended

### Tools Used

Manual Review

### Recommended Mitigation Steps

Make sure the withdrawFee() function can only be called once, after the endTime has passed.

## Medium Findings:

### [M-01] QuestFactory#mintReceipt() does not adhere to endTime\_

### Lines of code

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L219-L229

### Impact

Successful participants can still mint a receipt after endTime has passed. They will not be able to get their rewards if withdrawRemainingTokens() is called. protocolFee() can keep changing if there are more receiptRedeemers.

### Proof of Concept

A successful user can mint a receipt and trade their receipt for a token reward. However, there is no check for endTime; the participant can still participate and mint a receipt even after the quest ends.

```
function mintReceipt(string memory questId_, bytes32 hash_, bytes memory signature_) public {
    if (quests[questId_].numberMinted + 1 > quests[questId_].totalParticipants) revert OverMaxAllowedToMint();
    if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
    if (keccak256(abi.encodePacked(msg.sender, questId_)) != hash_) revert InvalidHash();
    if (recoverSigner(hash_, signature_) != claimSignerAddress) revert AddressNotSigned();


    quests[questId_].addressMinted[msg.sender] = true;
    quests[questId_].numberMinted++;
    emit ReceiptMinted(msg.sender, questId_);
    rabbitholeReceiptContract.mint(msg.sender, questId_);
}
```

Successful participants who participated after the endTime will not get their rewards if withdrawRemainingTokens() is called already.

### Tools Used

VSCode

### Recommended Mitigation Steps

Make sure participants cannot mint any receipt after endTime. Add a endTime\_ check in the mintReceipt() function. Store the endTime variable in the QuestFactory and use it in mintReceipt().

```
    uint endTime;
    function createQuest(
        address rewardTokenAddress_,
        uint256 endTime_,
        uint256 startTime_,
        uint256 totalParticipants_,
        uint256 rewardAmountOrTokenId_,
        string memory contractType_,
        string memory questId_
    ) public onlyRole(CREATE_QUEST_ROLE) returns (address) {
        if (quests[questId_].questAddress != address(0)) revert QuestIdUsed();
+     endTime = endTime_;
        if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {
    function mintReceipt(string memory questId_, bytes32 hash_, bytes memory signature_) public {
+       if (block.timestamp < endTime) revert NotAllowedToMintAfterQuestEnd();
        if (quests[questId_].numberMinted + 1 > quests[questId_].totalParticipants) revert OverMaxAllowedToMint();
        if (quests[questId_].addressMinted[msg.sender] == true) revert AddressAlreadyMinted();
        if (keccak256(abi.encodePacked(msg.sender, questId_)) != hash_) revert InvalidHash();
        if (recoverSigner(hash_, signature_) != claimSignerAddress) revert AddressNotSigned();


        quests[questId_].addressMinted[msg.sender] = true;
        quests[questId_].numberMinted++;
        emit ReceiptMinted(msg.sender, questId_);
        rabbitholeReceiptContract.mint(msg.sender, questId_);
    }
```

### [M-02] Successful participants who minted a ticket but didn't claim yet cannot do so if Erc1155Quest#withdrawRemainingTokens() is called

### Lines of code

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L54-L62
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L96-L118
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L81-L87

### Impact

Participants cannot exchange their ERC721 Receipt for their ERC1155 reward token if withdrawRemainingTokens() is called.

### Proof of Concept

The withdrawRemainingTokens() in ERC20Quest.sol and ERC1155Quest.sol is written differently. In ERC20Quest.sol, after the quest endtime, participants who still has a receipt but didn't call claim can do so because there will be a portion of the rewards left inside the ERC20Quest contract. withdrawRemainingTokens() in ERC20Quest.sol only withdraws the amount that is for those that didn't manage to mint a receipt.

function withdrawRemainingTokens(address to*) public override onlyOwner {
super.withdrawRemainingTokens(to*);

    uint unclaimedTokens = (receiptRedeemers() - redeemedTokens) * rewardAmountInWeiOrTokenId;
    uint256 nonClaimableTokens = IERC20(rewardToken).balanceOf(address(this)) - protocolFee() - unclaimedTokens;
    IERC20(rewardToken).safeTransfer(to_, nonClaimableTokens);

}
However, in ERC1155Quest.sol, when withdrawRemainingTokens() is called, it just withdraws the leftover tokens in the contract. Those that minted a ERC721 receipt but did not call claim cannot receive their reward after withdrawRemainingToken() is called. Also, there is no protocol fee to pay, unlike ERC20Quest.sol.

function withdrawRemainingTokens(address to*) public override onlyOwner {
super.withdrawRemainingTokens(to*);
IERC1155(rewardToken).safeTransferFrom(
address(this),
to\_,
rewardAmountInWeiOrTokenId,
IERC1155(rewardToken).balanceOf(address(this), rewardAmountInWeiOrTokenId),
'0x00'
);

### Tools Used

Manual Review

### Recommended Mitigation Steps

Make sure the functions in both ERC1155Quest and ERC20Quest work the same way, otherwise it will be unfair for the users (cannot claim in ERC1155Quest) or for the quest deployer (no need to pay protocol fees in ERC1155Quest).

### [M-03] Erc20Quest#withdrawRemainingTokens() may count remaining tokens incorrectly if withdrawFee() is called first.

### Lines of code

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L81-L87
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L91-L93
https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L96-L98

### Summary

Erc20Quest#withdrawRemainingTokens() may count remaining tokens incorrectly if Erc20Quest#withdrawFee() is called first.

### Impact

Quest deployer cannot get back the funds from those participants that did not successfully mint a ticket.

### Proof of Concept

In Erc20Quest#withdrawRemainingTokens(), the owner can withdraw the remaining tokens that is left in the contract.

```
function withdrawRemainingTokens(address to_) public override onlyOwner {
    super.withdrawRemainingTokens(to_);


    uint unclaimedTokens = (receiptRedeemers() - redeemedTokens) * rewardAmountInWeiOrTokenId;
    uint256 nonClaimableTokens = IERC20(rewardToken).balanceOf(address(this)) - protocolFee() - unclaimedTokens;
    IERC20(rewardToken).safeTransfer(to_, nonClaimableTokens);
}
```

Firstly, the function tabulates the amount of unclaimed tokens, which is the total amount of people that has minted a receipt minus those that already claimed, multiplied by the rewardAmount.

Next, the function calculates the nonClaimableTokens, which is the total amount in the contract minus protocolFee() and unclaimedTokens. ProtocolFee() is calculated by taking the total amount of people that has minted a receipt _ amount _ questFee and divided by BPS.

```
function protocolFee() public view returns (uint256) {
    return (receiptRedeemers() * rewardAmountInWeiOrTokenId * questFee) / 10_000;
}
```

However, the function did not take into account that the protocolFee() might already been withdrawn.

```
    uint256 nonClaimableTokens = IERC20(rewardToken).balanceOf(address(this)) - protocolFee() - unclaimedTokens;
```

If the protocolFee() is already withdrawn, then the nonClimableTokens should not account for the protocolFee anymore. Otherwise, the quest deployer will not be able to withdraw their reward.

Eg There is 1500 USDC in the contract. 500 USDC is already distributed to those that minted and claimed.

There is now 1000 USDC in the contract. 200 USDC is protocol fee, 700 USDC is unclaimed tickets and 100 USDC is leftover for failed quests. Quest deployer calls withdrawRemainingTokens and gets 100 USDC back.

However, if protocolFees is withdrawn already (1000-200 = 800), there will be 800 USDC in the contract (address(this)). 200 USDC is protocol fee (protocolFee), 700 USDC is unclaimed tickets (unclaimedTokens), 100 USDC is leftover. Quest deployer calls withdrawRemainingTokens and attempt to withdraw 100 USDC . However, function will result in an overflow because 800 - 200 - 700 = -100.

### Tools Used

Manual Review

### Recommended Mitigation Steps

The function should check whether the protocol fee is withdrawn already. A suggestion is to create a modifier in the protocolFee with a boolean, and check that the protocolFee() is not withdrawn yet before calling withdrawRemainingTokens(). Another suggestion would be to put withdrawFee() and withdrawRemainingTokens() in the same function

## Low Issues

### [L-01] Use safetransferownership instead of transferownership function.

transferOwnership function is used to change Ownership from QuestFactory.sol. Use a 2 structure transferOwnership (Ownable2Step) which is safer. safeTransferOwnership, use it is more secure due to 2-stage ownership transfer.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L70-L75

### [L-02] Sanity checks in QuestFactory.sol

When creating a quest, make sure to check the 0 value of rewardTokenAddress*, totalParticipants, rewardAmountOrTokenId, contractType, questId etc. Also, sanity check for endTime > startTime*, endTime* > block.timestamp and startTime* > block.timestamp should come earlier.

```
function createQuest(
    address rewardTokenAddress_,
    uint256 endTime_,
    uint256 startTime_,
    uint256 totalParticipants_,
    uint256 rewardAmountOrTokenId_,
    string memory contractType_,
    string memory questId_
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L61-L68

### [L-03] Confusing pausing function application

In Quest.sol, the pause functions pauses the quest and the unPause function unpauses the quest. However, the pause and unpause function still works even after the endTime has passed. The claim() function can still be paused by the quest deployer after the endTime has passed, which is a centralization issue and shouldn't be the case. Pause and unpause should only work during the quest duration, and not before or after. Also, minting receipts can still be allowed when the quest is paused.

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L57-L65

## Non-Critical Issues

### [N-01] Non-library/interface files should use fixed compiler versions, not floating ones

Non-library/interface files should use fixed compiler versions, not floating ones

eg. 0.8.15 instead of ^0.8.15

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc1155Quest.sol#L2 https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L2 https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L2 https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L2 https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleReceipt.sol#L2 https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/RabbitHoleTickets.sol#L2 https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/ReceiptRenderer.sol#L2 https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/TicketRenderer.sol#L2

### [N-02] Use bytes.concat() instead of abi.encodePacked()

Rather than using abi.encodePacked for appending bytes, since version 0.8.4, bytes.concat() is enabled. Since version 0.8.4 for appending bytes, bytes.concat() can be used instead of abi.encodePacked(,)

```
    if (quests[questId_].questAddress != address(0)) revert QuestIdUsed();

    if (keccak256(abi.encodePacked(contractType_)) == keccak256(abi.encodePacked('erc20'))) {
        if (rewardAllowlist[rewardTokenAddress_] == false) revert RewardNotAllowed();


        Erc20Quest newQuest = new Erc20Quest(
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/QuestFactory.sol#L70-L75

### [N-03] Update Import Usages for more readable and modern code

import '@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol';
import '@openzeppelin/contracts-upgradeable/utils/cryptography/ECDSAUpgradeable.sol';
import '@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol';
eg.

```
import {Initializable} from "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
```

### [N-04] Confusing variable name

In Erc20Quest.sol#withdrawRemainingTokens(), nonClaimableTokens should be renamed to unsuccessfulTokensToBeReclaimed. nonClaimableTokens sounds as though it is not claimable by anyone, which induces confusion, as the intention of the function is to just withdraw the excess tokens that have been deposited but participants were unable to successfully complete the quest.

```
function withdrawRemainingTokens(address to_) public override onlyOwner {
    super.withdrawRemainingTokens(to_);


    uint unclaimedTokens = (receiptRedeemers() - redeemedTokens) * rewardAmountInWeiOrTokenId;
    uint256 nonClaimableTokens = IERC20(rewardToken).balanceOf(address(this)) - protocolFee() - unclaimedTokens;
    IERC20(rewardToken).safeTransfer(to_, nonClaimableTokens);
}
```

https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Erc20Quest.sol#L81-L87

### [N-05] Improve test coverage

The test coverage rate of the project is 89%. Testing all functions is best practice in terms of security criteria. Due to its capacity, test coverage is expected to be 100%.

### [N-06] Function writing does not comply with the solidity style guide

Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

- constructor
- receive function (if exists)
- fallback function (if exists)
- external
- public
- internal
- private
  within a grouping, place the view and pure functions last
  All contracts, eg: https://github.com/rabbitholegg/quest-protocol/blob/8c4c1f71221570b14a0479c216583342bd652d8d/contracts/Quest.sol#L1-L151
