SPARK任务
---------

### 步骤（1）：新建工作流任务

将图中的“+”控件拖拽到画布中，搜索spark任务，选择spark任务，确认。然后在画布中找到新建的任务，双击编辑。

![](media/21577f9f5736dbe51fd0640e0c826d23.png)

### 步骤（2）：填写基本信息

![](media/5cfd9fd70f9189f1064a15a5f2f3bd9a.png)

### 步骤（3）：配置调度设置

![](media/a5501083ab6f7f42869a53084dc5040c.png)

### 步骤（4）：参数配置

#### 上传脚本 or 选择脚本

![](media/05e21f0db0439067e0fa0cfff65f1ced.png)

选择你需要上传的脚本上传即可。

然后选择脚本

![](media/5362b95ba6e6ad9c6bd793a4199e890d.png)

![](media/f03027fc174bc61627d4caae14ea8d66.png)

输入执行spark任务的执行参数即可

### 步骤（5）：等待一段时间，等待系统执行任务

右键任务查看运行状态。

![](media/d26437dbd0e237acbb44720fb67121ec.png)

等待执行成功。

数据接入：
----------

Hdfs导出Hive（DATAHUB）
-----------------------

### 步骤（1）：创建工作流任务

首先我们需要创建一个Hdfs导出hive（datahub）工作流：（因为这个工作流只需要按照默认配置来进行即可，所以不再赘言）

![](media/36503aea8fc0c245e68a0da863b04e7f.png)

![](media/3163f3a8312cc21c5de22762e2ca325d.png)

![](media/c65023983cb9d7689fbfe05a997c3252.png)

审批之后，先不要执行：

![](media/2e06df97c074245a4373949150ceabbe.png)

因为这是一个数据接入的任务，所以需要配置一些数据接入的东西。

### 步骤（2）：新建hippo topic

查看说明文档，第一步的新建项目已经默认执行了，所以直接跳到第二步，新建hippo
topic。

![](media/ae1b763dbb9019bbe78aae86407ffd79.png)

![](media/37cbc32db3522612c96ce3f615cf5bf5.png)

![](media/d08d446e2320f054b7ffd69b2fd10ef7.png)

![](media/b06f48c274f005e761eca18ceef60de8.png)

可以在管理页面看到如下图：

![](media/d2dc512416d2a4ae0ad8e6b4714e7400.png)

![](media/9b679eb36bea0d3f82dbb7ee8e74e32f.png)

### 步骤（3）：新建数据接入

然后我们需要新建一个数据接入：

![](media/6a92359c5d5d999e3ad3805e03d97bae.png)

#### 配置基本信息

![](media/b894da5a0e324647a4f9059368444393.png)

#### 配置接口信息：

![](media/b4d664a27cbbe36eb27dafc2acac2684.png)

![](media/42f695f28f89205855f0f7af7273a944.png)

#### 选择topic，新建topology

![](media/271af6bb6e016608737f8e1e7bb25dc4.png)

#### 选择topology：

![](media/6d73d44e73750ec0a0e2ff7073ed5f48.png)

#### 点击资源验证：

![](media/107c1217782d8e3b125561c587a09e24.png)

#### 提示验证成功，即可提交申请：

![](media/57874f11be4c37cc674dc7d402602d4e.png)

#### 进入审批单页面：

![](media/3b8b46f3413f58a05326bd05cc329bbc.png)

#### 审批，检查一下是否配置正确，正确的话就通过，否则驳回：

![](media/5e1b5d44c1c6c87b8d064c158d06f538.png)

#### 审批成功：

![](media/aa06deab93e03465dd6168633103fb13.png)

### 步骤（4）：agent部署

#### 4.1 全量数据采集

所需软件包：

1.  apache-flume-1.7.0-bin.tar.gz

2.  data-hub-1.0.0-SNAPSHOT.tar.gz

3.  metric-2.0-bin.tar.gz

安装步骤：

1.  安装Flume agent

2.  安装TMetric agent

##### 4.1.1 安装Flume agent

###### 1. 解压apache-flume-1.7.0-bin.tar.gz到flume目录

