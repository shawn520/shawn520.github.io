---
title: spring Boot继承MyBatis 如何实现多数据源动态加载？
categories:
- 好好学习
tags:
  - JDBC
date: 2019-02-11 21:19:46
---

在公司的项目中遇到需要根据获取的数据库信息来动态连接数据库，执行SQL语句。通过JDBC很好实现，但是就不能使用MyBatisGenerator生成的SQL代码，需要手动写SQL了。网上能查到的教程都是Mybatis静态连接，或者多个数据源之间来回切换，不符合动态连接数据库的需求。那么如何在MyBatis里面如何实现呢？

<!-- more -->



## 1.首先再resources目录下新建一个jdbc配置文件.

```xml
huiyuan.port=com.mysql.jdbc.Driver
usercloud.port=jdbc:mysql://127.0.0.1:3306/***?characterEncoding=UTF-8
jdbc.username=root
jdbc.password=admin
```

## 2.配置mybatis-config.xml

```xml
<configuration>

    <properties resource="mybatis/jdbc.properties"/>
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true" />
    </settings>

    <typeAliases>
        <package name="com.***.pojo"/>
    </typeAliases>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="UNPOOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="com/***/mapper/DataMediaSourceMapper.xml"/>
    </mappers>
</configuration>
```

## 3.配置工具类

关键在第三步，SqlSessionFactoryBuilder根据动态获取的数据库配置信息构建出 SqlSessionFactory 的实例，然后根据SqlSessionFactory 获取SqlSession 的实例  ，然后根据SqlSession 执行已映射的 SQL 语句 。

```java
public class DataSourceConfig {

    static SqlSession session;

    public static SqlSession getSession() {
        return session;
    }

    public static void setSession(SqlSession session) {
        DataSourceConfig.session = session;
    }

    /**
     * 动态获取数据库连接SqlSession
     * @param database
     * @param username
     * @param password
     * @return
     */
    public static SqlSession getSqlSession(String database, String username, String password) {
        Properties properties = new Properties();
        properties.setProperty("jdbc.driver", "com.mysql.jdbc.Driver");
        properties.setProperty("jdbc.url",database +"?characterEncoding=UTF-8");
        properties.setProperty("jdbc.username", username);
        properties.setProperty("jdbc.password", password);
        String resource = "mybatis/mybatis-config.xml";
        Reader reader = null;
        try {
            reader = Resources.getResourceAsReader(resource);
        } catch (IOException e) {
            e.printStackTrace();
        }
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = builder.build(reader,properties);
        session = factory.openSession();
        setSession(session);
        return session;
    }
    
}
```

## 参考资料

<http://www.mybatis.org/mybatis-3/zh/getting-started.html> 

