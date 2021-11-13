---
title: Spring（六）- Bean 的作用域和生命周期
categories: Spring 学习
tag:
    - Spring
    - Bean
date: 2021-04-06
url: spring-study-06
---

在之前的几篇文章中，叙述的内容都是围绕 Bean 开展的，Spring 容器所管理的就是 Bean，在本文中，将叙述 Bean 在 Spring 中的作用域和生命周期。

## 1. Bean 的作用域

Spring 中的 Bean 作用域指的是 Bean 是单实例还是多实例。下面是 Bean 的几个作用域：

- `singleton` 单实例，在 Spring 容器中只存在一个实例，也是 Spring 的默认作用域
- `prototype` 多实例，每次从容器中获取 Bean，都会返回一个新的实例
- `request` 每次 HTTP 请求都会创建一个新的 Bean，并且这个 Bean 只在当前请求中有效
- `session` 一个 HTTP Session 使用一个 Bean
- `application` 一个 `ServletContext` 使用一个 Bean
- `websocket` 一个 WebSocket 使用一个 Bean

### 1.1 Singleton

Spring 容器中一个 Bean 只存在一个实例，即使多次获取 Bean，容器都只会返回同一个 Bean 实例。

新建一个示例来看：

```java
public class User {
}
```

配置文件：

```xml
<bean id="user" class="com.careyq.scope.User"/>
```

测试代码：

```java
public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
    User user01 = context.getBean("user", User.class);
    User user02 = context.getBean("user", User.class);
    System.out.println(user01);
    System.out.println(user02);
}
```

测试结果：

```shell
com.careyq.scope.User@e45f292
com.careyq.scope.User@e45f292
```

多次获取 Bean 实例，得到的结果仍然是同一个。

### 1.2 Prototype

想要每次获取 Bean 返回的是不同的实例，需要在配置文件中添加一个属性 `scope`，该属性的默认值是 `singleton`，多实例值是 `prototype`。

修改配置文件：

```xml
<bean id="user" class="com.careyq.scope.User" scope="prototype"/>
```

测试结果：

```shell
com.careyq.scope.User@2f686d1f
com.careyq.scope.User@3fee9989
```

`Singleton` 和 `Prototype` 之间还存在另外一个不同点，Spring 默认使用的是前者，它在加载配置文件的时候就会创建单实例对象，而 `Prototype` 则是在使用 `getBean()` 方法的时候才会创建对象。关于加载配置文件就会创建对象与使用 `ApplicationContext` 有关，可以查看 [BeanFactory 和 ApplicationContext 基本区别](/article/46)。

### 1.3 其他作用域

剩下的几个作用域，需要在 Web 环境下才能配置，没有上面两个使用多，了解一下就可以了。用法如下：

```xml
<bean id="user" class="com.careyq.scope.User" scope="request"/>
<bean id="user" class="com.careyq.scope.User" scope="session"/>
<bean id="user" class="com.careyq.scope.User" scope="application"/>
<bean id="user" class="com.careyq.scope.User" scope="websocket"/>
```

因为系列文章目前还未涉及到 Web，就不对其进行测试了。

## 2. Bean 的生命周期

所谓生命周期，就是对象创建到销毁的过程。而对于 Bean 来说，原始的使用方式，Bean 的生命周期存在几个步骤：

1. 执行构造进行实例化
2. 执行 set 方法进行属性赋值
3. 执行初始化方法
4. 执行销毁方法

使用代码进行描述：

```java
public class User {
    private String name;

    public User() {
        System.out.println("1. 实例化");
    }

    public void setName(String name) {
        this.name = name;
        System.out.println("2. 属性注入");
    }

    public void init() {
        System.out.println("3. 初始化");
    }

    public void destroy() {
        System.out.println("4. 销毁");
    }
}
```

```xml
<bean id="user" class="com.careyq.lifecycle.User" init-method="init" destroy-method="destroy">
    <property name="name" value="CareyQ"/>
</bean>
```

```java
public static void main(String[] args) {
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
    User user = context.getBean("user", User.class);
    context.close(); // 关闭上下文，用于销毁
}
```

运行结果：

```shell
1. 实例化
2. 属性注入
3. 初始化
4. 销毁
```

我在看这一部分的其他文章的时候，牵扯到 `BeanPostProcessor`、`InstantiationAwareBeanPostProcessor` 等内容，这些东西的存在我个人认为前提是 Bean 实现了相关的接口，在实例化或者初始化时，在它们前后添加了一些方法步骤而已，属于扩展的东西。

我不想在本系列文章中对其进行展开描述，应该会把这些放到本站 Spring 加料分类中叙述。