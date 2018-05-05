---
layout: post
keywords: Facebook Thrift
title: 说说facebook thrift
date: 2017-03-28
categories: Java
tags:
      - RPC
      - Thrift
---

Apache Thrift 是 Facebook 实现的一种高效的、支持多种编程语言的远程服务调用的框架。
### 前言
目前流行的服务调用方式有很多种，例如基于 SOAP 消息格式的 Web Service，基于 JSON 消息格式的 RESTful 服务等。其中所用到的数据传输方式包括 XML，JSON 等，然而 XML 相对体积太大，传输效率低，JSON 体积较小，新颖，但还不够完善。本文将介绍由 Facebook 开发的远程服务调用框架 Apache Thrift，它采用接口描述语言定义并创建服务，支持可扩展的跨语言服务开发，所包含的代码生成引擎可以在多种语言中，如 C++, Java, Python, PHP, Ruby, Erlang, Perl, Haskell, C#, Cocoa, Smalltalk 等创建高效的、无缝的服务，其传输数据采用二进制格式，相对 XML 和 JSON 体积更小，对于高并发、大数据量和多语言的环境更有优势。本文将详细介绍 Thrift 的使用，并且提供丰富的实例代码加以解释说明，帮助使用者快速构建服务。

### 安装
在Mac上通过终端先安装brew，再在终端输入brew命令行来安装thrift的，当然还有其他的方式，大家可以去网上了解一下，而我用brew是对其偏爱，是因为brew作为Mac OSX上的软件包管理工具，能在Mac中方便的安装软件或者卸载软件， 只需要一个命令， 而不用麻烦的终端命令，非常方便，另外brew 又叫Homebrew。

```
brew install thrift
```

安装成功后，使用如下命令查看：
```
thrift -version
```
之前的版本为0.9.3，使用 `brew upgrade thrift` 更新，之后的版本为：0.10.0
<!-- more -->
### Thrift的组成

1、类型系统以及 IDL(interface definition language) 编译器：负责由用户给定的 IDL 文件生成相应语言的接口代码  
2、TProtocol：实现 RPC 的协议层，可以选择多种不同的对象串行化方式，如 JSON, Binary。  
3、TTransport：实现 RPC 的传输层，同样可以选择不同的传输层实现，如socket, 非阻塞的 socket, MemoryBuffer 等。  
4、TProcessor：作为协议层和用户提供的服务实现之间的纽带，负责调用服务实现的接口。  
5、TServer：聚合 TProtocol, TTransport 和 TProcessor 几个对象。  

### Thrift架构
<img src="http://oe7n2xiy9.bkt.clouddn.com/thrift/thrift-framework.png" />

图中前面3个部分是：

1、你通过Thrift脚本文件生成的代码

2、图中的褐色框部分是你根据生成代码构建的客户端和处理器的代码

3、图中红色的部分是两端产生的计算结果

从TProtocol下面3个部分是Thrift的传输体系和传输协议以及底层I/O通信，Thrift并且提供 堵塞、非阻塞，单线程、多线程的模式运行在服务器上，还可以配合服务器/容器一起运行，可以和现有JEE服务器/Web容器无缝的结合。

### 数据类型

#### 基本类型
- **bool**：布尔值，true 或 false，对应 Java 的 boolean
- **byte**：8 位有符号整数，对应 Java 的 byte
- **i16**：16 位有符号整数，对应 Java 的 short
- **i32**：32 位有符号整数，对应 Java 的 int
- **i64**：64 位有符号整数，对应 Java 的 long
- **double**：64 位浮点数，对应 Java 的 double
- **string**：未知编码文本或二进制字符串，对应 Java 的 String

#### 结构体类型
**struct**：定义公共的对象，类似于 C 语言中的结构体定义，在 Java 中是一个 JavaBean

#### 容器类型
- **list**：对应 Java 的 ArrayList
- **set**：对应 Java 的 HashSet
- **map**：对应 Java 的 HashMap

#### 异常类型
- **exception**：对应 Java 的 Exception

#### 服务类型
- **service**：对应服务的类

### 协议
Thrift 可以让用户选择客户端与服务端之间传输通信协议的类别，在传输协议上总体划分为文本 (text) 和二进制 (binary) 传输协议，为节约带宽，提高传输效率，一般情况下使用二进制类型的传输协议为多数，有时还会使用基于文本类型的协议，这需要根据项目 / 产品中的实际需求。常用协议有以下几种：

1、TBinaryProtocol – 二进制编码格式进行数据传输
```java
TBinaryProtocol.Factory protocolFactory = new TBinaryProtocol.Factory();
```
2、TCompactProtocol – 高效率的、密集的二进制编码格式进行数据传输
```java
TCompactProtocol.Factory protocolFactory = new TCompactProtocol.Factory();
```
3、TJSONProtocol – 使用JSON的数据编码协议进行数据传输
```java
TJSONProtocol.Factory protocolFactory = new TJSONProtocol.Factory();
```
4、TTupleProtocol - 继承于TCompactProtocol，Struct的编解码时使用更省空间

