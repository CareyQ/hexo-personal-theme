---
title: Spring（三）- 使用 XML 管理 Bean
categories: Spring 学习
date: 2021-04-03
url: spring-study-03
---

## 1. IoC 管理 Bean

Spring 中的 IoC 容器就是用于管理 Bean 的，通常我们在使用 Spring 管理 Bean 时，指的就是 Bean 的创建和其属性的设置。在上一节 [IoC 初识](/article/47) 中，文章中的例子就是 Spring 常见的一种 Bean 管理方式。

在 Spring 中，IoC 管理 Bean 的方式存在两种：

- 基于 XML 配置文件方式
- 基于注解方式

本篇文章主要是对 XML 配置文件方式管理 Bean 的讲解，注解方式将在下一章进行叙述。

先准备一个类

```java
public class User {
    private String name;
    private int age;
}
```

再准备一个 XML 配置文件，在配置文件中会对这个类进行实例化以及属性的赋值操作。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd">

</beans>
```

最后准备一个测试类：

```java
public class UserTest {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        User user = context.getBean("user", User.class);
        System.out.println(user);
    }
}
```

## 2. Bean 的创建

在 XML 配置文件中，使用 `bean` 标签实现对象的创建，通常会在标签中添加对应的属性。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- Bean 创建 -->
	<bean id="user" class="com.careyq.User" />
</beans>
```

在 `bean` 标签中有两个属性：`id` 和 `class`。

- `id` 是唯一标识，可以随意命名，但是并不建议这样做，通常会和类名一致
- `class` 是类的全路径，包 + 类

使用测试类进行测试，结果如下，Bean 被创建了：

```shell
com.careyq.User@e45f292
```

Bean 的创建默认使用了类中的无参构造，众所周知，我们没有在类中定义无参构造，编译时会自动添加一个构造方法，现在我们在类中添加一个有参构造。

```java
public class User {
    private String name;
    private int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

再执行测试类，将会报错：

```shell
Caused by: java.lang.NoSuchMethodException: com.careyq.User.<init>()
```

## 3. Bean 的属性注入

在 XML 配置文件中，Bean 的属性注入存在多种方法。

### 3.1 set 方法注入

使用了 set 方法进行属性注入，需要在类中添加对应的 set 方法：

```java
public class User {
    private String name;
    private int age;

    public void setName(String name) {
        this.name = name;
    }

    public void setAge(int age) {
        this.age = age;
    }

    // 省略 toString()...
}
```

修改 XML 配置文件：

```xml
<bean id="user" class="com.careyq.User" >
    <property name="name" value="CareyQ" />
    <property name="age" value="18" />
</bean>
```

运行测试类，打印结果：

```shell
User{name='CareyQ', age=18}
```

使用 set 进行属性注入，需要注意一个问题，初学者总会有一个错觉，认为 `property` 标签中的 `name` 属性值对应了类中的属性名，其实并不是如此。我们修改类如下：

```java
public class User {
    private String bname; // 修改了属性名
    private int bage;

    public void setName(String name) {
        this.bname = name;
    }

    public void setAge(int age) {
        this.bage = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "bname='" + bname + '\'' +
                ", bage=" + bage +
                '}';
    }
}
```

再运行结果：

```shell
User{bname='CareyQ', bage=18}
```

我将属性 `name` 修改为了 `bname`，同样能够正确的注入。其实 `property` 标签中的 `name` 属性值是通过分析 set 方法得到的，set 方式注入使用到了反射机制，在框架中，凡是使用到反射进行值注入的，属性名都不是 Bean 中所定义的属性变量名，而是通过 Java 中的内省机制分析出来的属性名。

我们可以再次修改类：

```java
public class User {
    private String name;
    private int age;

  	// 修改了 set 方法名
    public void setBName(String name) {
        this.name = name;
    }

    public void setBAge(int age) {
        this.age = age;
    }
	// 省略 toString()...
}
```

执行结果：

```shell
Caused by: org.springframework.beans.NotWritablePropertyException: Invalid property 'name' of bean class [com.careyq.User]: Bean property 'name' is not writable or has an invalid setter method. Did you mean 'BName'?
```

它告诉我们 `name` 这个属性是无效的，由此可见，使用 set 方式注入，是通过分析类中的 `setXxx()` 方法实现的。

### 3.2 有参构造方法注入

既然是有参构造方法注入，那么类中肯定需要定义有参构造方法：

```java
public class User {
    private String name;
    private int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
	// 省略 toString()...
}
```

编写配置文件：

```xml
<bean id="user" class="com.careyq.User" >
    <constructor-arg name="name" value="CareyQ" />
    <constructor-arg name="age" value="18" />
</bean>
```

测试结果：

```shell
User{name='CareyQ', age=18}
```

在 `constructor-arg` 标签中，`name` 属性对应有参构造的参数名，`value` 对应想要注入的值。这里需要注意，如果有多个构造方法，它会根据参数个数及参数类型自动匹配对应的构造方法。

使用有参构造方式，还有另外一种写法，使用 `index` 属性，如下：

```xml
<bean id="user" class="com.careyq.User" >
    <constructor-arg index="0" value="CareyQ" />
    <constructor-arg index="1" value="18" />
</bean>
```

`index` 是索引的意思，和数组的索引差不多，所以它的值应该与构造方法参数位置相对应，否则可能出错。

### 3.3 p 名称空间注入

在 set 方法注入时，Bean 中一个属性对应一个 `property` 标签，假设属性比较多，配置文件就会变大，于是就出现了 p 名称空间注入，可以简化配置文件，修改配置文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"	<!--新增的约束-->
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="user" class="com.careyq.User" p:name="CareyQ" p:age="18" />
</beans>
```

