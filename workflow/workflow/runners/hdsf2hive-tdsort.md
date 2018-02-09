hdfs2hive-tdsort 

### 功能说明
支持实时接入，将实时接入写入hdfs 的数据，写入hive.  

### 其他说明
**务必确保该任务类型全局只有一个。**  

### 任务设置
#### 1. 基本信息  
参考 [基本信息设置](/workflow/workflow/runnerBasicInfo.md)  
#### 2. 调度  
**只能设置为一次性非周期。**  
参考 [调度设置](/workflow/workflow/runnerCycle.md)  

#### 3. 参数
参数设置参考
![hdfs2hive-tdsort](/workflow/workflow/images/hdfs2hive-datahub.png) 

1. 数据入库模式   
有两种模式可选，append和truncate  
append模式不会删除原有数据，所有重跑实例，可能会有重复数据。  
truncate 模式会删除原有数据。如果目标hive 表是分区表，则会删除数据时间对应的分区，如果hive 不是分区表，则会将整个hive表记录删除。  
默认（推荐）使用append 模式，英文实时接入的数据可能存在多次落地的情况。  

2. 轮询周期  
如果没有数据待导入，任务休眠时间。时间为秒。 

### demo
如上图所示

### demo资源
