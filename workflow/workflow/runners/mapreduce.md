mapreduce 任务

### 功能说明
执行mapreduce任务。  

### 其他说明
任务执行者为任务第一个责任人（portal登录用户)

### 任务设置
#### 1. 基本信息  
参考 [基本信息设置](/workflow/workflow/runnerBasicInfo.md)  
#### 2. 调度  
参考 [调度设置](/workflow/workflow/runnerCycle.md)  

#### 3. 参数
参数配置如下：
![mr](/workflow/workflow/images/mapreduce.png)

1. 源服务器  
源服务器是ftp 服务器连接信息。[更多服务器配置](/workflow/services/readme.md)  

2. 输出目录  
指定mr 数据输出目录，系统会在执行任务前，清理输出目录。  

3. 执行脚本  
mr 可执行脚本,可以包括多个文件，最终压缩为zip 

4. 执行脚本参数  
执行mr 需要用到的参数，通常格式为 jar名 函数入口 参数  
比如： hadoop-mapreduce-examples-2.6.0.2.2.0.0-2041.jar wordcount args1 args2   
**这里不需要添加hadoop 安装目录**

5. 任务超时时间  
执行hadoop jar 命令超时时间，超过设置的时间，任务会被终止，实例失败。

6. 前置check命令  
执行mr 之前，用户可以执行前置命令，比如判断hdfs 数据目录是否存在，或者执行一些前置shell 命令（执行shell 脚本），可以输入多个前置命令。  

7. 前置check命令超时时间  
前置check 命令如果存在多个，每个前置命令执行时间不超过5min,总执行时间不超过该设置值（min）
 
8. 后置done命令  
任务执行后的done命令,在hdfs中touchz一个文件或执行清理脚本。对在hdfs中touch 一个文件，采用“done hdfsdir”; 对于清理脚本，采用“ ***.sh  argv”


### demo
如上图所示

### demo资源
输出目录：/project/tbds_autotest/autotest/mr/out  
命令参数：hadoop-mapreduce-examples-2.6.0.2.2.0.0-2041.jar wordcount /project/tbds_autotest/autotest/hdfs2hive/* /project/tbds_autotest/autotest/mr/out  
命令行脚本：[demo下载](https://share.weiyun.com/e0dd52520c1f68f4f5663ea3cf0435bb)
