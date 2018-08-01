hdfs导出hive（75）

### 功能说明
HDFS数据导出HIVE表

### 其他说明
读入HDFS和写HIVE执行者为任务第一个责任人（portal登录用户)

### 任务设置
#### 1. 基本信息  
参考 [基本信息设置](/workflow/workflow/runnerBasicInfo.md)  
#### 2. 调度  
参考 [调度设置](/workflow/workflow/runnerCycle.md)  

#### 3. 参数
任务参数配置如下图：
![hdfs2hive](/workflow/workflow/images/hdfs2hive1.png)

1. 源服务器  
待导入数据所在的 HDFS server  
更多信息参考 [服务器配置](/workflow/services/readme.md)

2. 目标服务器  
存储最终结果的 HIVE server   
更多信息参考 [服务器配置](/workflow/services/readme.md)

3. 源目录  
待导出数据所在HDFS 目录  
支持[时间隐式参数](/workflow/workflow/more/implicitVariable.md)

4. 源文件名  
默认为 * ,支持linux 格式的通配符  
支持[时间隐式参数](/workflow/workflow/more/implicitVariable.md)

5. 源文件字符集  
指定hdfs server 存放数据的编码格式。   
该设置在创建hive 外表时用于
```set serdeproperties ('charset'='编码');'```

6. 源文件列名  
源文件的栏位名称，以英文逗号分割（结尾不能是逗号）,必须保证列数和文件内容一致.
创建hive外表（临时表）所用表列名  

7. 字段映射关系  
hive表列名,以英文逗号分隔,表示的列的内容顺序,需和DB列字段保持一致。决定从临时表往目的表里写的字段顺序。日期和常量需要用中括号包起来，例如：[${YYYYMMDD}], [\'test\']

8. DB名称  
待写入数据的hive db

9. 目标表名  
待写入数据的hive 表名  
如果目标hive 表有分区字段,字段值类型最好是bigint（分区格式为YYYYMM也可以是int类型）,分区类型为list

10. 分区格式  
指定hive 表分区格式。
通常和任务调度周期对应，如任务调度周期为天，则分区格式为${YYYYMMDD}。  
如果hive 目标表有分区字段（分区字段值和分区格式必须对应）则一定要分区格式，若无分区字段，分区格式设置无效。

11. 入库为空时任务处理  
无源文件或入库记录为0时,可以指定任务为成功或失败。   
选择成功，表示无源文件或入库记录为0时 ，任务成功，反之失败。  

12. 数据入库模式  
有两种模式可选，append和truncate  
append模式不会删除原有数据，重跑实例，可能会有重复数据。  
truncate 模式会删除原有数据。如果目标hive 表是分区表，则会删除数据时间对应的分区，如果hive 不是分区表，则会将整个hive表记录删除。如果hive 表是分区表，但是对应的分区值不是指定的分区格式，则清理分区不会成功，原数据将会被保存，重跑实例将会出现重复数据。

13. map个数  
暂时没有启用

14. TDW参数  
暂时没有启用

15. 任务超时（分钟）  
暂时没有启用

### demo
如上图所示  

### demo资源
