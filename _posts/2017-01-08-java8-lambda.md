---
layout: post
keywords: Java lambda
title: Java8 新特性 - lambda表达式
date: 2017-01-08
categories: Java
tags:
      - Java
      - Lambda
---

lambda表达式是Java8给我们带来的几个重量级新特性之一，借用lambda表达式，可以让我们的Java程序设计更加简洁。

### 行为参数化
行为参数化简单的说就是函数的主体仅包含模板类通用代码，而一些会随着业务场景而变化的逻辑则以参数的形式传递到函数之中，采用行为参数化可以让程序更加的通用，以应对频繁变更的需求。

考虑一个业务场景，假设我们需要通过程序对苹果进行筛选，我们先定义一个苹果的实体：
```java
public class Apple {

    /** 编号 */
    private long id;

    /** 颜色 */
    private Color color;

    /** 重量 */
    private float weight;

    /** 产地 */
    private String origin;

    public Apple() {
    }

    public Apple(long id, Color color, float weight, String origin) {
        this.id = id;
        this.color = color;
        this.weight = weight;
        this.origin = origin;
    }

    // 省略getter和setter
}
```
<!-- more -->
用户最开始的需求可能只是简单的希望能够通过程序筛选出绿色的苹果，于是我们可以很快的通过程序实现：
```java
/**
 * 筛选绿苹果
 *
 * @param apples
 * @return
 */
public static List<Apple> filterGreenApples(List<Apple> apples) {
    List<Apple> filterApples = new ArrayList<>();
    for (final Apple apple : apples) {
        if (Color.GREEN.equals(apple.getColor())) {
            filterApples.add(apple);
        }
    }
    return filterApples;
}
```

如果过了一段时间用户提出了新的需求，希望能够通过程序筛选出红色的苹果，于是我们又针对性的添加了筛选红色苹果的功能：
```java
/**
 * 筛选红苹果
 *
 * @param apples
 * @return
 */
public static List<Apple> filterRedApples(List<Apple> apples) {
    List<Apple> filterApples = new ArrayList<>();
    for (final Apple apple : apples) {
        if (Color.RED.equals(apple.getColor())) {
            filterApples.add(apple);
        }
    }
    return filterApples;
}
```

更好的实现是把颜色作为一个参数传递到函数中，这样就可以应对以后用户提出的各种颜色筛选请求了：

```java
/**
 * 自定义筛选颜色
 *
 * @param apples
 * @param color
 * @return
 */
public static List<Apple> filterApplesByColor(List<Apple> apples, Color color) {
    List<Apple> filterApples = new ArrayList<>();
    for (final Apple apple : apples) {
        if (color.equals(apple.getColor())) {
            filterApples.add(apple);
        }
    }
    return filterApples;
}
```
这样设计了之后，再也不用担心用户的颜色筛选需求变化了，但是不幸的是，某一天用户提了一个需求要求能够选择重量达到某一标准的苹果，有了前面的教训，我们也把重量的标准作为参数传递给筛选函数，于是得到：

```java
/**
 * 筛选指定颜色，且重要符合要求
 *
 * @param apples
 * @param color
 * @param weight
 * @return
 */
public static List<Apple> filterApplesByColorAndWeight(List<Apple> apples, Color color, float weight) {
    List<Apple> filterApples = new ArrayList<>();
    for (final Apple apple : apples) {
        if (color.equals(apple.getColor()) && apple.getWeight() >= weight) {
            filterApples.add(apple);
        }
    }
    return filterApples;
}
```

这样通过传递参数的方式真的好吗？如果筛选条件越来越多，组合模式越来越复杂，我们是不是需要考虑到所有的情况，并针对每一种情况都有相应的应对策略呢，并且这些函数仅仅是筛选条件的部分不一样，其余部分都是相同的模板代码（遍历集合），这个时候我们就可以将行为 参数化 ，让函数仅保留模板代码，而把筛选条件抽离出来当做参数传递进来，在java8之前，我们通过定义一个过滤器接口来实现：
```java
@FunctionalInterface
public interface AppleFilter {

    /**
     * 筛选条件抽象
     *
     * @param apple
     * @return
     */
    boolean accept(Apple apple);

}

/**
 * 将筛选条件封装成接口
 *
 * @param apples
 * @param filter
 * @return
 */
public static List<Apple> filterApplesByAppleFilter(List<Apple> apples, AppleFilter filter) {
    List<Apple> filterApples = new ArrayList<>();
    for (final Apple apple : apples) {
        if (filter.accept(apple)) {
            filterApples.add(apple);
        }
    }
    return filterApples;
}
```

