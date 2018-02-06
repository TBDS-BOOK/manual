所需软件包：

 - Linux平台：122011_ggs_Adapters_Linux_x64.zip
 - Windows平台：V100448-01.zip (Oracle GoldenGate for Big Data 12.2.0.1.0 on Microsoft Windows x64)

### 初始化

1) 在/data/ggsAdapters中解压安装包

2) ./ggsci

3) GGSCI > create subdirs

### 配置manager

4) GGSCI > edit params mgr
```
PORT 7809
PURGEOLDEXTRACTS /data/ggsAdapters/dirdat/*, USECHECKPOINTS
```

### 配置replicat

5) cp AdapterExamples/big-data/flume/* dirprm/

6) vi dirprm/flume.props
```
gg.handlerlist = flumehandler
gg.handler.flumehandler.type=flume
gg.handler.flumehandler.RpcClientPropertiesFile=custom-flume-rpc.properties

gg.handler.flumehandler.format=delimitedtext
gg.handler.flumehandler.format.fieldDelimiter=|
gg.handler.flumehandler.format.includePosition=false
gg.handler.flumehandler.format.pkUpdateHandling=update

gg.handler.flumehandler.mode=tx
gg.handler.flumehandler.EventMapsTo=op
gg.handler.flumehandler.PropagateSchema=true
gg.handler.flumehandler.includeTokens=false

goldengate.userexit.timestamp=utc
goldengate.userexit.writers=javawriter
javawriter.stats.display=TRUE
javawriter.stats.full=TRUE

gg.log=log4j
gg.log.level=INFO

gg.report.time=30sec

## /data/apache-flume-1.7.0-bin 为flume安装路径
gg.classpath=dirprm/:/data/apache-flume-1.7.0-bin/lib/*:

javawriter.bootoptions=-Xmx512m -Xms32m -Djava.class.path=ggjava/ggjava.jar
```

7) vi custom-flume-rpc.properties
```
client.type=default
hosts=h1
## flume agent IP与端口
hosts.h1=localhost:41414
batch-size=100
connect-timeout=20000
request-timeout=20000
```

8) ./ggsci

9) GGSCI > edit params rflume
```
REPLICAT rflume

## dbo.def为源端数据库表的定义，需要在源端使用defgen工具生成后拷贝过来
SOURCEDEFS dirdef/dbo.def

-- Trail file for this example is located in "AdapterExamples/trail" directory
-- Command to add REPLICAT
-- add replicat rflume, exttrail AdapterExamples/trail/tr
-- Win平台下为 ggjava.dll
TARGETDB LIBFILE libggjava.so SET property=dirprm/flume.props
REPORTCOUNT EVERY 1 MINUTES, RATE
GROUPTRANSOPS 10000

MAP dbo.*, TARGET dbo.*;
```

### 启动

10) `export JAVA_HOME=/usr/jdk64/jdk1.8.0_111`

11) `export LD_LIBRARY_PATH=$JAVA_HOME/jre/lib/amd64:$JAVA_HOME/jre/lib/amd64/server:$JAVA_HOME/jre/lib/amd64/server/libjsig.so:$JAVA_HOME/jre/lib/amd64/server/libjvm.so` Win平台下为`libjvm.dll`和`libjsig.dll`，确保这两个文件存在

12) ./ggsci

13) GGSCI > start mgr

14) GGSCI > add replicat rflume, exttrail /data/ggsAdapters/dirdat/et, begin now

15) GGSCI > start rflume

### 诊断

1. GGSCI命令行中运行`info all`确保mgr与rflume都处于`RUNNING`状态；
2. 如果出错可查看错误日志`ggserr.log`，或`dirrpt/RFLUME.rpt`；
3. 确保dirdat目录下写入了etXXXXXXXXX文件，该文件由源端pump进程写入，存储了源端的增量数据；
