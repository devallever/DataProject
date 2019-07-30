---
title: Java连接数据库
date: 2017-04-21 13:49:27
tags:
- java
- sql
categories: [Java]
---
![](http://www.bing.com/az/hprichbg/rb/SolarFarm_ZH-CN4853771923_1366x768.jpg)


> MySQL provides standards-based drivers for JDBC, ODBC, and .Net enabling developers to build database applications in their language of choice. In addition, a native C library allows developers to embed MySQL directly into their applications.

------

### 1.下载相关的驱动程序
如 Java与MySQL的连接,可到MySQL官网上下载
[下载地址](https://www.mysql.com/products/connector/)

### 2.数据库的URL格式
如: jdbc:mysql://127.0.0.1/dbname
在连接数据库时,我们必须使用各种与数据库类型相关的参数,例如主机名,端口号和数据库名
jdbc的一般语法:jdbc:subprotocol:other stuff
- *subprotocol:连接数据库的具体驱动程序*
- *other stuff:随subprotocol的不同而不同,应查阅供应商的相关文档*

### 3.注册驱动器类
```java
Class.forName("com.mysql.jdbc.Driver");
```
字符串的内容为驱动器类所在包的全路径
这条语句使得驱动器类被加载,由此将执行可以注册驱动器的静态初始化器

### 4.连接到数据库
```java
Connection conn = DriverManager.getConnection(sqlUrl, username, password);
```
以上三个参数类型都是字符串类型, 连接成功会返回Connection对象,用它去执行SQL语句.

### 5.操作数据库
#### 5.1 基本方法
```java
String sql = "select nickname, phone from tuser where username='xm'";
```
执行sql命令首先创建Statement对象或其子类PreparedStatement对象
```java
statement = conn.createStatement();
preparedStatement = conn.preparedStatement(sql);
ResultSet resultSet = statement.executeQuery(sql);
或
resultSet = preparedStatement.executeQuery();
```
executeQuery()方法可以执行select 语句, executeUpdate()方法可以执行insert, update和delete之类的操作. 也可以执行create table 和drop table之类的数据定义语句. execute()方法可以执行任意的sql语句.

