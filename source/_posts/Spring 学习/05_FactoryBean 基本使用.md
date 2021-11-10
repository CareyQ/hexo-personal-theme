---
title: Spring（五）- FactoryBean 基本使用
categories: Spring 学习
date: 2021-04-05
url: spring-study-05
---

Spring 容器中存在两种 Bean：`普通 Bean` 和 `工厂 Bean`。前者通常是 Spring 内部使用的方式，而后者可以自己产生对象，通过实现 `org.springframework.beans.factory.FactoryBean` 工厂类接口，开发者可以自定义 Bean 的创建过程。看不懂？没关系，接着往下看可能会明白。

## 1. 实现 FctoryBean 接口

首先看一下 FactoryBean 接口：

```java
public interface FactoryBean<T> {
    @Nullable
    T getObject() throws Exception;

    @Nullable
    Class<?> getObjectType();

    default boolean isSingleton() {
        return true;
    }
}
```

- `getObject()` 会返回一个对象，这个对象被 Spring 容器所使用
- `getObjectType()` 返回 FactoryBean 生成对象的类型
- `isSingleton()` 表示对象是否是一个单例，在 Spring 5.0 后该方法利用 JDK 1.8 的特性使用了 `default` 方法，并返回 `true`

接下来使一个示例展示这个接口的基本用法：

```java
public class User {
    private String name;

    // 省略有参构造、getter、setter、toString()
}
```

实现 FactoryBean 接口：

```java
public class UserFactory implements FactoryBean<User> {

    private int factoryId;
    private String username;

    @Override
    public User getObject() throws Exception {
        return new User(username);
    }

    @Override
    public Class<?> getObjectType() {
        return User.class;
    }

    @Override
    public boolean isSingleton() {
        return false;
    }

    // 省略 getter、setter、toString()
}
```

`UserFactory` 实现了 `FactoryBean`，可以用它生产 `User` 对象。

## 2. 使用 XML 配置

在配置文件中，对 `UserFactory` 进行配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans ...>

    <bean id="user" class="com.careyq.factorybean.UserFactory">
        <property name="factoryId" value="001"/>
        <property name="username" value="CareyQ"/>
    </bean>
</beans>
```

编写测试类进行测试：

```java
public class FactoryBeanXmlTest {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        User user = context.getBean("user", User.class);
        System.out.println(user);
    }
}
```

结果如下：

```shell
User{name='CareyQ'}
```

根据测试结果，我们在配置文件中配置的是 `UserFactory` 工厂类，能通过它得到 `User` 对象。Spring 使用 `FactoryBean` 生成的 `User` 对象，配置的时候并没有对其进行配置。

当然，我们也可以直接获取 `FactoryBean` 本身：

```java
public class FactoryBeanXmlTest {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        UserFactory userFactory = context.getBean("&user", UserFactory.class);
        System.out.println(userFactory);
    }
}
```

> 在唯一标识前添加 `&` 符号。

结果如下：

```she
UserFactory{factoryId=1, username='CareyQ'}
```

## 3. 使用 Java 配置

使用 Java 配置的方式与 XML 配置不同，需要显式地调用 `getObject()` 方法，新建一个配置类如下：

```java
@Configuration
public class Config {
    @Bean(name = "user")
    public UserFactory userFactory() {
        UserFactory factory = new UserFactory();
        factory.setFactoryId(1);
        factory.setUsername("CareyQ");
        return factory;
    }

    @Bean
    public User user() throws Exception {
        return userFactory().getObject();
    }
}
```

测试类：

```java
public static void main(String[] args) {
    ApplicationContext context = new AnnotationConfigApplicationContext(Config.class);
    User user = context.getBean("user", User.class);
    UserFactory userFactory = context.getBean("&user", UserFactory.class);
    System.out.println(user);
    System.out.println(userFactory);
}
```

结果：

```shell
User{name='CareyQ'}
UserFactory{factoryId=1, username='CareyQ'}
```

> 需要明白的是：FactoryBean 是一种 Bean，不是工厂
