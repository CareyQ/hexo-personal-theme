---
title: Spring（八）- JdbcTemplate 基本使用
categories: Spring 学习
date: 2021-04-08
url: spring-study-08
---

JdbcTemplate 是 Spring 框架使用 AOP 思想封装的 JDBC 一系列操作的工具。

## 1. 准备

添加依赖：

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.3.6</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.3.6</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.25</version>
        </dependency>
    </dependencies>
```

准备数据库：

```sql
CREATE DATABASE user_db;

USE user_db;

DROP TABLE IF EXISTS `user`;

CREATE TABLE `user` (
    `id` INT(11) NOT NULL AUTO_INCREMENT,
    `username` VARCHAR(255) COLLATE utf8_general_ci DEFAULT NULL,
    `address` VARCHAR(255) COLLATE utf8_general_ci DEFAULT NULL,
    PRIMARY KEY(`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 COLLATE=utf8_general_ci;
```

准备实体类：

```java
public class User {
    private Integer id;
    private String username;
    private String address;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", address='" + address + '\'' +
                '}';
    }
}
```

## 2. 开始使用

创建 XML 配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--数据库的连接池-->
    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource" id="dataSource">
        <property name="username" value="root"/>
        <property name="password" value="123456"/>
        <property name="url" value="jdbc:mysql:///user_db?useUnicode=true&amp;characterEncoding=utf8"/>
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
    </bean>

    <!--JdbcTemplate 对象-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>
</beans>
```

进行测试：

```java
public class UserTest {

    private JdbcTemplate jdbcTemplate;

    @Before
    public void before() {
        ApplicationContext context = new ClassPathXmlApplicationContext("jdbc.xml");
        jdbcTemplate = context.getBean(JdbcTemplate.class);
    }

    // 插入一条数据
    @Test
    public void insert() {
        jdbcTemplate.update("insert into user (username, address) values (?,?);", "CareyQ", "careyq.cool");
    }

    // 更新一条数据
    @Test
    public void update() {
        jdbcTemplate.update("update user set username=? where id=?", "Lenovo", 1);
    }

    // 删除一条数据
    @Test
    public void delete() {
        jdbcTemplate.update("delete from user where id=?", 1);
    }

    // 查询表中的数据总数
    @Test
    public void selectCount() {
        Integer count = jdbcTemplate.queryForObject("select count(*) from user", Integer.class);
        System.out.println(count);
    }

    // 查询一条数据，返回对象
    @Test
    public void selectUser() {
        User user = jdbcTemplate.queryForObject("select * from user where id = ?", new BeanPropertyRowMapper<>(User.class), 1);
        System.out.println(user);
    }

    // 查询多条数据，返回对象集合
    @Test
    public void selectUsers() {
        List<User> userList = jdbcTemplate.query("select * from user", new BeanPropertyRowMapper<>(User.class));
        System.out.println(userList);
    }

    // 批量添加数据
    @Test
    public void batchInsert() {
        ArrayList<Object[]> objects = new ArrayList<>();
        Object[] o1 = {3, "李四", "中国"};
        Object[] o2 = {4, "王五", "杭州"};
        Object[] o3 = {5, "大雄", "日本"};
        objects.add(o1);
        objects.add(o2);
        objects.add(o3);
        jdbcTemplate.batchUpdate("insert into user values (?,?,?)", objects);
    }

    // 批量修改数据
    @Test
    public void batchUpdate() {
        ArrayList<Object[]> objects = new ArrayList<>();
        Object[] o1 = {"李四11", "中国", 3};
        Object[] o2 = {"王五22", "杭州", 4};
        Object[] o3 = {"大雄33", "日本", 5};
        objects.add(o1);
        objects.add(o2);
        objects.add(o3);
        jdbcTemplate.batchUpdate("update user set username=?, address=? where id=?", objects);
    }

    // 批量删除数据
    @Test
    public void batchDelete() {
        ArrayList<Object[]> objects = new ArrayList<>();
        Object[] o1 = {3};
        Object[] o2 = {4};
        Object[] o3 = {5};
        objects.add(o1);
        objects.add(o2);
        objects.add(o3);
        jdbcTemplate.batchUpdate("delete from user where id=?", objects);
    }
}
```