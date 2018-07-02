hive 导出到DB(mysql)

### 功能说明
hive 查询记录导入关系型数据库（mysql）

### 其他说明
hive 查询使用，任务第一责任人连接（portal前台登陆用户）

### 任务设置
#### 1. 基本信息  
参考 [基本信息设置](/workflow/workflow/runnerBasicInfo.md)  
#### 2. 调度  
参考 [调度设置](/workflow/workflow/runnerCycle.md)  

#### 3. 参数
参数配置如下图  
![hive2mysql](/workflow/workflow/images/hive2mysql.png)

1. 源服务器  
待导入数据所在的 HIVE server  
更多信息参考 [服务器配置](/workflow/services/readme.md)

2. 目标服务器  
存储最终结果的 MYSQL server   
更多信息参考 [服务器配置](/workflow/services/readme.md)

3. 源db名称  
hive db

4. 源SQL  
hive sql 查询语句  

5. 源SQL查询结果字段数量  
对复杂的sql 查询，需要指定源sql查询结果的字段个数，简单查询可以不填。  

6. 分隔符  
创建hive 外表，以及读取临时hdfs 数据会用到

7. 目标表（mysql）  
mysql 表名，此处不要加db名称，在目标服务器已经指定了。  

8. 目标表列名  
目标表列名，跟hdfs临时数据字段保持一致。 

9. 脏数据阀值  
允许出错的百分比，20代表允许有20%的数据可以读或者写失败，0代表不允许有任何数据读写失败。读写各算一次失败。

11. 入库为空时任务处理  
无源文件或入库记录为0时,可以指定任务为成功或失败。   
选择成功，表示无源文件或入库记录为0时 ，任务成功，反之失败。  

12. 数据入库模式  
有两种模式可选，append和truncate  
append模式不会删除原有数据，所有重跑实例，可能会有重复数据。  
truncate 模式会删除原有数据。如果目标hive 表是分区表，则会删除数据时间对应的分区，如果hive 不是分区表，则会将整个hive表记录删除。

12. 是否分区  
是否分区是指mysql 表是否为分区表。  
    * 如果选择否，直接写入数据。  
    * 如果选择是，则需要进一步制定分区字段格式，目前只支持时间格式，通常任务调度周期和字段分区类型对应。比如任务调度周期是天，数据按天分区，分区字段选择P_${YYYYMMDD} 。在将数据写入mysql 中，执行的sql 语句中会嵌入partition 关键字。  

13. beeline输出格式  
连接的源服务器，对应的beeline 输出格式

14. 任务超时时间  
当前没有使用

15. 临时存储中间结果HDFS环境  
存放临时结果的hdfs 连接地址。

16. 数据临时存放目录  
存放临时结果的hdfs上的目录，默认在/user/${portalUser}/hm/ 目录下

11. 读并发度  
读hdfs 数据的并发线程数。

12. 写并发度  
数据写入mysql的并发线程数。 

### demo
如上图所示

### demo资源