###### 2. 解压data-hub-1.0.0-SNAPSHOT.tar.gz到data-hub目录

###### 3. 修改data-hub/conf/common.properties配置

| **配置项**            | **配置说明**                                                                   | **默认值** |
|-----------------------|--------------------------------------------------------------------------------|------------|
| data.hub.manager.ip   | TBDS集群中DataHubServer的IP。agent会定期心跳给DataHubServer。                  |            |
| data.hub.manager.port | TBDS集群中DataHubServer的heartbeat.port。                                      |            |
| data.hub.agent.name   | agent名称。可根据业务随意配置。                                                |            |
| data.hub.name         | 新建接入时，配置的接入名称。标识该agent是属于哪个接入。一个agent对应一个接入。 |            |
| data.hub.source       | agent所属接入的源数据库类型，可配置为SQLServer或者MySQL或者Oracle等。          |            |
| data.hub.target       | agent所属接入的流向目标，可配置为HDFS或者DB。                                  |            |

示例如下：

data.hub.manager.ip=10.0.0.15

data.hub.manager.port=9292

data.hub.agent.name=agent-hdfs-test

data.hub.name=hdfs2hive

data.hub.source=MySQL

data.hub.target=HDFS

######  拷贝data-hub/conf/flume.conf到flume/conf目录

cp flume.conf \$FLUME_HOME/conf

###### 修改flume/conf/flume.conf配置

| **配置项**                                    | **配置说明**                                                                                                           | **默认值** |
|-----------------------------------------------|------------------------------------------------------------------------------------------------------------------------|------------|
| type                                          | Flume agent的channel类型，必须配置为com.tencent.tbds.datahub.agent.flume.channel.hippo.HippoChannel                    |            |
| hippo.topic                                   | Hippo topic。接入配置时选择的topic。                                                                                   |            |
| hippo.controllerIpList                        | Hippo controller的IP列表。                                                                                             |            |
| hippo.producerGroup                           | Hippo生产组名，可根据业务配置。                                                                                        |            |
| hippo.producer.userName                       | 权限相关。生产者用户名。可在topic详情页面获得。                                                                        | admin      |
| hippo.producer.secretId                       | 权限相关。生产者secretId。可在topic详情页面获得。                                                                      |            |
| hippo.producer.secretKey                      | 权限相关。生产者secretKey。可在topic详情页面获得。                                                                     |            |
| hippo.producer.sendAsync.fqueue.path          | 消息发送失败时的暂存目录。                                                                                             |            |
| hippo.producer.sendAsync.enabled              | Hippo消息使用异步发送还是同步发送。机器配置较差时建议使用同步发送。                                                    | true       |
| hippo.producer.sendAsync.timeout              | 消息发送超时时间。毫秒为单位。默认20秒超时。                                                                           | 20000      |
| hippo.producer.sendAsync.rateLimit            | 消息发送限流。默认值为1000，表示每秒最多发送1000条消息。                                                               | 1000       |
| hippo.producer.sendAsync.fqueue.poll.interval | 超时重传间隔的毫秒数。默认一分钟重传一次。                                                                             | 60000      |
| hippo.lfReplace.enabled                       | 是否替换消息中的换行符。注意，如果数据中存在换行符而又不进行替换的话，则流向HDFS的文件里也会换行，导入Hive就会有问题。 | true       |
| hippo.lfReplace.replacement                   | 使用什么字符串来替换换行符。                                                                                           | \\\\n      |

###### 需配置HippoChannel与SQLSource。

###### HippoChannel配置说明：

###### SQLSource配置说明：

