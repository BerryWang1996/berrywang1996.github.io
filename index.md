## What's MySplitter? 

[[中文]](/index_zh) [[Github]](https://github.com/BerryWang1996/MySplitter) [[Apache License 2.0]](https://github.com/BerryWang1996/MySplitter/blob/master/LICENSE)

MySplitter is a lightweight read / write separation, multiple data sources, high availability, load balancing database connection middleware.

When you need a program to connect two or more databases or even different types of databases; the database is configured for read-write separation, but the read operation has multiple data sources and cannot be load-balanced; hopefully the data source is not available when there is a reminder, And can switch to available data sources, let MySplitter help you!

With simple configuration, MySplitter can manage data sources and you can easily use multiple data sources.

Configuration quick preview：
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
<!-- It is developing now -->
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

MySplitter 默认读取 `mysplitter.yml` 配置文件。请确该文件在项目的 `resources` 目录下。（暂不支持Spring Boot配置文件配置以及自定义配置文件名称）

### 2.Configure multiple data sources

### 3.Configure read / write separation

### 4.Configure your own read and write parser

### 5.Configure multiple databases

### 6.Configure multiple databases routing handler

### 7.Configure load balance

### 8.Configure high available

### 9.Configure sql filter (unsupported now)

### 10.Configure data sources monitor (unsupported now)

### 11.Encrypt data sources password (unsupported now)

## FAQ