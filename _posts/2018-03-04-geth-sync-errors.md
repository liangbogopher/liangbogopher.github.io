---
layout: post
keywords: 以太坊 区块链 geth
title: 以太坊 geth 节点同步遇到的问题
date: 2018-03-04
categories: 区块链
tags:
    - Geth
    - Ethereum
---
本人在搭建geth节点遇到过许多问题，其中异常一到四引用自网上的博客，在文末有链接地址。
### 服务器配置
服务器配置比较简单，在阿里云上购买的2核4GLinux服务器，操作系统为centos 7.4，另外挂载了一个500G的高速云盘。 
   
如果大家条件允许，可将服务器配置进行升级，比如4核8G，8核16G等，如果配置过低会遇到后面提到的一些问题。

### 节点启动

安装官网提供参数正常启动节点，其中cache参数值配置为512，大家可根据自己的服务器情况适当扩大，有助于节点数据的同步。

### 数据同步

此步骤也是最容易出现问题的地方。针对此步骤的问题详细介绍一下。

<!-- more -->

### 异常一
```
goroutine 10855 [IO wait]:
internal/poll.runtime_pollWait(0x7f4a6599ebb0, 0x72, 0x0)
    /home/travis/.gimme/versions/go1.9.2.linux.amd64/src/runtime/netpoll.go:173 +0x57
internal/poll.(*pollDesc).wait(0xc43863a198, 0x72, 0xffffffffffffff00, 0x184e740, 0x18475a0)
    /home/travis/.gimme/versions/go1.9.2.linux.amd64/src/internal/poll/fd_poll_runtime.go:85 +0xae
internal/poll.(*pollDesc).waitRead(0xc43863a198, 0xc462457a00, 0x20, 0x20)
    /home/travis/.gimme/versions/go1.9.2.linux.amd64/src/internal/poll/fd_poll_runtime.go:90 +0x3d
internal/poll.(*FD).Read(0xc43863a180, 0xc462457a40, 0x20, 0x20, 0x0, 0x0, 0x0)
    /home/travis/.gimme/versions/go1.9.2.linux.amd64/src/internal/poll/fd_unix.go:126 +0x18a
net.(*netFD).Read(0xc43863a180, 0xc462457a40, 0x20, 0x20, 0x0, 0xc42158dcc0, 0x302b35d6a3a0)
    /home/travis/.gimme/versions/go1.9.2.linux.amd64/src/net/fd_unix.go:202 +0x52
net.(*conn).Read(0xc421aac000, 0xc462457a40, 0x20, 0x20, 0x0, 0x0, 0x0)
    /home/travis/.gimme/versions/go1.9.2.linux.amd64/src/net/net.go:176 +0x6d
io.ReadAtLeast(0x7f4a603b02f8, 0xc421aac000, 0xc462457a40, 0x20, 0x20, 0x20, 0xf1e900, 0x464600, 0x7f4a603b02f8)
    /home/travis/.gimme/versions/go1.9.2.linux.amd64/src/io/io.go:309 +0x86
io.ReadFull(0x7f4a603b02f8, 0xc421aac000, 0xc462457a40, 0x20, 0x20, 0x20, 0x0, 0x6fc23a9b4)
    /home/travis/.gimme/versions/go1.9.2.linux.amd64/src/io/io.go:327 +0x58
github.com/ethereum/go-ethereum/p2p.(*rlpxFrameRW).ReadMsg(0xc43432b650, 0xbe9568cbea77ec48, 0x5186ea4942, 0x19d4c80, 0x0, 0x0, 0x19d4c80, 0x28, 0x11, 0x0)
    /home/travis/gopath/src/github.com/ethereum/go-ethereum/p2p/rlpx.go:650 +0x100
github.com/ethereum/go-ethereum/p2p.(*rlpx).ReadMsg(0xc440545da0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0)
    /home/travis/gopath/src/github.com/ethereum/go-ethereum/p2p/rlpx.go:95 +0x148
github.com/ethereum/go-ethereum/p2p.(*Peer).readLoop(0xc4315cc660, 0xc4315cd0e0)
    /home/travis/gopath/src/github.com/ethereum/go-ethereum/p2p/peer.go:251 +0xad
created by github.com/ethereum/go-ethereum/p2p.(*Peer).run
    /home/travis/gopath/src/github.com/ethereum/go-ethereum/p2p/peer.go:189 +0xf2

goroutine 14632 [select]:
net.(*netFD).connect.func2(0x18583c0, 0xc42f87c8a0, 0xc488fcaf00, 0xc4caba3da0, 0xc4caba3d40)
    /home/travis/.gimme/versions/go1.9.2.linux.amd64/src/net/fd_unix.go:129 +0xf2
created by net.(*netFD).connect
    /home/travis/.gimme/versions/go1.9.2.linux.amd64/src/net/fd_unix.go:128 +0x2a3

goroutine 7089 [select]:
github.com/ethereum/go-ethereum/p2p.(*Peer).run(0xc427e1af60, 0xd80820, 0xc44e84bd80, 0x0)
    /home/travis/gopath/src/github.com/ethereum/go-ethereum/p2p/peer.go:199 +0x2fe
github.com/ethereum/go-ethereum/p2p.(*Server).runPeer(0xc4201e2fc0, 0xc427e1af60)
    /home/travis/gopath/src/github.com/ethereum/go-ethereum/p2p/server.go:790 +0x122
created by github.com/ethereum/go-ethereum/p2p.(*Server).run
    /home/travis/gopath/src/github.com/ethereum/go-ethereum/p2p/server.go:570 +0x139c

goroutine 14620 [select]:
net.(*netFD).connect.func2(0x18583c0, 0xc47df23560, 0xc4289ccc80, 0xc4551937a0, 0xc455193740)
    /home/travis/.gimme/versions/go1.9.2.linux.amd64/src/net/fd_unix.go:129 +0xf2
created by net.(*netFD).connect
    /home/travis/.gimme/versions/go1.9.2.linux.amd64/src/net/fd_unix.go:128 +0x2a3
```

