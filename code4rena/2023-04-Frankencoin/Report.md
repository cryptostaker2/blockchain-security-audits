# Full Report

## Medium Findings:

### [M-01] Challengers and bidders can collude together to restrict the minting of position owner

### Lines of code
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L312

### Proof of Concept
There is no restrictions as to how many challenges can occur at one given auction time. A challenger can potentially create an insanely large amount of challenges with a tiny amount of collateral for each challenge.

    function launchChallenge(address _positionAddr, uint256 _collateralAmount) external validPos(_positionAddr) returns (uint256) {
        IPosition position = IPosition(_positionAddr);
        IERC20(position.collateral()).transferFrom(msg.sender, address(this), _collateralAmount);
        uint256 pos = challenges.length;
        challenges.push(Challenge(msg.sender, position, _collateralAmount, block.timestamp + position.challengePeriod(), address(0x0), 0));
        position.notifyChallengeStarted(_collateralAmount);
        emit ChallengeStarted(msg.sender, address(position), _collateralAmount, pos);
        return pos;
    }
Let's say if the fair price of 1000 ZCHF is 1 WETH, and the position owner sets his position at the fair price. Rationally, there will be no challenges and bidders because the price is fair. However, the challenger can attack the position owner this way:

Set a small amount of collateralAmount to challenge, ie 0.0001 WETH.
The bidder comes and bid a price over 1000* ZCHF (Not the actual amount*. The actual amount is an equivalent amount scaled to the collateral, but for simplicity sake let's just say 1000 ZCHF, ideally its like 1000 * 0.0001 worth)
Because the bid is higher than 1000* ZCHF, tryAvertChallenge() succeeds and the bidder buys the collateral from the challenger.
When tryAvertChallenge() succeeds, restrictMinting(1 days) is called to suspend the owner from minting for 1 additional day
If the challenger and the bidder is colluding or even the same person, then the bidder does not lose out because he is essentially buying the collateral for a higher price from himself.
The challenger and bidder can repeat this attack and suspend the owner from minting. Such attack is possible because there is nothing much to lose other than a small amount of gas fees
```
    function tryAvertChallenge(uint256 _collateralAmount, uint256 _bidAmountZCHF) external onlyHub returns (bool) {
        if (block.timestamp >= expiration){
            return false; // position expired, let every challenge succeed
        } else if (_bidAmountZCHF * ONE_DEC18 >= price * _collateralAmount){
            // challenge averted, bid is high enough
            challengedAmount -= _collateralAmount;
            // Don't allow minter to close the position immediately so challenge can be repeated before
            // the owner has a chance to mint more on an undercollateralized position
//@audit-- calls restrictMinting if passed
            restrictMinting(1 days);
            return true;
        } else {
            return false;
        }
    }
    function restrictMinting(uint256 period) internal {
        uint256 horizon = block.timestamp + period;
        if (horizon > cooldown){
            cooldown = horizon;
        }
    }
```
### Impact

Minting for Position owner will be suspended for a long time.

### Tools Used
VSCode

### Recommended Mitigation Steps
In this Frankencoin protocol, the challenger never really loses.

If the bid ends lower than the liquidation price, then the bidder wins because he bought the collateral at a lower market value. The challenger also wins because he gets his reward. The protocol owner loses because he sold his collateral at a lower market value.

If the bid ends higher than the liquidation price, then the bidder loses because he bought the collateral at a higher market value. The challenger wins because he gets to trade his collateral for a higher market value. The protocol owner neither wins nor loses.

The particular POC above is one way a challenger can abuse his power to create many challenges without any sort of consequence in order to attack the owner. In the spirit of fairness, the challenger should also lose if he challenges wrongly.

Every time a challenger issues a challenge, he should pay a small fix sum of money that will go to the owner if the bidder sets an amount higher than fair market value. (because that means that the protocol owner was right about the fair market value all along.)

Although the position owner can be a bidder himself, if the position owner bids on his own position in order to win this small amount of money, the position owner will lose at the same time because he is buying the collateral at a higher-than-market price from the challenger, so this simultaneous gain and loss will balance out.

### [M-02] Challenger will not be able to get back their collateral if the collateral from the owner's position does not accept zero value transfer 

### Lines of code
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L268-L276


### Impact
Challenger's collateral will be stuck in contract if collateral does not accept zero value transfer.

### Proof of Concept
Assume that when a position is created, the _initialCollateral is equal to the _minCollateral. This means that the position owner only deposits _initialCollateral amount into the new position created.

MintingHub.sol#openPosition()

        IERC20(_collateralAddress).transferFrom(msg.sender, address(pos), _initialCollateral);
Now, we know that there can be more than one challenge occuring at the same time. Let's say 3 challenges occur, position owner has 1 ETH max, and the 3 challengers put up 1 ETH, 1 ETH and 0.5 ETH respectively. launchChallenge will check whether the challenger's collateral size must be more than minimumCollateral and the challenger's size must be more than the collateralBalance inside the position, which is true at the start of the challenge.

    function launchChallenge(address _positionAddr, uint256 _collateralAmount) external validPos(_positionAddr) returns (uint256) {
        IPosition position = IPosition(_positionAddr);
        IERC20(position.collateral()).transferFrom(msg.sender, address(this), _collateralAmount);
        uint256 pos = challenges.length;
        challenges.push(Challenge(msg.sender, position, _collateralAmount, block.timestamp + position.challengePeriod(), address(0x0), 0));
        position.notifyChallengeStarted(_collateralAmount);
        emit ChallengeStarted(msg.sender, address(position), _collateralAmount, pos);
        return pos;
    }
    function notifyChallengeStarted(uint256 size) external onlyHub {
        // require minimum size, note that collateral balance can be below minimum if it was partially challenged before
        if (size < minimumCollateral && size < collateralBalance()) revert ChallengeTooSmall();
        challengedAmount += size;
    }
Assuming challenge has started and the first challenge has passed its challenge duration. The bidders in challenge 1 will get his fair share of collateral from the position owner, and the position will be left with no more collateral.

Now, challenge two finishes. Challenger 2 wants to collect his collateral and he calls end(), which calls position.notifyChallengeSucceeded().

        (address owner, uint256 effectiveBid, uint256 volume, uint256 repayment, uint32 reservePPM) = challenge.position.notifyChallengeSucceeded(recipient, challenge.bid, challenge.size);
In notifyChallengeSucceeded, because _size is greater than colBal, (1 ETH > 0), the _bid will be redimension. However, _bid will equal to 0 because _divD18(_mulD18(_bid, 0), 1ETH) equals to zero. The _size will be equal to colBal.

Next, internalWithdrawCollateral will be called, with _bidder as the _bidder and _size as 0.

        internalWithdrawCollateral(_bidder, _size); // transfer collateral to the bidder and emit update
However, if the collateral is unable to perform zero value transfers, then internalWithdrawCollateral will fail.

    function internalWithdrawCollateral(address target, uint256 amount) internal returns (uint256) {
        IERC20(collateral).transfer(target, amount);
        uint256 balance = collateralBalance();
        if (balance < minimumCollateral){
            cooldown = expiration;
        }
        emitUpdate();
        return balance;
    }
### Tools Used
Manual Review

### Recommended Mitigation Steps
Make sure the collateral accepts zero value transfer as well so that the whole end() function can execute properly. Otherwise, skip the transfer.

    function internalWithdrawCollateral(address target, uint256 amount) internal returns (uint256) {
        if(amount > 0){
        IERC20(collateral).transfer(target, amount);
}
        uint256 balance = collateralBalance();
        if (balance < minimumCollateral){
            cooldown = expiration;
        }
        emitUpdate();
        return balance;
    }