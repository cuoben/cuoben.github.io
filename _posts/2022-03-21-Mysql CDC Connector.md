---
layout: post
title: "Mysql CDC Connector"
description: "Mysql CDC Connector"
categories: [flink]
tags: [connector]
redirect_from:
- /2022/03/21/
---
# Mysql CDC Connector

MySQL CDC 连接器允许从 MySQL 数据库读取快照数据和增量数据。底层通过[Debezium](https://debezium.io/documentation/reference/1.4/connectors/index.html)监控读取binlog日志。

## maven依赖

```XML
<dependency>

  <groupId>com.ververica</groupId>

  <artifactId>flink-connector-mysql-cdc</artifactId>

  <version>2.0.2</version>

</dependency>
```

## SQL 客户端 JAR

下载[flink-sql-connector-mysql-cdc-2.0.2.jar](https://repo1.maven.org/maven2/com/ververica/flink-sql-connector-mysql-cdc/2.0.2/flink-sql-connector-mysql-cdc-2.0.2.jar)放到<FLINK_HOME>/lib/

## 配置mysql

创建一个对所有数据库具有适当权限的 MySQL 用户以满足用于Debezium Mysql连接器。

- 授予用户所需的权限：

`mysql> GRANT SELECT, SHOW DATABASES, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'user' IDENTIFIED BY 'password';`

**注意**：`scan.incremental.snapshot.enabled`启用后不再需要RELOAD（使连接器能够使用该FLUSH语句来清除或重新加载内部缓存、刷新表或获取锁。这仅在执行快照时使用）权限（默认启用）。使连接器能够使用该FLUSH语句来清除或重新加载内部缓存、刷新表或获取锁。这仅在执行快照时使用。

- 刷新权限：

`mysql> FLUSH PRIVILEGES;`

请参阅有关[权限说明](https://debezium.io/documentation/reference/1.5/connectors/mysql.html#mysql-creating-user)的更多信息。

## 可选配置

### 为每个阅读器设置不同的 SERVER ID

每个读取binlog的mysql client都应该要有一个唯一id。Mysql以id为粒度保存binlog消费位置。所以，要是多个作业共同使用同一个id，则会从错误的binlog位置读取。因此，可以通过sql Hints（通过添加选项和sql语句一起使用以达到更改查询计划）为每个阅读器设置不同的服务器id。比如源并行度为4，我们则可以为4个源阅读器中的每一个分配唯一的服务器 id。

`SELECT * FROM source_table /*+ OPTIONS('server-id'='5401-5404') */ ;`

### 设置 MySQL 会话超时

当mysql数据量过大导致一致性快照的制作耗费大量时间，使建立的mysql连接超时。

可以通过增大mysql 配置文件中`interactive_timeout`和`wait_timeout`参数来防止超时。

- interactive_timeout：服务器在关闭交互式连接之前等待活动的秒数。
- wait_timeout：服务器在关闭非交互式连接之前等待活动的秒数

详细信息参阅[Mysql文档](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_wait_timeout)。

## 创建Mysql CDC表

sql client Demo:

```SQL
-- 设置checkpoint间隔                      

Flink SQL> SET 'execution.checkpointing.interval' = '3s';   



-- DDL

Flink SQL> CREATE TABLE orders (

     order_id INT,

     order_date TIMESTAMP(0),

     customer_name STRING,

     price DECIMAL(10, 5),

     product_id INT,

     order_status BOOLEAN,

     PRIMARY KEY(order_id) NOT ENFORCED

     ) WITH (

     'connector' = 'mysql-cdc',

     'hostname' = 'localhost',

     'port' = '3306',

     'username' = 'root',

     'password' = '123456',

     'database-name' = 'mydb',

     'table-name' = 'orders');

  

Flink SQL> SELECT * FROM orders;
```

## 连接器选项

| **Option**                              | **Required** | **Default** | **Type** | **Description**                                              |
| --------------------------------------- | ------------ | ----------- | -------- | ------------------------------------------------------------ |
| connector                               | required     | (none)      | String   | 指定要使用的连接器，这里应该是'mysql-cdc'.                   |
| hostname                                | required     | (none)      | String   | MySQL 数据库服务器的 IP 地址或主机名。                       |
| username                                | required     | (none)      | String   | 连接到 MySQL 数据库服务器时要使用的mysql用户名。             |
| password                                | required     | (none)      | String   | 连接到 MySQL 数据库服务器时使用的密码。                      |
| database-name                           | required     | (none)      | String   | 要监控的 MySQL 服务器的数据库名称。database-name 还支持正则表达式来监控多个匹配正则表达式的表 |
| table-name                              | required     | (none)      | String   | 要监控的 MySQL 数据库的表名。table-name 还支持正则表达式来监控多个匹配正则表达式的表。 |
| port                                    | optional     | 3306        | Integer  | MySQL 数据库服务器的整数端口号。                             |
| server-id                               | optional     | (none)      | String   | 数据库客户端的数字ID或数字ID范围，数字ID语法如'5400'，数字ID范围语法如'5400-5408'，配合选项'scan.incremental.snapshot'使用，在启用时推荐使用数字ID范围。id唯一标识连接mysql集群的进程即客户端。默认情况下，会在5400到6400之间生成一个随机数，但建议设置一个明确的值。 |
| scan.incremental.snapshot.enabled       | optional     | true        | Boolean  | 增量快照是一种新的读取表快照的机制。与旧的快照机制相比，增量快照有很多优点，包括：（1）源在快照读取时可以并行，（2）源在快照读取时可以在块粒度上执行检查点，（3）源不需要在快照读取之前获取全局读锁（FLUSH TABLES WITH READ LOCK）。如果您希望源并行运行，每个并行读取器应该有一个唯一的服务器 id，因此 'server-id' 必须是一个类似 '5400-6400' 的范围，并且该范围必须大于并行度。有关更多详细信息，请参阅[增量快照](https://tqjdnyv0bs.larksuite.com/docs/docusMNYSwwHpCwj4q5zamz7lrQ#RvUHKb)读取部分。 |
| scan.incremental.snapshot.chunk.size    | optional     | 8096        | Integer  | 表快照的块大小（行数），表在读取表的快照时被拆分为多个块。   |
| scan.snapshot.fetch.size                | optional     | 1024        | Integer  | 读取表快照时每次轮询的最大获取大小。                         |
| scan.startup.mode                       | optional     | initial     | String   | MySQL CDC 消费者的可选启动模式，有效枚举为“initial”和“latest-offset”。有关更多详细信息，请参阅[启动位置](https://tqjdnyv0bs.larksuite.com/docs/docusMNYSwwHpCwj4q5zamz7lrQ#yERZIs)部分。 |
| server-time-zone                        | optional     | UTC         | String   | 数据库服务器中的会话时区，例如“亚洲/上海”。它控制 MYSQL 中的 TIMESTAMP 类型如何转换为 STRING。 |
| debezium.min.row.count.to.stream.result | optional     | 1000        | Integer  | 在快照操作期间，连接器将查询每个包含的表以生成该表中所有行的读取事件。此参数确定 MySQL 连接是否将表的所有结果拉入内存（速度快但需要大量内存），或者是否将结果流式传输（可能更慢，但适用于非常大的表）。该值指定在连接器传输结果之前表必须包含的最小行数，默认为 1,000。将此参数设置为“0”以跳过所有表大小检查并始终在快照期间流式传输所有结果，表的行数大于该值使用流式传输，小于该值将所有结果拉入内存。 |
| connect.timeout                         | optional     | 30s         | Duration | 在超时之前，连接器在尝试连接到 MySQL 数据库服务器后应等待的最长时间。 |
| debezium.*                              | optional     | (none)      | String   | 将 Debezium 的属性传递给 Debezium 嵌入式引擎，该引擎用于从 MySQL 服务器捕获数据更改。例如：'debezium.snapshot.mode' = 'never'。查看有关[Debezium 的 MySQL 连接器属性](https://debezium.io/documentation/reference/1.5/connectors/mysql.html#mysql-connector-properties)的更多信息 |

- snapshot.mode

Debezium 支持五种模式:

1. `initial` ：默认模式，在没有找到 offset 时(记录在 Kafka topic 的 `connect-offsets` 中，Kafka connect 框架维护)，做一次 snapshot——遍历有 SELECT 权限的表，收集列名，并且将每个表的所有行 select 出来写入 Kafka；
2. `when_needed`: 跟 `initial` 类似，只是增加了一种情况，当记录的 offset 对应的 binlog 位置已经在 MySQL 服务端被 purge 了时，就重新做一个 snapshot。
3. `never`: 不做 snapshot，也就是不拿所有表的列名，也不导出表数据到 Kafka，这个模式下，要求[从最开头](https://dieken.gitlab.io/posts/use-debezium-to-replicate-mysql-binlog/)消费 binlog，以获取完整的 DDL 信息，MySQL 服务端的 binlog 不能被 purge 过，否则由于 DML binlog 里只有 database name、table name、column type 却没有 column name，Debezium 会报错 `Encountered change event for table some_db.some_table whose schema isn't known to this connector`；
4. `schema_only`: 这种情况下会拿所有表的列名信息，但不会导出表数据到 Kafka，而且只从 Debezium 启动那刻起的 binlog 末尾开始消费，所以很适合不关心历史数据，只关心最近变更的场合。
5. `schema_only_recovery`: 在 Debezium 的 schema_only 模式出错时，用这个模式恢复，一般不会用到。

## 原理

### 增量快照

增量快照读取是一种新的读取表快照的机制。与旧的快照机制相比，增量快照有很多优点，包括：

- MySQL CDC Source在快照读取时可以并行
- MySQL CDC Source 可以在snapshot读取时以chunk粒度进行checkpoint
- SQL CDC Source在快照读取前不需要获取全局读锁(FLUSH TABLES WITH READ LOCK)

如果您希望源并行运行，每个并行读取器应该有一个唯一的服务器 id，因此 'server-id' 必须是一个类似 '5400-6400' 的范围，并且该范围必须大于并行度。

在增量快照读取过程中，MySQL CDC Source首先通过表的主键对快照chunk进行拆分（splits），然后MySQL CDC Source将这些chunk分配给多个reader来读取snapshot chunk的数据。

### 增量快照读取的工作原理

MySQL CDC 源启动时，它会并行读取表的快照，然后以单并行方式读取表的 binlog。

在快照阶段，根据表的主键和表行的大小将快照切割成多个快照块。快照块被分配给多个快照读取器。每个快照读取器使用块读取算法读取其接收到的块，并将读取的数据发送到下游。源管理块的进程状态（已完成或未完成），因此快照阶段的源可以支持块级别的检查点。如果发生故障，可以恢复源并继续从最后完成的块中读取块。

在所有快照块完成后，源将继续在单个任务中读取 binlog。为了保证快照记录和binlog记录的全局数据顺序，binlog reader会开始读取数据，直到snapshot chunks完成后有一个完整的checkpoint，以确保所有的快照数据都被下游消费了。binlog reader 在 state 中跟踪消耗的 binlog 位置，因此 binlog phase 的 source 可以支持行级别的 checkpoint。

Flink 定期为源执行检查点，在故障转移的情况下，作业将从上次成功的检查点状态重新启动并恢复，并保证恰好一次语义。

### 快照块拆分

在执行增量快照读取时，MySQL CDC 源需要一个用于拆分表的标准。MySQL CDC Source 使用拆分列将表拆分为多个拆分（块）。默认情况下，MySQL CDC 源会识别表的主键列，并使用主键中的第一列作为拆分列。如果表中没有主键，增量快照读取将失败，您可以禁用scan.incremental.snapshot.enabled回退到旧快照读取机制。

对于数字和自动增量拆分列，MySQL CDC Source 按固定步长有效拆分块。例如，如果您有一个表，其主键列id是自动增量 BIGINT 类型，最小值为0，最大值为100，表选项scan.incremental.snapshot.chunk.size值为25，则该表将被拆分为以下块：

```Plain%20Text
(-∞, 25),

 [25, 50),

 [50, 75),

 [75, 100),

 [100, +∞)
```

对于其他主键列类型，MySQL CDC Source 以`SELECT MAX(STR_ID) AS chunk_high FROM (SELECT * FROM TestTable WHERE STR_ID > 'uuid-001' limit 25)` 的形式执行语句来获取每个块的低值和高值，拆分块集如下：

```Plain%20Text
(-∞, 'uuid-001'),

 ['uuid-001', 'uuid-009'),

 ['uuid-009', 'uuid-abc'),

 ['uuid-abc', 'uuid-def'),

 [uuid-def, +∞)
```



### 块读取算法

对于上面的例子MyTable，如果 MySQL CDC Source 并行度设置为 4，MySQL CDC Source 将运行 4 个读取器，每个读取器执行偏移信号算法以获得快照块的最终一致输出。该偏移信号算法简单描述如下：

1. 记录当前binlog位置为LOWoffset
2. 通过执行语句读取并缓存快照chunk记录 SELECT * FROM MyTable WHERE id > chunk_low AND id <= chunk_high
3. 记录当前binlog位置作为HIGH偏移量
4. 从LOWoffset到HIGHoffset读取属于snapshot chunk的binlog记录
5. 将读取到的binlog记录Upsert到缓冲的chunk记录中，将buffer中的所有记录作为snapshot chunk的最终输出（都作为INSERT记录）发出
6. HIGH在single binlog reader中继续读取并发出属于offset之后的chunk的binlog记录。

**注意**：如果主键的实际值在其范围内不均匀分布，这可能会导致增量快照读取时任务不平衡。

## **启动阅读位置**

config 选项scan.startup.mode指定 MySQL CDC 使用者的启动模式。有效的枚举是：



-  initial （默认）：首次启动时对监控的数据库表进行初始快照，并继续读取最新的binlog。



- latest-offset: 第一次启动时不要对监控的数据库表执行快照，只从 binlog 的末尾读取，这意味着只有自连接器启动以来的更改。



注意：scan.startup.modeoption的机制依赖于 Debezium 的snapshot.mode配置。所以请不要同时使用它们。如果在表 DDL 中同时指定scan.startup.mode和debezium.snapshot.mode选项，它可能scan.startup.mode不起作用。



## DataStream

MySQL CDC Source 的增量快照读取功能目前仅在 SQL 中公开，如果您使用的是 DataStream，请使用旧版 MySQL Source：

```Java
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

import org.apache.flink.streaming.api.functions.source.SourceFunction;

import com.ververica.cdc.debezium.StringDebeziumDeserializationSchema;

import com.ververica.cdc.connectors.mysql.MySqlSource;



public class MySqlBinlogSourceExample {

  public static void main(String[] args) throws Exception {

    Properties debeziumProperties = new Properties();

    debeziumProperties.put("snapshot.locking.mode", "none");// do not use lock

    SourceFunction<String> sourceFunction = MySqlSource.<String>builder()

        .hostname("yourHostname")

        .port(yourPort)

        .databaseList("yourDatabaseName") // set captured database

        .tableList("yourDatabaseName.yourTableName") // set captured table

        .username("yourUsername")

        .password("yourPassword")

        .deserializer(new StringDebeziumDeserializationSchema()) // converts SourceRecord to String

        .debeziumProperties(debeziumProperties)

        .build();





    StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

    

    env.enableCheckpointing(3000); // checkpoint every 3000 milliseconds

    env

      .addSource(sourceFunction)

      .print().setParallelism(1); // use parallelism 1 for sink to keep message ordering



    env.execute();

  }

}
```

## 类型映射

| **MySQL type**                    | **Flink SQL type**                                    |
| --------------------------------- | ----------------------------------------------------- |
| TINYINT                           | TINYINT                                               |
| SMALLINT、TINYINT UNSIGNED        | SMALLINT                                              |
| INT、MEDIUMINT、SMALLINT UNSIGNED | INT                                                   |
| BIGINT、INT UNSIGNED              | BIGINT                                                |
| BIGINT UNSIGNED                   | DECIMAL(20, 0)                                        |
| BIGINT                            | BIGINT                                                |
| FLOAT                             | FLOAT                                                 |
| DOUBLE、DOUBLE PRECISION          | DOUBLE                                                |
| NUMERIC(p, s)、DECIMAL(p, s)      | DECIMAL(p, s)                                         |
| BOOLEAN、TINYINT(1)               | BOOLEAN                                               |
| DATE                              | DATE                                                  |
| TIME [(p)]                        | TIME [(p)] [WITHOUT TIMEZONE]                         |
| DATETIME [(p)]                    | TIMESTAMP [(p)] [WITHOUT TIMEZONE]                    |
| TIMESTAMP [(p)]                   | TIMESTAMP [(p)]、TIMESTAMP [(p)] WITH LOCAL TIME ZONE |
| CHAR(n)、VARCHAR(n)、TEXT         | STRING                                                |
| BINARY、VARBINARY、BLOB           | BYTES                                                 |

