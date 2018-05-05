---
layout: post
keywords: 以太坊 区块链 geth
title: 一步步教你发行属于自己的以太坊代币
date: 2018-03-08
categories: 区块链
tags: Ethereum
---

### 代币Token
如果不那么追求精确的定义，代币就是数字货币，比特币、以太币就是一个代币。
利用以太坊的智能合约可以轻松编写出属于自己的代币，代币可以代表任何可以交易的东西，如：积分、财产、证书等等。
因此不管是出于商业，还是学习很多人想创建一个自己的代币，引用网上的一个图看看创建的代币是什么样子。
<img src="https://learnblockchain.cn/images/token_info.jpeg" />

你也可以在如下地址查看Tokens：[https://etherscan.io/tokens](https://etherscan.io/tokens)

今天我们就来详细讲一讲怎样创建一个这样的代币。
<!-- more -->
### ERC20 Token
也许你经常看到ERC20和代币一同出现， ERC20是以太坊定义的一个[代币标准](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md)。
要求我们在实现代币的时候必须要遵守的协议，如指定代币名称、总量、实现代币交易函数等，只有支持了协议才能被以太坊钱包支持。

其接口如下：
```
contract ERC20Interface {

    string public constant name = "Token Name";
    string public constant symbol = "SYM";
    uint8 public constant decimals = 18;  // 18 is the most common number of decimal places

    function totalSupply() public constant returns (uint);
    function balanceOf(address tokenOwner) public constant returns (uint balance);
    function allowance(address tokenOwner, address spender) public constant returns (uint remaining);
    function transfer(address to, uint tokens) public returns (bool success);
    function approve(address spender, uint tokens) public returns (bool success);
    function transferFrom(address from, address to, uint tokens) public returns (bool success);

    event Transfer(address indexed from, address indexed to, uint tokens);
    event Approval(address indexed tokenOwner, address indexed spender, uint tokens);
}
```
简单说明一下：  
name：代币名称  
symbol：代币符号  
decimals：代币小数点位数，代币的最小单位， 18表示我们可以拥有 .000000000000000001单位个代币。  
totalSupply()：发行代币总量。  
balanceOf()：查看对应账号的代币余额。  
transfer()：实现代币交易，用于给用户发送代币（从我们的账户里）。  
transferFrom()：实现代币用户之间的交易。  
allowance()：控制代币的交易，如可交易账号及资产。  
approve()：允许用户可花费的代币数。

### 一个最简单的合约代码，详细看注释：
```
pragma solidity ^0.4.18;

/**
 * @title 基础版的代币合约
 */
contract token {
    /*记录所有余额的映射*/
    mapping (address => uint256) public balanceOf;

    /* 初始化合约，并且把初始的所有代币都给这合约的创建者
     * @param initialSupply 代币的总数
     */
    function token(uint256 initialSupply) {
        //给指定帐户初始化代币总量，初始化用于奖励合约创建者
        balanceOf[msg.sender] = initialSupply;
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
}
```

### 改善代币
实际使用过程中，交易的过程，需要通知到客户端，并且记录到区块中，我们可以使用event事件来指定，如下代码进行声明：
```
//在区块链上创建一个事件，用以通知客户端
event Transfer(address indexed from, address indexed to, uint256 value);
```

设置一些代币的基本信息
```
/* 公共变量 */
string public name; //代币名称
string public symbol; //代币符号比如'$'
uint8 public decimals = 18;  //代币单位，展示的小数点后面多少个0,和以太币一样后面是是18个0
uint256 public totalSupply; //代币总量
```

某些特定的场景中，不允许某个帐户花费超过指定的上限，避免大额支出，我们可以添加一个 `approve` 方法，来设置一个允许支出最大金额的列表。
```
mapping (address => mapping (address => uint256)) public allowance;

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
```
同样在 solidity 中，合约之间也可以相互调用，我们可以增加一个 approveAndCall 方法，用于在设置帐户最大支出金额后，可以做一些其他操作。
```
interface tokenRecipient { function receiveApproval(address _from, uint256 _value, address _token, bytes _extraData) public; }

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
```
我们可以增加一个 burn 方法，用于管理员减去指定帐户的指定金额。进行该方法操作时，通知客户端记录到区块链中。
```
//减去用户余额事件
event Burn(address indexed from, uint256 value);  

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
```
完整的代码如下：
```
pragma solidity ^0.4.18;

interface tokenRecipient { function receiveApproval(address _from, uint256 _value, address _token, bytes _extraData) public; }


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
        balanceOf[msg.sender] = totalSupply;

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
```

### 如何部署
下载 Ethereum Wallet 钱包，打开界面，点击CONTRACTS ==> DEPLOY NEW CONTRACT, 选择部署该合约的账户地址，复制源码，粘贴到 SOLIDITY CONTRACT SOURCE CODE. 在右侧选择`token`, `Initial Supply` 输入初始金额，`Token name` 输入我们的代币名称 ，`Token symbol` 输入代币符号, 然后点击 部署，输入部署帐户的密码。 

等待其他区块认证后，你就拥有自己的代币了。

<hr/>参考文档：  
[代币标准](https://theethereum.wiki/w/index.php/ERC20_Token_Standard)  
[Create your own crypto-currency with ethereum](https://ethereum.org/token)  
[一步步教你创建自己的数字货币（代币）进行ICO](https://learnblockchain.cn/2018/01/12/create_token/)  
[Go-Ethereum 1.7.2 结合 Mist 0.9.2 实现代币智能合约的实例](https://mshk.top/2017/11/go-ethereum-1-7-2-mist-0-9-2-token/)