| **配置项**                          | **配置说明**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | **默认值** |
|-------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------|
| type                                | Flume agent的source类型，必须配置为com.tencent.tbds.tdbank.flume.source.sql.SQLSource                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |            |
| hibernate.connection.url            | JDBC连接URL。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |            |
| hibernate.connection.user           | JDBC连接用户名。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |            |
| hibernate.connection.password       | JDBC连接用户密码。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |            |
| hibernate.connection.isolation      | JDBC连接事务隔离级别。设置为2，即READ COMMITTED级别。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |            |
| hibernate.dialect                   | 根据不同数据库配置dialect。参考[https://docs.jboss.org/hibernate/orm/4.3/manual/en-US/html/ch03.html\#configuration-optional-dialects。](https://docs.jboss.org/hibernate/orm/4.3/manual/en-US/html/ch03.html#configuration-optional-dialects%E3%80%82)                                                                                                                                                                                                                                                                                                                                                 |            |
|                                     | 注意，采集SQLServer数据时，根据对应版本配置为：                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |            |
|                                     | com.tencent.tbds.tdbank.flume.source.sql.utils.SQLServerCustomDialect                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |            |
|                                     | com.tencent.tbds.tdbank.flume.source.sql.utils.SQLServer2005CustomDialect                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |            |
|                                     | com.tencent.tbds.tdbank.flume.source.sql.utils.SQLServer2008CustomDialect                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |            |
|                                     | com.tencent.tbds.tdbank.flume.source.sql.utils.SQLServer2012CustomDialect                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |            |
| hibernate.connection.driver_class   | JDBC连接需要的驱动类名。需要把驱动jar包放到flume的lib目录下。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |            |
| dbType                              | 如果通过JDBC URL无法解析出数据库类型，则需手动配置。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |            |
| dbHost                              | 如果通过JDBC URL无法解析出数据库主机，则需手动配置。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |            |
| database                            | 要采集数据的源库。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |            |
| tables                              | 要采集数据的源表。可配置多个表，逗号分隔。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |            |
| tables.\$表名.customQuery           | 完整的自定义SQL。可支持多表关联查询。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |            |
| tables.\$表名.whereClause           | WHERE子句。如果设置了column.name则WHERE子句中必须包含:index，该字符串表示当前采集到的该字段的最大值。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |            |
|                                     | 另外可以使用:bizdate，表示当前日期（不包括时间，例如2018-01-16）。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |            |
| tables.\$表名.startFrom             | 如果设置了column.name则为该字段最小值（不包括）。如果是时间戳类型字段，格式必须为yyyy-MM-dd HH:mm:ss.SSS，否则必须为整数。                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |            |
|                                     | 如果未设置column.name则为采集的起始偏移，必须为大于等于0的整数。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |            |
|                                     | 注意，如果配置了startFrom，则每次拉起cron任务时，都会将采集偏移重置回startFrom。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |            |
| tables.\$表名.column.name           | 通过某个字段来采集数据。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |            |
| tables.\$表名.column.isTimestamp    | true或者false。该字段是否为时间戳类型。默认为false。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |            |
| cronExpression                      | cron表达式，参考[http://www.quartz-scheduler.org/documentation/quartz-2.x/tutorials/crontrigger.html。](http://www.quartz-scheduler.org/documentation/quartz-2.x/tutorials/crontrigger.html%E3%80%82)                                                                                                                                                                                                                                                                                                                                                                                                   |            |
|                                     | 全量数据采集将根据cron表达式指定的时间点来执行。如果recoveryMode为false则cronExpression必须配置。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |            |
| recoveryMode                        | 可配置为true。若为true，则表示数据采集会立即执行，采集完成后agent进程自动退出。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | false      |
| output.delimiter                    | 字段分隔符，需要与接入配置一致，默认为竖线。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |            |
|                                     | 注意，若使用特殊字符，如\\u0001或者\\t，则此处应配置为\\\\u0001或者\\\\t，即应加上转义                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |            |
| query.pageSize                      | 分页查询大小。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | 10000      |
| query.numThreads                    | 全量数据采集线程池大小。一张表占用一个线程。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | 10         |
| query.timeout                       | SQL查询超时设置，以秒为单位。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | 60         |
| status.filePath                     | 用于存放当前数据采集进度的目录。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |            |
| hibernate.connection.provider_class | 设置为org.hibernate.connection.C3P0ConnectionProvider可使用C3P0连接池。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |            |
| hibernate.c3p0.min_size             | C3P0连接池最小值。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |            |
| hibernate.c3p0.max_size             | C3P0连接池最大值。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |            |

###### 配置示例：

\# nohup bin/flume-ng agent
-Dlog4j.configuration=/data/data-hub/conf/log4j.properties --classpath
/data/data-hub/conf:/data/data-hub/lib/\* --conf ./conf/ -f conf/flume.conf -n
agent1 \> stdout 2\>&1 &

\# nohup bin/flume-ng agent -Dlog4j.configuration=conf/log4j.properties
--classpath /data/data-hub/conf:/data/data-hub/lib/\* --conf ./conf/ -f
conf/flume.conf -n agent1 \> stdout 2\>&1 &

agent1.sources = src1

agent1.channels = ch1

\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\# HippoChannel
\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#

agent1.channels.ch1.type =
com.tencent.tbds.datahub.agent.flume.channel.hippo.HippoChannel

agent1.channels.ch1.hippo.topic = hdfs_hive_topic

agent1.channels.ch1.hippo.controllerIpList = 10.0.0.36:8066

agent1.channels.ch1.hippo.producerGroup = group_db_001

agent1.channels.ch1.hippo.producer.secretId =
pjQebM0BKskHT9j6EvOEuV3tN1FjbBgKv4e5

agent1.channels.ch1.hippo.producer.secretKey = SNGpA0R2QzgoyCrUldTqGdBC0m5WIR8B

agent1.channels.ch1.hippo.producer.sendAsync.fqueue.path =
/data/jordonchen/hdfstest/fqueue

agent1.sources.src1.type = avro

agent1.sources.src1.bind = 0.0.0.0

agent1.sources.src1.port = 41414

agent1.sources.src1.channels = ch1

\#\#agent1.sources.src1.type = netcat

\#\#agent1.sources.src1.bind = 0.0.0.0

\#\#agent1.sources.src1.port = 6666

\#\#agent1.sources.src1.channels = ch1

\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#全量数据采集\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#

agent1.sources.sqlSource.channels = ch1

agent1.sources.sqlSource.type =
com.tencent.tbds.tdbank.flume.source.sql.SQLSource

agent1.sources.sqlSource.hibernate.connection.url =
jdbc:mysql://139.199.154.43:3306/tdbank

agent1.sources.sqlSource.hibernate.connection.user = root

agent1.sources.sqlSource.hibernate.connection.password = portal\@Tbds.com

agent1.sources.sqlSource.hibernate.connection.isolation = 2

agent1.sources.sqlSource.hibernate.dialect = org.hibernate.dialect.MySQL5Dialect

agent1.sources.sqlSource.hibernate.connection.driver_class =
com.mysql.jdbc.Driver

agent1.sources.sqlSource.database = tdbank

agent1.sources.sqlSource.tables = student,teacher

agent1.sources.sqlSource.hibernate.connection.autocommit = true

\#\# 参考
https://docs.jboss.org/hibernate/orm/4.3/manual/en-US/html/ch03.html\#configuration-optional-dialects

\#\#agent1.sources.sqlSource.hibernate.dialect =
org.hibernate.dialect.MySQL5Dialect

\#\# 将DB的JDBC驱动jar包扔入\$FLUME_HOME/lib或者\$DATAHUB_HOME/lib

\#\#agent1.sources.sqlSource.hibernate.connection.driver_class =
com.mysql.jdbc.Driver

\#\# 接入接口名称为 TESTDB_TESTTABLE

\#\#agent1.sources.sqlSource.database = testDB

\#\#agent1.sources.sqlSource.table = testTable

\#\#agent1.sources.sqlSource.columns.to.select = \*

\#\#agent1.sources.sqlSource.whereClause =

\#\#agent1.sources.sqlSource.recoveryMode = true

\#\# 参考
http://www.quartz-scheduler.org/documentation/quartz-2.x/tutorials/crontrigger.html

\#\#agent1.sources.sqlSource.cronExpression = 0 0/5 \* \* \* ?

\#\# Status file is used to save last readed row

\#\#agent1.sources.sqlSource.status.file.path = /data/flume

\#\#agent1.sources.sqlSource.status.file.name = sqlSource.status

\#\#agent1.sources.sqlSource.hibernate.connection.provider_class =
org.hibernate.connection.C3P0ConnectionProvider

\#\#agent1.sources.sqlSource.hibernate.c3p0.min_size=1

\#\#agent1.sources.sqlSource.hibernate.c3p0.max_size=10

agent1.channels.ch1.type =
com.tencent.tbds.datahub.agent.flume.channel.hippo.HippoChannel

agent1.channels.ch1.hippo.topic = hdfs2hive611

\#agent1.channels.ch1.hippo.dynamicIF.enabled = false

\#agent1.channels.ch1.hippo.bizInterface = test_interface

agent1.channels.ch1.hippo.controllerIpList = 10.0.0.36:8066

agent1.channels.ch1.hippo.producerGroup = test_group

agent1.channels.ch1.hippo.producer.secretId =
pjQebM0BKskHT9j6EvOEuV3tN1FjbBgKv4e5

agent1.channels.ch1.hippo.producer.secretKey = SNGpA0R2QzgoyCrUldTqGdBC0m5WIR8B

agent1.channels.ch1.hippo.producer.sendAsync.fqueue.path = /data/fqueue

###### 修改flume/conf/log4j.properties配置

增加以下配置，用于生成指标日志文件，供TMetric agent采集：

log4j.logger.metrics=INFO, METRICS

log4j.appender.METRICS=org.apache.log4j.rolling.RollingFileAppender

log4j.appender.METRICS.rollingPolicy=org.apache.log4j.rolling.TimeBasedRollingPolicy

log4j.appender.METRICS.rollingPolicy.ActiveFileName=\${flume.log.dir}/metrics.log

log4j.appender.METRICS.rollingPolicy.FileNamePattern=\${flume.log.dir}/metrics.log.%d{yyyy-MM-dd}

log4j.appender.METRICS.layout=org.apache.log4j.PatternLayout

log4j.appender.METRICS.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %m%n

###### 启动Flume agent：

nohup bin/flume-ng agent -Dlog4j.configuration=conf/log4j.properties --classpath
/data-hub目录/conf:/data-hub目录/lib/\* --conf conf/ -f conf/flume.conf -n
agent1 \> stdout 2\>&1 &

##### 4.1.2 安装TMetric agent

###### 1. 解压metric-2.0-bin.tar.gz到tmetric目录

###### 2. 修改tmetric/conf/agent.ini配置

只需修改以下配置：

; agent主机地址，使用本机ip

hostName=10.0.0.2

; agent监听的rpc端口，必填项

port=8003

; TMetric master的IP与配置的端口

masterAddressList=tbds-10-0-0-7:9000

; 修改为本地地址

bdbFilePath=/data/tmetric/bdb

metricSpliter=\#

; 修改为本地地址，配置为flume agent指标日志文件。需保证TMetric
agent对metrics.log有可读可执行权限

metricFiles=/data/flume/logs/metrics.log

; 修改为本地地址

failoverQueuePath=/data/tmetric/failover

###### 3. 启动TMetric agent：bin/agent.sh start

bin/agent.sh start

#### 4.2 增量数据采集

增量数据采集有两种方式：使用Oracle
GoldenGate（以下简称OGG）采集，或者使用Flume的SQLSource采集。

-   使用OGG可以采集到所有的增量数据，但是配置较为繁琐；

-   使用SQLSource可以用程序生成配置文件，但是无法采集到所有的增量数据，限制如下：

    -   无法采集到delete的数据；

    -   增量insert/update的数据必须具有自增的主键或者时间戳字段；

##### 4.2.1 使用OGG

所需软件包：

1.  apache-flume-1.7.0-bin.tar.gz

2.  data-hub-1.0.0-SNAPSHOT.tar.gz

3.  metric-2.0-bin.tar.gz

4.  OGG相关软件包

安装步骤：

1.  安装Flume agent。同全量。需修改source配置；

2.  增量数据采集同样需要配置source与channel。channel与全量一致，使用HippoChannel。根据使用OGG或者使用SQLSource需要配置不同的source。

3.  1) 若使用OGG，则source需要AvroSource，配置如下：

4.  agent1.sources.src1.type = avro

5.  agent1.sources.src1.bind = 0.0.0.0

6.  agent1.sources.src1.port = 41414

7.  agent1.sources.src1.channels = hippoChannel

8.  2) 若使用SQLSource，可实现准实时增量数据采集，也可实现T+1增量数据采集：

