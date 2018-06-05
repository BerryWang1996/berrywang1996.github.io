## MySplitter 是什么？ [[English]](/index)

MySplitter 是轻量级的读/写分离，多数据源，高可用性，负载均衡数据库连接中间件。

当你需要一个程序连接2个或者更多的数据库，甚至是不同类型的数据库；数据库配置了读写分离，但是读操作有多个数据源且无法负载均衡；希望数据源不可使用时有提醒，并且可以切换到可用的数据源，那就让 MySplitter 来帮助你吧！

只需进行简单的配置，就可以让 MySplitter 管理数据源，你就可以轻松地使用多数据源了。

配置文件快速预览：
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
    <version>1.0.0</version>
</dependency>
```

Gradle:

```markdown
compile group: 'com.mysplitter', name: 'mysplitter', version: '1.0.0'
```

### 2.设置数据源

使用com.mysplitter.MySplitterDataSource获取连接、管理多个数据源。

### 3.创建配置文件

在项目的resources目录创建文件名为mysplitter.yml的配置文件，明确你需要几个数据库，是否需要读写分离、高可用和负载均衡。并参考下面的文档进行配置。

## 文档

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/BerryWang1996/berrywang1996.github.io/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
