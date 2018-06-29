## What's MySplitter? 

[[中文]](/index_zh) [[Github]](https://github.com/BerryWang1996/MySplitter) [[Apache License 2.0]](https://github.com/BerryWang1996/MySplitter/blob/master/LICENSE)

MySplitter is a lightweight read / write separation, multiple data sources, high availability, load balancing database connection middleware.

### When to use

When you need a program to connect two or more databases or even different types of databases; the database is configured for read-write separation, but the read operation has multiple data sources and cannot be load-balanced; hopefully the data source is not available when there is a reminder, And can switch to available data sources, let MySplitter help you!


### Feature

* Support multiple data sources (different types of databases, different types of connection pools)
* Support for multiple data source transactions
* Support multi-data source load balancing (polling, random weight)

### Future feature

* SQL filter
* Multiple data sources are highly available (dynamic switching)
* Data source exception reminder
* Multiple data source status monitoring
* Data source password encryption
* Spring Boot configuration file configuration

### Configuration quick preview：

With simple configuration, MySplitter can manage data sources and you can easily use multiple data sources.

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

## Start

### 1.Add dependency

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

### 2.Set data source

Use `com.mysplitter.MySplitterDataSource` to get connections and manage multiple data sources.

### 3.Create a configuration file

Create a configuration file named `mysplitter.yml` in the project's `resources` directory. Make sure how many databases you needed. Do you need read/write separation, high availability, and load balancing? Then refer to the following documents to configure.

## Configuration guide

### 1.Preparatory work

MySplitter reads the `mysplitter.yml` configuration file by default. Please make sure the file is in the project's `resources` directory. (Spring Boot configuration file configuration and custom configuration file names are not supported yet)

### 2.Common configuration

// TODO

### 3.Configure multiple data sources (read / write separation)

Example:

```markdown
mysplitter:
  databases:
    database-a: #You can customize the name of the database
      readers:
        reader-read-slave-1: # You can customize the name of the read data source node
          dataSourceClass: com.alibaba.druid.pool.DruidDataSource
          configuration: # Reader data source configuration information
            url: jdbc:mysql://localhost:3306/user
            username: root
            password: admin
            driverClassName: com.mysql.jdbc.Driver
      writers:
        writer-write-master-1: # You can customize the name of the read data source node
          dataSourceClass: com.alibaba.druid.pool.DruidDataSource
          configuration: # Writer data source configuration information
            url: jdbc:mysql://localhost:3306/user
            username: root
            password: admin
            driverClassName: com.mysql.jdbc.Driver
```

### 4.Configure your own read and write parser

When you do not configure a read/write parser, `com.mysplitter.DefaultReadAndWriteParser` will used by default. If your want to parse by yourself, customize the read and write parser and implement `com.mysplitter.advise.MySplitterReadAndWriteParserAdvise`.

Configuration file example:

```markdown
mysplitter:
  readAndWriteParser: com.xxx.MyReadAndWriteParser
```

Java example:

```markdown
public class MyReadAndWriteParser implements MySplitterReadAndWriteParserAdvise {

    /**
     * @param sql SQL statement to execute
     * @return readers:read data source  writers:write data source
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

### 5.Configure multiple databases

If it is a data source for multiple databases, you need to define the data source route and implement `com.mysplitter.advise.MySplitterDatabasesRoutingHandlerAdvise`.

Configuration file example:

```markdown
mysplitter:
  databasesRoutingHandler: com.xxx.MyDatabasesRoutingHandler
  common:
    dataSourceClass: com.zaxxer.hikari.HikariDataSource
  databases:
    database-a: # You can customize the name of the database, pay attention to the corresponding in the data source routing implementation class
      readers: # Means child nodes are read data sources
        reader-read-slave-1: # You can customize the name of the data source node
          configuration:
            url: jdbc:mysql://localhost:3306/user
            username: root
            password: admin
            driverClassName: com.mysql.jdbc.Driver
      writers: # Means child nodes are write data sources
        writer-write-master-1: # You can customize the name of the data source node
          configuration:
            url: jdbc:mysql://localhost:3306/user
            username: root
            password: admin
            driverClassName: com.mysql.jdbc.Driver
    database-b: # You can customize the name of the database, pay attention to the corresponding in the data source routing implementation class
      integrates: # Means child nodes are integrate data sources
        integrate-slave-1: # You can customize the name of the data source node
          configuration:
            jdbcUrl: jdbc:mysql://localhost:3306/dept
            username: root
            password: admin
            driverClassName: com.mysql.jdbc.Driver
```

Java example:

```markdown
public class MyDatabasesRoutingHandler implements MySplitterDatabasesRoutingHandlerAdvise {

    /**
     * @param sql SQL statement to execute
     * @return The database name defined in the configuration file
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
     * @param sql SQL statement to execute
     * @return To execute the sql statement. if you do not need to rewrite, return "sql" directly
     */
    @Override
    public String rewriteSql(String sql) {
        return sql;
    }

}
```

### 6.Configure load balance

When read and write is separated, there are multiple read/write nodes, and you can configure load balancing to achieve multi-data source load balancing of `jvm` level.

Configuration file example:

1. Polling load balance：

    ```markdown
    mysplitter:
      common:
        dataSourceClass: com.zaxxer.hikari.HikariDataSource
      databases:
        database-a:
          loadBalance:
            read:
              enabled: true
              strategy: polling
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
    
2. Random weight load balance：

    ```markdown
    mysplitter:
      common:
        dataSourceClass: com.zaxxer.hikari.HikariDataSource
      databases:
        database-a:
          loadBalance:
            read:
              enabled: true
              strategy: random # Now is the use of random weights for load balancing, weight needs to be defined at each node, the default is 1
            write:
              enabled: false
          readers:
            reader-read-slave-1:
              weight: 1 # The weight of this node is 1
              configuration:
                url: jdbc:mysql://localhost:3306/user
                username: root
                password: admin
                driverClassName: com.mysql.jdbc.Driver
            reader-read-slave-2:
              weight: 2 # The weight of this node is 3，The probability of accessing the first node at this time is 20%, and the probability of the second node is 80%
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

### 7.Configure high available

// TODO

### 8.Configure sql filter (unsupported now)

### 9.Configure data sources monitor (unsupported now)

### 10.Encrypt data sources password (unsupported now)

## FAQ

// TODO