# Full Report

## Medium Findings:

### [M-01] Owner of winning NFT ticket can frontrun buyer on secondary NFT market when claiming rewards

### Lines of code

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L259-L269

### Impact

Owner of winning NFT ticket can frontrun buyer on secondary NFT market when claiming rewards. Buyer on secondary market will lose funds.

### Proof of Concept

When a player gets a winning ticket, he can either claim the ticket directly by calling claimWinningTicket() or sell it on the secondary NFT market. If the player chooses to claim the ticket, the ticket will be marked as claimed and cannot be claimed anymore.

    function claimWinningTicket(uint256 ticketId) private onlyTicketOwner(ticketId) returns (uint256 claimedAmount) {
        uint256 winTier;
        (claimedAmount, winTier) = this.claimable(ticketId);
        if (claimedAmount == 0) {
            revert NothingToClaim(ticketId);
        }


        unclaimedCount[ticketsInfo[ticketId].drawId][ticketsInfo[ticketId].combination]--;
        markAsClaimed(ticketId);
        emit ClaimedTicket(msg.sender, ticketId, claimedAmount);
    }

The player can sell the winning ticket on an NFT marketplace, wait for a buyer to click on purchase, then frontrun the purchase by calling claimWinningTicket(). Now, the player has the money, and the buyer has the claimed NFT, resulting in a loss of funds for the buyer because he cannot claim anymore.

### Tools Used

Manual Review

### Recommended Mitigation Steps

Recommend making sure that NFT transfer is disabled right after it is claimed so that a claimed ticket cannot be bought anymore. Can also consider burning the claimed NFT.

## Low and Non-Critical Findings:

### [L-01] First draw schedule should not accept a time in the past

In the constructor of LotterySetup.sol, the protocol owner dictates the first draw schedule by filling in the firstDrawschedule variable.

```
        firstDrawSchedule = lotterySetupParams.drawSchedule.firstDrawScheduledAt;
```

There should exist a check that the firstDrawSchedule must be greater than the block.timestamp, else the firstDrawSchedule can be mistakenly set in the past.

```
if (
	firstDrawSchedule < block.timestamp){
		revert FirstDrawInThePast();
}
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L83

### [L-02] Avoid Division before Multiplication

Avoid writing division before multiplication, even if it looks safe.

```
            // 0.75%
            return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 75 / 100);
        }
        if (totalTicketsSoldPrevDraw < 1_000_000) {
            // 0.5%
            return totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 50 / 100);
        }
        return 5000;
    }
```

Looking at the calculation of percentages,

```
totalTicketsSoldPrevDraw.getPercentage(PercentageMath.ONE_PERCENT * 75 / 100)
is totalTicketsSoldPrevDraw * (PercentageMath.ONE_PERCENT * 75 / 100) / 100000 (which has two divisions)
```

which can be written as

```
totalTicketsSoldPrevDraw.getPercentage(750)
Same for 0.5% , write it as 500 instead of doing a calculation since PercentageMath has a constant variable PERCENTAGE_BASE of 100_000
```

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/ReferralSystem.sol#L122-L128

### [L-03] Check for address(0) or amount == 0 in the LotterySetupParams constructor

In the LotterySetup constructor, values are not checked to be 0. It is good practice for every variable to be checked for 0 value in case of a typing error.
```
-       if(ticketPrice == 0){revert ZeroAmount()};
-       if(selectionSize == 0){revert ZeroAmount()};
-       if(selectionMax == 0){revert ZeroAmount()};
-       if(expectedPayout == 0){revert ZeroAmount()};

          ticketPrice = lotterySetupParams.ticketPrice;
          selectionSize = lotterySetupParams.selectionSize;
          selectionMax = lotterySetupParams.selectionMax;
          expectedPayout = lotterySetupParams.expectedPayout;
```
  https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/LotterySetup.sol#L86-L89

### [L-04] Unbounded Loop when buying an excessive amount of tickets

Since tickets are only 1.5 DAI (for now), some whales have the capability to buy an extremely huge number of tickets at one go. The contract should be able to take the large purchase without reverting due to excessive gas.

The buyTickets() function in Lottery.sol loops through the drawId length, which refers to the number of tickets that a player wants to buy.
```
        ticketIds = new uint256[](tickets.length);
        for (uint256 i = 0; i < drawIds.length; ++i) {
            ticketIds[i] = registerTicket(drawIds[i], tickets[i], frontend, referrer);
        }
```
If the drawIds.length is too high (ie 100_000 tickets bought by the same person at one go), the function will revert due to reaching maximum gas limit. Consider implementing try catch, require a max number of tickets bought at one go and/or breaking up the loop into smaller loops so that each execution can be successful.

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L124-L127

### [L-05] Use safeMint() instead of mint() for ERC721 tokens

safeMint is there to prevent someone minting ERC721 to a contract which does not support ERC721 transfer. So the ERC721 token is stuck there forever. Since this lottery system allows anyone to buy a ticket, the players also includes smart contracts. If the smart contract doea not have onERC721Received, it will not be able to mint a ERC721 token and the token will be stuck there forever.
```
    function mint(address to, uint128 drawId, uint120 combination) internal returns (uint256 ticketId) {
        ticketId = nextTicketId++;
        ticketsInfo[ticketId] = TicketInfo(drawId, combination, false);
        _mint(to, ticketId);
    }
