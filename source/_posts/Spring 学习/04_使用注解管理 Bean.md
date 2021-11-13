---
title: Spring（四）- 使用注解管理 Bean
categories: Spring 学习
tag:
    - Spring
    - Bean
date: 2021-04-04
url: spring-study-04
---

在 Spring 中，管理 Bean 的方式除了通过 XML 配置文件外，还有一种是通过注解方式。当拥有许多 Bean 的时候，要实例化它们，就需要在 XML 配置文件中不断增加配置，导致配置文件体积增大等问题出现，而使用注解的目的就是为了简化 XML 配置。

## 1. Bean 的创建

针对 Bean 的创建，Spring 提供了如下几个注解：

- @Component：通用的 Bean 创建注解
- @Service：通常用于 Service 层
- @Controller：通常用于 Web 层（Controller 层）
- @Repository：通常用于 DAO 层

4 个注解功能完全一样，只是名字有区别，方便开发者使用而已，在 DAO 层使用 @Service 照样能成功。

### 1.1 开启组件扫描

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.careyq" />
</beans>
```

`base-package` 属性用于指定要扫描哪个包下的类，这里指的是扫描 `com.careyq` 下的所有类，不论下面是否存在其他包。除了这样直接指定父级包以外，还可以使用逗号分隔要扫描的包路径，例如：

```xml
<context:component-scan base-package="com.careyq.service, com.careyq.dao" />
```

### 1.2 使用注解

创建类，在类上直接加入注解，四个中任意一个都可以：

```java
@Component
public class User {
    private String name;
    private int age;
}
```

`@Component` 等价于 `<bean id="user" class="com.careyq.User" />`。

编写测试类：

```java
public class UserTest {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        User user = context.getBean("user", User.class);
        System.out.println(user);
    }
}
```

结果可以正常创建 Bean：

```shell
com.careyq.User@51b279c9
```

在使用此类注解时，后面可以附加值，例如 `@Component(value = "user")`，对应 XML 配置文件中的 `id="user"`。在这里没有写也能成功，说明它被默认实现了，通常是将类名 `User` 首字母小写为 `user`。

## 2. 组件扫描细化

在配置组件扫描时，可以配置只扫描哪一个注解，或者不扫描哪一个注解。

### 2.1 只扫描指定注解

```xml
<context:component-scan base-package="com.careyq" use-default-filters="false">
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
</context:component-scan>
```

添加 `use-default-filters` 属性，值为 `false`，表示关闭默认的扫描机制（扫描该路径下所有类），`expression` 属性表示只扫描的注解。在这里，**只会扫描** `com.careyq` 包下带有 `@Repository` 注解的 Bean。

修改配置文件后，再次运行测试类，则出现错误：

```shell
NoSuchBeanDefinitionException: No bean named 'user' available
```

可以把原来的 `@Component` 注解改变为 `@Repository`，再进行测试：

```java
@Repository
public class User {
    private String name;
    private int age;
}
```

结果：

```shell
com.careyq.User@140e5a13
```

### 2.2 不扫描指定注解

```xml
<context:component-scan base-package="com.careyq">
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
</context:component-scan>
```

注意和上一个不同的地方在于：

- `context:component-scan` 标签中没有 `use-default-filters` 属性
- 内部是 `context:exclude-filter` 而不是`include`

这样配置以后，将**不会扫描** `com.careyq` 包下带有 `@Repository` 注解的 Bean。

## 3. Bean 的属性注入

关于属性注入，常用的注解有如下几个：

- @Autowired：根据属性的类型进行注入
- @Qualifier：根据属性名称进行注入
- @Resource：以上二者都可
- @Value：普通类型的注入

前三个通常用于对象的注入，最后一个就是基本类型的注入。

### 3.1 @Autowired

创建 2 个类如下：

```java
@Component
public class User {
    private String name;
    @Autowired
    private Book book;

    // 省略 toString()
}

@Component
public class Book {
    private String bookName;

}
```

运行测试类，结果为：

```shell
User{name='null', book=com.careyq.Book@517cd4b}
```

> 这里不再需要定义 setter 方法，内部封装好了

### 3.2 @Qualifier

`@Qualifier` 通常与 `@Autowired` 一起使用。

```java
@Component
public class User {
    private String name;

    @Autowired
    @Qualifier(value = "book")
    private Book book;

    // 省略 toString()
}
```

`@Qualifier` 注解会根据名字去查找 Bean，我们可以把 `Book` 类指定一个名字：

```java
@Component(value = "book1")
public class Book {
    private String bookName;
}
```

运行测试类，出现报错：

```shell
NoSuchBeanDefinitionException: No qualifying bean of type 'com.careyq.Book' available: expected at least 1 bean which qualifies as autowire candidate.
```

把 `Book` 类中 `@Component(value = "book1")` 修改为 `@Component(value = "book")` 或者不写值就能执行成功了，这里不要忘了 Bean 创建注解会默认给我们添加一个名字。

### 3.2 @Resource

这个注解如果不添加值，则会自动根据类型注入。

```java
@Component
public class User {
    private String name;
    @Resource
    private Book book;

    // 省略 toString()
}

@Component
public class Book {
    private String bookName;
}
```

运行结果：

```shell
User{name='null', book=com.careyq.Book@2f490758}
```

如果添加了值，则会自动根据名字进行注入。

```java
@Component
public class User {
    private String name;
    @Resource(name = "book1")
    private Book book;

    // 省略 toString()
}
```

这时候运行就会报错，找不到 `book1` 这个 Bean，解决方法和 `@Qualifier` 一样，这里不再进行说明。

> 该注解并非 Spring 中的注解，它是 Java 自带的，位于 `javax` 注解包下，Spring 不建议使用该注解。

### 3.3 @Value

这个注解用于基本类型的注入，如下：

```java
@Component
public class User {
    @Value("CareyQ") // 不带 value 可以
    private String name;
    @Autowired
    private Book book;

    // 省略 toString()
}

@Component
public class Book {
    @Value(value = "西游记") // 带 value 可以
    private String bookName;

    // 省略 toString()
}
```

运行结果：

```shell
User{name='CareyQ', book=Book{bookName='西游记'}}
```

## 4. 完全使用注解

在上面的例子中，还是使用了 XML 配置文件，要是想完全使用注解开发，可以在新建一个类，如下：

```java
@Configurable
@ComponentScan("com.careyq") // 或者使用下面两个，均可，别名而已
// @ComponentScan(basePackages = "com.careyq")
// @ComponentScan(value = "com.careyq")
public class Config {
}
```

- @Configurable 表示这是一个配置类
- @ComponentScan 表示要扫描的包路径

新建了一个这样的类，就是替换了原本在配置文件中定义的组件扫描，这时可以删除配置文件了，修改对应的测试类如下：

```java
public class UserTest {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(Config.class);
        User user = context.getBean("user", User.class);
        System.out.println(user);
    }
}
```

这里使用的是 `new AnnotationConfigApplicationContext(配置类)`。

运行结果如下：

```shell
User{name='CareyQ', book=Book{bookName='西游记'}}
```