9.  2.1) 准实时增量数据采集，配置示例如下，

10. agent1.sources = sqlSource

11. agent1.channels = ch1

12. \#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\# HippoChannel
    \#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#

13. agent1.channels.ch1.type =
    com.tencent.tbds.datahub.agent.flume.channel.hippo.H

14. ippoChannel

15. agent1.channels.ch1.hippo.topic = topic_poc

16. agent1.channels.ch1.hippo.controllerIpList = 10.0.0.10:8066

17. agent1.channels.ch1.hippo.producerGroup = group_poc

18. agent1.channels.ch1.hippo.producer.secretId =
    WJbcIYI6Kkj6OH6Uy2QMufDVvewFrKCzD3hi

19. agent1.channels.ch1.hippo.producer.secretKey =
    AHf3V1fqZt4cN6wPO6cdk0M0kMQnuHer

20. agent1.channels.ch1.hippo.producer.sendAsync.fqueue.path = /data/fqueue

21. \#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\# 准实时增量数据采集
    \#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#

22. agent1.sources.sqlSource.channels = ch1

23. agent1.sources.sqlSource.type =
    com.tencent.tbds.tdbank.flume.source.sql.SQLSource

24. agent1.sources.sqlSource.hibernate.connection.url =
    jdbc:mysql://10.0.0.1:3306/test

