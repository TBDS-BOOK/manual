### 对接实时接入的hdfs导出hive 任务 常见问题定位和处理方式

### 1. 任务实例日志不新增
**问题表现**  
>任务实例最长每十分都会新的日志。使用当前时间和日志末尾处的日志打印时间比较，如果时间差异大，则出现这个问题。  

**可能原因：**  
1. 多个进程操作同一个hive 表。  
2. zookeeper 连接失败。   

**定位方式**  
1. [查看任务实例运行所在节点](/workflow/workflow/qa/common_operation.md)
2. 登陆对应机器节点，使用hdfs 用户，切到/usr/local/lhotse_runners 目录  
3. 执行ps -axu|grep realtaskid|grep -v grep 
4. 查看输出结果，如果有两个或两个以上的进程,则有问题。
5. 如果输出中有写入和查询的进程同时存在，则是hive 表被锁住。

### 2. 实时接入的数据没有落地到hive
**问题表现**  
> hdfs2hive_tdsort 任务运行正常，数据通过实时接入写入hdfs,但是查询hive表没有对应的记录

**可能原因：**  
1. 数据没有落地到hdfs
2. 数据落地hdfs但是没有写入到tdsort_db.tdsort_upload_info 表
3. 数据有落地，数据库中有记录，写入hive失败
   
**定位方式** 
1. 根据业务接入数据时间以及接入的topic_name和tid 确认hdfs 有数据  
/tdbank/data/hippo/{topic_name}/{tid}/{时间时间}/{接入时间}/in/  
例如：/tdbank/data/hippo/topic_hive/TEST_BLUES_TBLX/20171129/2017112906/in/
  
2. 根据topic_name和tid 查询tdsort_db，确认对应业务时间对应的记录存在。  
a. 查询数据库记录
```
select * from tdsort_upload_info where topic_name="{topic_name}" and tid="{tid}" \G  
```
例如：
```  
select * from tdsort_upload_info where topic_name="topic_hive" and tid="TEST_BLUES_TBLX" \G
```
b. 查询对应记录，如果有error 记录，则需要特别注意
```
select flag,count(1)  from tdsort_upload_info where topic_name="topic_hive" and tid="TEST_BLUES_TBLX" group by flag\G
```

3. 如果tdsort_upload_info记录中有pend 记录，但是没有在任务实例日志体现。通过如下记录查看
```
select m.project_id as projectid, m.user_name as portaluser, m.topic_name as topicName ,m.tid as tid ,m.db_name as dbName,m.table_name as tableName ,m.fields as fields ,m.time_partion as partition ,m.spliter as separate,i.partition_value as partitionValue ,m.hdfs_url as hdfsUrl ,m.hive_url as hiveUrl,i.path as path ,i.id  as upload_infoID from tdsort_upload_info i,hdfs2hive_metainfo m where i.topic_name = m.topic_name and i.tid=m.tid and i.flag = 'pend' order by dbName ,tableName, partitionValue limit 100
```

#### 000. 任务实例日志不新增
**问题表现**  
>问题

**可能原因：**  
1. 原因。  

**定位方式** 
1. 首先。  
