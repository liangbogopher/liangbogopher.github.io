---
layout: post
keywords: 以太坊 区块链 geth
title: 实现增发，兑换，冻结等高级功能的以太坊代币
date: 2018-03-09
categories: 区块链
tags: Ethereum 
---

在上一篇[一步步教你发行属于自己的以太坊代币](/2018/03/09/create_erc20_token/)中我们实现一个最基本功能的代币，本文将在上一遍文章的基础上，讲解如果添加更多的高级功能。

### 高级版的代币功能
一般的代币可以不设置管理者，就是所谓的去中心化。实际使用过程中，可能需要给予挖矿等功能，让别人能够购买你的代币，那么我们就需要设置一个帐户地址做为这个代币合约的管理者。
```
**
 * owned 是一个管理者
 */
contract owned {
    address public owner;

    /**
     * 初台化构造函数
     */
    function owned() {
        owner = msg.sender;
    }

    /**
     * 判断当前合约调用者是否是管理员
     */
    modifier onlyOwner {
        require (msg.sender == owner);
        _;
    }

    /**
     * 指派一个新的管理员
     * @param  newOwner address 新的管理员帐户地址
     */
    function transferOwnership(address newOwner) onlyOwner {
        owner = newOwner;
    }
}
```
上面的代码是一个非常简单的合约，我们可以在后面的代码中，使用 `继承` 来实现后续的功能。
```
/**
 * @title 高级版代币
 * 增加冻结用户、挖矿、根据指定汇率购买(售出)代币价格的功能
 */
contract MyAdvancedToken is owned{}
```
在 `MyAdvancedToken` 的所有方法中，可以使用 `owned` 的变量 `owner`和 `modifier onlyOwner`。
<!-- more -->
#### 1. 去中心化的管理者
我们也可以在构造函数中设置是否需要一个去中心化的管理者。
```
/*初始化合约，并且把初始的所有的令牌都给这合约的创建者
 * @param initialSupply 所有币的总数
 * @param tokenName 代币名称
 * @param tokenSymbol 代币符号
 * @param centralMinter 是否指定其他帐户为合约所有者,为0是去中心化
 */
function MyAdvancedToken(
  uint256 initialSupply,
  string tokenName,
  string tokenSymbol,
  address centralMinter
)  {
    //设置合约的管理者
    if(centralMinter != 0 ) owner = centralMinter;
}
```
#### 2. 挖矿
有的时候需要更多的代币流通，可以增加 `mintToken` 方法，创造更多的代币。
```
/**
 * 合约拥有者，可以为指定帐户创造一些代币
 * @param  target address 帐户地址
 * @param  mintedAmount uint256 增加的金额(单位是wei)
 */
function mintToken(address target, uint256 mintedAmount) onlyOwner {

    //给指定地址增加代币，同时总量也相加
    balanceOf[target] += mintedAmount;
    totalSupply += mintedAmount;
}
```
在方法的最后有一个 `onlyOwner`，说明 `mintToken` 是继承了 `onlyOwner`方法，会先调用 `modifier onlyOwner` 方法，然后将 `mintToken` 方法的内容，插入到下划线 _ 处调用。

#### 3. 冻结资产
有的场景中，某些用户违反了规定，需要冻结/解冻帐户，不想让他使用已经拥有的代币.可以增加以下代码来控制：
```
//是否冻结帐户的列表
mapping (address => bool) public frozenAccount;

//定义一个事件，当有资产被冻结的时候，通知正在监听事件的客户端
event FrozenFunds(address target, bool frozen);

/**
 * 增加冻结帐户名称
 *
 * 你可能需要监管功能以便你能控制谁可以/谁不可以使用你创建的代币合约
 *
 * @param  target address 帐户地址
 * @param  freeze bool    是否冻结
 */
function freezeAccount(address target, bool freeze) onlyOwner {
    frozenAccount[target] = freeze;
    FrozenFunds(target, freeze);
}
```
#### 4. 自动交易
到了现在，代币的功能很完善，大家也相信你的代币是有价值的，但你想要使用以太币 ether 或者其他代币来购买，让代币市场化，可以真实的交易，我们可以设置一个价格
```
//卖出的汇率,一个代币，可以卖出多少个以太币，单位是wei
uint256 public sellPrice;

//买入的汇率,1个以太币，可以买几个代币
uint256 public buyPrice;

/**
 * 设置买卖价格
 *
 * 如果你想让ether(或其他代币)为你的代币进行背书,以便可以市场价自动化买卖代币,我们可以这么做。如果要使用浮动的价格，也可以在这里设置
 *
 * @param newSellPrice 新的卖出价格
 * @param newBuyPrice 新的买入价格
 */
function setPrices(uint256 newSellPrice, uint256 newBuyPrice) onlyOwner {
    sellPrice = newSellPrice;
    buyPrice = newBuyPrice;
}
```

