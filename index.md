## What's MySplitter? [[中文]](/index_zh)

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
<dependency>
    <groupId>com.mysplitter</groupId>
    <artifactId>mysplitter</artifactId>
    <version>1.0.0</version>
</dependency>
```

Gradle:

```markdown
compile group: 'com.mysplitter', name: 'mysplitter', version: '1.0.0'
```

### 2.Set data source

Use `com.mysplitter.MySplitterDataSource` to get connections and manage multiple data sources.

### 3.Create a configuration file

Create a configuration file named mysplitter.yml in the project's resources directory. Make sure how many databases you needed. Do you need read/write separation, high availability, and load balancing? Then refer to the following documents to configure.

## Document

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/BerryWang1996/berrywang1996.github.io/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
