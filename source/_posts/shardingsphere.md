---
title: shardingsphere
date: 2022-03-19 21:35:45
tags: sharding, jdbc
---

# JDBC API及其上层生态

JDBC API是JDK提供的, Java程序访问数据库的统一接口. 我们实际开发中使用的数据库连接池、ORM/持久层框架、分库分表等也无一例外是基于JDBC API的上层拓展与实现. 本文我们从底层出发, 逐个了解上层的封装细节.

## JDBC API基本介绍

### Oracle官方定义: 

*The Java Database Connectivity (JDBC) API provides universal data access from the Java programming language. Using the JDBC API, you can access virtually any data source, from relational databases to spreadsheets and flat files.* 

简单来说, JDBC是JDK提供的Java程序访问数据库资源的上层统一接口.  当Java程序要访问数据库时，Java代码可直接调用 JDBC接口，而JDBC接口底层通过Java SPI机制找到数据库厂商的JDBC驱动来实现对数据库的读写操作。

<img src="./jdbc-level.png" alt="image-20220331212519941" style="zoom:47%;" />

### JDBC API基础使用方法:

JDBC API包含两个包: java.sql 和 javax.sql , 其核心类和基本用法如下:

<img src="./jdbc-package.png" alt="jdbc-package" style="zoom:43%;" />

+ 通过DriverManager.getConnection()获取数据库驱动连接

```java
public class DriverManager {

    // List of registered JDBC drivers
    private final static CopyOnWriteArrayList<DriverInfo> registeredDrivers = new CopyOnWriteArrayList<>();
    
    static {
        // 通过java SPI 注册声明的Driver实现到 registeredDrivers
        loadInitialDrivers();
        println("JDBC DriverManager initialized");
    }
    // ... 略
    public static synchronized void registerDriver(java.sql.Driver driver,
            DriverAction da)
        throws SQLException {

        /* Register the driver if it has not already been added to our list */
        if(driver != null) {
            registeredDrivers.addIfAbsent(new DriverInfo(driver, da));
        } else {
            // This is for compatibility with the original DriverManager
            throw new NullPointerException();
        }

        println("registerDriver: " + driver);
    }
    // ... 略
    // 获取连接时最终遍历registeredDriver并connect
    public static Connection getConnection(String url,
        java.util.Properties info) throws SQLException {

        return (getConnection(url, info, Reflection.getCallerClass()));
    }
}
```

+ 查询数据

```java
String JDBC_URL = "jdbc:mysql://localhost:3306/test";
String JDBC_USER = "root";
String JDBC_PASSWORD = "password";

try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
    try (PreparedStatement ps = conn.prepareStatement("SELECT id, grade, name, gender FROM students WHERE gender=? AND grade=?")) {
        ps.setObject(1, "M"); // 注意：索引从1开始
        ps.setObject(2, 3);
        try (ResultSet rs = ps.executeQuery()) {
            while (rs.next()) {
                long id = rs.getLong("id");
                long grade = rs.getLong("grade");
                String name = rs.getString("name");
                String gender = rs.getString("gender");
            }
        }
    }
}
```

+ 插入、更新、删除数据

```java
// insert and get genrated key
try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
    try (PreparedStatement ps = conn.prepareStatement(
            "INSERT INTO students (grade, name, gender) VALUES (?,?,?)",
            Statement.RETURN_GENERATED_KEYS)) {
        ps.setObject(1, 1); // grade
        ps.setObject(2, "Bob"); // name
        ps.setObject(3, "M"); // gender
        int n = ps.executeUpdate(); // 1
        try (ResultSet rs = ps.getGeneratedKeys()) {
            if (rs.next()) {
                long id = rs.getLong(1); // 注意：索引从1开始
            }
        }
    }
}

// update
try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
    try (PreparedStatement ps = conn.prepareStatement("UPDATE students SET name=? WHERE id=?")) {
        ps.setObject(1, "Bob"); // 注意：索引从1开始
        ps.setObject(2, 999);
        int n = ps.executeUpdate(); // 返回更新的行数
    }
}

// delete
try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
    try (PreparedStatement ps = conn.prepareStatement("DELETE FROM students WHERE id=?")) {
        ps.setObject(1, 999); // 注意：索引从1开始
        int n = ps.executeUpdate(); // 删除的行数
    }
}
```

+ 事务操作,  默认每条sql语句都会在一个独立的事务中并被自动提交,  控制事务就是取消自动提交、执行多条sql并手动提交

