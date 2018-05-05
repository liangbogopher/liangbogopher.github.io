---
layout: post
keywords: Java 设计模式
title: 单例模式总结
date: 2017-03-18
categories: Java
tags: 设计模式
---

单例模式 (Singleton) 也叫单态模式，是设计模式中最为简单的一种模式。

### 第一种（懒汉, 线程不安全）：
```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}

```

这种写法 lazy loading 很明显, 但是致命的是在多线程不能正常工作。
<!-- more -->
### 第二种（懒汉, 线程安全）：

```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {
    }

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
这种写法能够在多线程中很好的工作, 而且看起来它也具备很好的 lazy loading, 但是, 遗憾的是, 效率很低, 99% 情况下不需要同步。

### 第三种（饿汉）：

```java
public class Singleton {
    private static Singleton instance = new Singleton();

    private Singleton() {
    }

    public static Singleton getInstance() {
        return instance;
    }
}
```
这种方式基于 classloder 机制避免了多线程的同步问题, 不过, instance 在类装载时就实例化, 虽然导致类装载的原因有很多种, 在单例模式中大多数都是调用 getInstance 方法, 但是也不能确定有其他的方式（或者其他的静态方法）导致类装载, 这时候初始化 instance 显然没有达到 lazy loading 的效果。

### 第四种（饿汉, 变种）：

```java
public class Singleton {
    private Singleton instance = null;

    static {
        instance = new Singleton();
    }

    private Singleton() {
    }

    public static Singleton getInstance() {
        return this.instance;
    }
}
```
表面上看起来差别挺大, 其实更第三种方式差不多, 都是在类初始化即实例化 instance。

### 第五种（静态内部类）：

```java
public class Singleton {
    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    private Singleton() {
    }

    public static final Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```
这种方式同样利用了classloder的机制来保证初始化instance时只有一个线程, 它跟第三种和第四种方式不同的是（很细微的差别）：第三种和第四种方式是只要Singleton类被装载了, 那么instance就会被实例化（没有达到lazy loading效果）, 而这种方式是Singleton类被装载了, instance不一定被初始化。因为SingletonHolder类没有被主动使用, 只有显示通过调用getInstance方法时, 才会显示装载SingletonHolder类, 从而实例化instance。想象一下, 如果实例化instance很消耗资源, 我想让他延迟加载, 另外一方面, 我不希望在Singleton类加载时就实例化, 因为我不能确保Singleton类还可能在其他的地方被主动使用从而被加载, 那么这个时候实例化instance显然是不合适的。这个时候, 这种方式相比第三和第四种方式就显得很合理。

### 第六种（枚举）：

```java
public enum Singleton {
    INSTANCE;

    public void whateverMethod() {
    }
}
```

这种方式是 Effective Java作者 Josh Bloch 提倡的方式, 它不仅能避免多线程同步问题, 而且还能防止反序列化重新创建新的对象, 可谓是很坚强的壁垒啊, 不过, 个人认为由于 1.5 中才加入 enum 特性, 用这种方式写不免让人感觉生疏, 在实际工作中, 我也很少看见有人这么写过。

### 第七种（双重校验锁）：

```java
public class Singleton {
    private volatile static Singleton singleton;

    private Singleton() {
    }

