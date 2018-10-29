## What's MySplitter? 

[[中文]](/index_zh) [[Github]](https://github.com/BerryWang1996/MySplitter) [[Apache License 2.0]](https://github.com/BerryWang1996/MySplitter/blob/master/LICENSE)

MySplitter is a lightweight read / write separation, multiple data sources, high availability, load balancing database connection middleware.

When you need a program to connect two or more databases or even different types of databases; the database is configured for read-write separation, but the read operation has multiple data sources and cannot be load-balanced; hopefully the data source is not available when there is a reminder, And can switch to available data sources, let MySplitter help you!


### Feature

* Multiple data sources (different types of databases, different types of connection pools)
* Multiple data source transactions
* Multi-data source load balancing (polling, random weight)
* Data source password encryption
* Multiple data sources are highly available (dynamic switching)
* Data source exception reminder
* Data source status monitoring
* Configure the file name that needs to be read in Spring Boot configuration file

### Development plan

* SQL filter
* XML format configuration file
* Configure all configuration in Spring Boot configuration file

### Configuration quick preview：

With simple configuration, MySplitter can manage data sources and you can easily use multiple data sources.

```yaml
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

```xml
<dependency>
    <groupId>com.mysplitter</groupId>
    <artifactId>mysplitter</artifactId>
    <version>0.9.1</version>
</dependency>
```

Gradle:

```markdown
compile group: 'com.mysplitter', name: 'mysplitter', version: '0.9.1'
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

```yaml
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

```yaml
mysplitter:
  readAndWriteParser: com.xxx.MyReadAndWriteParser
```

Java example:

```java
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

```yaml
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

```java
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

When read and write is separated, there are multiple read/write nodes, and you can configure load balancing, for example:

1. Polling load balance：

    ```yaml
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

    ```yaml
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
              weight: 1 # The weight of this node is 1，The probability of accessing this node at this time is 25%
              configuration:
                url: jdbc:mysql://localhost:3306/user
                username: root
                password: admin
                driverClassName: com.mysql.jdbc.Driver
            reader-read-slave-2:
              weight: 3 # The weight of this node is 3，The probability of accessing this node at this time is 75%
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

### 7.High available data source

If the data source is not available, it is automatically removed from the data source node. The default is 30 seconds to retry connection until the data source is available.

If you need to configure the retry interval, set the `failTimeout` parameter. Refer to the following configuration:

1. define in "common":

    ```yaml
    mysplitter:
      readAndWriteParser: com.mysplitter.demo.datasource.ReadAndWriteParser
      illAlertHandler: com.mysplitter.demo.datasource.DataSourceIllAlertHandler
      common:
        dataSourceClass: com.alibaba.druid.pool.DruidDataSource
        loadBalance:
          read:
            enabled: true
            strategy: polling
            failTimeout: 1m
          write:
            enabled: false
    ```
    
2. define in "database node":

    ```yaml
    mysplitter:
      enablePasswordEncryption: true
      readAndWriteParser: com.mysplitter.demo.datasource.ReadAndWriteParser
      illAlertHandler: com.mysplitter.demo.datasource.DataSourceIllAlertHandler
      common:
        dataSourceClass: com.alibaba.druid.pool.DruidDataSource
      databases:
        database-a:
          loadBalance:
            read:
              enabled: true
              strategy: polling
              failTimeout: 1m
            write:
              enabled: false
          readers:
            reader-read-slave-1:
              configuration:
                url: jdbc:mysql://localhost:3306/user?useSSL=false
                username: root
                password: UtDAi2eqmspIDSHqpoGQU5JC9kpfFeZPBhUxkPnWtNwsTEYFkTh/QAa5wyU7LDufruSYN+0WCUTE6F5X++5tDA==
                publicKey: MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAKVbfAja9r0HF29S/ph/T+f6UbeNxn4giAzgxweKABRsJ2sI/MNhV8x7jTsCM15xDHKM4G++QqC1Bx0tdgG/BI0CAwEAAQ==
                driverClassName: com.mysql.jdbc.Driver
                maxWait: 1000
            reader-read-slave-2:
              configuration:
                url: jdbc:mysql://localhost:3306/user?useSSL=false
                username: root
                password: Oe7fcF2TLqytAlvy37C/IWfBhNhBFXmMGceE6GRxYyjJXh3TUdmq8EvebiFb0pB1hF9aH7thnnkthFiy5n3M8Q==
                publicKey: MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAMWmL+AzrbsKwfrtP/a/aQpQplNsoySxCHUQb0aJw2t8iemRtbxtxJhXmQqPMlAZdYppyK0wB48HTArD2am3/NMCAwEAAQ==
                driverClassName: com.mysql.jdbc.Driver
                maxWait: 1000
          writers:
            writer-write-master-1:
              configuration:
                url: jdbc:mysql://localhost:3306/user?useSSL=false
                username: root
                password: MAtsEynrB5qJp6oDfmae2Z2Hx1lqPwFDNMKnwUr/P7+HvYy8ZXIm6DKI5VWfLO34Bjcdy+Jsr4+/N++Bxx0Y5w==
                publicKey: MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAIvm9Ez/X3VOLUGNfATqtyQsK5+TOR66uK6MvHdX89N1K8S3l3bNVB2BKiPZ1hDxZNZfYtbQNUUHKjDyV+eUtq8CAwEAAQ==
                driverClassName: com.mysql.jdbc.Driver
                maxWait: 1000
    ```

### 8.Configure sql filter (unsupported now)

### 9.Configure data sources monitor

You can get the status of all current data sources by calling the `getStatus` method of `com.mysplitter.MySplitterDataSource`.

### 10.Configure data source exception alerter

Create a class and implement `com.mysplitter.advise.DataSourceIllAlerterAdvise` and configure it in the configuration file:

```yaml
mysplitter:
  illAlertHandler: com.mysplitter.demo.datasource.DataSourceIllAlertHandler
```

### 11.Encrypt data sources password

1. Execute the encryption command to obtain the private key, public key, and encrypted password. Encryption directly uses the encryption algorithm of `com.alibaba.druid` and the source code.

    ```markdown
    java -cp mysplitter-0.9.1.jar com.mysplitter.util.SecurityUtil password [password...]
    ```

2. Enable password encryption `enablePasswordEncryption: true` in the configuration file. Then modify and add `password` and `publicKey` in the configuration file.

   Here is an example taken with the configuration in the quick preview of the configuration file in the document:
   
   ```yaml
   mysplitter:
     enablePasswordEncryption: true
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
               password: M1jnmaflpqw/FoZ4mSdD2e7pvQPiBWxJrEa3xvFjRb5ZCIkH5lCbfUQlbDBg58YJAhfxo8dd3KiYlygZVa2Ecw==
               publicKey: MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAKZ+PehB68CplrpZJ0qyyp5NV40HAKVlc0zf5LMD7luVd9buSvLqgmJKnT/QhRYaFDXAIaXRCKxd0TUf1ZhEafECAwEAAQ==
       database-b:
         integrates:
           database-b-integrate-node:
             dataSourceClass: com.zaxxer.hikari.HikariDataSource
             configuration:
               driverClassName: oracle.jdbc.driver.OracleDriver
               jdbcUrl: jdbc:oracle:thin:@//localhost:1521/orcl 
               username: scott
               password: rJuZ5fak54A7n4iPFfRIP2TN8xPx8nEJGfsLQzhKF6uBnDr0bfbWL8yRIeqv8h2Jbf/YuJeQnR7oUUIEcOPklg==
               publicKey: MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBALaSX/yimY5Hzqd8InW2ETrodqcVTVfiGATESTFGDKNbKfFjJFLzzFJqvwg+ZmOhXjj2tVyb5j7qz4We94zIaH8CAwEAAQ==
   ```

## FAQ

// TODO