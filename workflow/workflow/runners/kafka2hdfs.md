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

Work进程数: 1  
Spout线程数: 1  
Bolt线程数: 1
kafka消费线程数：1

### demo资源