```java
Connection conn = openConnection();
// 设定隔离级别为READ COMMITTED, 不设置使用mysql默认隔离级别
conn.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);
try {
    // 关闭自动提交:
    conn.setAutoCommit(false);
    // 执行多条SQL语句:
    insert(); 
    update();
    delete();
    // 提交事务:
    conn.commit();
} catch (SQLException e) {
    // 回滚事务:
    conn.rollback();
} finally {
    conn.setAutoCommit(true);
    conn.close();
}
```

### DataSource接口:

官方javadoc:  

 **A factory for connections** to the physical data source that this DataSource object represents. An **alternative to the DriverManager** facility, a DataSource object is **the preferred means of getting a connection**. (总结)

<img src="/Users/talentxiet/Library/Application Support/typora-user-images/image-20220401190952211.png" alt="image-20220401190952211" style="zoom:50%;" />

DataSource一般有三种实现:

1. Basic 实现:  内部就是调用DriverManager中的数据库驱动获取连接. 例如Spring JDBC的DriverManagerDataSource.
2. Connection pooling实现: 用池化技术复用连接. 例如Hikari的HikariDataSource.
3. Distributed 实现: 可以产出能执行分布式SQL的Connection. 例如Sharding JDBC的ShardingDataSource.

<img src="./datasource-impl.png" alt="image-20220401191629549" style="zoom:50%;" />

## 生态一: 数据库连接池

每次读写访问都通过数据库驱动新建和销毁tcp连接的开销是昂贵的,  我们通常用连接池来初始化并复用已经创建好的连接. 连接池的开源实现非常多, 比如hikari, druid, tomcat-jdbc pool等. 不同实现模型和细节各有不同, 其优势也各有不同, 我们这里不一一展开了, 大家可以自行查看其benchmark相关文章.

<img src="./datasource-pool.png" alt="image-20220401193526994" style="zoom: 45%;" />

连接池的大体模型都如上图所示: 数据库驱动连接的借用、归还、以及创建和销毁. 我们这里列出一些较通用的连接池配置点, 在实际使用过程中大家注意根据实际需要正确的配置即可. 下边示例是hikari的参数配置项.

```sh
connectionTimeout: 请求连接的超时时间
idleTimeout: 空闲连接回收时间
maxLifetime: 连接最大存活时间
connectionTestQuery: borrow到连接后测试连接有效的query语句
minimumIdle: 连接池中最小的闲置连接数, 总数不超过maximux的前提下hikari尽可能保证此参数数量
maximumPoolSize: 连接池中最大的连接数
```

此类数据库连接池通常也会产出详细的监控指标信息, 收集这类指标可以很好的辅助我们排障和了解线上资源使用情况, 常见指标(hikari为例)如下: 

```
idleConnections: 当前闲置连接数
activeConnections: 当前活跃连接数
totalConnections: activeConnection + idleConnections
pendingThreads: 当前等待连接的线程数
maxConnections: 历史最大连接数
minConnections: 历史最小连接数
usageTime: 本次连接使用时间
acquireTime: 本次连接请求时间
```

## 生态二: spring-jdbc

spring-jdbc是Spring framework中的模块之一, 提供了Spring对原生JDBC API的实现和上层封装,其主要包含以下四个包:

+ **core**: 提供Spring封装后的核心功能接口及实现, 如 *JdbcOperations*, *JdbcTemplate*, *NamedParameterJdbcTemplate*
+ **datasource**: 提供DataSource的实现及上层功能封装, 如 *DriverManagerDataSource*和 *DataSourceTransactionManager*
+ **object**: 提供面向对象的SQL Query和Result返回结果封装
+ **support**: 提供上面包的一些通用support类

spring-jdbc (包括其依赖的spring-tx) 可以帮我们简化一些开发流程: 

+ 原始: 获取数据库连接、处理事务autocommit、声明SQL、预编译并执行、映射ResultSet结果集到期望对象、处理异常和事务、释放连接资源等一系列操作.
+  spring-jdbc:  配置DataSource和JdbcTemplate, 声明SQL和结果映射方式、调用JdbcTemplate接口执行, 获取对象结果.

简单使用示例如下:

+ 配置DataSource和JdbcTemplate

```java
@Configuration
@PropertySource("jdbc.properties")
public class AppConfig {

    @Value("${jdbc.url}")
    String jdbcUrl;
    @Value("${jdbc.username}")
    String jdbcUsername;
    @Value("${jdbc.password}")
    String jdbcPassword;

    @Bean
    DataSource createDataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(jdbcUrl);
        config.setUsername(jdbcUsername);
        config.setPassword(jdbcPassword);
        config.addDataSourceProperty("autoCommit", "true");
        config.addDataSourceProperty("connectionTimeout", "5");
        config.addDataSourceProperty("idleTimeout", "60");
        return new HikariDataSource(config);
    }

    @Bean
    JdbcTemplate createJdbcTemplate(@Autowired DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```

