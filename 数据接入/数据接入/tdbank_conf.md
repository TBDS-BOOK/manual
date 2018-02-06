## DataHub配置说明

### common.properties配置说明

| **配置项** | **配置说明** | **默认值** |
| --- | --- | --- |
| <font color='red'>data.hub.manager.ip</font> | TBDS集群中DataHubServer的IP。agent会定期心跳给DataHubServer。 |   |
| <font color='red'>data.hub.manager.port</font> | TBDS集群中DataHubServer的heartbeat.port。 |   |
| <font color='red'>data.hub.agent.name</font> | agent名称。可根据业务随意配置。 |   |
| <font color='red'>data.hub.name</font> | 新建接入时，配置的接入名称。标识该agent是属于哪个接入。一个agent对应一个接入。 |   |
| <font color='red'>data.hub.source</font> | agent所属接入的源数据库类型，可配置为SQLServer或者MySQL或者Oracle等。 |   |
| <font color='red'>data.hub.target</font> | agent所属接入的流向目标，可配置为HDFS或者DB。 |   ||

注：红色配置项为必填项

## Flume配置说明

### log4j.properties配置说明

增加以下配置，用于生成指标日志文件，供TMetric agent采集：

```
log4j.logger.metrics=INFO, METRICS
log4j.appender.METRICS=org.apache.log4j.rolling.RollingFileAppender
log4j.appender.METRICS.rollingPolicy=org.apache.log4j.rolling.TimeBasedRollingPolicy
log4j.appender.METRICS.rollingPolicy.ActiveFileName=${flume.log.dir}/metrics.log
log4j.appender.METRICS.rollingPolicy.FileNamePattern=${flume.log.dir}/metrics.log.%d{yyyy-MM-dd}
log4j.appender.METRICS.layout=org.apache.log4j.PatternLayout
log4j.appender.METRICS.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %m%n
```

### flume.conf配置说明

#### 全量数据采集

需配置HippoChannel与SQLSource。

HippoChannel配置说明：

| **配置项** | **配置说明** | **默认值** |
| --- | --- | --- |
| <font color='red'>type</font> | Flume agent的channel类型，必须配置为`com.tencent.tbds.datahub.agent.flume.channel.hippo.HippoChannel` |   |
| <font color='red'>hippo.topic</font> | Hippo topic。接入配置时选择的topic。 |   |
| <font color='red'>hippo.controllerIpList</font> | Hippo controller的IP列表。 |   |
| <font color='red'>hippo.producerGroup</font> | Hippo生产组名，可根据业务配置。 |   |
| hippo.producer.userName | 权限相关。生产者用户名。可在topic详情页面获得。 | admin |
| <font color='red'>hippo.producer.secretId</font> | 权限相关。生产者secretId。可在topic详情页面获得。 |   |
| <font color='red'>hippo.producer.secretKey</font> | 权限相关。生产者secretKey。可在topic详情页面获得。 |   |
| <font color='red'>hippo.producer.sendAsync.fqueue.path</font> | 消息发送失败时的暂存目录。 |   |
| hippo.producer.sendAsync.enabled | Hippo消息使用异步发送还是同步发送。机器配置较差时建议使用同步发送。 | true |
| hippo.producer.sendAsync.timeout | 消息发送超时时间。毫秒为单位。默认20秒超时。 | 20000 |
| hippo.producer.sendAsync.rateLimit | 消息发送限流。默认值为1000，表示每秒最多发送1000条消息。 | 1000 |
| hippo.producer.sendAsync.fqueue.poll.interval | 超时重传间隔的毫秒数。默认一分钟重传一次。 | 60000 |
| hippo.lfReplace.enabled | 是否替换消息中的换行符。注意，如果数据中存在换行符而又不进行替换的话，则流向HDFS的文件里也会换行，导入Hive就会有问题。 | true |
| hippo.lfReplace.replacement | 使用什么字符串来替换换行符。 | \\\n |


SQLSource配置说明：

