---
layout: post
keywords: 以太坊 区块链 geth
title: 通过智能合约进行ICO众筹
date: 2018-03-10
categories: 区块链
tags: Ethereum
---

前面我们有两遍文章写了如何发行代币，今天我们讲一下如何使用代币来公开募资，即编写一个募资合约。
### 众筹
先简单说下众筹的概念：一般是这样的，我一个非常好的想法，但是我没有钱来做这事，于是我把这个想法发给大家看，说：我做这件事需要5百万，大家有没有兴趣投些钱，如果大家在30天内投够了5百万我就开始做，到时大家都是原始股东，如果募资额不到5百万，大家投的钱就还给大家。

现在ICO众筹已经被各路大佬拿来割韭菜而被玩坏了（不管有无达标，都把钱卷走）。

其实区块链技术本事非常适合解决众筹的信任问题，借助于智能合约，可以实现当募资额完成时，募资款自动打到指定账户，当募资额未完成时，可退款。这个过程不需要看众筹大佬的人品，不用依靠第三方平台信用担保。
<!-- more -->
### 众筹智能合约代码
接下来就看看如何实现一个众筹智能合约。
```
pragma solidity ^0.4.18;

interface token {
    function transfer(address receiver, uint amount) public;
}

contract Crowdsale {
    address public beneficiary;  // 募资成功后的收款方
    uint public fundingGoal;     // 众筹目标，单位是ether
    uint public amountRaised;    // 已筹集金额数量， 单位是wei
    uint public deadline;        // 募资截止期
    
    uint public price;           // token与以太坊的汇率, token卖多少钱
    token public tokenReward;    // 要卖的token
    uint8 public decimals = 18;
    
    mapping(address => uint256) public balanceOf; //保存众筹地址
    
    bool fundingGoalReached = false;  // 众筹是否达到目标
    bool crowdsaleClosed = false;   //  众筹是否结束
   
    //记录已接收的ether通知
    event GoalReached(address recipient, uint totalAmountRaised);
    
    //转帐通知
    event FundTransfer(address backer, uint amount, bool isContribution);
    
    /**
     * 构造函数, 设置相关属性
     */
    function Crowdsale(
        address ifSuccessfulSendTo,
        uint fundingGoalInEthers,
        uint durationInMinutes,
        uint finneyCostOfEachToken,
        address addressOfTokenUsedAsReward) public {
            beneficiary = ifSuccessfulSendTo;
            fundingGoal = fundingGoalInEthers * 1 ether;
            deadline = now + durationInMinutes * 1 minutes;
            price = finneyCostOfEachToken * 1 finney;
            tokenReward = token(addressOfTokenUsedAsReward);   // 传入已发布的 token 合约的地址来创建实例
    }
    
    /**
     * 无函数名的Fallback函数，
     * 
     * 在向合约转账时，这个函数会被调用
     */
    function () payable public {
        require(!crowdsaleClosed);
        uint amount = msg.value;
        
        // 捐款人的金额累加
        balanceOf[msg.sender] += amount;
        
        // 捐款总额累加
        amountRaised += amount;
        
        tokenReward.transfer(msg.sender, amount / price * 10 ** uint256(decimals));
        FundTransfer(msg.sender, amount, true);
    }
    
    /**
    * 定义函数修改器modifier（作用和Python的装饰器很相似）
    * 用于在函数执行前检查某种前置条件（判断通过之后才会继续执行该方法）
    * _ 表示继续执行之后的代码
    */
    modifier afterDeadline() { if (now >= deadline) _; }
    
    /**
     * 判断众筹是否完成融资目标， 这个方法使用了afterDeadline函数修改器
     */
    function checkGoalReached() afterDeadline public {
        if (amountRaised >= fundingGoal) {
            // 达成众筹目标
            fundingGoalReached = true;
            GoalReached(beneficiary, amountRaised);
        }
        crowdsaleClosed = true;
    }
    
    /**
     * 完成融资目标时，融资款发送到收款方
     * 未完成融资目标时，执行退款
     */
    function safeWithdrawal() afterDeadline public {
        // 如果没有达成众筹目标
        if (!fundingGoalReached) {
            // 获取合约调用者已捐款余额
            uint amount = balanceOf[msg.sender];
            balanceOf[msg.sender] = 0;
            if (amount > 0) {
                if (msg.sender.send(amount)) {
                    FundTransfer(msg.sender, amount, false);
                } else {
                    balanceOf[msg.sender] = amount;
                }
            }
        }
        
        // 如果达成众筹目标，并且合约调用者是受益人
        if (fundingGoalReached && beneficiary == msg.sender) {
            if (beneficiary.send(amountRaised)) {
                FundTransfer(beneficiary, amountRaised, false);
            } else {
                // If we fail to send the funds to beneficiary, unlock funders balance
                fundingGoalReached = false;
            }
        }
    }
}
```
### 部署及说明
在部署这个合约之前，我们需要先部署一个代币合约，请参考[一步步教你发行属于自己的以太坊代币](/2018/03/09/create_erc20_token/)。

1. 创建众筹合约我们需要提供一下几个参数：
ifSuccessfulSendTo：募资成功后的收款方（其实这里可以默认为合约创建者）
fundingGoalInEthers：募资额度， 为了方便我们仅募3个ether
durationInMinutes：募资时间
finneyCostOfEachToken：每个代币的价格, 这里为了方便使用了单位finney及值为：1 （1 ether = 1000 finney）
addressOfTokenUsedAsReward：代币合约地址。

2. 参与人投资的时候实际购买众筹合约代币，所有需要先向合约预存代币，代币的数量为：募资额度 / 代币的价格 ， 这里为：3 x 1000/1 = 3000 （当能也可以大于3000）。
可以在 `Ethereum Wallet` 进行代币转账。
**以太坊计量单位：**
Kwei（Babbage）= 10的 3次方 Wei
Mwei（Lovelace）= 10的 6次方 Wei
Gwei（Shannon）= 10的 9次方 Wei
MicroEther（Szabo）= 10的 12次方 Wei
MilliEther（Finney）= 10的 15次方 Wei
Ether = 10的 18次方 Wei

3. 参与人投资行为即是向买众筹合约转账，转账时，会执行Fallback回退函数（即无名函数）向其账户打回相应的代币。

4. safeWithdrawl() 可以被参与人或收益人执行，如果融资不达标参与人可收回之前投资款，如果融资达标收益人可以拿到所有的融资款。 



<hr/>参考文档：  
[Create a crowdsale](https://ethereum.org/crowdsale)  
[如何通过以太坊智能合约来进行众筹（ICO）](https://learnblockchain.cn/2018/02/28/ico-crowdsale/)