```
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Ticket.sol#L23-L27

### [L-06] Frontend address should check for address(0)

When buying a ticket, make sure the frontend address cannot be address(0) otherwise no one can claim the frontend rewards.
```
    function buyTickets(
        uint128[] calldata drawIds,
        uint120[] calldata tickets,
        address frontend,
        address referrer
    )
```
https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/Lottery.sol#L110-L115

## Non-Critical Issues

### [N-01] Use require instead of assert

Assert should not be used except for tests, require should be used.

Prior to Solidity 0.8.0, pressing a confirm consumes the remainder of the process’s available gas instead of returning it, as request()/revert() did.

assert() and require(); The big difference between the two is that the assert()function when false, uses up all the remaining gas and reverts all the changes made. Meanwhile, a require() function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay. This is the most common Solidity function used by developers for debugging and error handling.

Assertion() should be avoided even after solidity version 0.8.0, because its documentation states “The Assert function generates an error of type Panic(uint256).Code that works properly should never Panic, even on invalid external input. If this happens, you need to fix it in your contract. There’s a mistake”.

            assert((winTier <= selectionSize) && (intersection == uint256(0)));

https://github.com/code-423n4/2023-03-wenwin/blob/91b89482aaedf8b8feb73c771d11c257eed997e8/src/TicketUtils.sol#L99

### [N-02] All functions should have NATSPEC comments in the interface file

In Lottery.sol, there are 3 functions that do not have any comments in the interface file. Even if the function is extremely clear cut, some comments on every function is always appreciated
```
    function returnUnclaimedJackpotToThePot() private {
        if (currentDraw >= LotteryMath.DRAWS_PER_YEAR) {
            uint128 drawId = currentDraw - LotteryMath.DRAWS_PER_YEAR;
            uint256 unclaimedJackpotTickets = unclaimedCount[drawId][winningTicket[drawId]];
            currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);
        }
    }


    function requireFinishedDraw(uint128 drawId) internal view override {
        if (drawId >= currentDraw) {
            revert DrawNotFinished(drawId);
        }
    }
```
```
    function mintNativeTokens(address mintTo, uint256 amount) internal override {
        ILotteryToken(address(nativeToken)).mint(mintTo, amount);
    }

}
```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L271-L288

### [N-03] Critical functions should emit an event

In Lottery.sol, there are some critical functions like claimWinningTickets() and claimable() that do not emit anything. Most, if not all functions should emit an event for the sake of clarity.

```
    function claimable(uint256 ticketId) external view override returns (uint256 claimableAmount, uint8 winTier) {
        TicketInfo memory ticketInfo = ticketsInfo[ticketId];
        if (!ticketInfo.claimed) {
            uint120 _winningTicket = winningTicket[ticketInfo.drawId];
            winTier = TicketUtils.ticketWinTier(ticketInfo.combination, _winningTicket, selectionSize, selectionMax);
            if (block.timestamp <= ticketRegistrationDeadline(ticketInfo.drawId + LotteryMath.DRAWS_PER_YEAR)) {
                claimableAmount = winAmount[ticketInfo.drawId][winTier];
            }
        }
    }
```

```
    function claimWinningTickets(uint256[] calldata ticketIds) external override returns (uint256 claimedAmount) {
        uint256 totalTickets = ticketIds.length;
        for (uint256 i = 0; i < totalTickets; ++i) {
            claimedAmount += claimWinningTicket(ticketIds[i]);
        }
        rewardToken.safeTransfer(msg.sender, claimedAmount);
    }
```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L159-L176

### [N-04] Increase Code Coverage
The coverage in some contracts is not 100%, such as the LotterySetup.sol which has test coverage of only 71.43%. Aim to reach a total of 100% (now 80.69%).

### [N-05] For modern and more readable code; update import usages
Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it. This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.

For example, in Lottery.sol, the import is as such:
```

import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import "@openzeppelin/contracts/utils/math/Math.sol";
import "src/ReferralSystem.sol";
import "src/RNSourceController.sol";
import "src/staking/Staking.sol";
import "src/LotterySetup.sol";
import "src/TicketUtils.sol";
A good example from the ArtGobblers project;

import {Owned} from "solmate/auth/Owned.sol";
import {ERC721} from "solmate/tokens/ERC721.sol";
import {LibString} from "solmate/utils/LibString.sol";
import {MerkleProofLib} from "solmate/utils/MerkleProofLib.sol";
import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
import {ERC1155, ERC1155TokenReceiver} from "solmate/tokens/ERC1155.sol";
import {toWadUnsafe, toDaysWadUnsafe} from "solmate/utils/SignedWadMath.sol";

```
https://github.com/code-423n4/2023-03-wenwin/blob/main/src/TicketUtils.sol#L99

### [N-06] Lock pragma version
It is considered best practice to use a locked Solidity version, thereby only allowing compilation with a specific version. So the instances of ^0.8.0 should be changed to 0.8.0.

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol

### [N-07] Standardize pragma version
Some code uses 0.8.7, some uses 0.8.17 whereas most uses 0.8.19. Use consistent pragma version (ideally 0.8.17) for best practice so that a complication can only be attributed to one specific version

https://github.com/code-423n4/2023-03-wenwin/blob/main/src/VRFv2RNSource.sol https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol

### [N-08] Function writing that does not comply with the Solidity Style Guide
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this. All contracts.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

- constructor
- receive function (if exists)
- fallback function (if exists)
- external
- public
- internal
- private
- within a grouping, place the view and pure functions last
```
