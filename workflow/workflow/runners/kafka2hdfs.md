kafka接入HDFS 


### 功能说明
实现从kafka消费数据，并将数据写入hdfs。  

### 其他说明
该任务是一个storm toplogy 实时任务。  
使用前请确保 jstorm 已经安装，且运行状态正常。  

### 任务设置
#### 1. 基本信息  
参考 [基本信息设置](/workflow/workflow/runnerBasicInfo.md)  
#### 2. 调度  
参考 [调度设置](/workflow/workflow/runnerCycle.md)  
#### 3. 参数
参数配置如下图所示：
![kafka2hdfs](/workflow/workflow/images/kafka2hdfs.png)

kafka 接入hbase 参数设置分成三个部分：kafka 连接信息，hdfs 连接信息，topology 配置信息。
###### 3.1 kafka 配置信息  
1. 消息中间件主题  
kafka topic 在创建任务前，需要确保topic已经存在，系统不会创建对应的topic。

2. 消息中间件消费组  
kafka消费组 可以随意指定，但需要确保消费组全局唯一。

3. kafka集群broker 列表     
kafka broker 列表，格式为 ip1:port,ip2:port   
ip地址为Kafka Broker 服务所在节点ip   
port  kafka 开放给client 连接端口，可以参考kafka broker 服务配置。 

###### 3.2 hdfs配置
1. 出库HDFS目录  
数据落地目录

2. HDFS地址  
数据落地所属的HDFS环境（连接地址）

3. 文件最大落地大小  
单位为：Byte 。如果文件最大落地大小设置的值小于64k，会使用64k ，如果设置的值大于64k，将使用实际设置的值。  
实际落地hdfs 的文件大小会稍微小于设置的文件最大落地大小。
  
4. 文件最小落地周期  
单位为小时，必须整数。  
 如果时间周期到了，系统会将缓存的数据保存到hdfs ,这是时候落地的数据可能少于64k ，也可能稍微大于设置的文件最小落地大小。  

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
出库HDFS目录：/project/tbds_autotest/autotest/kafka_export   
HDFS地址 ：hdfsCluster  
文件最大落地大小：1024000   
文件最小落地周期 ：24  
Work进程数: 1  
Spout线程数: 1  
Bolt线程数: 1
kafka消费线程数：1

### demo资源


### 问题定位方式
通过nimbus ，或者storm ui 获取topology日志
1. 确认从kafka 获取内容成功  
receive data ,the size 

2. 确认获取对应的配置成功（不能为0）  
spout receive message from kafka ,the data size   
这里需要进一步确认(如果有改记录那就不会往下走)：DataInterfaceKafkaSout.java:1030): onlineconfig is null  

3. 接近写成功会出现  
desc=[name=
 