然后增加买、卖的方法，每一次的交易，都会消耗掉一定的 `ether`。在 `Solidity 0.4.0` 之后，要接收 `ether` 的函数都要加一个 `payable` 属性，如果你开放的合约，需要别人转钱给你，就需要加 `payable`。

下面的方法，不会增加代币，只是改变调用合约者的代币数量，买、卖的价格单位不是 `ether`，而是 `wei`，这是以太币中最小的单位(就像美元里的美分,比特币里的聪)。`1 ether` = 1000000000000000000 `wei`。因此使用 `ether` 设置价格的时候,在最后加18个0。

当创建合约的时候,发送足够多的 `ether` 作为代币的背书,否则你的合约就是破产的,你的用户就不能够卖掉他们的代币。
```
/**
 * 使用以太币购买代币
 */
function buy() payable public {
  uint amount = msg.value / buyPrice;

  _transfer(this, msg.sender, amount);
}

/**
 * @dev 卖出代币
 * @return 要卖出的数量(单位是wei)
 */
function sell(uint256 amount) public {

    //检查合约的余额是否充足
    require(this.balance >= amount * sellPrice);

    _transfer(msg.sender, this, amount);

    msg.sender.transfer(amount * sellPrice);
}
```

#### 5. 全部代码
把所有的特性加上，完整的代码如下：
```
pragma solidity 0.4.18;

interface tokenRecipient { function receiveApproval(address _from, uint256 _value, address _token, bytes _extraData) public; }

/**
 * owned 是一个管理者
 */
contract owned {
    address public owner;

    /**
     * 初台化构造函数
     */
    function owned() {
        owner = msg.sender;
    }

    /**
     * 判断当前合约调用者是否是管理员
     */
    modifier onlyOwner {
        require (msg.sender == owner);
        _;
    }

    /**
     * 指派一个新的管理员
     * @param  newOwner address 新的管理员帐户地址
     */
    function transferOwnership(address newOwner) onlyOwner {
        owner = newOwner;
    }
}

/**
 * @title 基础版的代币合约
 */
contract token {
    /* 公共变量 */
    string public name; //代币名称
    string public symbol; //代币符号比如'$'
    uint8 public decimals = 18;  //代币单位，展示的小数点后面多少个0,和以太币一样后面是是18个0
    uint256 public totalSupply; //代币总量

    /*记录所有余额的映射*/
    mapping (address => uint256) public balanceOf;
    mapping (address => mapping (address => uint256)) public allowance;

    /* 在区块链上创建一个事件，用以通知客户端*/
    event Transfer(address indexed from, address indexed to, uint256 value);  //转帐通知事件
    event Burn(address indexed from, uint256 value);  //减去用户余额事件

    /* 初始化合约，并且把初始的所有代币都给这合约的创建者
     * @param initialSupply 代币的总数
     * @param tokenName 代币名称
     * @param tokenSymbol 代币符号
     */
    function token(uint256 initialSupply, string tokenName, string tokenSymbol) {

        //初始化总量
        totalSupply = initialSupply * 10 ** uint256(decimals);    //以太币是10^18，后面18个0，所以默认decimals是18

        //给指定帐户初始化代币总量，初始化用于奖励合约创建者
        //balanceOf[msg.sender] = totalSupply;
        balanceOf[this] = totalSupply;

        name = tokenName;
        symbol = tokenSymbol;
    }

    /**
     * 私有方法从一个帐户发送给另一个帐户代币
     * @param  _from address 发送代币的地址
     * @param  _to address 接受代币的地址
     * @param  _value uint256 接受代币的数量
     */
    function _transfer(address _from, address _to, uint256 _value) internal {

      //避免转帐的地址是0x0
      require(_to != 0x0);

      //检查发送者是否拥有足够余额
      require(balanceOf[_from] >= _value);

      //检查是否溢出
      require(balanceOf[_to] + _value > balanceOf[_to]);

      //保存数据用于后面的判断
      uint previousBalances = balanceOf[_from] + balanceOf[_to];

      //从发送者减掉发送额
      balanceOf[_from] -= _value;

      //给接收者加上相同的量
      balanceOf[_to] += _value;

      //通知任何监听该交易的客户端
      Transfer(_from, _to, _value);

      //判断买、卖双方的数据是否和转换前一致
      assert(balanceOf[_from] + balanceOf[_to] == previousBalances);

    }

    /**
     * 从主帐户合约调用者发送给别人代币
     * @param  _to address 接受代币的地址
     * @param  _value uint256 接受代币的数量
     */
    function transfer(address _to, uint256 _value) public {
        _transfer(msg.sender, _to, _value);
    }

    /**
     * 从某个指定的帐户中，向另一个帐户发送代币
     *
     * 调用过程，会检查设置的允许最大交易额
     *
     * @param  _from address 发送者地址
     * @param  _to address 接受者地址
     * @param  _value uint256 要转移的代币数量
     * @return success        是否交易成功
     */
    function transferFrom(address _from, address _to, uint256 _value) returns (bool success) {
        //检查发送者是否拥有足够余额
        require(_value <= allowance[_from][msg.sender]);   // Check allowance

        allowance[_from][msg.sender] -= _value;

        _transfer(_from, _to, _value);

        return true;
    }

    /**
     * 设置帐户允许支付的最大金额
     *
     * 一般在智能合约的时候，避免支付过多，造成风险
     *
     * @param _spender 帐户地址
     * @param _value 金额
     */
    function approve(address _spender, uint256 _value) public returns (bool success) {
        allowance[msg.sender][_spender] = _value;
        return true;
    }

    /**
     * 设置帐户允许支付的最大金额
     *
     * 一般在智能合约的时候，避免支付过多，造成风险，加入时间参数，可以在 tokenRecipient 中做其他操作
     *
     * @param _spender 帐户地址
     * @param _value 金额
     * @param _extraData 操作的时间
     */
    function approveAndCall(address _spender, uint256 _value, bytes _extraData) public returns (bool success) {
        tokenRecipient spender = tokenRecipient(_spender);
        if (approve(_spender, _value)) {
            spender.receiveApproval(msg.sender, _value, this, _extraData);
            return true;
        }
    }

    /**
     * 减少代币调用者的余额
     *
     * 操作以后是不可逆的
     *
     * @param _value 要删除的数量
     */
    function burn(uint256 _value) public returns (bool success) {
        //检查帐户余额是否大于要减去的值
        require(balanceOf[msg.sender] >= _value);   // Check if the sender has enough

        //给指定帐户减去余额
        balanceOf[msg.sender] -= _value;

        //代币问题做相应扣除
        totalSupply -= _value;

        Burn(msg.sender, _value);
        return true;
    }

    /**
     * 删除帐户的余额（含其他帐户）
     *
     * 删除以后是不可逆的
     *
     * @param _from 要操作的帐户地址
     * @param _value 要减去的数量
     */
    function burnFrom(address _from, uint256 _value) public returns (bool success) {

        //检查帐户余额是否大于要减去的值
        require(balanceOf[_from] >= _value);

        //检查 其他帐户 的余额是否够使用
        require(_value <= allowance[_from][msg.sender]);

        //减掉代币
        balanceOf[_from] -= _value;
        allowance[_from][msg.sender] -= _value;

        //更新总量
        totalSupply -= _value;
        Burn(_from, _value);
        return true;
    }
}

/**
 * @title 高级版代币
 * 增加冻结用户、挖矿、根据指定汇率购买(售出)代币价格的功能
 */
contract MyAdvancedToken is owned, token {

    //卖出的汇率,一个代币，可以卖出多少个以太币，单位是wei
    uint256 public sellPrice;

    //买入的汇率,1个以太币，可以买几个代币
    uint256 public buyPrice;

    //是否冻结帐户的列表
    mapping (address => bool) public frozenAccount;

    //定义一个事件，当有资产被冻结的时候，通知正在监听事件的客户端
    event FrozenFunds(address target, bool frozen);

    /*初始化合约，并且把初始的所有的令牌都给这合约的创建者
     * @param initialSupply 所有币的总数
     * @param tokenName 代币名称
     * @param tokenSymbol 代币符号
     * @param centralMinter 是否指定其他帐户为合约所有者,为0是去中心化
     */
    function MyAdvancedToken(
      uint256 initialSupply,
      string tokenName,
      string tokenSymbol,
      address centralMinter
    ) token (initialSupply, tokenName, tokenSymbol) {

        //设置合约的管理者
        if(centralMinter != 0 ) owner = centralMinter;

        sellPrice = 2;     //设置1个单位的代币(单位是wei)，能够卖出2个以太币
        buyPrice = 4;      //设置1个以太币，可以买0.25个代币
    }


    /**
     * 私有方法，从指定帐户转出余额
     * @param  _from address 发送代币的地址
     * @param  _to address 接受代币的地址
     * @param  _value uint256 接受代币的数量
     */
    function _transfer(address _from, address _to, uint _value) internal {

        //避免转帐的地址是0x0
        require (_to != 0x0);

        //检查发送者是否拥有足够余额
        require (balanceOf[_from] > _value);

        //检查是否溢出
        require (balanceOf[_to] + _value > balanceOf[_to]);

        //检查 冻结帐户
        require(!frozenAccount[_from]);
        require(!frozenAccount[_to]);



        //从发送者减掉发送额
        balanceOf[_from] -= _value;

        //给接收者加上相同的量
        balanceOf[_to] += _value;

        //通知任何监听该交易的客户端
        Transfer(_from, _to, _value);

    }

    /**
     * 合约拥有者，可以为指定帐户创造一些代币
     * @param  target address 帐户地址
     * @param  mintedAmount uint256 增加的金额(单位是wei)
     */
    function mintToken(address target, uint256 mintedAmount) onlyOwner {

        //给指定地址增加代币，同时总量也相加
        balanceOf[target] += mintedAmount;
        totalSupply += mintedAmount;


        Transfer(0, this, mintedAmount);
        Transfer(this, target, mintedAmount);
    }

    /**
     * 增加冻结帐户名称
     *
     * 你可能需要监管功能以便你能控制谁可以/谁不可以使用你创建的代币合约
     *
     * @param  target address 帐户地址
     * @param  freeze bool    是否冻结
     */
    function freezeAccount(address target, bool freeze) onlyOwner {
        frozenAccount[target] = freeze;
        FrozenFunds(target, freeze);
    }

    /**
     * 设置买卖价格
     *
     * 如果你想让ether(或其他代币)为你的代币进行背书,以便可以市场价自动化买卖代币,我们可以这么做。如果要使用浮动的价格，也可以在这里设置
     *
     * @param newSellPrice 新的卖出价格
     * @param newBuyPrice 新的买入价格
     */
    function setPrices(uint256 newSellPrice, uint256 newBuyPrice) onlyOwner {
        sellPrice = newSellPrice;
        buyPrice = newBuyPrice;
    }

    /**
     * 使用以太币购买代币
     */
    function buy() payable public {
      uint amount = msg.value / buyPrice;

      _transfer(this, msg.sender, amount);
    }

    /**
     * @dev 卖出代币
     * @return 要卖出的数量(单位是wei)
     */
    function sell(uint256 amount) public {

        //检查合约的余额是否充足
        require(this.balance >= amount * sellPrice);

        _transfer(msg.sender, this, amount);

        msg.sender.transfer(amount * sellPrice);
    }
}
```
参考之前的方法，在 `Ethereum Wallet` 图形界面重新部署我们的合约。

<hr/>参考文档：  
[Go-Ethereum 1.7.2 结合 Mist 0.9.2 实现代币智能合约的实例](https://mshk.top/2017/11/go-ethereum-1-7-2-mist-0-9-2-token/)


