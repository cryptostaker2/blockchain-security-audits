# Full Report

## Medium Findings:

### [M-01] The winning bid of the auction in Auction.sol can be frontrunned or the last bidder can block-stuff to win the bid

### Lines of code

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/Auction.sol#L80-L91

### Impact

Last bid can be frontrunned and/or block-stuffed. Attacker can game the auction system.

Proof of Concept
In Auction.sol, a user calls addBid() and deposits ETH into the contract. The bid amount must be higher than the previous bid in order for the bid to successfully go through. The bid must also not be greater than the endBlock.

    function addBid(uint256 lotId) external payable override whenNotPaused {
        // reject payments of 0 ETH
        if (msg.value == 0) revert InSufficientETH();

        LotItem storage lotItem = lots[lotId];
        if (block.number > lotItem.endBlock) revert AuctionEnded();

        uint256 totalUserBid = lotItem.bids[msg.sender] + msg.value;

        if (totalUserBid < lotItem.highestBidAmount + bidIncrement) revert InSufficientBid();

        lotItem.highestBidder = msg.sender;
        lotItem.highestBidAmount = totalUserBid;
        lotItem.bids[msg.sender] = totalUserBid;

        emit BidPlaced(lotId, msg.sender, totalUserBid);
    }

Since there is no increment to the endBlock once a bid starts, a user can frontrun the bid at the very last moment in order to become the highest bidder. Worst still, the user who is currently the highest bidder can execute a block stuffing attack to ensure that he will win the auction.

The highestBidder then calls claimSD() to claim the SD token auctioned.

    function claimSD(uint256 lotId) external override {
        LotItem storage lotItem = lots[lotId];
        if (block.number <= lotItem.endBlock) revert AuctionNotEnded();
        if (msg.sender != lotItem.highestBidder) revert notQualified();
        if (lotItem.sdClaimed) revert AlreadyClaimed();

        lotItem.sdClaimed = true;
        if (!IERC20(staderConfig.getStaderToken()).transfer(lotItem.highestBidder, lotItem.sdAmount)) {
            revert SDTransferFailed();
        }
        emit SDClaimed(lotId, lotItem.highestBidder, lotItem.sdAmount);
    }

Setting as Medium severity because:

Frontrunning is possible to ensure a winning bid
Block-stuffing is possible, although it is costly in ETH and the attacker has to ensure that the SD value in the auction is favourable.

### Tools Used

Remix IDE

### Recommended Mitigation Steps

Recommend increasing the block.number of the endBlock by a few blocks after every bid in the last few blocks before the auction ends.

### [M-02] StaderOracle#latestRoundData() may return a stale price

### Lines of code

https://github.com/code-423n4/2023-06-stader/blob/7566b5a35f32ebd55d3578b8bd05c038feb7d9cc/contracts/StaderOracle.sol#L637-L651

### Impact

Stale price may be used.

### Proof of Concept

In StaderOracle#getPORFeedData(), the function uses Chainlink's latestRoundData API but only used one return variable, totalETHBalanceInInt / totalETHXSupplyInInt. Both RoundId and the updatedAt timing is not checked, which may lead to a stale price.

    function getPORFeedData()
        internal
        view
        returns (
            uint256,
            uint256,
            uint256
        )
    {
        (, int256 totalETHBalanceInInt, , , ) = AggregatorV3Interface(staderConfig.getETHBalancePORFeedProxy())
            .latestRoundData();
        (, int256 totalETHXSupplyInInt, , , ) = AggregatorV3Interface(staderConfig.getETHXSupplyPORFeedProxy())
            .latestRoundData();
        return (uint256(totalETHBalanceInInt), uint256(totalETHXSupplyInInt), block.number);
    }

### Tools Used

Manual Review

### Recommended Mitigation Steps

Add additional checks to check for price stalesness.

```
    {
+       (uint80 roundID, int256 totalETHBalanceInInt, ,uint256 updatedAt ,uint80 answeredInRound ) = AggregatorV3Interface(staderConfig.getETHBalancePORFeedProxy())
            .latestRoundData();
+       require(answeredInRound >= roundId, "answer is stale");
+       require(updatedAt > 0, "round is incomplete");
+       require(totalETHBalanceInInt > 0, "Invalid feed answer");

+       (uint80 roundID2, int256 totalETHXSupplyInInt, ,uint256 updatedAt2 ,uint80 answeredInRound2) = AggregatorV3Interface(staderConfig.getETHXSupplyPORFeedProxy())
            .latestRoundData();
+       require(answeredInRound2 >= roundId2, "answer is stale");
+       require(updatedAt2 > 0, "round is incomplete");
+       require(totalETHXSupplyInInt > 0, "Invalid feed answer");

        return (uint256(totalETHBalanceInInt), uint256(totalETHXSupplyInInt), block.number);
    }
```