测试结果：

```shell
User{name='CareyQ', age=18}
```

p 名称空间注入本质上也是使用了 set 方法，这种方式在实际中使用到的并不多，了解即可。

> 这里需要注意把类修改回带无参构造，拥有 set 方法

## 4. 注入特殊值

### 4.1 注入 NULL

有时候属性值需要是一个 `NULL`，这时就可以修改配置文件如下：

```xml
<bean id="user" class="com.careyq.User">
    <property name="age" value="18" />
    <property name="name">
        <null />
    </property>
</bean>
```

测试结果：

```xml
User{name='null', age=18}
```

### 4.2 注入特殊字符

需要注入特殊符号，例如 `<`、`>`，这种符号可能会被识别为标签符号，产生 `XmlBeanDefinitionStoreException` 异常。这时候可以使用转义字符进行解决：

```xml
<bean id="user" class="com.careyq.User">
    <property name="age" value="18" />
    <property name="name" value="&lt;CareyQ&gt;" />
</bean>
```

测试结果：

```shell
User{name='<CareyQ>', age=18}
```

还有另外一种方式，使用 `CDATA` 实现：

```xml
<bean id="user" class="com.careyq.User">
    <property name="age" value="18"/>
    <property name="name">
        <value><![CDATA[<CareyQ>]]></value>
    </property>
</bean>
```

`CDATA` 结构为 `<![CDATA[你想要注入的值]]>`，在 IDEA 中，直接输入 `CD` 可达到自动输入。

## 5. 注入 Bean（对象）

在类中可能还存在另外一个对象，编写类如下：

```java
public class User {
    private String name;
    private Book book;

	// 省略 setter 和 toString()
}
```

```java
public class Book {
    private String bookName;

    // 省略 setter 和 toString()
}
```

### 5.1 使用外部 Bean

编写配置文件：

```xml
<bean id="user" class="com.careyq.User">
    <property name="name" value="CareQ"/>
    <property name="book" ref="book"/>
</bean>
<bean id="book" class="com.careyq.Book">
    <property name="bookName" value="西游记"/>
</bean>
```

测试结果：

```shell
User{name='CareQ', book=Book{bookName='西游记'}}
```

这里通过使用 `ref` 属性引入另外一个 Bean，填写的是这个 Bean 的唯一标识 `id`。

### 5.2 使用内部 Bean

```xml
<bean id="user" class="com.careyq.User">
    <property name="name" value="CareQ"/>
    <property name="book">
        <bean class="com.careyq.Book">
            <property name="bookName" value="葵花宝典" />
        </bean>
    </property>
</bean>
```

内部 Bean 有点内部类的意思，直接在 `property` 标签内部添加一个 `bean` 标签即可。

## 6. 注入集合

### 6.1 注入数组

准备一个类：

```java
public class User {
    private String name;
    private String[] favorites;

	// 省略 setter 和 toString()
}
```

配置文件：

```xml
<bean id="user" class="com.careyq.User">
    <property name="name" value="CareQ"/>
    <property name="favorites">
        <array>
            <value>吃</value>
            <value>喝</value>
            <value>玩</value>
            <value>乐</value>
        </array>
    </property>
</bean>
```

结果：

```shell
User{name='CareQ', favorites=[吃, 喝, 玩, 乐]}
```

### 6.2 注入 List 集合

准备一个类：

```java
public class User {
    private String name;
    private List<Book> books;

    // 省略 setter 和 toString()
}
```

配置文件：

```xml
<bean id="user" class="com.careyq.User">
    <property name="name" value="CareQ"/>
    <property name="books">
        <list>
            <ref bean="bk1"/>
            <bean class="com.careyq.Book">
                <property name="bookName" value="九阴真经"/>
            </bean>
        </list>
    </property>
</bean>

<bean id="bk1" class="com.careyq.Book">
    <property name="bookName" value="葵花宝典"/>
</bean>
```

结果：

```shell
User{name='CareQ', books=[Book{bookName='葵花宝典'}, Book{bookName='九阴真经'}]}
```

> 如果是基本类型，就像数组一样，外层标签使用 `list`，内部标签使用 `value` 即可。

### 6.3 注入 Map 集合

准备一个类：

```java
public class User {
    private String name;
    private Map<String, Object> map;

	// 省略 setter 和 toString()
}
```

配置文件：

```xml
<bean id="user" class="com.careyq.User">
    <property name="name" value="CareQ"/>
    <property name="map">
        <map>
            <entry key="girlfriend" value="汪啊汪" />
            <entry key="age" value="18" />
        </map>
    </property>
</bean>
```

结果：

```shell
User{name='CareQ', map={girlfriend=汪啊汪, age=18}}
```

> 这些使用大同小异，无非是标签不同而已。

### 6.3 抽取公共集合

配置文件中，需要先引入 util 约束，然后进行配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"	<!--新增-->
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/util	<!--新增-->
        http://www.springframework.org/schema/util/spring-util.xsd">	<!--新增-->

    <util:list id="favorites">
        <value>吃</value>
        <value>喝</value>
        <value>玩</value>
        <value>乐</value>
    </util:list>

    <bean id="user" class="com.careyq.User">
        <property name="name" value="CareQ"/>
        <property name="favorites" ref="favorites"/>
    </bean>
</beans>
```

> 其他类型的抽取，大同小异，这里不再演示。