**程序同步抛出此异常，一般情况下为内存或IO不足导致程序挂掉。一般情况下重启程序即可。**

### 异常二
```
WARN [02-03|12:54:57] Synchronisation failed, dropping peer    peer=3616e2d0bcacf32f err="retrieved hash chain is invalid"
WARN [02-03|12:56:02] Ancestor below allowance                 peer=64e4dd3f53e5c01e number=4843643 hash=000000…000000 allowance=4843643
WARN [02-03|12:56:02] Synchronisation failed, dropping peer    peer=64e4dd3f53e5c01e err="retrieved ancestor is invalid"

// 和以下异常

WARN [02-03|12:58:55] Synchronisation failed, dropping peer    peer=dbf24adb86cfa3e6 err="no peers available or all tried for download"
WARN [02-03|12:59:23] Synchronisation failed, retrying         err="receipt download canceled (requested)"
WARN [02-03|13:00:17] Synchronisation failed, retrying         err="peer is unknown or unhealthy"
WARN [02-03|13:03:06] Synchronisation failed, retrying         err="block download canceled (requested)"
WARN [02-03|13:03:07] Synchronisation failed, retrying         err="peer is unknown or unhealthy"
```
日志一致卡在此处，说明geth没有链接到其他有效的节点，通过cosole后台执行以下命令可看到链接的节点数为0：
```
> net.peerCount
0
```
针对此警告等待即可，如果长时间无响应，建议重新启动节点，让节点重新寻找新的peers。同时也可以手动添加peer。星火计划提供的节点如下列表，可尝试添加：
```
[
  "enode://6427b7e7446bb05f22fe7ce9ea175ec05858953d75a5a6e4f99a6aec0779a8bd6276f1959a42fe5948acbe14bcd0652082dc546d3b37ae8f2aea41eba4eca43b@121.201.14.181:30303",
  "enode://91922b12115c067005c574844c6bbdb114eb262f90b6355cec89e13b483c3e4669c6d63ec66b6e3ca7a3a462d28edb3c659e9fa05ed4c7234524e582a8816743@120.27.164.92:13333",
  "enode://3dde41a994b3b99f938f75ddf6d48318c78ddd869c70b48d00b922190bb434fc5474f6250c143723f4387273d123e02f6a38f07d0311f240d2915f6140e09850@207.226.141.212:30303",
  "enode://7ab8fa90b204f2146c00939b8474549c544caa3598a0894fa639a5cdbd992cbc6135fd776f8bcf97ae95fdaa3afbfa2d107fea71549119afd7ea57356b899be5@121.201.24.236:30303",
  "enode://db81152a8296089b04a21ad9bf347df3ff0450ffc8215d9f50c400ccf8d18963118010cacf03c4b71981cf9cac5394438cab3039e98db4d2aae5859ab7d1793e@139.198.1.244:30303",
  "enode://68dd1360f0a4ac362b41124692e31652ffe26f6f06a284ca11f3b514b3968594ac1f4320d1aa1ca343b06327c18a2e40eded74edfb3086e1baaa27ca24226b21@113.106.85.172:30303",
  "enode://58f6b6908286cefe43c166cfc4fed033c750caa1bc3f6e1e1e1507752c0b91248addb3122f8557c5f8912e702285a160ab3a10203ae1eff3807eda25d6ed6478@45.113.71.186:30303",
  "enode://87190a01c02cafb97e7f49672b4c3be2937cf79c3969e0b8e7b35cac28cebfbda52a13d56fd2113c726a1dd359c9476ccf7e60651439cef56e3a71039f6a4f5e@119.29.207.90:30303",
  "enode://d1fdd05a62fd9544eeb455e4f4d4bd8bb574138d82d8f909f3041d0792e3401f8695133d39ad0a3aa5d217e3c5bed0511b531505a67b03607a909ae9096720d2@120.26.129.121:30303",
  "enode://a1e9cf99eca94590ae776c8dd5c6c043a8c1f0375e9e391c9fb55133385bf453ac3d3fb3ead8e63415b2ef99d54a19e2a7bc830cd1fdbbb283818e3bcb0ea31e@182.254.209.254:30303",
  "enode://562796b19d43d79dfb6160abd2d7bb78a2f2efd9501a0a767c00677e0fb3a4407235f813c3003682c2a421a58709c52f595827bc15708cc5f534f55d0f8d03ad@121.40.199.54:30303",
  "enode://fa2c17dcc83a6e2825668210abf7480452de4b13d8bdea8f301c3b603701918bc4dade9e68d119d7a8214e90e7ea10a2782041c98951385d97bee73358fb08f4@120.26.124.58:30303",
  "enode://0b331b27e2976d797aed1d1464ac483a7f262860334cb5737a01a0188da08d79226a6973adc5f2a2c1a20192b399161eee23a0d56ecf472cbe4058d010ecc89f@47.89.49.61:30303",
  "enode://0639f20fdb5af1fecd2f2bc0ddb648885483a5945686530e6b046678635d3435dd7b92fe34209f81ec6f003482aa78e407e5e6eb1b10be4773a2adbcf1fc1ba6@118.192.161.147:30303",
  "enode://fd2a5d30e4f3917ee640876cc57d72a8bf5ecf049e9106c95e60cf306dd7a5dd68d1a295f3718af44a7083252686926d6e8a402f1abe6f805e10e7281967db28@121.201.29.82:30303",
  "enode://0d1b9eed7afe2d5878d5d8a4c2066b600a3bcac2e5730586421af224e93a58cd03cac75bf0b2a62fd8049cd3692a085758cc1e407c8b2c94bb069814a5e8d0f0@209.9.106.245:30303",
  "enode://ca087a651571d04953187753af969f7deb1582af2a06a3048b90adb3f87d4c41973aac4b5e80449efc97154dac769a5ea447b123c3aaf7a2c23825a1558804dc@182.150.37.23:30303",
 "enode://9b53b9d41d964f71db60d2198cfa9013fc7808d707c5e0a32da1e22d3cacd6adbae46901df6506a752d9d4e3791df29171315fbb86f7b09331a25458158fe65b@182.150.37.24:30303"
]
```