    public static Singleton getSingleton() {
        if (singleton == null) {
            synchronized (Singleton.class) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```

这个是第二种方式的升级版, 俗称双重检查锁定, 也有瑕疵。主要在于singleton = new Singleton()这句，这并非是一个原子操作，事实上在 JVM 中这句话大概做了下面 3 件事情。

- 给 singleton 分配内存
- 调用 Singleton 的构造函数来初始化成员变量，形成实例
- 将singleton对象指向分配的内存空间（执行完这步 singleton才是非 null 了）但是在 JVM 的即时编译器中存在指令重排序的优化。也就是说上面的第二步和第三步的顺序是不能保证的，最终的执行顺序可能是 1-2-3 也可能是 1-3-2。如果是后者，则在 3 执行完毕、2 未执行之前，被线程二抢占了，这时 instance 已经是非 null 了（但却没有初始化），所以线程二会直接返回 instance.

在JDK1.5之后, 双重检查锁定才能够正常达到单例效果，1.5之前有个坑。

说这个坑之前我们要先来看看volatile这个关键字。其实这个关键字有两层语义。第一层语义相信大家都比较熟悉，就是可见性。可见性指的是在一个线程中对该变量的修改会马上由工作内存（Work Memory）写回主内存（Main Memory），所以会马上反应在其它线程的读取操作中。顺便一提，工作内存和主内存可以近似理解为实际电脑中的高速缓存和主存，工作内存是线程独享的，主存是线程共享的。volatile的第二层语义是禁止指令重排序优化。大家知道我们写的代码（尤其是多线程代码），由于编译器优化，在实际执行的时候可能与我们编写的顺序不同。编译器只保证程序执行结果与源代码相同，却不保证实际指令的顺序与源代码相同。这在单线程看起来没什么问题，然而一旦引入多线程，这种乱序就可能导致严重问题。volatile关键字就可以从语义上解决这个问题。 但是很不幸，禁止指令重排优化这条语义直到jdk1.5以后才能正确工作。此前的JDK中即使将变量声明为volatile也无法完全避免重排序所导致的问题。所以，在jdk1.5版本前，双重检查锁形式的单例模式是无法保证线程安全的。

### 总结

有两个问题需要注意：

1. 如果单例由不同的类装载器装入, 那便有可能存在多个单例类的实例。假定不是远端存取, 例如一些servlet容器对每个servlet使用完全不同的类 装载器, 这样的话如果有两个servlet访问一个单例类, 它们就都会有各自的实例。

2. 如果 Singleton 实现了 java.io.Serializable 接口, 那么这个类的实例就可能被序列化和复原。不管怎样, 如果你序列化一个单例类的对象, 接下来复原多个那个对象, 那你就会有多个单例类的实例。

对第一个问题修复的办法：

```java
private static Class getClass(String classname) throws ClassNotFoundException {
        ClassLoader classLoader = Thread.currentThread().getContextClassLoader();

        if (classLoader == null) {
            classLoader = Singleton.class.getClassLoader();
        }

        return (classLoader.loadClass(classname));
}

```
对第二个问题修复的办法：

```java
public class Singleton implements java.io.Serializable {
    public static Singleton INSTANCE = new Singleton();

    protected Singleton() {
    }

    private Object readResolve() {
        return INSTANCE;
    }
}
```

对我来说, 我比较喜欢第三种和第五种方式, 简单易懂, 而且在JVM层实现了线程安全（如果不是多个类加载器环境）, 一般的情况下, 我会使用第三种方式, 只有在要明确实现lazy loading效果时才会使用第五种方式, 另外, 如果涉及到反序列化创建对象时我会试着使用枚举的方式来实现单例, 不过, 我一直会保证我的程序是线程安全的, 而且我永远不会使用第一种和第二种方式, 如果有其他特殊的需求, 我可能会使用第七种方式, 毕竟, JDK1.5已经没有双重检查锁定的问题了。 不过一般来说, 第一种不算单例, 第四种和第三种就是一种, 如果算的话, 第五种也可以分开写了。所以说, 一般单例都是五种写法。懒汉, 恶汉, 双重校验锁, 枚举和静态内部类。

### 三大要点

- 线程安全
- 延迟加载
- 序列化与反序列化安全

除了枚举形式, 其他实现方式都有两个共同的缺点

- 都需要额外的工作(Serializable、transient、readResolve())来实现序列化，否则每次反序列化一个序列化的对象实例时都会创建一个新的实例。

- 可能会有人使用反射强行调用我们的私有构造器（如果要避免这种情况，可以修改构造器，让它在创建第二个实例的时候抛异常）。

引自：[单例模式总结](https://segmentfault.com/a/1190000008759300)