| **配置项** | **配置说明** | **默认值** |
| --- | --- | --- |
| <font color='red'>type</font> | Flume agent的source类型，必须配置为`com.tencent.tbds.tdbank.flume.source.sql.SQLSource` |   |
| <font color='red'>hibernate.connection.url</font> | JDBC连接URL。 |   |
| <font color='red'>hibernate.connection.user</font> | JDBC连接用户名。 |   |
| <font color='red'>hibernate.connection.password</font> | JDBC连接用户密码。 |   |
| <font color='red'>hibernate.connection.isolation</font> | JDBC连接事务隔离级别。设置为2，即READ COMMITTED级别。 |   |
| <font color='red'>hibernate.dialect</font> | 根据不同数据库配置dialect。参考https://docs.jboss.org/hibernate/orm/4.3/manual/en-US/html/ch03.html#configuration-optional-dialects。<br/>注意，采集SQLServer数据时，根据对应版本配置为：<br/>com.tencent.tbds.tdbank.flume.source.sql.utils.SQLServerCustomDialect <br/> com.tencent.tbds.tdbank.flume.source.sql.utils.SQLServer2005CustomDialect <br/> com.tencent.tbds.tdbank.flume.source.sql.utils.SQLServer2008CustomDialect <br/> com.tencent.tbds.tdbank.flume.source.sql.utils.SQLServer2012CustomDialect |   |
| <font color='red'>hibernate.connection.driver\_class</font> | JDBC连接需要的驱动类名。需要把驱动jar包放到flume的lib目录下。 |   |
| dbType | 如果通过JDBC URL无法解析出数据库类型，则需手动配置。 |   |
| dbHost | 如果通过JDBC URL无法解析出数据库主机，则需手动配置。 |   |
| <font color='red'>database</font> | 要采集数据的源库。 |   |
| <font color='red'>tables</font> | 要采集数据的源表。可配置多个表，逗号分隔。 |   |
| tables.$表名.customQuery | 完整的自定义SQL。可支持多表关联查询。 |   |
| tables.$表名.whereClause | WHERE子句。如果设置了column.name则WHERE子句中必须包含`:index`，该字符串表示当前采集到的该字段的最大值。<br/>另外可以使用`:bizdate`，表示当前日期（不包括时间，例如2018-01-16）。 |   |
| tables.$表名.startFrom | 如果设置了column.name则为该字段最小值（不包括）。如果是时间戳类型字段，格式必须为`yyyy-MM-dd HH:mm:ss.SSS`，否则必须为整数。<br/>如果未设置column.name则为采集的起始偏移，必须为大于等于0的整数。<br/>注意，如果配置了startFrom，则每次拉起cron任务时，都会将采集偏移重置回startFrom。 |   |
| tables.$表名.column.name | 通过某个字段来采集数据。 |   |
| tables.$表名.column.isTimestamp | true或者false。该字段是否为时间戳类型。默认为false。 |   |
| cronExpression | cron表达式，参考http://www.quartz-scheduler.org/documentation/quartz-2.x/tutorials/crontrigger.html。<br/>全量数据采集将根据cron表达式指定的时间点来执行。如果recoveryMode为false则cronExpression必须配置。 |   |
| recoveryMode | 可配置为true。若为true，则表示数据采集会立即执行，采集完成后agent进程自动退出。 | false |
| output.delimiter | 字段分隔符，需要与接入配置一致，默认为竖线。<br/>注意，若使用特殊字符，如\u0001或者\t，则此处应配置为\\\u0001或者\\\t，即应加上转义 | |
| query.pageSize | 分页查询大小。 | 10000 |
| query.numThreads | 全量数据采集线程池大小。一张表占用一个线程。 | 10 |
| query.timeout | SQL查询超时设置，以秒为单位。 | 60 |
| <font color='red'>status.filePath</font> | 用于存放当前数据采集进度的目录。 |   |
| hibernate.connection.provider\_class | 设置为`org.hibernate.connection.C3P0ConnectionProvider`可使用C3P0连接池。 |   |
| hibernate.c3p0.min\_size | C3P0连接池最小值。 |   |
| hibernate.c3p0.max\_size | C3P0连接池最大值。 |   ||

配置示例：

