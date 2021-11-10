---
title: Spring（七）- AOP 基本使用AOP 基本使用
categories: Spring 学习
date: 2021-04-07
url: spring-study-07
---

Spring 框架中的核心除了 IOC 容器外，还有一个就是 AOP（Aspect Oriented Programming），面向切面编程。AOP 指的是在

程序运行期间，不改变源代码的情况下，动态地将某段代码切入到指定方法、位置中，以达到增强方法的编程方式。常见的使用场景例如：日志、事务、数据库操作等。

## 1. 动态代理

AOP 底层是动态代理。Java 中的代理有两种方式：`JDK` 和 `CGLIB`。

- JDK 基于反射机制，使用实现接口的方式达到方法的增强
- CGLIB 基于继承机制，重写父类方法达到方法的增强

在 Spring 中，要被代理的对象实现了接口，默认采用 `JDK` 的动态代理，没有实现接口，则使用 `CGLIB`。

### 1.1 JDK 动态代理

JDK 的动态代理使用了 `Proxy` 类中的 `newProxyInstance` 方法，如下所示：

```java
static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)
```

- `loader` 代表类加载器
- `interfaces` 被代理对象所实现的接口
- `h` 表示实现了 `InvocationHandler` 接口的类，这个类中有增强的方法

创建一个接口：

```java
public interface Calculator {
    int add(int a, int b);
}
```

创建实现类：

```java
public class CalculatorImpl implements Calculator {
    @Override
    public int add(int a, int b) {
        return a + b;
    }
}
```

实现 `InvocationHandler` 接口：

```java
public class CalculatorInvoke implements InvocationHandler {
    // 要代理的对象，使用有参构造传递
    private CalculatorImpl obj;
    public CalculatorInvoke(CalculatorImpl obj) {
        this.obj = obj;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println(method.getName() + "方法开始执行...");
        System.out.println("参数有：" + Arrays.toString(args));
        Object invoke = method.invoke(obj, args);
        System.out.println(method.getName() + "方法执行结束...");
        return invoke;
    }
}
```

测试：

```java
public class CalculatorProxy {
    public static void main(String[] args) {
        CalculatorImpl calculatorImpl = new CalculatorImpl();
        Calculator calculator = (Calculator) Proxy.newProxyInstance(
                CalculatorProxy.class.getClassLoader(),
                calculatorImpl.getClass().getInterfaces(),
                new CalculatorInvoke(calculatorImpl));
        System.out.println("结果为：" + calculator.add(1, 2));
    }
}
```

输出结果：

```shell
add方法开始执行...
参数有：[1, 2]
add方法执行结束...
结果为：3
```

> 第三个参数可以写为匿名内部类，在这里为了效果，分开编写。

## 2. AOP 中的几个术语

在上一小节的例子中，类里面有 `add` 方法，当然也可以添加 `sub`、`div` 等方法，我们都可以对这些方法进行增强，这些方法称之为 **连接点**。

在这些方法中，实际上只对 `add` 方法做了增强，其他的虽然可以增强，但是我们不需要，那这个真正做了增强的方法称之为 **切入点**。

对 `add` 方法增强了一些输出语句，这一部分则称之为 **通知**。

在将通知添加到切入点的这个过程称之为 **切面**，它是一个动作。

## 3. AOP 使用

在 Spring 框架中，使用 AOP 操作通常是基于 `AspectJ` 来实现的，`AspectJ` 不是 Spring 中的一部分，是一个独立的框架，为了方便使用，就将 `AspectJ` 和 `Spring` 一起结合使用。

首先，使用 Maven 导入相关依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.3.6</version>
    </dependency>
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.9.6</version>
    </dependency>
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjrt</artifactId>
        <version>1.9.6</version>
    </dependency>
</dependencies>
```

### 3.1 使用注解方式

首先创建配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                            http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/context
                            http://www.springframework.org/schema/context/spring-context.xsd
                            http://www.springframework.org/schema/aop
                            http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--开启注解扫描-->
    <context:component-scan base-package="com.careyq.aop_anno"/>
    <!--开启Aspect生成代理对象-->
    <aop:aspectj-autoproxy/>
</beans>
```

创建一个类：

```java
@Component
public class Calculator {
    public void add() {
        System.out.println("add 方法内");
    }
}
```

接下来，定义通知（增强）：

```java
@Component
@Aspect
public class CalculatorProxy {
    // @Before 表示前置通知，在目标方法执行之前执行，注解中填入要切入点
    @Before("execution(* com.careyq.aop_anno.Calculator.add())")
    public void before() {
        System.out.println("方法之前执行...");
    }
}
```

最后，创建测试类：

```java
public class AopTest {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        Calculator calculator = context.getBean("calculator", Calculator.class);
        calculator.add();
    }
}
```

运行结果：

```shell
方法之前执行...
add 方法内
```

### 3.2 execution 表达式

execution 的格式为：

```java
execution(<修饰符><返回类型><方法名>(<参数>)<异常>)
```

其中，返回类型、方法名、参数是必填项，其他的可以省略。在上一节的例子中，表达式为：

```java
execution(* com.careyq.aop_anno.Calculator.add())
```

- `*` 表示任意返回值
- `com.careyq.aop_anno.Calculator.add()` 是该方法的全路径，无参方法

下面再列举几个例子：

```java
execution(public * *(..))	// 任何公共的方法，两个点表示任何参数
execution(* com.careyq.aop_anno.Calculator.*(..))	// Calculator 类下的任何方法
execution(* com.careyq.aop_anno.*.*(..))	// aop_anno 包下的任意类的任何方法
```

