# Full Report

## High Findings:

### [H-01] Bidder's payment is not refunded if Auction is cancelled prematurely via cancelAuction()

### Summary

If the auction creator decides to cancel the Auction when bidding is on going and bid is still less than reserve price, the last bidder who have already bid for a price will not get his money back.

### Vulnerability Detail

When bidders bid for a price in createBid(), their payment is handled through \_handleOutGoingPayment and \_handleIncomingPayment. The logic of both \_handle functions is such that the current bidder is supposed to pay for the lastBidder's refund and the remaining amount is transferred to \_handleIncomingPayment to pay off the lien debts (if any).

Eg Bid is currently at $500

- Alice decides to bid $650
- The bid is 5% above $500, so bid succeeds, and \_handleOutGoingPayment is called
- \_handleOutGoingPayment uses address(msg.sender) as a from and address to as a to, which means that Alice (the msg.sender) has to pay $500 to the previous bidder

```
  function _handleOutGoingPayment(address to, uint256 amount) internal {
    TRANSFER_PROXY.tokenTransferFrom(weth, address(msg.sender), to, amount);
  }
```

- With the remaining $150, \_handleIncomingPayment is called
- \_handleIncomingPayment pays initiatorFee first before paying back the loan

```
        if (transferAmount >= lien.amount) {
          payment = lien.amount;
          transferAmount -= payment;
        } else {
          payment = transferAmount;
          transferAmount = 0;
        }


        if (payment > 0) {
          LIEN_TOKEN.makePayment(tokenId, payment, lien.position, payer);
        }
```

- If there is no loan, the $150 goes to the owner of the NFT

```
    } else {
      TRANSFER_PROXY.tokenTransferFrom(
        weth,
        payer,
        COLLATERAL_TOKEN.ownerOf(tokenId),
        transferAmount
      );
    }
```

For Alice to get her money back, she has to either get outbid by another bidder (so \_handleOutGoingPayment will pay her back), or she wins the auction and wins the NFT. However, the auction owner can cancel the auction by calling cancelAuction(). If that happens, Alice will not get her bid money back.

### Impact

A bidder will lose all his deposit if the Auction owner decides to cancel the Auction.

### Code Snippet

https://github.com/sherlock-audit/2022-10-astaria/blob/main/lib/astaria-gpl/src/AuctionHouse.sol#L109-L116

### Tool used

Manual Review

### Recommendation

Before cancelling the auction, make sure lastBidder is paid with the proper refund amount.

## Medium Findings:

### [M-01] AuctionHouse firstBidTime is set to block.timestamp which breaks the calculation of the Auction's total duration

### Summary

AuctionHouse.sol firstBidTime is initialized to block.timestamp instead of 0, which makes the duration of the Auction incorrect.

### Vulnerability Detail

```
newAuction.firstBidTime = block.timestamp.safeCastTo64();
```

Since firstBidTime is set to block.timestamp and there is no bidder yet when the Auction starts, this if conditional will never run.

```
if (firstBidTime == 0) {
  auctions[tokenId].firstBidTime = block.timestamp.safeCastTo64();
} else if (lastBidder != address(0)) {
  uint256 lastBidderRefund = amount - vaultPayment;
  _handleOutGoingPayment(lastBidder, lastBidderRefund);
}
```

The firstBidTime thus acts as the start time of the Auction instead of the time of the first bid, which defies the protocol logic

```
// If this is the first valid bid, we should set the starting time now.
```

### Impact

Auction timing might cause confusion for bidders.

### Code Snippet

https://github.com/sherlock-audit/2022-10-astaria/blob/main/lib/astaria-gpl/src/AuctionHouse.sol#L80

### Tool used

Manual Review

### Recommendation

Change Line 80

```
newAuction.firstBidTime = block.timestamp.safeCastTo64();
```

to

```
newAuction.firstBidTime = 0;
```