```
agent1.sources = sqlSource
agent1.channels = ch1

##################### HippoChannel #####################
agent1.channels.ch1.type = com.tencent.tbds.datahub.agent.flume.channel.hippo.HippoChannel
agent1.channels.ch1.hippo.topic = topic_poc
agent1.channels.ch1.hippo.controllerIpList = 10.0.0.10:8066
agent1.channels.ch1.hippo.producerGroup = group_poc
agent1.channels.ch1.hippo.producer.secretId = WJbcIYI6Kkj6OH6Uy2QMufDVvewFrKCzD3hi
agent1.channels.ch1.hippo.producer.secretKey = AHf3V1fqZt4cN6wPO6cdk0M0kMQnuHer
agent1.channels.ch1.hippo.producer.sendAsync.fqueue.path = /data/fqueue

##################### 全量数据采集 #####################
agent1.sources.sqlSource.channels = ch1
agent1.sources.sqlSource.type = com.tencent.tbds.tdbank.flume.source.sql.SQLSource
agent1.sources.sqlSource.hibernate.connection.url = jdbc:mysql://10.0.0.1:3306/test
agent1.sources.sqlSource.hibernate.connection.user = tbds
agent1.sources.sqlSource.hibernate.connection.password = tbds
agent1.sources.sqlSource.hibernate.connection.isolation = 2
agent1.sources.sqlSource.hibernate.dialect = org.hibernate.dialect.MySQL5Dialect
agent1.sources.sqlSource.hibernate.connection.driver_class = com.mysql.jdbc.Driver
agent1.sources.sqlSource.database = db_poc
agent1.sources.sqlSource.tables = tbl_poc1, tbl_poc2

## 增加以下配置可使用索引字段（假设为id）提升查询性能
## 另外也可以使用自定义SQL
## agent1.sources.sqlSource.tables.tbl_poc1.whereClause = WHERE id > :index ORDER BY id
## agent1.sources.sqlSource.tables.tbl_poc1.column.name = id

agent1.sources.sqlSource.recoveryMode = true
agent1.sources.sqlSource.status.filePath = /data/sqlStatus

agent1.sources.sqlSource.hibernate.connection.provider_class = org.hibernate.connection.C3P0ConnectionProvider
agent1.sources.sqlSource.hibernate.c3p0.min_size=1
agent1.sources.sqlSource.hibernate.c3p0.max_size=10
```

#### 增量数据采集

增量数据采集同样需要配置source与channel。channel与全量一致，使用HippoChannel。根据使用OGG或者使用SQLSource需要配置不同的source。

1) 若使用OGG，则source需要AvroSource，配置如下：

```
agent1.sources.src1.type = avro
agent1.sources.src1.bind = 0.0.0.0
agent1.sources.src1.port = 41414
agent1.sources.src1.channels = hippoChannel
```

2) 若使用SQLSource，可实现准实时增量数据采集，也可实现T+1增量数据采集：

2.1) 准实时增量数据采集，配置示例如下，

```
agent1.sources = sqlSource
agent1.channels = ch1

##################### HippoChannel #####################
agent1.channels.ch1.type = com.tencent.tbds.datahub.agent.flume.channel.hippo.H
ippoChannel
agent1.channels.ch1.hippo.topic = topic_poc
agent1.channels.ch1.hippo.controllerIpList = 10.0.0.10:8066
agent1.channels.ch1.hippo.producerGroup = group_poc
agent1.channels.ch1.hippo.producer.secretId = WJbcIYI6Kkj6OH6Uy2QMufDVvewFrKCzD3hi
agent1.channels.ch1.hippo.producer.secretKey = AHf3V1fqZt4cN6wPO6cdk0M0kMQnuHer
agent1.channels.ch1.hippo.producer.sendAsync.fqueue.path = /data/fqueue

##################### 准实时增量数据采集 #####################
agent1.sources.sqlSource.channels = ch1
agent1.sources.sqlSource.type = com.tencent.tbds.tdbank.flume.source.sql.SQLSource
agent1.sources.sqlSource.hibernate.connection.url = jdbc:mysql://10.0.0.1:3306/test
agent1.sources.sqlSource.hibernate.connection.user = tbds
agent1.sources.sqlSource.hibernate.connection.password = tbds
agent1.sources.sqlSource.hibernate.connection.isolation = 2
agent1.sources.sqlSource.hibernate.dialect = org.hibernate.dialect.MySQL5Dialect
agent1.sources.sqlSource.hibernate.connection.driver_class = com.mysql.jdbc.Driver
agent1.sources.sqlSource.database = db_poc
agent1.sources.sqlSource.tables = tbl_poc1, tbl_poc2

## 每5分钟采集一次数据
agent1.sources.sqlSource.cronExpression = 0 0/5 * * * ?
agent1.sources.sqlSource.status.filePath = /data/sqlStatus

## 注意，如果没有设置startFrom，首次采集实际上是会执行全量采集的
## 配置startFrom有副作用，见配置说明
agent1.sources.sqlSource.tables.tbl_poc1.whereClause = WHERE id > :index ORDER BY id
agent1.sources.sqlSource.tables.tbl_poc1.column.name = id

agent1.sources.sqlSource.tables.tbl_poc2.whereClause = WHERE modify_time > :index ORDER BY modify_time
agent1.sources.sqlSource.tables.tbl_poc2.column.name = modify_time
agent1.sources.sqlSource.tables.tbl_poc2.column.isTimestamp = true

agent1.sources.sqlSource.hibernate.connection.provider_class = org.hibernate.connection.C3P0ConnectionProvider
agent1.sources.sqlSource.hibernate.c3p0.min_size=1
agent1.sources.sqlSource.hibernate.c3p0.max_size=10
```

