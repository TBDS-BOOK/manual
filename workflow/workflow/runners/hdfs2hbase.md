hdfs导入hbase 


### 功能说明
实现HDFS中的数据写入hbase。  

### 其他说明


### 任务设置
#### 1. 基本信息  
参考 [基本信息设置](runnerBasicInfo.md)  
#### 2. 调度  
参考 [调度设置](runnerCycle.md)  
#### 3. 参数
HDFS导出hbase 参数设置只需要 hbase 配置信息。  
HDFS 连接信息使用的集群默认HDFS地址。  

###### 3.1 hbase配置
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