Also make sure that the code in endAuction() is changed because if firstBidTime is 0, the auction can end immediately after it starts.

```
require(
  ++ auctions[auctionId].firstBidTime !== 0 &&
  block.timestamp >= auctions[auctionId].firstBidTime + auctions[auctionId].duration,
  "Auction hasn't completed"
);
```

### [M-02] The calculation of extended Auction timing when a bidder calls createBid() is incorrect

### Summary

When a bidder wants to call for a bid, the protocol makes sure that the bid lasts for 15 minutes. If the bidder calls for a bid 5 minutes before the end duration, the duration will be extended for 10 minutes to accommodate the 15 minutes buffer. This extension will run until max duration is reached. After that, the bidder cannot call for bids anymore. However, the code incorrectly calculates the extension time.

### Vulnerability Detail

```
if (firstBidTime + duration - block.timestamp < timeBuffer) {
```

In createBid(), a bidder calls for a bid and passes through some checks. The first check checks if firstBidTime + duration - block.timestamp < timeBuffer. Since the variables are a little hard to picture, let's take a real life example.

firstBidTime = Monday 8am
duration = 1 day
block.timestamp = Tuesday 7.50am (the time when the bidder calls his bid)
timeBuffer = 15 minutes (never change)

Using the variables above, Since Monday 8am + 1 day - Tuesday 7.50am < 15 minutes, or rather

Monday 8am + 1 day = Tuesday 8am
Tuesday 8am - Tuesday 7.50am = 10 minutes

Since 10 minutes < 15 minutes, the following code will run to calculate for the newDuration.

```
if (firstBidTime + duration - block.timestamp < timeBuffer) {
  uint64 newDuration = uint256(
    duration + (block.timestamp + timeBuffer - firstBidTime)
  ).safeCastTo64();
```

duration + (block.timestamp + timeBuffer - firstBidTime),
1 day + (Tuesday 7.50am + 15 minutes - Monday 8am)
1 day + (Tuesday 8.05am - Monday 8am)
1 day + (1 day 5 minutes)
2 day 5 minutes

newDuration is therefore 1 day 5 minutes. Next, the code will check if newDuration <= auctions[tokenId].maxDuration, where maxDuration is essentially duration + 1 day.

```
newAuction.maxDuration = (duration + 1 days).safeCastTo64();

if (newDuration <= auctions[tokenId].maxDuration)
```

Since 2 day 5 minutes is not less than 2 days, the following else statement will run.

```
    auctions[tokenId].duration =
      auctions[tokenId].maxDuration -
      firstBidTime;
```

duration = 2 days - firstBidTime
duration = 2 days - Monday 8am, which is an underflow and function will revert.

```
if (newDuration <= auctions[tokenId].maxDuration) {
    auctions[tokenId].duration = newDuration;
  } else {
    auctions[tokenId].duration =
      auctions[tokenId].maxDuration -
      firstBidTime;
  }
```

To summarize, the function will always revert because duration is calculated incorrectly which will lead to an underflow.

### Impact

Incorrect calculation of function will result in underflow. When bidder has 15 minutes left before the Auction ends, he cannot bid anymore as max duration and duration does not work as intended.

### Code Snippet

https://github.com/sherlock-audit/2022-10-astaria/blob/main/lib/astaria-gpl/src/AuctionHouse.sol#L127-L154

### Tool used

Manual Review

### Recommendation

Change

```
    duration + (block.timestamp + timeBuffer - firstBidTime)
```

to

```
   (block.timestamp + timeBuffer - firstBidTime) - duration
```

Also, change

```
 if (newDuration <= auctions[tokenId].maxDuration) {
    auctions[tokenId].duration = newDuration;
  } else {
```

to

```
 if (newDuration <= auctions[tokenId].maxDuration) {
    auctions[tokenId].duration += newDuration;
  } else {
```

Essentially, newDuration is the extended time given and duration should be added together with newDuration, up until duration reaches maxDuration.