2.2) T+1增量数据采集，配置示例如下，

```
agent1.sources = sqlSource
agent1.channels = ch1

##################### HippoChannel #####################
agent1.channels.ch1.type = com.tencent.tbds.datahub.agent.flume.channel.hippo.H
ippoChannel
agent1.channels.ch1.hippo.topic = topic_poc
agent1.channels.ch1.hippo.controllerIpList = 10.0.0.10:8066
agent1.channels.ch1.hippo.producerGroup = group_poc
agent1.channels.ch1.hippo.producer.secretId = WJbcIYI6Kkj6OH6Uy2QMufDVvewFrKCzD3hi
agent1.channels.ch1.hippo.producer.secretKey = AHf3V1fqZt4cN6wPO6cdk0M0kMQnuHer
agent1.channels.ch1.hippo.producer.sendAsync.fqueue.path = /data/fqueue

##################### T+1增量数据采集 #####################
agent1.sources.sqlSource.channels = ch1
agent1.sources.sqlSource.type = com.tencent.tbds.tdbank.flume.source.sql.SQLSource
agent1.sources.sqlSource.hibernate.connection.url = jdbc:mysql://10.0.0.1:3306/test
agent1.sources.sqlSource.hibernate.connection.user = tbds
agent1.sources.sqlSource.hibernate.connection.password = tbds
agent1.sources.sqlSource.hibernate.connection.isolation = 2
agent1.sources.sqlSource.hibernate.dialect = org.hibernate.dialect.MySQL5Dialect
agent1.sources.sqlSource.hibernate.connection.driver_class = com.mysql.jdbc.Driver
agent1.sources.sqlSource.database = db_poc
agent1.sources.sqlSource.tables = tbl_poc2

## 每天凌晨1点钟采集一次数据
agent1.sources.sqlSource.cronExpression = 0 0 1 * * ?
agent1.sources.sqlSource.status.filePath = /data/sqlStatus

## 注意，如果没有设置startFrom，首次采集实际上是会执行全量采集的
## 配置startFrom有副作用，见配置说明
agent1.sources.sqlSource.tables.tbl_poc2.whereClause = WHERE modify_time > :index AND modify_time < :bizdate ORDER BY modify_time
agent1.sources.sqlSource.tables.tbl_poc2.column.name = modify_time
agent1.sources.sqlSource.tables.tbl_poc2.column.isTimestamp = true

agent1.sources.sqlSource.hibernate.connection.provider_class = org.hibernate.connection.C3P0ConnectionProvider
agent1.sources.sqlSource.hibernate.c3p0.min_size=1
agent1.sources.sqlSource.hibernate.c3p0.max_size=10
```

## TMetric配置说明

### agent.ini配置说明

只需修改以下配置：

```
; agent主机地址，使用本机ip
hostName=10.0.0.2
; agent监听的rpc端口，必填项
port=8003
; TMetric master的IP与配置的端口
masterAddressList=tbds-10-0-0-7:9000
; 修改为本地地址
bdbFilePath=/data/tmetric/bdb
metricSpliter=#
; 修改为本地地址，配置为flume agent指标日志文件。需保证TMetric agent对metrics.log有可读可执行权限
metricFiles=/data/flume/logs/metrics.log
; 修改为本地地址
failoverQueuePath=/data/tmetric/failover
```

## OGG安装说明

### OGG Flume Adapter安装说明

参考[OGG-Flume-Adapter-部署文档](ogg_flume_adapter.md)。

### OGG for MySQL/SQLServer/Oracle等安装说明

根据源库不同的类型不同的版本需要下载不同的OGG安装包，并且需要不同的配置。可根据不同环境联系我们提供安装文档，在此不做赘述。
