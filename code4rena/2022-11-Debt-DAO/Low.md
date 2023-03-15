## Table of Contents
- [L-01] Best Practices to prevent Re-Entrancy
- [N-01] Avoid using sensitive terms
- [N-02] Constants should be defined rather than using magic numbers
- [N-03] if( should be if ( to match other lines in the file
- [N-04] Typos
- [N-05] Unused/Empty receive()/fallback() function
### [L-01] Best Practices to prevent Re-Entrancy
Follow the checks  - effects - interaction practice when writing functions. In addCollateral(), the token is transferred to LineLib first before the amount in self.deposited is changed. It should be the other way round

https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/utils/EscrowLib.sol#L87-L101

### [N-01] Avoid using sensitive terms
Use alternative variants, e.g. allowlist/denylist instead of whitelist/blacklist

https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/modules/credit/SpigotedLine.sol#L235-L241

### [N-02] Constants should be defined rather than using magic numbers
https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/utils/EscrowLib.sol#L42-L43

### [N-03] if( should be if ( to match other lines in the file
For universal coding style

https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/modules/credit/LineOfCredit.sol#L65 https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/modules/credit/LineOfCredit.sol#L124

### [N-04] Typos
interset -> interest

https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/utils/CreditLib.sol#L123

bwithdrawn -> withdrawn

https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/utils/CreditLib.sol#L200

repais -> repaid

https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/interfaces/ILineOfCredit.sol#L169

priviliegad -> privileged 

https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/modules/credit/EscrowedLine.sol#L85

https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/modules/credit/EscrowedLine.sol#L74

https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/modules/credit/EscrowedLine.sol#L45