### 异常三
geth莫名其妙自动关闭，日志未呈现异常。此问题之前的文章也提到过，因为服务器内存不足触发Linux的OOM killer操作，被杀掉了。此问题除了升级内存，没有太好的办法，只能频繁的监控程序，发现问题重启即可。

其中折中的办法是设置swap，但是设置swap会大幅降低同步速度。

### 异常四
疯狂打印类似以下的日志：
```
INFO [02-03|13:07:24] Imported new state entries               count=1142 elapsed=5.888ms   processed=84671 pending=1907  retry=0   duplicate=0 unexpected=170
```
长时间打印以上日志，区块同步高度未变化，在这个日志中没有其他操作日志。如果时间长达几个小时，那么趁早放弃吧，此问题是因为基础设施比如网络、硬盘等原因导致的，短则几天、长则几周，都不好说。

这种问题即使重启服务器还会重新进入这个步骤，就不浪费精力和时间了。好多朋友遇到的都是这个问题，特别是window系统下启动，有的卡到百分之九十九，一直同步不完，基本上都是在执行上面的操作。

**亲身经历**

昨天晚上6点部署好服务器开始节点同步，刚开始由于交易较少同步速度很快。早上起床发现凌晨2点多节点卡死，一直没同步。七点多重启重新同步，这中间经历了多次挂掉，多次程序异常，多次`oom killer`。

当节点数据同步到距离最新高度200块左右的时候一直加载结构体，是一个比较漫长的阶段，大家就耐心等待了，这期间最好不要重启。

### 异常五
在`centos linux`系统中运行geth时，遇见`glibc`版本过低的情况，系统版本为`2.12`需要升级到`2.14`，结果还升级失败一次，导致整个服务器进不去，非常麻烦。这里附上我最后成功的教程地址。切记注意`make install`时出现的错误，因为第一次我就是没有注意到，导致失败的。可以参考如下文章解决：
[Centos6.4升级glibc_2.14](http://blog.csdn.net/ai2000ai/article/details/78983461)

### 异常六
在geth运行过程，发现报错信息 `too many open files`，很明显打开文件数过多了。
可以参考如下文章解决：
[Linux最大文件打开数](https://www.cnblogs.com/pangguoping/p/5791432.html)

### 最后：
本人在CentOS Linux上使用geth同步主网络时，完成同步大概使用了70G左右，但是使用个人电脑 Mac Pro的Ethereum Wallet钱包同步时，磁盘使用了113G，同步到区块460多万左右，区块ID上升很慢，到目前为止总区块大概520万块左右，估计能把磁盘撑满来。现在下载一个geth客户端，使用快速模式`--syncmode fast`并且使用`--ipcpath`指定下ipc的地址，防止和Ethereum Wallet钱包rinkeby网络启动的ipc地址重复。

<hr/>**参考：**
[以太坊geth节点同步亲测经历](http://blog.csdn.net/wo541075754/article/details/79247282)



