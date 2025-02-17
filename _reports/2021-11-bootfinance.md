---
sponsor: "Boot Finance"
slug: "2021-11-bootfinance"
date: "2022-01-21" 
title: "Boot Finance contest"
findings: "https://github.com/code-423n4/2021-11-bootfinance-findings/issues"
contest: 51
---

# Overview

## About C4

Code4rena (C4) is an open organization consisting of security researchers, auditors, developers, and individuals with domain expertise in smart contracts.

A C4 code contest is an event in which community participants, referred to as Wardens, review, audit, or analyze smart contract logic in exchange for a bounty provided by sponsoring projects.

During the code contest outlined in this document, C4 conducted an analysis of Boot Finance contest smart contract system written in Solidity. The code contest took place between November 4—November 10 2021.

## Wardens

28 Wardens contributed reports to the Boot Finance contest:

1. [jonah1005](https://twitter.com/jonah1005w)
1. WatchPug ([jtp](https://github.com/jack-the-pug) and [ming](https://github.com/mingwatch))
1. pants
1. [pauliax](https://twitter.com/SolidityDev)
1. Reigada
1. [Meta0xNull](https://twitter.com/Meta0xNull)
1. [cmichel](https://twitter.com/cmichelio)
1. [gpersoon](https://twitter.com/gpersoon)
1. [gzeon](https://twitter.com/gzeon)
1. [leastwood](https://twitter.com/liam_eastwood13)
1. [fr0zn](https://twitter.com/_fr0zn)
1. [defsec](https://twitter.com/defsec_)
1. [JMukesh](https://twitter.com/MukeshJ_eth)
1. 0v3rf10w
1. hyh
1. [nathaniel](https://twitter.com/n4th4n131?t&#x3D;ZXGbALC3q6JMMoolZddgHg&amp;s&#x3D;09)
1. elprofesor
1. [Ruhum](https://twitter.com/0xruhum)
1. mics
1. [ye0lde](https://twitter.com/_ye0lde)
1. [loop](https://twitter.com/loop_225)
1. [rfa](https://www.instagram.com/riyan_rfa/)
1. [TomFrenchBlockchain](https://github.com/TomAFrench)
1. [tqts](https://tqts.ar/)
1. [pmerkleplant](https://twitter.com/merkleplant_eth)
1. 0x0x0x
1. PranavG
1. [jah](https://twitter.com/jah_s3)

This contest was judged by [0xean](https://github.com/0xean).

Final report assembled by [itsmetechjay](https://twitter.com/itsmetechjay) and [CloudEllie](https://twitter.com/CloudEllie1).

# Summary

The C4 analysis yielded an aggregated total of 55 unique vulnerabilities and 156 total findings. All of the issues presented here are linked back to their original finding.

Of these vulnerabilities, 9 received a risk rating in the category of HIGH severity, 12 received a risk rating in the category of MEDIUM severity, and 34 received a risk rating in the category of LOW severity.

C4 analysis also identified 39 non-critical recommendations and 62 gas optimizations.

# Scope

The code under review can be found within the [C4 Boot Finance contest repository](https://github.com/code-423n4/2021-11-bootfinance), and is composed of 27 smart contracts written in the Solidity programming language and includes 5588 lines of Solidity code and 107 lines of JavaScript.

# Severity Criteria

C4 assesses the severity of disclosed vulnerabilities according to a methodology based on [OWASP standards](https://owasp.org/www-community/OWASP_Risk_Rating_Methodology).

Vulnerabilities are divided into three primary risk categories: high, medium, and low.

High-level considerations for vulnerabilities span the following key areas when conducting assessments:

- Malicious Input Handling
- Escalation of privileges
- Arithmetic
- Gas use

Further information regarding the severity criteria referenced throughout the submission review process, please refer to the documentation provided on [the C4 website](https://code423n4.com).

# High Risk Findings (9)
## [[H-01] Contract BasicSale is missing an approve(address(vestLock), 2**256-1) call](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/135)
_Submitted by Reigada, also found by WatchPug_

#### Impact

As we can see in the contracts `AirdropDistribution` and `InvestorDistribution`, they both have the following `approve() call: mainToken.approve(address(vestLock), 2\*\*256-1);`
- <https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/AirdropDistribution.sol#L499>
- <https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/InvestorDistribution.sol#L80>

This is necessary because both contracts transfer tokens to the vesting contract by calling its `vest()` function:
- <https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/AirdropDistribution.sol#L544>
- <https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/AirdropDistribution.sol#L569>
- <https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/InvestorDistribution.sol#L134>
- <https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/InvestorDistribution.sol#L158>

The code of the `vest()` function in the Vesting contract performs a transfer from `msg.sender` to Vesting contract address -> `vestingToken.transferFrom(msg.sender, address(this), \_amount);`
<https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/Vesting.sol#L95>

Same is done in the BasicSale contract:
<https://github.com/code-423n4/2021-11-bootfinance/blob/main/tge/contracts/PublicSale.sol#L225>

The problem is that this contract is missing the `approve()` call. For that reason, the contract is totally useless as the function `\_withdrawShare()` will always revert with the following message:
revert reason: ERC20: transfer amount exceeds allowance. This means that all the `mainToken` sent to the contract would be stuck there forever. No way to retrieve them.

How this issue was not detected in the testing phase?
Very simple. The mock used by the team has an empty `vest()` function that performs no transfer call.
<https://github.com/code-423n4/2021-11-bootfinance/blob/main/tge/contracts/helper/MockVesting.sol#L10>

#### Proof of Concept

See below Brownie's custom output:
```
Calling -> publicsale.withdrawShare(1, 1, {'from': user2})
Transaction sent: 0x9976e4f48bd14f9be8e3e0f4d80fdb8f660afab96a7cbd64fa252510154e7fde
Gas price: 0.0 gwei   Gas limit: 6721975   Nonce: 5
BasicSale.withdrawShare confirmed (ERC20: transfer amount exceeds allowance)   Block: 13577532   Gas used: 323334 (4.81%)

Call trace for '0x9976e4f48bd14f9be8e3e0f4d80fdb8f660afab96a7cbd64fa252510154e7fde':
Initial call cost  \[21344 gas]
BasicSale.withdrawShare  0:3724  \[16114 / -193010 gas]
├── BasicSale.\_withdrawShare  111:1109  \[8643 / 63957 gas]
│   ├── BasicSale.\_updateEmission  116:405  \[53294 / 55739 gas]
│   │   └── BasicSale.getDayEmission  233:248  \[2445 gas]
│   ├── BasicSale.\_processWithdrawal  437:993  \[-7726 / -616 gas]
│   │   ├── BasicSale.getEmissionShare  484:859  \[4956 / 6919 gas]
│   │   │   │
│   │   │   └── MockERC20.balanceOf  \[STATICCALL]  616:738  \[1963 gas]
│   │   │           ├── address: mockerc20.address
│   │   │           ├── input arguments:
│   │   │           │   └── account: publicsale.address
│   │   │           └── return value: 100000000000000000000
│   │   │
│   │   └── SafeMath.sub  924:984  \[191 gas]
│   └── SafeMath.sub  1040:1100  \[191 gas]
│
├── MockERC20.transfer  \[CALL]  1269:1554  \[1115 / 30109 gas]
│   │   ├── address: mockerc20.address
│   │   ├── value: 0
│   │   ├── input arguments:
│   │   │   ├── recipient: user2.address
│   │   │   └── amount: 27272727272727272727
│   │   └── return value: True
│   │
│   └── ERC20.transfer  1366:1534  \[50 / 28994 gas]
│       └── ERC20.\_transfer  1374:1526  \[28944 gas]
└── Vesting.vest  \[CALL]  1705:3712  \[-330491 / -303190 gas]
│   ├── address: vesting.address
│   ├── value: 0
│   ├── input arguments:
│   │   ├── \_beneficiary: user2.address
│   │   ├── \_amount: 63636363636363636363
│   │   └── \_isRevocable: 0
│   └── revert reason: ERC20: transfer amount exceeds allowance <-------------
│
├── SafeMath.add  1855:1883  \[94 gas]
├── SafeMath.add  3182:3210  \[94 gas]
├── SafeMath.add  3236:3264  \[94 gas]
│
└── MockERC20.transferFrom  \[CALL]  3341:3700  \[99923 / 27019 gas]
│   ├── address: mockerc20.address
│   ├── value: 0
│   ├── input arguments:
│   │   ├── sender: publicsale.address
│   │   ├── recipient: vesting.address
│   │   └── amount: 63636363636363636363
│   └── revert reason: ERC20: transfer amount exceeds allowance
│
└── ERC20.transferFrom  3465:3700  \[-97648 / -72904 gas]
└── ERC20.\_transfer  3473:3625  \[24744 gas]
```
#### Tools Used

Manual testing

#### Recommended Mitigation Steps

The following `approve()` call should be added in the constructor of the BasicSale contract:
`mainToken.approve(address(vestLock), 2\*\*256-1);`

**[chickenpie347 (Boot Finance) confirmed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/135)** 

## [[H-02] Can not update target price](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/143)
_Submitted by jonah1005, also found by WatchPug_

#### Impact

The sanity checks in `rampTargetPrice` are broken
[SwapUtils.sol#L1571-L1581](https://github.com/code-423n4/2021-11-bootfinance/blob/main/customswap/contracts/SwapUtils.sol#L1571-L1581)

```solidity
        if (futureTargetPricePrecise < initialTargetPricePrecise) {
            require(
                futureTargetPricePrecise.mul(MAX_RELATIVE_PRICE_CHANGE).div(WEI_UNIT) >= initialTargetPricePrecise,
                "futureTargetPrice_ is too small"
            );
        } else {
            require(
                futureTargetPricePrecise <= initialTargetPricePrecise.mul(MAX_RELATIVE_PRICE_CHANGE).div(WEI_UNIT),
                "futureTargetPrice_ is too large"
            );
        }
```

If `futureTargetPricePrecise` is smaller than `initialTargetPricePrecise` 0.01 of `futureTargetPricePrecise` would never larger than `initialTargetPricePrecise`.

Admin would not be able to ramp the target price. As it's one of the most important features of the customswap, I consider this is a high-risk issue

#### Proof of Concept

Here's a web3.py script to demo that it's not possible to change the target price even by 1 wei.

```python
    p1, p2, _, _ =swap.functions.targetPriceStorage().call()
    future = w3.eth.getBlock(w3.eth.block_number)['timestamp'] + 200 * 24 * 3600

    # futureTargetPrice_ is too small
    swap.functions.rampTargetPrice(p1 -1, future).transact()
    # futureTargetPrice_ is too large
    swap.functions.rampTargetPrice(p1 + 1, future).transact()
```

#### Tools Used

None

#### Recommended Mitigation Steps

Would it be something like:

```solidity
        if (futureTargetPricePrecise < initialTargetPricePrecise) {
            require(
                futureTargetPricePrecise.mul(MAX_RELATIVE_PRICE_CHANGE + WEI_UNIT).div(WEI_UNIT) >= initialTargetPricePrecise,
                "futureTargetPrice_ is too small"
            );
        } else {
            require(
                futureTargetPricePrecise <= initialTargetPricePrecise.mul(MAX_RELATIVE_PRICE_CHANGE + WEI_UNIT).div(WEI_UNIT),
                "futureTargetPrice_ is too large"
            );
        }
```

I believe the dev would spot this mistake if there's a more relaxed timeline.

**[chickenpie347 (Boot Finance) confirmed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/143)** 

## [[H-03] `SwapUtils.sol` Wrong implementation](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/252)
_Submitted by WatchPug_

Based on the context, the `tokenPrecisionMultipliers` used in price calculation should be calculated in realtime based on `initialTargetPrice`, `futureTargetPrice`, `futureTargetPriceTime` and current time, just like `getA()` and `getA2()`.

However, in the current implementation, `tokenPrecisionMultipliers` used in price calculation is the stored value, it will only be changed when the owner called `rampTargetPrice()` and `stopRampTargetPrice()`.

As a result, the `targetPrice` set by the owner will not be effective until another `targetPrice` is being set or `stopRampTargetPrice()` is called.

#### Recommendation

Consider adding `Swap.targetPrice` and changing the `_xp()` at L661 from:

<https://github.com/code-423n4/2021-11-bootfinance/blob/f102ee73eb320532c5a7c1e833f225c479577e39/customswap/contracts/SwapUtils.sol#L661-L667>

```solidity=661
function _xp(Swap storage self, uint256[] memory balances)
    internal
    view
    returns (uint256[] memory)
{
    return _xp(balances, self.tokenPrecisionMultipliers);
}
```

To:

```solidity=661
function _xp(Swap storage self, uint256[] memory balances)
    internal
    view
    returns (uint256[] memory)
{
    uint256[2] memory tokenPrecisionMultipliers = self.tokenPrecisionMultipliers;
    tokenPrecisionMultipliers[0] = self.targetPrice.originalPrecisionMultipliers[0].mul(_getTargetPricePrecise(self)).div(WEI_UNIT)
    return _xp(balances, tokenPrecisionMultipliers);
}
```

**[chickenpie347 (Boot Finance) confirmed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/252)**

## [[H-04] Swaps are not split when trade crosses target price](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/216)
_Submitted by cmichel, also found by gzeon_

The protocol uses two amplifier values A1 and A2 for the swap, depending on the target price, see `SwapUtils.determineA`.
The swap curve is therefore a join of two different curves at the target price.
When doing a trade that crosses the target price, it should first perform the trade partially with A1 up to the target price, and then the rest of the trade order with A2.

However, the `SwapUtils.swap / _calculateSwap` function does not do this, it only uses the "new A", see `getYC` step 5.

```solidity
// 5. Check if we switched A's during the swap
if (aNew == a){     // We have used the correct A
    return y;
} else {    // We have switched A's, do it again with the new A
    return getY(self, tokenIndexFrom, tokenIndexTo, x, xp, aNew, d);
}
```

#### Impact

Trades that cross the target price and would lead to a new amplifier being used are not split up and use the new amplifier for the *entire trade*.
This can lead to a worse (better) average execution price than manually splitting the trade into two transactions, first up to but below the target price, and a second one with the rest of the trader order size, using both A1 and A2 values.

In the worst case, it could even be possible to make the entire trade with one amplifier and then sell the swap result again using the other amplifier making a profit.

#### Recommended Mitigation Steps

Trades that lead to a change in amplifier value need to be split up into two trades using both amplifiers to correctly calculate the swap result.

**[chickenpie347 (Boot Finance) confirmed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/216)**

## [[H-05] Claim airdrop repeatedly](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/129)
_Submitted by gpersoon, also found by elprofesor, fr0zn, and pauliax_

#### Impact

Suppose someone claims the last part of his airdrop via `claimExact()` of `AirdropDistribution.sol`
Then `airdrop\[msg.sender].amount` will be set to 0.

Suppose you then call validate() again.
The check `airdrop\[msg.sender].amount == 0` will allow you to continue, because amount has just be set to 0.
In the next part of the function, `airdrop\[msg.sender]` is overwritten with fresh values and `airdrop\[msg.sender]`.claimed will be reset to 0.

Now you can claim your airdrop again (as long as there are tokens present in the contract)

Note: The function `claim()` prevents this from happening via `assert(airdrop\[msg.sender].amount - claimable != 0);`, which has its own problems, see other reported issues.

#### Proof of Concept

// <https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/vesting/contracts/AirdropDistribution.sol#L555-L563>

`function claimExact(uint256 \_value) external nonReentrant {
require(msg.sender != address(0));
require(airdrop\[msg.sender].amount != 0);`

        uint256 avail = _available_supply();
        uint256 claimable = avail * airdrop[msg.sender].fraction / 10**18; //
        if (airdrop[msg.sender].claimed != 0){
            claimable -= airdrop[msg.sender].claimed;
        }

        require(airdrop[msg.sender].amount >= claimable); // amount can be equal to claimable
        require(_value <= claimable);                       // _value can be equal to claimable
        airdrop[msg.sender].amount -= _value;      // amount will be set to 0 with the last claim

// <https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/vesting/contracts/AirdropDistribution.sol#L504-L517>
function validate() external nonReentrant {
...
require(airdrop\[msg.sender].amount == 0, "Already validated.");
...
Airdrop memory newAirdrop = Airdrop(airdroppable, 0, airdroppable, 10\*\*18 \* airdroppable / airdrop_supply);
airdrop\[msg.sender] = newAirdrop;
validated\[msg.sender] = 1;   // this is set, but isn't checked on entry of this function

#### Recommended Mitigation Steps

Add the following to `validate() :
require(validated\[msg.sender]== 0, "Already validated.");`

**[chickenpie347 (Boot Finance) confirmed and resolved](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/129#issuecomment-964720927):**
 > Addressed in issue #101 



## [[H-06] Ideal balance is not calculated correctly when providing imbalanced liquidity](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/150)
_Submitted by jonah1005_

#### Impact

When a user provides imbalanced liquidity, the fee is calculated according to the ideal balance. In saddle finance, the optimal balance should be the same ratio as in the Pool.

Take, for example, if there's 10000 USD and 10000 DAI in the saddle's USD/DAI pool, the user should get the optimal lp if he provides lp with ratio = 1.

However, if the `customSwap` pool is created with a target price = 2. The user would get 2 times more lp if he deposits DAI.
[SwapUtils.sol#L1227-L1245](https://github.com/code-423n4/2021-11-bootfinance/blob/main/customswap/contracts/SwapUtils.sol#L1227-L1245)
The current implementation does not calculates ideal balance correctly.

If the target price is set to be 10, the ideal balance deviates by 10.
The fee deviates a lot. I consider this is a high-risk issues.

#### Proof of Concept

We can observe the issue if we initiates two pools DAI/LINK pool and set the target price to be 4.

For the first pool, we deposit more DAI.

```python
    swap = deploy_contract('Swap' 
        [dai.address, link.address], [18, 18], 'lp', 'lp', 1, 85, 10**7, 0, 0, 4* 10**18)
    link.functions.approve(swap.address, deposit_amount).transact()
    dai.functions.approve(swap.address, deposit_amount).transact()
    previous_lp = lptoken.functions.balanceOf(user).call()
    swap.functions.addLiquidity([deposit_amount, deposit_amount // 10], 10, 10**18).transact()
    post_lp = lptoken.functions.balanceOf(user).call()
    print('get lp', post_lp - previous_lp)
```

For the second pool, one we deposit more DAI.

```python
    swap = deploy_contract('Swap' 
        [dai.address, link.address], [18, 18], 'lp', 'lp', 1, 85, 10**7, 0, 0, 4* 10**18)
    link.functions.approve(swap.address, deposit_amount).transact()
    dai.functions.approve(swap.address, deposit_amount).transact()
    previous_lp = lptoken.functions.balanceOf(user).call()
    swap.functions.addLiquidity([deposit_amount, deposit_amount // 10], 10, 10**18).transact()
    post_lp = lptoken.functions.balanceOf(user).call()
    print('get lp', post_lp - previous_lp)
```

We can get roughly 4x more lp in the first case

#### Tools Used

None

#### Recommended Mitigation Steps

The current implementation uses `self.balances`

<https://github.com/code-423n4/2021-11-bootfinance/blob/main/customswap/contracts/SwapUtils.sol#L1231-L1236>

```soliditiy
            for (uint256 i = 0; i < self.pooledTokens.length; i++) {
                uint256 idealBalance = v.d1.mul(self.balances[i]).div(v.d0);
                fees[i] = feePerToken
                    .mul(idealBalance.difference(newBalances[i]))
                    .div(FEE_DENOMINATOR);
                self.balances[i] = newBalances[i].sub(
                    fees[i].mul(self.adminFee).div(FEE_DENOMINATOR)
                );
                newBalances[i] = newBalances[i].sub(fees[i]);
            }
```

Replaces `self.balances` with `_xp(self, newBalances)` would be a simple fix.
I consider the team can take balance's weighted pool as a reference. [WeightedMath.sol#L149-L179](https://github.com/balancer-labs/balancer-v2-monorepo/blob/7ff72a23bae6ce0eb5b134953cc7d5b79a19d099/pkg/pool-weighted/contracts/WeightedMath.sol#L149-L179)

**[chickenpie347 (Boot Finance) confirmed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/150)**

## [[H-07] `customPrecisionMultipliers` would be rounded to zero and break the pool](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/183)
_Submitted by jonah1005_

#### Impact

`CustomPrecisionMultipliers` are set in the constructor:

```solidity
        customPrecisionMultipliers[0] = targetPriceStorage.originalPrecisionMultipliers[0].mul(_targetPrice).div(10 ** 18);
```

`originalPrecisionMultipliers` equal to 1 if the token's decimal = 18. The targe price could only be an integer.

If the target price is bigger than 10\*\*18, the user can deposit and trade in the pool. Though, the functionality would be far from the spec.

If the target price is set to be smaller than 10\*\*18, the pool would be broken and all funds would be stuck.

I consider this is a high-risk issue.

#### Proof of Concept

Please refer to the implementation.
[Swap.sol#L184-L187](https://github.com/code-423n4/2021-11-bootfinance/blob/main/customswap/contracts/Swap.sol#L184-L187)

We can also trigger the bug by setting a pool with target price = 0.5. (0.5 \* 10\*\*18)

#### Tools Used

None

#### Recommended Mitigation Steps

I recommend providing extra 10\*\*18 in both multipliers.

```solidity
        customPrecisionMultipliers[0] = targetPriceStorage.originalPrecisionMultipliers[0].mul(_targetPrice).mul(10**18).div(10 ** 18);
        customPrecisionMultipliers[1] = targetPriceStorage.originalPrecisionMultipliers[1].mul(10**18);
```

The customswap only supports two tokens in a pool, there's should be enough space. Recommend the devs to go through the trade-off saddle finance has paid to support multiple tokens. The code could be more clean and efficient if the pools' not support multiple tokens.

**[chickenpie347 (Boot Finance) confirmed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/183)**

## [[H-08] Unable to claim vesting due to unbounded timelock loop](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/120)
_Submitted by nathaniel, also found by WatchPug, leastwood, and pauliax_

#### Impact

The timelocks for any *beneficiary* are unbounded, and can be vested by someone who is not the *beneficiary*. When the array becomes significantly big enough, the vestments will no longer be claimable for the *beneficiary*.

The `vest()` function in Vesting.sol does not check the *beneficiary*, hence anyone can vest for anyone else, pushing a new timelock to the `timelocks[_beneficiary]`.
The `_claimableAmount()` function (used by `claim()` function), then loops through the `timelocks[_beneficiary]` to determine the amount to be claimed.
A malicious actor can easy repeatedly call the `vest()` function with minute amounts to make the array large enough, such that when it comes to claiming, it will exceed the gas limit and revert, rendering the vestment for the beneficiary unclaimable.
The malicious actor could do this to each *beneficiary*, locking up all the vestments.

#### Proof of Concept

- <https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/Vesting.sol#L81>
- <https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/Vesting.sol#L195>
- <https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/Vesting.sol#L148>

#### Tools Used

Manual code review

#### Recommended Mitigation Steps

*   Create a minimum on the vestment amounts, such that it won't be feasible for a malicious actor to create a large amount of vestments.
*   Restrict the vestment contribution of a *beneficiary* where `require(beneficiary == msg.sender)`

**[chickenpie347 (Boot Finance) confirmed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/120)**

## [[H-09] addInvestor() Does Not Check Availability of investors_supply](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/201)
_Submitted by Meta0xNull_

#### Impact

When add investor, `addInvestor()` does not check how many tokens is available from `investors_supply`. The total tokens allocated for Investors could more than `investors_supply`.

Possible Attack Scenario:

1.  Attacker who have Admin Private key call `addInvestor()` and `Input \_amount >= investors_supply`.
2.  Attacker can Claim All Available Tokens Now.

#### Proof of Concept

<https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/InvestorDistribution.sol#L85-L94>

#### Tools Used

Manual Review

#### Recommended

1.  Add `require(\_amount <= (investors_supply - Allocated_Amount))`
2.  When Add an Investor add the amount to `Allocated_Amount` with SafeMath

**[chickenpie347 (Boot Finance) acknowledged](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/201#issuecomment-970317628):**
 > While this is true, the addInvestor would be a one-time routine at deployment which would precisely send the allocated number of tokens to the contract as per to the allocatations.



 
# Medium Risk Findings (12)
## [[M-01] Unchecked transfers](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/31)
_Submitted by Reigada, also found by Ruhum, loop, cmichel, defsec, pauliax, WatchPug, and 0v3rf10w_

#### Impact

Multiple calls to `transferFrom` and transfer are frequently done without checking the results. For certain ERC20 tokens, if insufficient tokens are present, no revert occurs but a result of “false” is returned. It’s important to check this. If you don’t, in this concrete case, some airdrop eligible participants could be left without their tokens. It is also a best practice to check this.

#### Proof of Concept
```
AirdropDistributionMock.sol:132:        mainToken.transfer(msg.sender, claimable_to_send);
AirdropDistributionMock.sol:157:        mainToken.transfer(msg.sender, claimable_to_send);
AirdropDistribution.sol:542:        mainToken.transfer(msg.sender, claimable_to_send);
AirdropDistribution.sol:567:        mainToken.transfer(msg.sender, claimable_to_send);

InvestorDistribution.sol:132:        mainToken.transfer(msg.sender, claimable_to_send);
InvestorDistribution.sol:156:        mainToken.transfer(msg.sender, claimable_to_send);
InvestorDistribution.sol:207:        mainToken.transfer(msg.sender, bal);

Vesting.sol:95:        vestingToken.transferFrom(msg.sender, address(this), \_amount);

PublicSale.sol:224:            mainToken.transfer(\_member, v_value);
```
#### Tools Used

Manual testing

#### Recommended Mitigation Steps

Check the result of `transferFrom` and transfer. Although if this is done, the contracts will not be compatible with non standard ERC20 tokens like USDT. For that reason, I would rather recommend making use of SafeERC20 library: `safeTransfer` and `safeTransferFrom`.

**[chickenpie347 (Boot Finance) confirmed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/31)**

## [[M-02] Unchecked low-level calls](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/145)
_Submitted by 0v3rf10w, also found by Reigada_

#### Impact

Unchecked low-level calls

#### Proof of Concept

Unchecked cases at 2 places :-
`BasicSale.receive()` (2021-11-bootfinance/tge/contracts/PublicSale.sol#148-156) ignores return value by `burnAddress.call{value: msg.value}()` (2021-11-bootfinance/tge/contracts/PublicSale.sol#154)

BasicSale.burnEtherForMember(address) (2021-11-bootfinance/tge/contracts/PublicSale.sol#158-166) ignores return value by `burnAddress.call{value: msg.value}()` (2021-11-bootfinance/tge/contracts/PublicSale.sol#164)

#### Tools Used

Manual

#### Recommended Mitigation Steps

The return value of the low-level call is not checked, so if the call fails, the Ether will be locked in the contract. If the low level is used to prevent blocking operations, consider logging failed calls.

**[chickenpie347 (Boot Finance) confirmed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/145)**

## [[M-03] Investor can't claim the last tokens (via claim() )](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/131)
_Submitted by gpersoon_

#### Impact

Suppose you are an investor and want to claim the last part of your claimable tokens (or your entire set of claimable tokens if you haven't claimed anything yet).
Then you call the function `claim()` of `InvestorDistribution.sol`, which has the following statement:
`require(investors\[msg.sender].amount - claimable != 0);`
This statement will prevent you from claiming your tokens because it will stop execution.

Note: with the function `claimExact()` it is possible to claim the last part.

#### Proof of Concept

// <https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/vesting/contracts/InvestorDistribution.sol#L113-L128>

function `claim() external nonReentrant {
...
require(investors\[msg.sender].amount - claimable != 0);
investors\[msg.sender].amount -= claimable;`

#### Tools Used

#### Recommended Mitigation Steps

Remove the require statement.

**[chickenpie347 (Boot Finance) commented](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/131#issuecomment-964723021):**
 > Duplicate of issue #130 

**[chickenpie347 (Boot Finance) commented](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/131#issuecomment-964723644):**
 > I just noticed it's different files. The AirdropDistrbution.sol and InvestorDistribution.sol contracts were built on the same base, with slight changes.

**[0xean (judge) commented](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/131#issuecomment-1007869122):**
 > Downgrading to medium risk as an alternative path does exist for claiming the drop. Funds are not lost, but the availability of them is compromised. Per Docs:
> 
> ```
> 2 — Med: Assets not at direct risk, but the function of the protocol or its availability could be impacted, or leak value with a hypothetical attack path with stated assumptions, but external requirements.
> 3 — High: Assets can be stolen/lost/compromised directly (or indirectly if there is a valid attack path that does not have hand-wavy hypotheticals).
> ```



## [[M-04] Get virtual price is not monotonically increasing ](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/185)
_Submitted by jonah1005_

#### Impact

There's a feature of `virtualPrice` that is monotonically increasing regardless of the market. This function is heavily used in multiple protocols. e.g.(curve metapool, mim, ...) This is not held in the current implementation of customSwap since `customPrecisionMultipliers` can be changed by changing the target price.

There are two issues here:
The meaning of `virtualPrice` would be vague.
This may damage the lp providers as the protocol that adopts it may be hacked.

I consider this is a medium-risk issue.

#### Proof of Concept

We can set up a mockSwap with extra `setPrecisionMultiplier` to check the issue.

```solidity
    function setPrecisionMultiplier(uint256 multipliers) external {
        swapStorage.tokenPrecisionMultipliers[0] = multipliers; 
    }
```

```python
    print(swap.functions.getVirtualPrice().call())
    swap.functions.setPrecisionMultiplier(2).transact()
    print(swap.functions.getVirtualPrice().call())

# output log:
#   1000000000000000000
#   1499889859738721606
```

#### Tools Used

None

#### Recommended Mitigation Steps

Dealing with the target price with multiplier precision seems clever as we can reuse most of the existing code. However, the precision multiplier should be an immutable parameter. Changing it after the pool is set up would create multiple issues. This function could be implemented in a safer way IMHO.

The quick fix would be to remove the `getVirtualPrice` function. I can't come up with a safe way if other protocol wants to use this function.

**[chickenpie347 (Boot Finance) confirmed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/185)**

## [[M-05] Stop ramp target price would create huge arbitrage space.](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/208)
_Submitted by jonah1005_

### Stop ramp target price would create huge arbitrage space.

#### Impact

`stopRampTargetPrice` would set the `tokenPrecisionMultipliers` to `originalPrecisionMultipliers[0].mul(currentTargetPrice).div(WEI_UNIT);`
Once the `tokenPrecisionMultipliers` is changed, the price in the AMM pool would change. Arbitrager can sandwich `stopRampTargetPrice` to gain profit.

Assume the decision is made in the DAO, an attacker can set up the bot once the proposal to `stopRampTargetPrice` has passed. I consider this is a medium-risk issue.

#### Proof of Concept

The `precisionMultiplier` is set here:
[Swap.sol#L661-L666](https://github.com/code-423n4/2021-11-bootfinance/blob/main/customswap/contracts/Swap.sol#L661-L666)

We can set up a mockSwap with extra `setPrecisionMultiplier` to check the issue.

```solidity
    function setPrecisionMultiplier(uint256 multipliers) external {
        swapStorage.tokenPrecisionMultipliers[0] = multipliers; 
    }
```

```python
print(swap.functions.getVirtualPrice().call())
swap.functions.setPrecisionMultiplier(2).transact()
print(swap.functions.getVirtualPrice().call())

# output log:
#     1000000000000000000
#     1499889859738721606
```

#### Tools Used

None

#### Recommended Mitigation Steps

Dealing with the target price with multiplier precision seems clever as we can reuse most of the existing code. However, the precision multiplier should be an immutable parameter. Changing it after the pool is setup would create multiple issues. This function could be implemented in a safer way IMHO.

A quick fix I would come up with is to ramp the `tokenPrecisionMultipliers` as the `aPrecise` is ramped. As the `tokenPrecision` is slowly increased/decreased, the arbitrage space would be slower and the profit would (probably) distribute evenly to lpers.

Please refer to `_getAPreceise`'s implementation
[SwapUtils.sol#L227-L250](https://github.com/code-423n4/2021-11-bootfinance/blob/main/customswap/contracts/SwapUtils.sol#L227-L250)

**[chickenpie347 (Boot Finance) confirmed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/208)**

## [[M-07] `MainToken.set_mint_multisig()` doesn't check that `_minting_multisig` doesn't equal zero](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/66)
_Submitted by pants_

The function `MainToken.set_mint_multisig()` doesn't check that `_minting_multisig` doesn't equal zero before it sets it as the new `minting_multisig`.

#### Impact

This function can be invoked by mistake with the zero address as `_minting_multisig`, causing the system to lose its `minting_multisig` forever, without the option to set a new `minting_multisig`.

#### Tool Used

Manual code review.

#### Recommended Mitigation Steps

Check that `_minting_multisig` doesn't equal zero before setting it as the new `minting_multisig`.

**[chickenpie347 (Boot Finance) confirmed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/66)** 

## [[M-08] `LPToken.set_minter()` doesn't check that `_minter` doesn't equal zero](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/69)
_Submitted by pants_

The function `LPToken.set_minter()` doesn't check that `_minter` doesn't equal zero before it sets it as the new minter.

#### Impact

This function can be invoked by mistake with the zero address as `_minter`, causing the system to lose its minter forever, without the option to set a new minter.

#### Tool Used

Manual code review.

#### Recommended Mitigation Steps

Check that `_minter` doesn't equal zero before setting it as the new minter.

**[chickenpie347 (Boot Finance) confirmed)](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/69)**

## [[M-09] NFT flashloans can bypass sale constraints](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/276)
_Submitted by pauliax_

#### Impact

Public sale has a constraint that for the first 4 weeks only NFT holders can access the sale:
`if (currentEra < firstPublicEra) {
require(nft.balanceOf(msg.sender) > 0, "You need NFT to participate in the sale.");
}`

However, this check can be easily bypassed with the help of flash loans. You can borrow the NFT, participate in the sale and then return this NFT in one transaction. It takes only 1 NFT that could be flashloaned again and again to give access to the sale for everyone (`burnEtherForMember`).

#### Recommended Mitigation Steps

I am not sure what could be the most elegant solution to this problem. You may consider transferring and locking this NFT for at least 1 block but then the user will need to do an extra tx to retrieve it back. You may consider taking a snapshot of user balances so the same NFT can be used by one address only but then this NFT will lose its extra benefit of selling it during the pre-sale when it acts as a pre-sale token. You may consider checking that the caller is EOA but again there are ways to bypass that too.

**[chickenpie347 (Boot Finance) acknowledged](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/276)**

## [[M-10] Can't claim last part of airdrop](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/130)
_Submitted by gpersoon_

#### Impact

Suppose you are eligible for the last part of your airdrop (or your entire airdrop if you haven't claimed anything yet).
Then you call the function claim() of AirdropDistribution.sol, which has the following statement:
`assert(airdrop\[msg.sender].amount - claimable != 0);`
This statement will prevent you from claiming your airdrop because it will stop execution.

Note: with the function `claimExact()` it is possible to claim the last part.

#### Proof of Concept

// <https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/vesting/contracts/AirdropDistribution.sol#L522-L536>

function `claim() external nonReentrant {
..
assert(airdrop\[msg.sender].amount - claimable != 0);
airdrop\[msg.sender].amount -= claimable;`

#### Recommended Mitigation Steps

Remove the assert statement.
Also add the following to `validate()` , to prevent claiming the airdrop again:
`require(validated\[msg.sender]== 0, "Already validated.");`

**[chickenpie347 (Boot Finance) confirmed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/130#issuecomment-965227983):**
 > Patched it with `assert(airdrop[msg.sender].amount - claimable >= 0);` the >=0 check is just to ensure the claimant does not end up claiming more than allocated due to any fringe case.

**[0xean (judge) commented](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/130#issuecomment-1007869115):**
 > Downgrading to medium risk as an alternative path does exist for claiming the drop. Funds are not lost, but the availability of them is compromised. Per Docs:
> 
> ```
> 2 — Med: Assets not at direct risk, but the function of the protocol or its availability could be impacted, or leak value with a hypothetical attack path with stated assumptions, but external requirements.
> 3 — High: Assets can be stolen/lost/compromised directly (or indirectly if there is a valid attack path that does not have hand-wavy hypotheticals).
> ```



## [[M-11] Overwrite benRevocable](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/132)
_Submitted by gpersoon, also found by pauliax, WatchPug, cmichel, hyh, and leastwood_

#### Impact

Anyone can call the function `vest()` of `Vesting.sol`, for example with a smail `\_amount` of tokens, for any `\_beneficiary`.

The function overwrites the value of `benRevocable\[\_beneficiary]`, effectively erasing any previous value.

So you can set any `\_beneficiary` to Revocable.
Although `revoke()` is only callable by the owner, this is circumventing the entire mechanism of `benRevocable`.

#### Proof of Concept

// <https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/vesting/contracts/Vesting.sol#L73-L98>

function `vest(address \_beneficiary, uint256 \_amount, uint256 \_isRevocable) external payable whenNotPaused {
...
if(\_isRevocable == 0){
benRevocable\[\_beneficiary] = \[false,false];  // just overwrites the value
}
else if(\_isRevocable == 1){
benRevocable\[\_beneficiary] = \[true,false]; // just overwrites the value
}`

#### Recommended Mitigation Steps

Whitelist the calling of `vest()`
Or check if values for `benRevocable` are already set.

**[chickenpie347 (Boot Finance) confirmed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/132)**

## [[M-12] No Transfer Ownership Pattern](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/35)
_Submitted by defsec, also found by Ruhum, elprofesor, pauliax, Reigada, and mics_

#### Impact

The current ownership transfer process involves the current owner calling `Swap.transferOwnership()`. This function checks the new owner is not the zero address and proceeds to write the new owner's address into the owner's state variable. If the nominated EOA account is not a valid account, it is entirely possible the owner may accidentally transfer ownership to an uncontrolled account, breaking all functions with the `onlyOwner()` modifier.

#### Proof of Concept

1.  Navigate to "<https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/customswap/contracts/Swap.sol#L30>"
2.  The contract has many `onlyOwner` function.
3.  The contract is inherited from the Ownable which includes `transferOwnership`.

#### Tools Used

None

#### Recommended Mitigation Steps

Implement zero address check and consider implementing a two step process where the owner nominates an account and the nominated account needs to call an `acceptOwnership()` function for the transfer of ownership to fully succeed. This ensures the nominated EOA account is a valid and active account.

**[chickenpie347 (Boot Finance) confirmed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/35)** 

**[0xean (judge) commented](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/35#issuecomment-1007872687):**
 > upgrading to med severity as this could impact availability of protocol 
> 
> 2 — Med: Assets not at direct risk, but the function of the protocol or its availability could be impacted, or leak value with a hypothetical attack path with stated assumptions, but external requirements.

# Low Risk Findings (34)
- [[L-01] Missing Zero-check](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/146) _Submitted by 0v3rf10w, also found by Reigada_
- [[L-02] Reentrancy](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/148) _Submitted by 0v3rf10w, also found by defsec_
- [[L-03] No checking of admin_fee, wether it is <= max_admin_fee ](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/127) _Submitted by JMukesh_
- [[L-04] No event was emitted while setting fees and admin_fees in constructor](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/128) _Submitted by JMukesh_
- [[L-05] wrong operator used in checking the fees, adminfee, withdrawfee](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/254) _Submitted by JMukesh, also found by loop and ye0lde_
- [[L-06] revoke() Does Not Check Zero Address for _addr](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/202) _Submitted by Meta0xNull_
- [[L-07] Incorrect event parameter used in emit](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/105) _Submitted by Reigada_
- [[L-08] Tokens not recoverable](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/108) _Submitted by Reigada_
- [[L-09] Vesting.revoke is missing a require statement](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/116) _Submitted by Reigada_
- [[L-10] Require statement missing in fallback and burnEtherForMember() functions](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/137) _Submitted by Reigada_
- [[L-11] Check ERC20 token `approve()` function return value](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/109) _Submitted by Ruhum_
- [[L-12] Vesting contract locks tokens for less time than expected](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/82) _Submitted by TomFrench, also found by pauliax_
- [[L-13] Tokens with decimals larger than 18 are not supported](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/221) _Submitted by WatchPug_
- [[L-14] `SwapUtils.sol` Inconsistent parameter value of `lpTokenSupply` among `Liquidity` related events](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/237) _Submitted by WatchPug_
- [[L-15] # Missing parameter validation](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/209) _Submitted by cmichel_
- [[L-16] `BasicSale` has unused ERC20 code](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/210) _Submitted by cmichel_
- [[L-17] `BasicSale` uses inaccurate `secondsPerDay` value](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/211) _Submitted by cmichel_
- [[L-18] can withdraw shares on behalf of anyone](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/215) _Submitted by cmichel_
- [[L-19] Lack of maximum and minimum vesting amount check on the vesting function](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/32) _Submitted by defsec_
- [[L-20] Airdrop Supply differs](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/99) _Submitted by fr0zn, also found by pauliax_
- [[L-21] Vesting.benVested storage variable can be simplified, while _claimableAmount's "s <= benTotal[_addr]" check is redundant and to be removed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/186) _Submitted by hyh_
- [[L-22] Incorrect `require` Statement in `Vesting.claim()`](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/306) _Submitted by leastwood_
- [[L-23] Usage of deprecated safeApprove](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/166) _Submitted by mics_
- [[L-24] claimExact does not check claimable amount](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/126) _Submitted by nathaniel, also found by fr0zn_
- [[L-25] admin can override investor and stuck its funds in the system ](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/11) _Submitted by pants, also found by defsec_
- [[L-26] Array out-of-bounds errors in `BTCPoolDelegator`](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/37) _Submitted by pants_
- [[L-27] Array out-of-bounds errors in `ETHPoolDelegator`](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/38) _Submitted by pants_
- [[L-28] Array out-of-bounds errors in `USDPoolDelegator`](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/39) _Submitted by pants_
- [[L-29] payable vest](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/273) _Submitted by pauliax_
- [[L-30] _recordBurn does not handle 0 _eth appropriately](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/274) _Submitted by pauliax_
- [[L-31] Usage of assert](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/279) _Submitted by pauliax, also found by leastwood_
- [[L-32] Validations](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/301) _Submitted by pauliax_
- [[L-33] Unclear Commented Out Code](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/140) _Submitted by ye0lde, also found by loop_
- [[L-34] Don't allow swapping the same token](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/89) _Submitted by Ruhum, also found by rfa_

# Non-Critical Findings (39)
- [[N-01] claimExact() Missing Validation As In claim()](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/194) _Submitted by Meta0xNull, also found by pauliax_
- [[N-02] `Vesting.sol#calcClaimableAmount()` Claimed amount should be excluded in claimable amount](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/248) _Submitted by WatchPug_
- [[N-03] Invalid return value when calculating claimable amount ](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/92) _Submitted by fr0zn_
- [[N-04] Missing revert message](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/168) _Submitted by mics_
- [[N-05] `Token.transfer()` and `Token.transferFrom()` emit `Transfer` events when the transferred amount is zero](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/43) _Submitted by pants_
- [[N-06] `Token.transferFrom()` emits `Transfer` events when `_from` equals `_to`](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/44) _Submitted by pants_
- [[N-07] `Token.approve()` emits `Approval` events when the allowence hasn't changed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/46) _Submitted by pants_
- [[N-08] `PoolGauge.deposit()` emits `Deposit` events when the deposited amount is zero](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/48) _Submitted by pants_
- [[N-09] `PoolGauge.withdraw()` emits `Withdraw` events when the withdrawn amount is zero](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/49) _Submitted by pants_
- [[N-10] `MainToken.transfer()` and `MainToken.transferFrom()` emit `Transfer` events when the transferred amount is zero](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/51) _Submitted by pants_
- [[N-11] `MainToken.transferFrom()` emits `Transfer` events when `_from` equals `_to`](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/52) _Submitted by pants_
- [[N-12] `MainToken.approve()` emits `Approval` events when the allowance hasn't changed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/54) _Submitted by pants_
- [[N-13] `MainToken.mint()` and `MainToken.mint_dev()` emit `Transfer` events when the minted amount is zero](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/60) _Submitted by pants_
- [[N-14] `MainToken.burn()` emits `Transfer` events when the burned amount is zero](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/61) _Submitted by pants_
- [[N-15] `MainToken.set_minter()` emits `SetMinter` events when the minter hasn't changed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/62) _Submitted by pants_
- [[N-16] `MainToken.set_admin()` emits `SetAdmin` events when the admin hasn't changed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/63) _Submitted by pants_
- [[N-17] `MainToken.set_mint_multisig()` emits `SetMintMultisig` events when `minting_multisig` hasn't changed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/64) _Submitted by pants_
- [[N-18] `MainToken.__init__()` emits `Transfer` events when the amount minted for `msg.sender` is zero (and it is always the case)](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/67) _Submitted by pants_
- [[N-19] `LPToken.__init__()` emits `Transfer` events when the amount minted for `msg.sender` is zero](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/68) _Submitted by pants_
- [[N-20] `LPToken.transfer()` and `LPToken.transferFrom()` emit `Transfer` events when the transferred amount is zero](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/70) _Submitted by pants_
- [[N-21] `LPToken.transferFrom()` emits `Transfer` events when `_from` equals `_to`](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/71) _Submitted by pants_
- [[N-22] `LPToken.approve()` emits `Approval` events when the allowance hasn't changed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/73) _Submitted by pants_
- [[N-23] Missing emit of initial `SetAdmin` event in `MainToken.__init__()`](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/78) _Submitted by pants_
- [[N-24] Missing emit of initial `ApplyOwnership` event in `GaugeController.__init__()`](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/79) _Submitted by pants_
- [[N-25] `GaugeController.apply_transfer_ownership()` emits `ApplyOwnership` events when the admin hasn't changed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/80) _Submitted by pants_
- [[N-26] `GaugeController.commit_transfer_ownership()` emits `CommitOwnership` events when the future admin hasn't changed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/81) _Submitted by pants_
- [[N-27] block timestamp](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/147) _Submitted by 0v3rf10w_
- [[N-28] Unnecessary imports](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/91) _Submitted by PranavG_
- [[N-29] Code Style: consistency](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/228) _Submitted by WatchPug_
- [[N-30] Typos](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/230) _Submitted by WatchPug_
- [[N-31] Missing error messages in require statements](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/247) _Submitted by WatchPug_
- [[N-32] Renaming variables for clarity](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/125) _Submitted by nathaniel_
- [[N-33] burnAddress is not actually meant to burn anything](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/275) _Submitted by pauliax_
- [[N-34] Events should be written in CapWords](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/162) _Submitted by pmerkleplant_
- [[N-35] Functions should be written in mixedCase](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/163) _Submitted by pmerkleplant_
- [[N-36] Contract `Vesting` should inherit from interface `IVesting`](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/164) _Submitted by pmerkleplant_
- [[N-37] Constants should be written in UPPER_CASE](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/165) _Submitted by pmerkleplant_
- [[N-38] Function _getDayEmission can be simplified (PublicSale.sol)](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/118) _Submitted by ye0lde_
- [[N-39] use of floating pragma](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/257) _Submitted by JMukesh, also found by loop_

# Gas Optimizations (62)
- [[G-01] Unnecessary require statement in vesting.claim()](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/115) _Submitted by Reigada, also found by Meta0xNull_
- [[G-02] uint256 is always >= 0](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/133) _Submitted by 0x0x0x, also found by Reigada, WatchPug, loop, and pauliax_
- [[G-03] Packing of state variable ](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/258) _Submitted by JMukesh_
- [[G-04] validate() to Verify Airdrop Address On Chain is Unnecessary](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/195) _Submitted by Meta0xNull, also found by cmichel and pauliax_
- [[G-05] State variables could be declared constant](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/88) _Submitted by PranavG, also found by defsec, ye0lde, WatchPug, nathaniel, and pmerkleplant_
- [[G-06] Use of uint256 parameter instead of bool](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/112) _Submitted by Reigada, also found by fr0zn_
- [[G-07] If statement in _updateEmission() can be removed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/34) _Submitted by Reigada, also found by cmichel and fr0zn_
- [[G-08] No usage of immutable keyword leaves free gas savings on the table](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/1) _Submitted by TomFrench, also found by WatchPug, jah, PranavG, Reigada, nathaniel, pants, pauliax, and pmerkleplant_
- [[G-09] Remove unnecessary variables can save some gas](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/223) _Submitted by WatchPug_
- [[G-10] `SwapUtils.sol#getYD()` Remove redundant code can save gas](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/233) _Submitted by WatchPug_
- [[G-11] External call can be done later to save gas](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/236) _Submitted by WatchPug_
- [[G-12] `SwapUtils.sol#getD()` Remove unnecessary variable and internal call can make the code simpler and save some gas](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/238) _Submitted by WatchPug, also found by hyh_
- [[G-13] Use literal `2` instead of read from storage for `pooledTokens.length` can save gas](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/241) _Submitted by WatchPug, also found by ye0lde, 0x0x0x, JMukesh, Ruhum, WatchPug, pauliax, gzeon, and loop_
- [[G-14] Cache external call results can save gas](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/242) _Submitted by WatchPug, also found by hyh_
- [[G-15] `Vesting.sol#_claimableAmount()` Remove unnecessary storage variables can save gas](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/246) _Submitted by WatchPug_
- [[G-16] Gas: Unnecessary length check in `Swap.constructor`](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/217) _Submitted by cmichel_
- [[G-17] Gas: Unnecessary msg.sender != 0 check](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/218) _Submitted by cmichel, also found by rfa, WatchPug, and pauliax_
- [[G-18] Adding unchecked directive can save gas](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/261) _Submitted by defsec, also found by mics, pants, and pauliax_
- [[G-19] Upgrade pragma to at least 0.8.4](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/29) _Submitted by defsec_
- [[G-20] Gas optimization on InvestorDistribution.sol](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/94) _Submitted by fr0zn_
- [[G-21] Duplicated code and usage of assert](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/96) _Submitted by fr0zn, also found by nathaniel and pauliax_
- [[G-22] Redundant check on claim](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/97) _Submitted by fr0zn_
- [[G-23] SwapUtils.getVirtualPrice double calling to storage reading function _xp(self)](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/193) _Submitted by hyh_
- [[G-24] SwapUtils's addLiquidity does multiple LP token total supply calls](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/197) _Submitted by hyh_
- [[G-25] SwapUtils.calculateTokenAmount does repetitive checks of static condition](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/200) _Submitted by hyh_
- [[G-26] SwapUtils's getD, getY, getYD functions do repetitive calculations of contant expression within the cycles](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/207) _Submitted by hyh_
- [[G-27] No need to initialize variables with default values](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/5) _Submitted by jah, also found by ye0lde, Meta0xNull, Reigada, WatchPug, pants, and pauliax_
- [[G-28] `Timelock` Struct Packing in `Vesting.sol`](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/307) _Submitted by leastwood_
- [[G-29] safeERC20 library imported but not used](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/154) _Submitted by loop_
- [[G-30] unnecessary variable y in getYD ](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/170) _Submitted by mics_
- [[G-31] Use calldata instead of memory for function parameters](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/172) _Submitted by mics_
- [[G-32] Public functions can be external](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/121) _Submitted by nathaniel, also found by pants, Reigada, hyh, leastwood, and defsec_
- [[G-33] double reading of memory inside a loop without caching](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/13) _Submitted by pants, also found by Ruhum, WatchPug, and mics_
- [[G-34] Rearrange state variables](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/138) _Submitted by pants_
- [[G-63] optimizing for loops by caching array length](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/14) _Submitted by pants, also found by pauliax, WatchPug, and hyh_
- [[G-36] internal functions could be set private](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/18) _Submitted by pants_
- [[G-37] `PoolGauge.withdraw()` can be optimized when `_value` equals zero](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/47) _Submitted by pants_
- [[G-38] Unnecessary use of safeMath](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/7) _Submitted by pants, also found by Ruhum, Reigada, TomFrench, loop, pauliax, and defsec_
- [[G-39] too many bits to describe small quantities ](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/8) _Submitted by pants_
- [[G-40] multiple reading of state variables without caching](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/9) _Submitted by pants, also found by pauliax_
- [[G-41] modifyInvestor does not need to check if _investor is not empty](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/281) _Submitted by pauliax_
- [[G-42] function claim optimizations](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/283) _Submitted by pauliax_
- [[G-43] Itteration over all the timelocks when revoking the user](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/285) _Submitted by pauliax_
- [[G-44] Optimize structs](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/290) _Submitted by pauliax_
- [[G-45] Useless nonReentrant](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/293) _Submitted by pauliax_
- [[G-46] _recordBurn _payer](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/300) _Submitted by pauliax_
- [[G-47] Remove unused variables](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/161) _Submitted by pmerkleplant, also found by Reigada and pauliax_
- [[G-48] Redundant check](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/2) _Submitted by tqts_
- [[G-49] Numerous gas optimizations in SwapUtils.sol](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/6) _Submitted by tqts_
- [[G-50] Cache Reference To State Variables "currentDay, currentEra, emission" in _updateEmission (PublicSale.sol)](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/117) _Submitted by ye0lde, also found by WatchPug_
- [[G-51] Unnecessary "else if" in function vest (Vesting.sol)](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/22) _Submitted by ye0lde_
- [[G-52] Long Revert Strings](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/23) _Submitted by ye0lde, also found by WatchPug and pants_
- [[G-53] Unused Named Returns (PublicSale.sol)](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/25) _Submitted by ye0lde, also found by WatchPug and pants_
- [[G-56] _withdrawShare Can Be Rewritten To Be More Efficient (PublicSale.sol)](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/26) _Submitted by ye0lde_
- [[G-57] _processWithdrawal Can Be Rewritten To Be More Efficient (PublicSale.sol)](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/27) _Submitted by ye0lde_
- [[G-58] getEmissionShare Can Be Rewritten To Be More Efficient (PublicSale.sol)](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/28) _Submitted by ye0lde_
- [[G-59] From' and 'to' tokens are read from storage multiple times in SwapUtils's swap function](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/190) _Submitted by hyh_
- [[G-60] Multiple double storage reading _xp(self) function calls](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/191) _Submitted by hyh_
- [[G-61] Redundant hardhat console import ](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/173) _Submitted by pants, also found by WatchPug_
- [[G-62] Use of uint8 for counter in for loop increases gas costs](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/175) _Submitted by pants, also found by Reigada and pauliax_
- [[G-64] Use bytes32 instead of string when possible](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/176) _Submitted by pants_
- [[G-65] Using ++i consumes less gas than i++](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/104) _Submitted by Reigada_

# Disclosures

C4 is an open organization governed by participants in the community.

C4 Contests incentivize the discovery of exploits, vulnerabilities, and bugs in smart contracts. Security researchers are rewarded at an increasing rate for finding higher-risk issues. Contest submissions are judged by a knowledgeable security researcher and solidity developer and disclosed to sponsoring developers. C4 does not conduct formal verification regarding the provided code but instead provides final verification.

C4 does not provide any guarantee or warranty regarding the security of this project. All smart contract software should be used at the sole risk and responsibility of users.
