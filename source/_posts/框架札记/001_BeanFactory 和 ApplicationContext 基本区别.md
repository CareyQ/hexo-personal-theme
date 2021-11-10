---
title: BeanFactory 和 ApplicationContext 基本区别
categories: 框架札记
desc: ，了解一波 Bean 的即时加载和延时加载。
date: 2021-05-06
url: framework-001
---

Spring 框架中的 IOC 容器有两个：`BeanFactory` 和 `ApplicationContext`，`BeanFactory` 是 Spring 框架 IOC 容器的基础，`ApplicationContext` 是其子接口，拥有其的全部功能的同时还扩展了其他功能。

- `BeanFactory` 是延时加载的，在加载配置文件的时候 Bean 不会被实例化，当使用 `getBean()` 的时候才会实例化该 Bean
- `ApplicationContext` 是即时加载的，在加载配置文件的时候就会配置所有的 Bean
- 在这个方面，`BeanFactory` 是属于轻量级的

下面将通过一个简单的例子来描述。

> 以下例子均为单例模式

## BeanFactory（延时加载）

首先创建一个类 User：

```java
public class User {
    public static boolean isInstance = false;

    public void change() {
        setIsInstance(true);
    }
	// 省略 setters and getters
}
```

创建配置文件，并将 `change()` 方法定义为初始化方法：

```xml
<bean id="user" class="com.careyq.User" init-method="change" />
```

创建一个测试类进行测试：

```java
@Test
public void test() {
    ClassPathResource res = new ClassPathResource("application.xml");
    XmlBeanFactory factory = new XmlBeanFactory(res);
    System.out.println(User.isInstance);	// false
}
```

结果输出 `false`，可见使用 `BeanFactory` 时，只有容器被初始化了，Bean 并没有实例化。

改变测试类，添加 `getBean()` 方法：

```java
@Test
public void test() {
    ClassPathResource res = new ClassPathResource("application.xml");
    XmlBeanFactory factory = new XmlBeanFactory(res);
    factory.getBean("user");
    System.out.println(User.isInstance);	// true
}
```

输出结果为 `true`，验证了结论，`BeanFactory` 只有在获取 Bean 的时候才会对 Bean 进行实例化，按需加载。

## ApplicationContext（即时加载）

修改测试类，使用 `ApplicationContext`：

```java
@Test
public void test2() {
    ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
    System.out.println(User.isInstance);	// true
}
```

在这里并没有使用 `getBean()` 方法，但是结果仍然为 `true`，这就说明了 `ApplicationContext` 在加载配置文件时，就会实例化所有的 Bean。

## 二者选用

虽然 `BeanFactory` 相比 `ApplicationContext` 较为节省资源，但是官方仍然推荐开发者使用 `ApplicationContext`，`ApplicationContext` 增强了 `BeanFactory`，有更多的功能，`ApplicationContext` 提供了国际化、事件发布、基于注解的依赖注入等功能，更加适合用于企业应用程序开发。

同时，`ApplicationContext` 支持所有的 Bean 作用域，而 `BeanFactory` 仅支持 `Singleton` 和 `Prototype` 两个作用域，这在需要开发复杂的程序时，`ApplicationContext` 更加灵活。
