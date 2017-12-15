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


String tableName = getExtPropValueWithDefault("hbase.table", "");// hbase表名
String hbaseColumnList = getExtPropValueWithDefault("hbase.column.list", "");//列列表
String hbaseRowRules = getExtPropValueWithDefault("hbase.row.rule", "");//行key规则
String tdwColumnList = getExtPropValueWithDefault("tdw.column.list", "");//列规则
String tsIndex = getExtPropValueWithDefault("hbase.ts.index", "");//时间戳所在列索引
String dataPath = getExtPropValueWithDefault("dataPath", "");//HDFS路径
String fieldCount = getExtPropValueWithDefault("tdw.column.number", "");//HDFS存放数据记录字段个数
String fieldDelim = getExtPropValueWithDefault("tdw.column.fieldDelim", "");//HDFS存放数据字段之间分隔符
String hbase_root = getExtPropValueWithDefault("hbase_root","/hbase-unsecure");//hbase在zk上的根路径
        
        
1. hbase表名  
格式为dbName:tableName ,如：hbase_autotest:auto_kafka_hbase 
2. 列列表 
列簇1名:列1名,列2名;列簇2名:列3名|列3值,列4名|列4值。
3. 行key规则 

4. 列规则

5. 时间戳所在列索引  
6. HDFS路径  
7. HDFS存放数据记录字段个数  
8. HDFS存放数据字段之间分隔符  
切分消息使用的分隔符。   
9. hbase在zk上的根路径  
为hbase 在zk 上的根目录, 默认为：/hbase-unsecure  

10. zookeeper连接地址  

11. 成功记录数占比  

  

### demo   
HBase表名：hbase_autotest:auto_kafka_hbase  
列列表 ：s1:id,s2:name,s2:age  
行key规则：RANDOM(5)  
列规则：1,2,3  
时间戳所在列索引：0  
HDFS路径：/project/tbds_autotest/autotest/hdfs2hbase_src  
HDFS存放数据记录字段个数：4  
HDFS存放数据字段之间分隔符：,  
hbase在zk上的根路径：/hbase-unsecure  
zookeeper连接地址：10.151.139.109,10.254.83.14,10.254.83.70  
成功记录数占比：90

### demo资源
