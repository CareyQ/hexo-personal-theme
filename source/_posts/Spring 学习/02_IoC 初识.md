---
title: Spring（二）- IoC 初识
categories: Spring 学习
tag:
    - Spring
    - 控制反转
date: 2021-04-02
url: spring-study-02
---

## 1. IoC 概述

控制反转（Inversion of Control，IoC）是 Spring 框架中的一个核心，主要用于管理框架中的对象（Bean）。这里要明白，IoC 不是一种技术，而是一种概念、一种思想，使用这种思想来设计开发程序，能够使程序更加优良，耦合度更低。

IoC 实际上就是指对一个对象的控制权限进行反转，控制权限转移了，创建对象的方式也就改变了。传统的对象由开发者进行创建，使用 IoC 方式后，对象的管理交给了 Spring，这就是所谓的控制反转。

在以前的开发方式中，我们通常使用如下的写法：

```java
public class User {
    private String name;
    private int age;
    // 省略 setters and getters
}

public class UserService {
    public void init() {
        User user = new User();
        user.setName("CareyQ");
        user.setAge(18);
    }
}
```

User 的创建在 UserService 中，二者之间的耦合度就较高，其他对象想要使用 User，需要重新创建，同时，对象的创建、初始化等操作都是开发者完成的。使用 Spring 就能摆脱这种方式，这些操作将交给 IoC 容器来管理。

在 Spring 官方文档中，IoC 也被称为依赖注入（Dependency Injection，DI），[原文链接](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-introduction)。DI 是 IoC 的一种实现，能够对 Bean 的属性进行注入，需要在 Bean 创建了的基础上才能进行操作，之后会出一篇二者之间更详细的文章，在此仅粗略描述。

## 2. IoC 初识

新建一个普通的 Maven 项目，并添加 `spring-context` 依赖。

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.3.6</version>
</dependency>
```

在 `resources ` 目录下创建 XML 配置文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="user" class="com.careyq.User" >
        <property name="name" value="CareyQ" />
        <property name="age" value="18" />
    </bean>
</beans>
```

> 各个标签和属性的作用将会在之后的章节进行叙述，这里就先练练手

编写主类/测试类。

```java
public class UserService {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        User user = context.getBean("user", User.class);
        System.out.println(user.getName() + "->" + user.getAge());	// CareyQ->18
    }
}
```

- 程序首先会加载 XML 配置文件，并初始化一个实例
- 之后通过 `getBean()` 从容器中获取对象

这个过程中，开发者并没有手动的去创建对象，而是使用了配置文件的方式，这样的好处就在于两个类之间的耦合度变小了，例如 User 类改变了包位置，只需要修改配置文件即可，无需再到使用到 User 的类中一个个修改（这个例子，请不要结合 IDE 的辅助功能）。

## 3. IoC 底层初识

根据上一小节中的主类可以看出，Spring 在管理 Bean 的过程中存在如下几步：

- 加载配置文件
- 获取 Bean

在这两步中，IoC 底层使用到了 `XML 解析`、`工厂模式`、`反射`。加载配置文件用到了 XML 解析和工厂模式，获取 Bean 用到了反射。

IoC 容器通常有两种：`ApplicationContext` 和 `BeanFactory`，二者的基本区别可 [点此查看](/blog/framework-001)，推荐使用 `ApplicationContext`，它在加载配置文件的时候就对 Bean 进行了实例化。

在 `ApplicationContext` 接口中存在两个实现类：`FileSystemXmlApplicationContext` 和 `ClassPathXmlApplicationContext`。

- `FileSystemXmlApplicationContext` 需要填入 XML 配置文件的绝对路径，基于盘符的，例如：E:\spring\src\main\resources\application.xml
- `ClassPathXmlApplicationContext` 需要填入 XML 配置文件的相对路径，基于 classpath
