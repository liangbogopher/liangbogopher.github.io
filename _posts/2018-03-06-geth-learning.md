---
layout: post
keywords: 以太坊 区块链 geth
title: Go-Ethereum 实战
date: 2018-03-06
categories: 区块链
tags:
    - Geth
    - Ethereum
---

之前有篇文章介绍了[geth命令详解](/2018/03/04/geth-command)。下面介绍一下在具体实战中如何使用。

Geth是典型的开发以太坊时使用的客户端，基于Go语言开发。 Geth提供了一个交互式命令控制台，通过命令控制台中包含了以太坊的各种功能（API）。

如果你不习惯使用命令控制台，可以下载以太坊官方钱包Ethereum Wallet，
下载地址如下：[https://www.ethereum.org/](https://www.ethereum.org/)

### geth的安装

Mac OSX可以使用brew安装
```
brew tap ethereum/ethereum
brew install ethereum
```
如果你是其他的操作系统可以参考如下地址：
https://github.com/ethereum/go-ethereum/wiki/Building-Ethereum
<!-- more -->
### geth 启动
geth的启动方式有多种，这边重点介绍其中两种

最简单的启动方式如下：
```
geth console
```
启动成功后，你就能看到>提示符了。
这样你就可以输入`admin`, `eth`, `personal`命令查看方法清单了。

还有一种你可以启动RPC模式
```
geth --rinkeby --syncmode fast --cache 1024 --rpc --rpcapi admin,eth,miner,personal --ipcpath /Users/liangbo/Library/Ethereum/geth.ipc 2>>geth_rinkeby.log
```
上面连接的是rinkeby网络节点，如果你还想连接主节点的话：
```
geth --syncmode fast --cache 1024 --port 30304 --rpc --rpcapi admin,eth,miner,personal --rpcport 8546 --ipcpath /Users/liangbo/Library/Ethereum/geth/geth.ipc 2>>geth.log
```
注意：  
1）默认的网卡监听端口是30303，HTTP-RPC默认监听端口是8545。  
2）如果需要上线的话注意需要指定`--rpcaddr`，默认是localhost。  
3）节点数据的同步记得加上`--syncmode fast`，快速模式下主网络同步下来的数据也要70多G，而且以后还得持续增加。全区块同步我本机同步了2个星期还没同步下来，磁盘都快占满了。后来放弃这种模式。

geth后面参数的具体含义，可以参考文章：[以太坊geth命令详解](/2018/03/04/geth-command/)

### 连接geth节点
```
$ geth attach /Users/liangbo/Library/Ethereum/geth.ipc
$ geth attach http://localhost:8545
$ geth attach ws://localhost:8546 (如果启动了WS-RPC的话)
```
成功后进入console模式。

### geth根目录说明
Mac OS：
```
/Users/xxx/Library/Ethereum
```
其中xxx为mac电脑的用户名

Linux:
```
/root/.ethereum
```
当然了你也可以通过使用`--datadir` 来指定目录地址。

### geth rpc调用
例如：查看用户列表
```
curl -l -H "Content-type: application/json" -X POST -d '{"jsonrpc":"2.0","method":"eth_accounts","params":[],"id":1}' localhost:8545
```
Result:
```
{"jsonrpc":"2.0","id":1,"result":["0x537e0ad4f869fa026cd2eb7523fdf7e361539a85","0xb0cab88a2d0ddb9efbf46facac2e4491677a3787","0x848e4c128827eb56ba3d42e29c53a3bfd7bd92cf","0x4ab5463c0b0f7a2d36e56e312a160d255918fa39","0x513ca936839c7b05af9e31c4a67685e457e2d8f4"]}
```
JSON-RPC API官方地址如下：[https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_gettransactionreceipt](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_gettransactionreceipt)

到这里我们就实现了console和rpc的方式来和以太坊交互了。


