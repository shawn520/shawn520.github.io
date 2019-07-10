---
title: Java Web项目如何获取表的ddl信息？
date: 2019-04-04 09:01:26
tags: JDBC
categories: 好好学习
copyright: true
---

背景：公司项目是做MySQL数据向同构和异构数据库同步和消息订阅的。在做向消息中间件RMQ同步时，需要根据源表的表信息，在目标数据库创建一个相同的逻辑表。

如果是用shell的话：`show create table tableName`就可以搞定啦。
![TIM截图20190404082907](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL3NoYXduNTIwL3N0b3JhZ2UvbWFzdGVyL3BpY3R1cmVzL3Nob3ctY3JlYXRlLXRhYmxlLnBuZw)
这里一共有两个参数，第一个是table，第二个是create Table, 我们需要的就是create Table的建表语句。
但是这个用Java怎么实现呢？

<!-- more-->

Mybatis这块暂且还不知道怎么实现，先通过jdbc来实现查询表的ddl语句。
思路一：通过执行`show create table tableName`
思路二：`show create table tablename` 和`show databases` 可能查的是MySQL的数据库information_schema。
```sql
SELECT * FROM information_schema.columns where TABLE_SCHEMA = "test" AND TABLE_NAME = "example" LIMIT 10;

SELECT * FROM information_schema.tables where TABLE_SCHEMA = "test" AND TABLE_NAME = "example" LIMIT 10;
```
可以根据以上等信息去组装ddl语句，比较复杂，而且账号的权限不一定够。所以这里按思路一的方式。
首先是数据库的连接：
```java
public class DBUtil {
    static final Logger logger = LoggerFactory.getLogger(DBUtil.class);
    private static final String DRIVER = "com.mysql.jdbc.Driver";
    private static final String USER = "root";
    private static final String PASSWORD = "admin";
    private static final String DATABASE = "jdbc:mysql://127.0.0.1:3306/TEST";

    public static Connection getConnection() {
        Connection connection = null;
        String URL = DATABASE +"?characterEncoding=UTF-8";
        try{
            Class.forName(DRIVER);
            connection = DriverManager.getConnection(URL,USER,PASSWORD);
        } catch (ClassNotFoundException | SQLException e) {
            logger.error(URL + "连接MySQL数据库失败！", e);
            return null;
        }
        return connection;
    }

}
```

获取ddl语句的实现：
```java
public void testDB() {
    String tableName = "example";
    Connection conn = DBUtil.getConnection();
    String sql = String.format("SHOW CREATE TABLE %s", tableName);//查询sql
    //String sql = "SHOW CREATE TABLE ?";
    PreparedStatement ps = null;
    try {
        ps = conn.prepareStatement(sql);
        //ps.setString(1, tableName);
        ResultSet resultSet = ps.executeQuery();
        while (resultSet.next()) {
            System.out.println(resultSet.getString(1));//第一个参数获取的是tableName
            System.out.println(resultSet.getString(2));//第二个参数获取的是表的ddl语句
        }
    } catch (SQLException e) {
        e.printStackTrace();
    } finally {
        if(null != ps){
            try {
                ps.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if(null != conn) {
            try {
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }

}
```
查询结果:
![TIM截图20190404085841](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93czMuc2luYWltZy5jbi9sYXJnZS9iNzI3YzY1M2x5MWcxcWJsaGZicTRqMjBtbTA0aW14OC5qcGc)

**注意**：这里用`ps.setString()` 会在tableName上加上引号，导致MySQLSyntaxErrorException。
```sql
show create table 'tableName'
```
```log
com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''example'' at line 1
```