+ 使用JdbcTemplate最为Dao层返回结果

```java
public List<User> getUsers(int pageIndex, int pageSize) {
    int limit = pageSize;
    int offset = limit * (pageIndex - 1);
    return jdbcTemplate.query("SELECT * FROM users LIMIT ? OFFSET ?", new Object[] { limit, offset },
            new BeanPropertyRowMapper<>(User.class));
}
```

JdbcTemplate只是jdbc api的一些简单封装, 在实际开发中, 大多还是使用MyBatis、Hibernate此类的ORM框架, 更好的对底层操作进行封闭.  

但spring-jdbc依赖的spring-tx包中对事务进行了抽象, 给我们提供了简单的 @Transactional 注解来为代码逻辑进行AOP事务增强.  其中增强逻辑的实现全部依靠 DataSourceTransactionManager 类来完成, 包括当前事务上下文判断、从DataSource中获取Connection、设置隔离级别并关闭auto commit、设置线程共享的ThreadLocal存储dataSource获取到的Connection、以及最终的Connection commit or rollback.

理解事务本质 以及 Spring对事务的封装:

从原生JDBC API我们知道, 一个事务和一个Connection是强绑定的. 默认一个Connection的autoCommit会设置成true, 也就是每条执行的SQL默认都在一个事务里,并在执行完成后自动提交了. 若我们想让代码实现多个SQL在一个事务里执行, 本质上就是让这些SQL在一个autoCommit设置为false的Connection中执行, 并最终手动commit. DataSourceTransactionManager本质上也就是做了这个工作, 在@Transactional 注解的方法开始前, 获取到新的Connection, 设置隔离级别与autoCommit, 执行若干代码逻辑, 最终提交或回滚.  当遇到传播级别为REQUIRES_NEW的增强方法或想另起一个新事务执行一些SQL时, 就需再从DataSource获取一个新的Connection来进行.

``` java
protected Object invokeWithinTransaction(Method method, Class<?> targetClass, final InvocationCallback invocation)
			throws Throwable {

		final TransactionAttribute txAttr = getTransactionAttributeSource().getTransactionAttribute(method, targetClass);
    // 获取DataSourceTransactionManager
		final PlatformTransactionManager tm = determineTransactionManager(txAttr);
		final String joinpointIdentification = methodIdentification(method, targetClass, txAttr);
		
      // TransactionManager判断是否需要创建新事务, 即获取Connection
			TransactionInfo txInfo = createTransactionIfNecessary(tm, txAttr, joinpointIdentification);
			Object retVal = null;
			try {
				// AOP 增强的原始方法
				retVal = invocation.proceedWithInvocation();
			}
			catch (Throwable ex) {
				// 遇到异常选择性回滚事务
				completeTransactionAfterThrowing(txInfo, ex);
				throw ex;
			}
			finally {
				cleanupTransactionInfo(txInfo);
			}
      // 正常执行提交事务
			commitTransactionAfterReturning(txInfo);
			return retVal;
}
```

## 生态三: ORM/持久层框架-MyBatis