25. agent1.sources.sqlSource.hibernate.connection.user = tbds

26. agent1.sources.sqlSource.hibernate.connection.password = tbds

27. agent1.sources.sqlSource.hibernate.connection.isolation = 2

28. agent1.sources.sqlSource.hibernate.dialect =
    org.hibernate.dialect.MySQL5Dialect

29. agent1.sources.sqlSource.hibernate.connection.driver_class =
    com.mysql.jdbc.Driver

30. agent1.sources.sqlSource.database = db_poc

31. agent1.sources.sqlSource.tables = tbl_poc1, tbl_poc2

32. \#\# 每5分钟采集一次数据

33. agent1.sources.sqlSource.cronExpression = 0 0/5 \* \* \* ?

34. agent1.sources.sqlSource.status.filePath = /data/sqlStatus

35. \#\# 注意，如果没有设置startFrom，首次采集实际上是会执行全量采集的

36. \#\# 配置startFrom有副作用，见配置说明

37. agent1.sources.sqlSource.tables.tbl_poc1.whereClause = WHERE id \> :index
    ORDER BY id

38. agent1.sources.sqlSource.tables.tbl_poc1.column.name = id

39. agent1.sources.sqlSource.tables.tbl_poc2.whereClause = WHERE modify_time \>
    :index ORDER BY modify_time

