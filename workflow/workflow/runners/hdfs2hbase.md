hdfs导入hbase


### 功能说明
实现HDFS中的数据写入hbase。  

### 其他说明

### 任务设置
#### 1. 基本信息  
参考 [基本信息设置](/workflow/workflow/runnerBasicInfo.md)  

#### 2. 调度  
参考 [调度设置](/workflow/workflow/runnerCycle.md)  

#### 3. 参数
任务配置如下图所示：
![hdfs2hbase](/workflow/workflow/images/hdfs2hbase1.png)

HDFS导出hbase 参数设置只需要 hbase 配置信息。  
HDFS 连接信息使用的集群默认HDFS地址。  

###### 3.1 HDFS配置
1. HDFS目录  
待写入hbase的hdfs 文件所在目录（或文件）。**不支持目录不存在**   
该参数直接作为job inputpath 参数（替换here）  
FileInputFormat.addInputPath(job, here);

2. 文本文件字段个数  
hdfs存放的数据文件，每行记录被分隔符切分的个数。 
表示对应切分的数据字段个数  
实际使用位置为map 类map 方法，对应value 进行切分后的个数判断。  

3. 文本文件字段分隔符  
指定hdfs存放的数据文件，每行记录被切分为字段值的分隔符。  
如果是 . 、 | 和 * 等转义字符，必须得加 \\
如果是 多个分隔符，可以用 | 作为连字符。

###### 3.2 hbase 配置  
1. hbase表名  
格式为dbName:tableName ,如：hbase_autotest:auto_kafka_hbase  

2. 列列表  
使用逗号分隔，用来表示hbase 需要写入数据的字段。  
格式为：列簇1名:列1名,列簇1名:列2名,列簇2名:列3名,列簇2名:列4值,列簇3名:列5值。

3. 行key规则  
指定行主键的生成方式：  
a. RANDOM(n)表示生成生成一个随机长度n个数字作为hbase的row key.  
b. REVERSE(n) 表示使用hdfs记录中被切分后的第n个部分反转值作为主键.  
c. n 表示使用hdfs记录中被切分后的第n个部分作为主键.   
d. ```'__'``` 表示使用常量```__```作为row key.   
e. PADLEFT(n;m;str) 表示使用hdfs记录中被切分后的第n个部分，作为row key 最右边的部分，余下的m-n个部分由str组成。比如PADLEFT(0;5;c) 表示row key 最右边的部分为hdfs记录被切分的第一部分内容，余下不足的部分由字符串c 填充。  
f. PADRIGHT(n;m;a) 表示使用hdfs记录中被切分后的第n个部分，作为row key 最左边的部分，余下的m-n个部分由str填充。比如PADRIGHT(0;5;c) 表示row key 最左边的部分为hdfs记录被切分后的第一个内容，余下不足的部分由字符串c 填充。  
——     
**PS:** 当然你也可以使用以上几者的组合比如``` RANDOM(5),'__',0``` ，主键将是随机的五个数字加```__```加hdfs记录被切分后的第一个部分作为rowkey

4. 列规则  
使用逗号分隔，跟列列表对应。用来表示对应的列名被写入的数值为hdfs记录被切分生成数组的下标。  
如hdfs 其中一行的为: hello,word,i,come,from,20171216120000   
列列表为 f1:c1,f1:c2,f1:c3,f2:c4,f3:c5,f4:c6  
列规则为 0,1,2,3,4,5  
则表示 hello 将被写入 列簇为f1 列名为c1的位置

5. 时间戳所在列索引  
为-1时，使用HConstants.LATEST_TIMESTAMP  
其他情况下，将使用 列规则对应位置的数值为下标的被切分的数组的数值。该数值必须能够被转为long 类型，否则该记录写入hbase失败。  
例如：   
如hdfs 其中一行的为: hello,word,i,come,from,20171216120000   
列列表为 f1:c1,f1:c2,f1:c3,f2:c4,f3:c5,f4:c6  
列规则为 1,1,2,3,4,5  
时间戳所在列索引为5 ，则表示hdfs上的该记录对应的时间戳为20171216120000  
如果时间戳所在列索引为0 ，表示将word 作为时间戳，那么记录写入hbase则会失败。

6. zookeeper连接地址  
hbase 所在zk 集群（可以不加2181，也可以参考8080端口 hbase配置的hbase.zookeeper.quorum 属性值）    

7. hbase在zk上的根路径  
为hbase 在zk 上的根目录, 默认为：/hbase-unsecure

###### 3.3 其他配置项  
1. 成功记录数占比  
写入成功的记录数超过该设置的值，任务成功，否则失败。 必须为正整数。填60表示，百分之60记录数写入hbase 成功，任务实例才执行成功。填100表示，只要有一条记录写入hbase 失败，则任务实例失败。   

### demo  
更多参考上图。   
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


### 其他说明
目前hdfs2hbase 只支持text 文本，不支持其他如csv,orc等格式。  
