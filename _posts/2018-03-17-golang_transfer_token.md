---
layout: post
keywords: 以太坊 区块链 geth
title: golang 调用智能合约实现以太坊代币转账
date: 2018-03-17
categories: 区块链
tags: Ethereum
---

### 首先获取go-ethereum代码
- `go get github.com/ethereum/go-ethereum`
- 然后我们go-ethereum目录，如果你的golang环境没有问题，那么应该是这个路径。
- `cd $GOPATH/src/github.com/ethereum/go-ethereum`
- 当你进入目录，看到代码已经完整拉取下来，那么我们就可以进行下一步了。

### 连接RPC节点
```
import (
    "github.com/ethereum/go-ethereum/rpc"
    "github.com/ethereum/go-ethereum/ethclient"
)

client, err := ethclient.Dial("http://127.0.0.1:8545")
if err != nil {
    fmt.Printf("err: %v \n", err)
    return
}
```
<!-- more -->
### 创建测试账户
```
import (
    "github.com/ethereum/go-ethereum/accounts/keystore"
)

ks := keystore.NewKeyStore("/Users/liangbo/Library/Ethereum/rinkeby/keystore", keystore.StandardScryptN, keystore.StandardScryptP)
address, _ := ks.NewAccount("password")

key_json, err := ks.Export(address, "password", "password")
if err != nil {
    fmt.Printf("err: %v \n", err)
    return
}

addr = address.Address.Hex()
key = string(key_json)
fmt.Printf("address: %s, private_key: %s \n", addr, key)
```

从上面的代码我们可以看到，我创建了一个以太坊的账户，并且密码设置为password，并导出。最终`addr`变量是账户地址， `key`变量就是账户的私钥，是一段json文本。

### 生成代币token文件
- 打开 `cd $GOPATH/src/github.com/ethereum/go-ethereum/cmd/abigen` 你能看到`main.go`文件

- 执行 `go build main.go`，会在目录下生成一个 `main` 的二进制文件。

- 执行命令：`./main --sol /Users/liangbo/Desktop/token.sol --pkg main --out token.go`

- 在当前目录生成了一个 `token.go` 文件。 

### 开始转账
```
import (
    "github.com/ethereum/go-ethereum/accounts/abi/bind"
)

keystore := "xxxx"  // 你的秘钥json
password := "xxxx" // 你的密码

//首先导入上面生成的账户密钥（json）和密码
auth, err := bind.NewTransactor(strings.NewReader(keystore), password)

// 查看合约地址
// https://etherscan.io/token/0x86fa049857e0209aa7d9e616f7eb3b3b78ecfdb0
// https://rinkeby.etherscan.io/token/0x2F814dBebf9d5ac77bCda9a192B387FC43325873 wtccoin
// https://rinkeby.etherscan.io/token/0x5Eb4db894e254510f83Ae4c4311Dd431C92E94Ba gogocoin
token, err := NewMyAdvancedToken(common.HexToAddress("0x5Eb4db894e254510f83Ae4c4311Dd431C92E94Ba"), client)
if err != nil {
    fmt.Printf("err: %v \n", err)
    return
}

balance, _ := token.BalanceOf(nil, common.HexToAddress(to_address))
fmt.Println("balance:%v", balance.String())

//每个代币都会有相应的位数，例如代币是18位，那么我们转账的时候，需要在金额后面加18个0
decimal, err := token.Decimals(nil)
if err != nil {
    fmt.Printf("err: %v \n", err)
    return
}

amount := big.NewFloat(100.00)
//这是处理位数的代码段
tenDecimal := big.NewFloat(math.Pow(10, float64(decimal)))
convertAmount, _ := new(big.Float).Mul(tenDecimal, amount).Int(&big.Int{})

// 查看交易信息
// https://rinkeby.etherscan.io/tx/0xe891847b31413c9a48b1de2ccd6397b4bd54e849a888e6f1e048fae701a8870c
tx, err := token.Transfer(auth, common.HexToAddress(to_address), convertAmount)
if nil != err {
    fmt.Printf("err: %v \n", err)
    return
}

fmt.Printf("result: %v\n", tx)
```

<hr/>参考文档：  
[用Golang实现以太坊代币转账](https://blog.sodroid.com/2017/11/05/%E7%94%A8Golang%E5%AE%9E%E7%8E%B0%E4%BB%A5%E5%A4%AA%E5%9D%8A%E4%BB%A3%E5%B8%81%E8%BD%AC%E8%B4%A6/)