40. agent1.sources.sqlSource.tables.tbl_poc2.column.name = modify_time

41. agent1.sources.sqlSource.tables.tbl_poc2.column.isTimestamp = true

42. agent1.sources.sqlSource.hibernate.connection.provider_class =
    org.hibernate.connection.C3P0ConnectionProvider

43. agent1.sources.sqlSource.hibernate.c3p0.min_size=1

44. agent1.sources.sqlSource.hibernate.c3p0.max_size=10

45. 2.2) T+1增量数据采集，配置示例如下，

46. agent1.sources = sqlSource

47. agent1.channels = ch1

48. \#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\# HippoChannel
    \#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#

49. agent1.channels.ch1.type =
    com.tencent.tbds.datahub.agent.flume.channel.hippo.H

50. ippoChannel

51. agent1.channels.ch1.hippo.topic = topic_poc

52. agent1.channels.ch1.hippo.controllerIpList = 10.0.0.10:8066

53. agent1.channels.ch1.hippo.producerGroup = group_poc

54. agent1.channels.ch1.hippo.producer.secretId =
    WJbcIYI6Kkj6OH6Uy2QMufDVvewFrKCzD3hi

55. agent1.channels.ch1.hippo.producer.secretKey =
    AHf3V1fqZt4cN6wPO6cdk0M0kMQnuHer