通过上面行为抽象化之后，我们可以在具体调用的地方设置筛选条件，并将条件作为参数传递到方法中：
```java
public static void main(String[] args) {
    List<Apple> apples = new ArrayList<>();

    // 筛选苹果
    List<Apple> filterApples = filterApplesByAppleFilter(apples, new AppleFilter() {
        @Override
        public boolean accept(Apple apple) {
            // 筛选重量大于100g的红苹果
            return Color.RED.equals(apple.getColor()) && apple.getWeight() > 100;
        }
    });
}
```
上面的行为参数化方式采用匿名类来实现，这样的设计在jdk内部也经常采用，比如java.util.Comparator，java.util.concurrent.Callable等，使用这一类接口的时候，我们都可以在具体调用的地方用过匿名类来指定函数的具体执行逻辑，不过从上面的代码块来看，虽然很极客，但是不够简洁，在java8中我们可以通过lambda来简化：

```java
// 筛选苹果
List<Apple> filterApples = filterApplesByAppleFilter(apples,
        (Apple apple) -> Color.RED.equals(apple.getColor()) && apple.getWeight() >= 100);
```

通过lambda表达式极大的精简了代码，下面来学习java的lambda表达式吧~

### lambda表达式定义
我们可以将lambda表达式定义为一种 简洁、可传递的匿名函数，首先我们需要明确lambda表达式本质上是一个函数，虽然它不属于某个特定的类，但具备参数列表、函数主体、返回类型，以及能够抛出异常；其次它是匿名的，lambda表达式没有具体的函数名称；lambda表达式可以像参数一样进行传递，从而极大的简化代码的编写。格式定义如下：

```
格式一： 参数列表 -> 表达式
格式二： 参数列表 -> {表达式集合}
```
需要注意的是，lambda表达式隐含了return关键字，所以在单个的表达式中，我们无需显式的写return关键字，但是当表达式是一个语句集合的时候，则需要显式添加return，并用花括号{ }将多个表达式包围起来，下面看几个例子：

```java
(String s) -> s.length() 

// 始终返回42的无参方法
() -> 42 

// 包含多行表达式，则用花括号括起来
(int x, int y) -> {
    int z = x * y;
    return x + z;
}
```

### 函数式接口
函数式接口定义为只具备 一个抽象方法 的接口。java8在接口定义上的改进就是引入了默认方法，使得我们可以在接口中对方法提供默认的实现，但是不管存在多少个默认方法，只要具备一个且只有一个抽象方法，那么它就是函数式接口，如下（引用上面的AppleFilter）：

```java
@FunctionalInterface
public interface AppleFilter {

    /**
     * 筛选条件抽象
     *
     * @param apple
     * @return
     */
    boolean accept(Apple apple);

}
```

AppleFilter仅包含一个抽象方法accept(Apple apple)，依照定义可以将其视为一个函数式接口，在定义时我们为该接口添加了@FunctionalInterface注解，用于标记该接口是函数式接口，不过这个接口是可选的，当添加了该接口之后，编译器就限制了该接口只允许有一个抽象方法，否则报错，所以推荐为函数式接口添加该注解。

#### jdk自带的函数式接口
jdk为lambda表达式已经内置了丰富的函数式接口，如下表所示(仅列出部分)：
<img src="http://oe7n2xiy9.bkt.clouddn.com/java8/lambda_0001.png" />

