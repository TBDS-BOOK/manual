kafka接入hbase 


### 功能说明
实现从kafka消费数据，并将数据写入hbase。  

### 其他说明
该任务是一个storm toplogy 实时任务。  
使用前请确保 jstorm 已经安装，且运行状态正常。  

### 任务设置
#### 1. 基本信息  
参考 [基本信息设置](runnerBasicInfo.md)  
#### 2. 调度  
参考 [调度设置](runnerCycle.md)  
#### 3. 参数
kafka 接入hbase 参数设置分成三个部分：kafka配置信息，hbase 配置信息，topology 配置信息。
###### 3.1 kafka 配置信息  
1. 消息中间件主题  
kafka topic 在创建任务前，需要确保topic已经存在，系统不会创建对应的topic。

2. 消息中间件消费组  
kafka消费组 可以随意指定，但需要确保消费组全局唯一。

3. kafka集群broker 列表     
kafka broker 列表，格式为 ip1:port,ip2:port   
ip地址为Kafka Broker 服务所在节点ip   
port  kafka 开放给client 连接端口，可以参考kafka broker 服务配置。 

###### 3.2 hbase配置
1. HBase表名  
格式为dbName:tableName ,如：hbase_autotest:auto_kafka_hbase 
2. HBase行主键  
每次插入一条记录，会根据配置的行主键信息生成行主键，格式${topic}:#{abc}:${1}。  
topic 为kafka 主题，  
#{}内容（ 如：abc）为随意指定的一个固定值，可以方便查询。  
最后一个参数${n}为 消息被分隔符切分之后，对应的数值，n 是整数。  
假如现在从kafka 集群 主题为firstTopic 消费了一条内容为“a,b,c,d”的记录，设置分割符为 “,” ,行主键设置是${topic}:#{ABC}:${1}。则这条记录插入hbase ，生成的主键为 主题为firstTopicABCa。  
3. HBase列簇  
列簇1名:列1名|列1值,列2名|列2值;列簇2名:列3名|列3值,列4名|列4值。  
比如：s1:#{name}|${0};s2:#{age}|${1}  
${n} 其中n 为整数，为消费消息被切分之后，组成的数组所在索引对应的值。  

4. HBase字段描述   
字段名1,字段类型1,字段备注1:字段名2,字段类型2,字段备注2，比如：name,STRING,desc:age,STRING,desc。  
当前只支持STRING类型。  

5. 分隔符  
切分消息使用的分隔符。 

6. Zookeeper根节点  
为hbase 在zk 上的根目录, 默认为：/hbase-unsecure   

###### 3.3 toplogy配置
1. Work进程数  
该任务所占storm 集群的槽位，默认为1.  

2. Spout线程数   
设定spout 启动的线程数  

2. Bolt线程数   
设定bolt 启动线程数  

4. kafka消费线程数   
设定kafka 消费线程数  


### demo  
消息中间件主题： kafka_export_hbase    
消息中间件消费组：kafka_export_hbase   
kafka集群broker list：10.254.83.70:6668   
HBase表名：hbase_autotest:auto_kafka_hbase  
HBase行主键：${topic}:#{cd}:${1}  
HBase列簇： s1:#{name}|${0};s2:#{age}|${1}  
HBase字段描述：name,STRING,desc:age,STRING,desc  
分隔符：,
Zookeeper根节点:/hbase-unsecure  
Work进程数: 1  
Spout线程数: 1  
Bolt线程数: 1
kafka消费线程数：1

### demo资源