> 本节我们挑重点讲解, 详细源码分析可参考此文章:  [mybatis design concept](https://learn.lianglianglee.com/%E4%B8%93%E6%A0%8F/%E6%B7%B1%E5%85%A5%E5%89%96%E6%9E%90%20MyBatis%20%E6%A0%B8%E5%BF%83%E5%8E%9F%E7%90%86-%E5%AE%8C/03%20%20MyBatis%20%E6%BA%90%E7%A0%81%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%8F%8A%E6%95%B4%E4%BD%93%E6%9E%B6%E6%9E%84%E8%A7%A3%E6%9E%90.md)

MyBatis官方Introduction: 

MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。**MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作**。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、Map、集合等接口和 Java POJO为数据库中的记录。

MyBatis 分为三层架构，分别是**基础支撑层、核心处理层**和**接口层**，如下图所示：

<img src="./mybatis-arch.png" alt="image-20220405125449569" style="zoom:30%;" />

+ 基础支撑层是整个 MyBatis 框架的地基，为整个 MyBatis 框架提供了底层基础的功能，其中每个模块都提供了一个内聚的、单一的能力;	
+ 核心处理层是 MyBatis 核心实现所在，其中涉及 MyBatis 的初始化以及执行一条 SQL 语句的全流程。
+ 接口层是 MyBatis 暴露给调用的接口集合，这些接口都是使用 MyBatis 时最常用的一些接口，例如，SqlSession 接口、SqlSessionFactory 接口等。

MyBatis执行过程以及对JDBC API的封装:

<img src="./mybatis-execute.png" alt="image-20220405132244101" style="zoom:35%;" />

MyBatis与Spring的集成:

我们在Spring框架内使用mybatis的时候, 要引入mybatis和mybatis-spring两个依赖, 前者是mybatis的所有核心逻辑的代码, 而后者负责整合spring框架能力, 其又引入了spring-jdbc以及spring-tx. 

<img src="./mybatis-package.png" alt="image-20220419141734725" style="zoom:50%;" />

一个 Transaction 的例子, 多个Dao在一个事务中执行的过程:

```java
    @Transactional
    public void createOrder(Long userId, Long productId, Integer count) {
        // 1. 创建订单
        orderDao.insertOrder(Order.builder()
                .userId(userId)
                .productId(productId)
                .count(count)
                .build());

        // 2. 减少商品库存
        productDao.minusCount(productId, count);

        // 3. 增加用户积分
        userDao.addPoint(userId, count);
    }
```

MyBatis框架的SQL执行是交给Executor负责的, Executor每次新建的时候会传入一个Transaction对象, 在与Spring整合的过程中, 会将MyBatis Configuration中的 TransactionFactory指定为 SpringManagedTransactionFactory, 其最终目标呢, 就是创建出 SpringManagedTransaction给 Executor使用. 

SpringManagedTransaction有什么特别的呢? 

``` java
// SpringManagedTransaction 中的获取连接
private void openConnection() throws SQLException {
    // 通过Spring-jdbc暴露的 DataSourceUtils 获取Connection, 该Util会从当前线程上下文中判断是否存在事务, 并获取由之前Spring事务切面存入的 Connection
    this.connection = DataSourceUtils.getConnection(this.dataSource);
    this.autoCommit = this.connection.getAutoCommit();
    this.isConnectionTransactional = DataSourceUtils.isConnectionTransactional(this.connection, this.dataSource);
    // ...
}

// DataSourceUtils.getConnection方法内部逻辑
public static Connection getConnection(DataSource dataSource) {
    // 从全局的TreadLocal中获取当前线程是否已有事务Connection
    ConnectionHolder conHolder = TransactionSynchronizationManager.getResource(dataSource);
		// 有的话直接使用同一个Connection
    if (conHolder != null && (conHolder.hasConnection() || conHolder.isSynchronizedWithTransaction())) {
			conHolder.requested();
			if (!conHolder.hasConnection()) {
				logger.debug("Fetching resumed JDBC Connection from DataSource");
				conHolder.setConnection(fetchConnection(dataSource));
			}
			return conHolder.getConnection();
		}
    // 线程上下文中没有事务连接, 新获取Connection执行后续SQL
    Connection con = fetchConnection(dataSource);
    // ...
    return con;
}
```

## 生态四: 分库分表

### Sharding-JDBC (ShardingSphere-JDBC)

官方介绍:

定位为轻量级 Java 框架，在 Java 的 JDBC 层提供的额外服务。 它使用客户端直连数据库，以 jar 包形式提供服务，无需额外部署和依赖，可理解为增强版的 JDBC 驱动，完全兼容 JDBC 和各种 ORM 框架。

- 适用于任何基于 JDBC 的 ORM 框架，如：JPA, Hibernate, Mybatis, Spring JDBC Template 或直接使用 JDBC；
- 支持任何第三方的数据库连接池，如：DBCP, C3P0, BoneCP, HikariCP 等；
- 支持任意实现 JDBC 规范的数据库，目前支持 MySQL，PostgreSQL，Oracle，SQLServer 以及任何可使用 JDBC 访问的数据库。

![image-20220319220206599](./sharding-jdbc.png)

故名思义, Sharding-JDBC, 叫这个名字, 那它一定是基于JDBC API进行的能力拓展. 我们可以看下其core包下结构便一目了然了:

<img src="./sharding-core-package.png" alt="image-20220418225313643" style="zoom:50%;" />

为什么Sharding-JDBC能支持所有的ORM框架呢?  也正因为它直接在JDBC层进行的封装, 以 MyBatis 和 Sharding JDBC为例: 

<img src="./mybatis-sharding-jdbc.png" alt="image-20220419152546734" style="zoom:50%;" />

Sharding-JDBC 为 JDBC DataSource接口实现了 ShardingDataSource, 其内部管理了多个数据源, 而上层使用时对多数据源完全无感. 在PrepareStatement阶段, Sharding-JDBC会根据要执行的SQL以及配置的分库分表Rule,  解析并路由出若干条执行单元, 并分别从对应连接池中获取连接, 并执行改写后的SQL. 

<img src="./sharding-jdbc-sequence.png" alt="image-20220419153403836" style="zoom: 25%;" />

分库分表配置示例:

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
  ~ Licensed to the Apache Software Foundation (ASF) under one or more
  ~ contributor license agreements.  See the NOTICE file distributed with
  ~ this work for additional information regarding copyright ownership.
  ~ The ASF licenses this file to You under the Apache License, Version 2.0
  ~ (the "License"); you may not use this file except in compliance with
  ~ the License.  You may obtain a copy of the License at
  ~
  ~     http://www.apache.org/licenses/LICENSE-2.0
  ~
  ~ Unless required by applicable law or agreed to in writing, software
  ~ distributed under the License is distributed on an "AS IS" BASIS,
  ~ WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  ~ See the License for the specific language governing permissions and
  ~ limitations under the License.
  -->

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:sharding="http://shardingsphere.apache.org/schema/shardingsphere/sharding"
       xmlns:bean="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                        http://www.springframework.org/schema/beans/spring-beans.xsd 
                        http://www.springframework.org/schema/tx 
                        http://www.springframework.org/schema/tx/spring-tx.xsd
                        http://www.springframework.org/schema/context 
                        http://www.springframework.org/schema/context/spring-context.xsd
                        http://shardingsphere.apache.org/schema/shardingsphere/sharding
                        http://shardingsphere.apache.org/schema/shardingsphere/sharding/sharding.xsd http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">
    <context:component-scan base-package="org.apache.shardingsphere.example.core.mybatis" />
    
    <bean id="demo_ds_0" class="com.zaxxer.hikari.HikariDataSource" destroy-method="close">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/demo_ds_0?useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
        <property name="username" value="root"/>
        <property name="password" value=""/>
    </bean>
    
    <bean id="demo_ds_1" class="com.zaxxer.hikari.HikariDataSource" destroy-method="close">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/demo_ds_1?useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
        <property name="username" value="root"/>
        <property name="password" value=""/>
    </bean>
    
    <sharding:inline-strategy id="databaseStrategy" sharding-column="user_id" algorithm-expression="demo_ds_${user_id % 2}" />
    <sharding:inline-strategy id="orderTableStrategy" sharding-column="order_id" algorithm-expression="t_order_${order_id % 2}" />
    <sharding:inline-strategy id="orderItemTableStrategy" sharding-column="order_id" algorithm-expression="t_order_item_${order_id % 2}" />
    
    <bean:properties id="properties">
        <prop key="worker.id">123</prop>
    </bean:properties>
    
    <sharding:key-generator id="orderKeyGenerator" type="SNOWFLAKE" column="order_id" props-ref="properties" />
    <sharding:key-generator id="itemKeyGenerator" type="SNOWFLAKE" column="order_item_id" props-ref="properties" />
    
    <sharding:data-source id="shardingDataSource">
        <sharding:sharding-rule data-source-names="demo_ds_0, demo_ds_1">
            <sharding:table-rules>
                <sharding:table-rule logic-table="t_order" actual-data-nodes="demo_ds_${0..1}.t_order_${0..1}" database-strategy-ref="databaseStrategy" table-strategy-ref="orderTableStrategy" key-generator-ref="orderKeyGenerator" />
                <sharding:table-rule logic-table="t_order_item" actual-data-nodes="demo_ds_${0..1}.t_order_item_${0..1}" database-strategy-ref="databaseStrategy" table-strategy-ref="orderItemTableStrategy" key-generator-ref="itemKeyGenerator" />
            </sharding:table-rules>
            <sharding:binding-table-rules>
                <sharding:binding-table-rule logic-tables="t_order,t_order_item"/>
            </sharding:binding-table-rules>
            <sharding:broadcast-table-rules>
                <sharding:broadcast-table-rule table="t_address"/>
            </sharding:broadcast-table-rules>
        </sharding:sharding-rule>
        <sharding:props>
            <prop key="sql.show">false</prop>
        </sharding:props>
    </sharding:data-source>
    
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="shardingDataSource" />
    </bean>
    <tx:annotation-driven />
    
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="shardingDataSource"/>
        <property name="mapperLocations" value="classpath*:META-INF/mappers/*.xml"/>
    </bean>
    
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="org.apache.shardingsphere.example.core.mybatis.repository"/>
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    </bean>
</beans>

```

思考: 跨数据源事务一致性问题?

## 生态五:  分布式事务... 待完成