##### `Predicate<T>`
```java
@FunctionalInterface
public interface Predicate<T> {

    /**
     * Evaluates this predicate on the given argument.
     *
     * @param t the input argument
     * @return {@code true} if the input argument matches the predicate,
     * otherwise {@code false}
     */
    boolean test(T t);
}
```
Predicate的功能类似于上面的AppleFilter，利用我们在外部设定的条件对于传入的参数进行校验，并返回验证结果boolean，下面利用Predicate对List集合的元素进行过滤
```java
/**
 * 按照指定的条件对集合元素进行过滤
 *
 * @param list
 * @param predicate
 * @param <T>
 * @return
 */
public <T> List<T> filter(List<T> list, Predicate<T> predicate) {
    List<T> newList = new ArrayList<T>();
    for (final T t : list) {
        if (predicate.test(t)) {
            newList.add(t);
        }
    }
    return newList;
}
```

利用上面的函数式接口过滤字符串集合中的空字符串：
```java
demo.filter(list, (String str) -> null != str && !str.isEmpty());
```

##### `Consumer<T>`
```java
@FunctionalInterface
public interface Consumer<T> {

    /**
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
    void accept(T t);
}

```
Consumer提供了一个accept抽象函数，该函数接收参数，但不返回值，下面利用Consumer遍历集合：
```java
/**
 * 遍历集合，执行自定义行为
 *
 * @param list
 * @param consumer
 * @param <T>
 */
public <T> void filter(List<T> list, Consumer<T> consumer) {
    for (final T t : list) {
        consumer.accept(t);
    }
}
```

利用上面的函数式接口，遍历字符串集合，并打印非空字符串：
```java
demo.filter(list, (String str) -> {
        if (StringUtils.isNotBlank(str)) {
            System.out.println(str);
        }
    });
```

##### ` Function <T, R> `
```java
@FunctionalInterface
public interface Function<T, R> {

    /**
     * Applies this function to the given argument.
     *
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);
}
```

Funcation执行转换操作，输入是类型T的数据，返回R类型的数据，下面利用Function对集合进行转换：
```java
/**
 * 遍历集合，执行自定义转换操作
 *
 * @param list
 * @param function
 * @param <T>
 * @param <R>
 * @return
 */
public <T, R> List<R> filter(List<T> list, Function<T, R> function) {
    List<R> newList = new ArrayList<R>();
    for (final T t : list) {
        newList.add(function.apply(t));
    }
    return newList;
}
```
下面利用上面的函数式接口，将一个封装字符串（整型数字的字符串表示）的接口，转换成整型集合：
```java
demo.filter(list, (String str) -> Integer.parseInt(str));
```

##### 类型推断
在编码过程中，有时候可能会疑惑我们的调用代码会去具体匹配哪个函数式接口，实际上编译器会根据参数、返回类型、异常类型（如果存在）等做正确的判定。
在具体调用时，在一些时候可以省略参数的类型，从而进一步简化代码
```java
// 筛选苹果
List<Apple> filterApples = filterApplesByAppleFilter(apples,
        (Apple apple) -> Color.RED.equals(apple.getColor()) && apple.getWeight() >= 100);

// 某些情况下我们甚至可以省略参数类型，编译器会根据上下文正确判断
List<Apple> filterApples = filterApplesByAppleFilter(apples,
        apple -> Color.RED.equals(apple.getColor()) && apple.getWeight() >= 100);
```

##### 方法引用
采用方法引用可以更近一步的简化代码，有时候这种简化让代码看上去更加的直观，先看一个例子：
```java
/* ... 省略apples的初始化操作 */

// 采用lambda表达式
apples.sort((Apple a, Apple b) -> Float.compare(a.getWeight(), b.getWeight()));

// 采用方法引用
apples.sort(Comparator.comparing(Apple::getWeight));
```
方法引用通过::将方法隶属和方法自身连接起来，主要分为三类：

1) 静态方法引用
```
(args) -> ClassName.staticMethod(args)
转换成
ClassName::staticMethod

```

2) 某个类的成员方法的引用
```
(args) -> args.instanceMethod()
转换成
ClassName::instanceMethod  // ClassName是args的类型
```

3) 某个实例对象的成员方法的引用
```
(args) -> ext.instanceMethod(args)
转换成
ext::instanceMethod(args)
```
4) 构造器引用
语法是Class::new，或者更一般的形式：Class<T>::new。注意：这个构造器没有参数。

<hr>书籍推荐：Java 8实战
引自：[Java8新特性 - lambda表达式](https://my.oschina.net/wangzhenchao/blog/747674)