56. agent1.channels.ch1.hippo.producer.sendAsync.fqueue.path = /data/fqueue

57. \#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\# T+1增量数据采集
    \#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#

58. agent1.sources.sqlSource.channels = ch1

59. agent1.sources.sqlSource.type =
    com.tencent.tbds.tdbank.flume.source.sql.SQLSource

60. agent1.sources.sqlSource.hibernate.connection.url =
    jdbc:mysql://10.0.0.1:3306/test

61. agent1.sources.sqlSource.hibernate.connection.user = tbds

62. agent1.sources.sqlSource.hibernate.connection.password = tbds

63. agent1.sources.sqlSource.hibernate.connection.isolation = 2

64. agent1.sources.sqlSource.hibernate.dialect =
    org.hibernate.dialect.MySQL5Dialect

65. agent1.sources.sqlSource.hibernate.connection.driver_class =
    com.mysql.jdbc.Driver

66. agent1.sources.sqlSource.database = db_poc

67. agent1.sources.sqlSource.tables = tbl_poc2

68. \#\# 每天凌晨1点钟采集一次数据

69. agent1.sources.sqlSource.cronExpression = 0 0 1 \* \* ?

70. agent1.sources.sqlSource.status.filePath = /data/sqlStatus

71. \#\# 注意，如果没有设置startFrom，首次采集实际上是会执行全量采集的

72. \#\# 配置startFrom有副作用，见配置说明

73. agent1.sources.sqlSource.tables.tbl_poc2.whereClause = WHERE modify_time \>
    :index AND modify_time \< :bizdate ORDER BY modify_time

74. agent1.sources.sqlSource.tables.tbl_poc2.column.name = modify_time

75. agent1.sources.sqlSource.tables.tbl_poc2.column.isTimestamp = true

76. agent1.sources.sqlSource.hibernate.connection.provider_class =
    org.hibernate.connection.C3P0ConnectionProvider

77. agent1.sources.sqlSource.hibernate.c3p0.min_size=1

78. agent1.sources.sqlSource.hibernate.c3p0.max_size=10

79. 安装TMetric agent。同全量；

80. 安装OGG Flume
    Adapter，参考[TDBank采集接口详解](https://tbds-book.gitbooks.io/tbds/%E6%95%B0%E6%8D%AE%E6%8E%A5%E5%85%A5/%E6%95%B0%E6%8D%AE%E6%8E%A5%E5%85%A5/tdbank_conf.html)；

81. 安装OGG for
    MySQL/SQLServer/Oracle等，参考[TDBank采集接口详解](https://tbds-book.gitbooks.io/tbds/%E6%95%B0%E6%8D%AE%E6%8E%A5%E5%85%A5/%E6%95%B0%E6%8D%AE%E6%8E%A5%E5%85%A5/tdbank_conf.html)；

##### 4.2.2 使用SQLSource

所需软件包：

1.  apache-flume-1.7.0-bin.tar.gz

2.  data-hub-1.0.0-SNAPSHOT.tar.gz

3.  metric-2.0-bin.tar.gz

安装步骤：

1.  安装Flume agent。同全量；

2.  安装TMetric agent。同全量；

### 步骤（5）：启动HDFS导出HIVE（DATAHUB）任务

右键画板中的任务，运行

![](media/f447af7219a00ff9ac222f72c8f39822.png)

查看任务状态，运行中。

![](media/0a35173f5f309185d43f7c142333388d.png)

### 步骤（6）：使用hql查询数据

数据分析-\>数据交互-\>HIVE-\>输入HIVESQL进行数据的查询即可

![](media/29202d336a4d726ff91c15f4ef75b9d5.png)
