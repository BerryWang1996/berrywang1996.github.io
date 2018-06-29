## MySplitter 是什么？ 

[[English]](/index) [[Github]](https://github.com/BerryWang1996/MySplitter) [[Apache License 2.0]](https://github.com/BerryWang1996/MySplitter/blob/master/LICENSE)

MySplitter 是轻量级的读/写分离，多数据源，高可用性，负载均衡数据库连接中间件。

### 使用场景

当你需要一个程序连接2个或者更多的数据库，甚至是不同类型的数据库；数据库配置了读写分离，但是读操作有多个数据源且无法负载均衡；希望数据源不可使用时有提醒，并且可以切换到可用的数据源，那就让 MySplitter 来帮助你吧！

### 特性

* 当前特性：

  * 支持多数据源（不同类型数据库、不同类型连接池）
  * 支持多数据源事务
  * 支持多数据源负载均衡（轮询、随机权重）

* 将要支持的特性：

  * SQL过滤器
  * 多数据源高可用（动态切换）
  * 数据源异常提醒
  * 多数据源状态监控
  * 数据源密码加密
  * Spring Boot配置文件配置

### 配置文件快速预览：

只需进行简单的配置，就可以让 MySplitter 管理数据源，你就可以轻松地使用多数据源了。

```markdown
mysplitter:
  databasesRoutingHandler: com.xxx.your.databasesRoutingHandler
  databases:
    database-a:
      integrates:
        database-a-integrate-node:
          dataSourceClass: com.alibaba.druid.pool.DruidDataSource
          configuration:
            driverClassName: com.mysql.jdbc.Driver
            url: jdbc:mysql://localhost:3306/test
            username: root
            password: admin
    database-b:
      integrates:
        database-b-integrate-node:
          dataSourceClass: com.zaxxer.hikari.HikariDataSource
          configuration:
            driverClassName: oracle.jdbc.driver.OracleDriver
            jdbcUrl: jdbc:oracle:thin:@//localhost:1521/orcl 
            username: scott
            password: tiger
```

## 开始使用

### 1.导入依赖

Maven:

```markdown
<dependency>
    <groupId>com.mysplitter</groupId>
    <artifactId>mysplitter</artifactId>
    <version>0.9.0</version>
</dependency>
```

Gradle:

```markdown
compile group: 'com.mysplitter', name: 'mysplitter', version: '0.9.0'
```

### 2.设置数据源

使用 `com.mysplitter.MySplitterDataSource` 获取连接、管理多个数据源。

### 3.创建配置文件

在项目的 `resources` 目录创建文件名为 `mysplitter.yml` 的配置文件，明确你需要几个数据库，是否需要读写分离、高可用和负载均衡。并参考下面的文档进行配置。

## 配置向导

### 1.准备工作

MySplitter 默认读取 `mysplitter.yml` 配置文件。请确该文件在项目的 `resources` 目录下。（暂不支持Spring Boot配置文件配置以及自定义配置文件名称）

### 2.统一配置

待完善


### 3.配置多数据源/读写分离

下面将演示一个读写分离的多数据源配置：

```markdown
mysplitter:
  databases:
    database-a: #可以自定义数据库的名称
      readers:
        reader-read-slave-1: #可以自定义读数据源节点的名称
          dataSourceClass: com.alibaba.druid.pool.DruidDataSource
          configuration: # 读数据源的配置信息
            url: jdbc:mysql://localhost:3306/user
            username: root
            password: admin
            driverClassName: com.mysql.jdbc.Driver
      writers:
        writer-write-master-1: #可以自定义写数据源节点的名称
          dataSourceClass: com.alibaba.druid.pool.DruidDataSource
          configuration: # 写数据源的配置信息
            url: jdbc:mysql://localhost:3306/user
            username: root
            password: admin
            driverClassName: com.mysql.jdbc.Driver
```

### 4.配置自定义读写解析器

当不配置读写解析器时，将使用自带的读写解析器 `com.mysplitter.DefaultReadAndWriteParser` 。如果自带的解析器无法满足你的需求，请自定义读写解析器并实现 `com.mysplitter.advise.MySplitterReadAndWriteParserAdvise`。

配置文件示例：

```markdown
mysplitter:
  readAndWriteParser: com.xxx.MyReadAndWriteParser
```

Java代码示例：

```markdown
public class MyReadAndWriteParser implements MySplitterReadAndWriteParserAdvise {

    /**
     * @param sql 要执行的sql语句
     * @return readers:当前数据源是读数据源  writers:当前数据源是写数据源
     */
    @Override
    public String parseOperation(String sql) {
        if (sql.startsWith("SELECT") || sql.startsWith("select")) {
            return "readers";
        }
        return "writers";
    }

}
```

### 5.配置多数据库

如果是多个数据库的数据源，需要先定义数据源路由并实现 `com.mysplitter.advise.MySplitterDatabasesRoutingHandlerAdvise`。

配置文件示例：

```markdown
mysplitter:
  databasesRoutingHandler: com.xxx.MyDatabasesRoutingHandler
  common:
    dataSourceClass: com.zaxxer.hikari.HikariDataSource
  databases:
    database-a: #可以自定义数据库的名称，注意在数据源路由实现类中进行对应
      readers: #代表子节点都是读数据源
        reader-read-slave-1: #可以自定义数据源节点名称
          configuration:
            url: jdbc:mysql://localhost:3306/user
            username: root
            password: admin
            driverClassName: com.mysql.jdbc.Driver
      writers: #代表子节点都是写数据源
        writer-write-master-1: #可以自定义数据源节点名称
          configuration: # dataSource configuration
            url: jdbc:mysql://localhost:3306/user
            username: root
            password: admin
            driverClassName: com.mysql.jdbc.Driver
    database-b: #可以自定义数据库的名称，注意在数据源路由实现类中进行对应
      integrates: #代表子节点是不进行读写分离的数据源
        integrate-slave-1: #可以自定义数据源节点名称
          configuration:
            jdbcUrl: jdbc:mysql://localhost:3306/dept
            username: root
            password: admin
            driverClassName: com.mysql.jdbc.Driver
```

Java代码示例：

```markdown
public class MyDatabasesRoutingHandler implements MySplitterDatabasesRoutingHandlerAdvise {

    /**
     * @param sql 要执行的sql语句
     * @return 配置文件中定义的数据库名称
     */
    @Override
    public String routerHandler(String sql) {
        if (sql.contains("user")) {
            return "database-a";
        } else {
            return "database-b";
        }
    }
    
    /**
     * @param sql 要执行的sql语句
     * @return 要执行的sql语句，如果不需要重写，直接返回sql即可
     */
    @Override
    public String rewriteSql(String sql) {
        return sql;
    }

}
```

### 6.配置负载均衡

当读写分离，有多个读/写节点，可以通过配置负载均衡达到 `jvm` 级别的多数据源负载均衡。

配置文件示例：

1. 负载均衡轮询算法：

    ```markdown
    mysplitter:
      common:
        dataSourceClass: com.zaxxer.hikari.HikariDataSource
      databases:
        database-a:
          loadBalance:
            read:
              enabled: true
              strategy: polling #现在是使用轮询方式进行负载均衡
            write:
              enabled: false
          readers:
            reader-read-slave-1:
              configuration:
                url: jdbc:mysql://localhost:3306/user
                username: root
                password: admin
                driverClassName: com.mysql.jdbc.Driver
            reader-read-slave-2:
              configuration:
                url: jdbc:mysql://localhost:3307/user
                username: root1
                password: admin2
                driverClassName: com.mysql.jdbc.Driver
          writers:
            writer-write-master-1:
              configuration: # dataSource configuration
                url: jdbc:mysql://localhost:3306/user
                username: root
                password: admin
                driverClassName: com.mysql.jdbc.Driver
    ```

2. 负载均衡随机权重算法：

    ```markdown
    mysplitter:
      common:
        dataSourceClass: com.zaxxer.hikari.HikariDataSource
      databases:
        database-a:
          loadBalance:
            read:
              enabled: true
              strategy: random #现在是使用随机权重方式进行负载均衡，需要在每个节点定义weight，默认是1
            write:
              enabled: false
          readers:
            reader-read-slave-1:
              weight: 1 #此节点的权重是1
              configuration:
                url: jdbc:mysql://localhost:3306/user
                username: root
                password: admin
                driverClassName: com.mysql.jdbc.Driver
            reader-read-slave-2:
              weight: 2 #此节点的权重是3，此时访问第一个节点的概率为20%，第二个节点的概率为80%
              configuration:
                url: jdbc:mysql://localhost:3307/user
                username: root1
                password: admin2
                driverClassName: com.mysql.jdbc.Driver
          writers:
            writer-write-master-1:
              configuration: # dataSource configuration
                url: jdbc:mysql://localhost:3306/user
                username: root
                password: admin
                driverClassName: com.mysql.jdbc.Driver
    ```

### 7.配置高可用

待完善

### 8.配置过滤器（暂不支持）

### 9.多数据源状态监控（暂不支持）

### 10.数据源密码加密（暂不支持）

## 已知问题

待完善