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
2. 输出目录
3. 命令行脚本
4. 命令行参数
5. 任务超时
6. 前置check命令
7. check 命令超时
8. 后置done命令


### demo
如上图所示

### demo资源
输出目录：/project/tbds_autotest/autotest/mr/out  
命令参数：hadoop-mapreduce-examples-2.6.0.2.2.0.0-2041.jar wordcount /project/tbds_autotest/autotest/hdfs2hive/* /project/tbds_autotest/autotest/mr/out  
命令行脚本：[demo下载](https://share.weiyun.com/e0dd52520c1f68f4f5663ea3cf0435bb)