同时还可以使用逻辑运算符定义多个切入点

```java
// com.careyq.aop_anno 包下的任意类的任意方法或者 com.careyq.proxy 包下的及其子孙包下的任意类的任意方法
// .表示包下的全部类
// ..表示包、子孙包下的全部类
execution(* com.careyq.aop_anno.*.*(..)) || execution(* com.careyq.proxy..*(..))
```

除了 `||` 还有 `&&`、`!`，在这里只做了初步认识，之后会深入探讨学习。

### 3.3 五种通知

- 前置通知：`@Before`，在目标方法执行之前执行
- 后置通知：`@After`，在目标方法执行之后执行
- 返回通知：`@AfterReturning`，正常返回后执行，如有异常则不执行
- 异常通知：`@AfterThrowing`，异常后才执行
- 环绕通知：`@Around`，在目标方法前后都执行，如有异常，后面那个不执行

在上面的例子中进行扩展：

```java
@Component
@Aspect
public class CalculatorProxy {
    @Before("execution(* com.careyq.aop_anno..*(..)) ")
    public void before() {
        System.out.println("@Before 在目标方法之前执行...");
    }

    @After("execution(* com.careyq.aop_anno..*(..)) ")
    public void after() {
        System.out.println("@After 在目标方法之后执行...");
    }

    @AfterReturning("execution(* com.careyq.aop_anno..*(..)) ")
    public void afterReturning() {
        System.out.println("@AfterReturning 在目标方法正常返回之后执行...");
    }

    @AfterThrowing("execution(* com.careyq.aop_anno..*(..)) ")
    public void afterThrowing() {
        System.out.println("@AfterThrowing 在目标方法异常之后执行...");
    }

    @Around("execution(* com.careyq.aop_anno..*(..)) ")
    public void around(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("@Around 在目标方法【前】执行...");
        joinPoint.proceed();
        System.out.println("@Around 在目标方法【后】执行...");
    }
}
```

输出结果：

```shell
@Around 在目标方法【前】执行...
@Before 在目标方法之前执行...
add 方法内
@AfterReturning 在目标方法正常返回之后执行...
@After 在目标方法之后执行...
@Around 在目标方法【后】执行...
```

接下来，在 `add` 方法中制造一个异常：

```java
@Component
public class Calculator {
    public void add() {
        int i = 10/0;
        System.out.println("add 方法内");
    }
}
```

再进行测试：

```shell
@Around 在目标方法【前】执行...
@Before 在目标方法之前执行...
@AfterThrowing 在目标方法异常之后执行...
@After 在目标方法之后执行...
```

两次结果进行对比，可以得出结论：

- `@After` 后置通知，无论是否抛出异常会执行
- 五种通知的顺序为：`@Around[前]` > `@Before` > `@AfterReturning/@AfterThrowing` > `@After` > `@Around[后]`

### 3.4 抽取公共切入点

在 3.3 节中，多种方式的切入，都使用到了同一个切入点，以至于每一个注解后都带有一长串的表达式，这时可以将表达式抽取出来，简化写法，如下所示：

```java
@Component
@Aspect
public class CalculatorProxy {

    @Pointcut("execution(* com.careyq.aop_anno..*(..))")
    public void point() {
    }

    @Before("point()")
    public void before() {...}

    @After("point()")
    public void after() {...}

    @AfterReturning("point()")
    public void afterReturning() {...}

    @AfterThrowing("point()")
    public void afterThrowing() {...}

    @Around("point()")
    public void around(ProceedingJoinPoint joinPoint) throws Throwable {...}
}
```

### 3.5 使用配置文件方式

还是以上面的代码为基础，删除原来所有类中的注解：

```java
public class Calculator {
    public void add() {
        System.out.println("add 方法内");
    }
}
```

```java
public class CalculatorProxy {
    public void before() {
        System.out.println("@Before 在目标方法之前执行...");
    }

    public void after() {
        System.out.println("@After 在目标方法之后执行...");
    }

    public void afterReturning() {
        System.out.println("@AfterReturning 在目标方法正常返回之后执行...");
    }

    public void afterThrowing() {
        System.out.println("@AfterThrowing 在目标方法异常之后执行...");
    }

    public void around(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("@Around 在目标方法【前】执行...");
        joinPoint.proceed();
        System.out.println("@Around 在目标方法【后】执行...");
    }
}
```

修改配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                            http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/aop
                            http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean id="calculator" class="com.careyq.aop_xml.Calculator" />
    <bean id="calculatorProxy" class="com.careyq.aop_xml.CalculatorProxy"/>

    <aop:config>
        <!--切入点-->
        <aop:pointcut id="point" expression="execution(* com.careyq.aop_xml.Calculator.add(..))"/>
        <!--切面-->
        <aop:aspect ref="calculatorProxy">
            <aop:before method="before" pointcut-ref="point"/>
            <aop:after method="after" pointcut-ref="point"/>
            <aop:after-returning method="afterReturning" pointcut-ref="point"/>
            <aop:after-throwing method="afterThrowing" pointcut-ref="point"/>
            <aop:around method="around" pointcut-ref="point"/>
        </aop:aspect>
    </aop:config>
</beans>
```

测试类不变，输出结果为：

```shell
@Before 在目标方法之前执行...
@Around 在目标方法【前】执行...
add 方法内
@Around 在目标方法【后】执行...
@AfterReturning 在目标方法正常返回之后执行...
@After 在目标方法之后执行...
```

> 使用注解和 XML 配置，二者的执行顺序有所不同，具体原因我还没有深究。
