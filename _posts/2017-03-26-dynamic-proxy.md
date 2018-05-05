---
layout: post
keywords: Java 动态代理
title: 说说jdk代理以及cglib动态代理
date: 2017-03-26
categories: Java
tags: 动态代理
---

代理实现可以分为静态代理和动态代理, 下面使用jdk和cglib来分别说明。

### 静态代理
具体实现：

```java
public interface Subject {

    void request();

}

class RealSubject implements Subject {
    public void request(){
        System.out.println("Call RealSubject");
    }
}

class Proxy implements Subject {
    private Subject subject;

    public Proxy(Subject subject) {
        this.subject = subject;
    }

    public void request() {
        System.out.println("begin");
        subject.request();
        System.out.println("end");
    }
}

public class ProxyTest {

    public static void main(String args[]) {
        RealSubject subject = new RealSubject();
        Proxy p = new Proxy(subject);
        p.request();
    }
}
```
<!-- more -->
其中：  
1、RealSubject 是委托类，Proxy 是代理类；  
2、Subject 是委托类和代理类的接口；  
3、request() 是委托类和代理类的共同方法；  

静态代理实现中，一个委托类对应一个代理类，代理类在编译期间就已经确定。

### 动态代理

动态代理中，代理类并不是在Java代码中实现，而是在运行时期生成，相比静态代理，动态代理可以很方便的对委托类的方法进行统一处理，如添加方法调用次数、添加日志功能等等，动态代理分为jdk动态代理和cglib动态代理，下面通过一个例子看看如何实现jdk动态代理。

#### jdk动态代理

1、定义业务逻辑
```java
public interface IService {

    void add();

}

public class UserServiceImpl implements IService {

    @Override
    public void add() {
        System.out.println("call user add method.");
    }
}
```

2、定义代理类的实现
```java
public class JdkDynamicProxy implements InvocationHandler {

    //被代理对象
    private Object target;

    //传递代理目标的实例
    public JdkDynamicProxy(Object target) {
        this.target = target;
    }

    /**
     * 参数说明：
     * proxy是生成的代理对象，method是代理的方法，args是方法接收的参数
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //目标方法之前执行
        System.out.println("do something before...");

        Object result = method.invoke(target, args);

        //目标方法之后执行
        System.out.println("do something after...");
        return result;
    }

    /**
     * 生成代理对象
     */
    public Object getProxy() {
        ClassLoader loader = Thread.currentThread().getContextClassLoader();
        Class<?>[] interfaces = target.getClass().getInterfaces();
        return Proxy.newProxyInstance(loader, interfaces, this);
    }

}
```

3、使用动态代理
```java
public class ProxyTest {

    public static void main(String[] args) {
        //创建目标对象
        IService target = new UserServiceImpl();
        //将目标类和横切类编织在一起
        JdkDynamicProxy handler = new JdkDynamicProxy(target);

        IService proxy = (IService) handler.getProxy();

        proxy.add();
    }
}
```

执行结果：
```
do something before...
call user add method.
do something after...

Process finished with exit code 0
```

#### jdk动态代理使用的局限性
通过反射类Proxy和InvocationHandler回调接口实现的jdk动态代理，要求委托类必须实现一个接口，但事实上并不是所有类都有接口，对于没有实现接口的类，便无法使用该方方式实现动态代理。

#### cglib动态代理
使用cglib[Code Generation Library]实现动态代理，并不要求委托类必须实现接口，底层采用asm字节码生成框架生成代理类的字节码，下面通过一个例子看看使用CGLib如何实现动态代理。

1、定义业务逻辑
```java
public class UserServiceImpl {

    public void add() {
        System.out.println("add a user");
    }

    public void delete(int id) {
        System.out.println("delete a user, id: " + id );
    }

}
```
2、使用cglib定义方法的拦截器
```java
public class CglibProxy implements MethodInterceptor {

    //增强器，动态代码生成器
    private Enhancer enhancer = new Enhancer();

    /**
     * 拦截方法：在代理实例上拦截并处理目标方法的调用，返回结果
     *
     * target: 目标对象代理的实例
     * method: 目标对象调用父类方法的method实例
     * args: 调用父类方法传递参数
     * methodProxy: 代理的方法去调用目标方法
     */
    @Override
    public Object intercept(Object target, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        //目标方法之前执行
        System.out.println("Before: " + method);
        //通过代理类实例调用父类的方法，即是目标业务类方法的调用
        Object object = methodProxy.invokeSuper(target, args);
        //目标方法之后执行
        System.out.println("After: " + method + "\n");
        return object;
    }

    /**
     * 创建代理对象
     * @param clazz
     * @return 返回代理对象
     */
    public Object getProxy(Class clazz) {
        //设置目标类(被代理的类)
        enhancer.setSuperclass(clazz);
        //设置回调(在调用父类方法是，回调this.intercept())
        enhancer.setCallback(this);
        //通过字节码技术动态创建子类实例
        return enhancer.create();
    }
}
```
3、执行测试类
```java
public class ProxyTest {

    public static void main(String[] args) {
        CglibProxy proxy = new CglibProxy();
        UserServiceImpl userService = (UserServiceImpl) proxy.getProxy(UserServiceImpl.class);

        userService.add();

        userService.delete(1);
    }
}
```

执行结果：
```java
Before: public void me.ilbba.aop.example.cglib.UserServiceImpl.add()
add a user
After: public void me.ilbba.aop.example.cglib.UserServiceImpl.add()

Before: public void me.ilbba.aop.example.cglib.UserServiceImpl.delete(int)
delete a user, id: 1
After: public void me.ilbba.aop.example.cglib.UserServiceImpl.delete(int)


Process finished with exit code 0
```

#### jdk和cglib动态代理实现的区别

1、jdk动态代理生成的代理类和委托类实现了相同的接口；  
2、cglib动态代理中生成的字节码更加复杂，生成的代理类是委托类的子类，且不能处理被final关键字修饰的方法；  
3、jdk采用反射机制调用委托类的方法，cglib采用类似索引的方式直接调用委托类方法。  