5、TSimpleJSONProtocol – 只提供JSON只写的协议，适用于通过脚本语言解析

### Transport传输层
1、Transport
* TSocket - 使用阻塞式I/O进行传输，也是最常见的模式。使用经典的JDK Blocking IO的Transport实现。
* TNonblockingSocket - 使用JDK NIO的Transport实现，读写的byte[]会每次被wrap成一个ByteBuffer

2、WrapperTransport
包裹一个底层的Transport，并利用自己的Buffer进行额外的操作。
* TFramedTransport- 使用非阻塞方式，按块的大小，进行传输，类似于Java中的NIO。
* TFastFramedTransport 与TFramedTransport相比，始终使用相同的Buffer，提高了内存的使用率。
* TSaslClientTransport与TSaslServerTransport， 提供SSL校验
* TZlibTransport- 使用执行zlib压缩，不提供Java的实现。

### Processor层
1、TBaseProcessor  
2、TMultiplexedProcessor：支持一个Server支持部署多个Service的情况

### 服务端类型
 * TSimpleServer -  单线程服务器端使用标准的阻塞式I/O。
 * TThreadPoolServer -  多线程服务器端使用标准的阻塞式I/O。
 * TNonblockingServer -  多线程服务器端使用非阻塞式I/O，并且实现了Java中的NIO通道。
 * TThreadedSelectorServer - 多线程半同步半异步模型。 

### 编写thrift文件
```
namespace java me.ilbba.example.thrift.bean

struct Person {
    1: string name,
    2: i32 age
}

service HelloService {
    void sayHello(1: Person person)
}
```

```
thrift --gen java hello_service.thrift
```
执行如上命令，会生成gen-java的文件夹，里面会生成Person.java和HelloService.java。

### 编写HelloService.Iface实现类
```java
public class HelloServiceImpl implements HelloService.Iface {

    @Override
    public void sayHello(Person person) throws TException {
        System.out.println("hello: " + person.getName() + ", your age: " + person.getAge());
    }
}
```

### server端代码
```java
@Slf4j
public class ThriftServiceServer {

    private Thread serverThread;

    private TServer server;

    public void initialize() throws Exception {
        log.info("Start thrift server begin...");
        //传输通道 - 非阻塞方式
        TNonblockingServerTransport serverTransport = new TNonblockingServerSocket(8090);
        //异步IO，需要使用TFramedTransport，它将分块缓存读取。
        TTransportFactory transportFactory = new TFramedTransport.Factory();
        //使用高密度二进制协议
        TProtocolFactory proFactory = new TCompactProtocol.Factory();
        //设置处理器
        TMultiplexedProcessor processor = new TMultiplexedProcessor();
        processor.registerProcessor("HelloService", new HelloService.Processor<HelloService.Iface>(new HelloServiceImpl()));
        //创建服务器
        server = new TThreadedSelectorServer(
                new TThreadedSelectorServer.Args(serverTransport)
                        .protocolFactory(proFactory)
                        .transportFactory(transportFactory)
                        .processor(processor).workerThreads(2 * Runtime.getRuntime().availableProcessors())
        );

        serverThread = new Thread(() -> {
            if (server != null && !server.isServing()) {
                server.serve();
            }
        });
        serverThread.start();
    }

    public void close() {
        log.info("Stop thrift server ...");
        if (serverThread != null && serverThread.isAlive()) {
            serverThread.interrupt();
        }
        if (server != null && server.isServing()) {
            server.stop();
        }
    }

    public static void main(String[] args) throws Exception {
        ThriftServiceServer server = new ThriftServiceServer();
        server.initialize();
    }
}
```

### client端代码
```java
public class HelloServiceClient {

    public void startClient() {
        TTransport transport;
        try {
            transport = new TSocket("localhost", 8090);
            TMultiplexedProtocol multiplexedProtocol = new TMultiplexedProtocol(
                    new TCompactProtocol(new TFramedTransport(transport)), "HelloService");

            HelloService.Client client = new HelloService.Client(multiplexedProtocol);
            transport.open();

            Person person = new Person();
            person.setName("bob");
            person.setAge(25);
            client.sayHello(person);

            transport.close();
        } catch (TTransportException e) {
            e.printStackTrace();
        } catch (TException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        HelloServiceClient client = new HelloServiceClient();
        client.startClient();
    }
}
```

这时启动ThriftServiceServer后，每次调用HelloServiceClient，查看server端日志输出。

代码可以参考[github](https://github.com/amuguelove/thrift-learning-example/tree/master)

