提交spark 任务

### 功能说明
向tbds 集群提交spark任务。 

### 其他说明
spark执行者为任务第一个责任人（portal登录用户)。  
该任务同时支持 python sql  
所有spark 任务都在yarn 上执行。

### 任务设置
#### 1. 基本信息  
参考 [基本信息设置](/workflow/workflow/runnerBasicInfo.md)  
#### 2. 调度  
参考 [调度设置](/workflow/workflow/runnerCycle.md)  

#### 3. 参数
参数配置如下图所示：
![spark](/workflow/workflow/images/spark.png)

1. 程序包ftp路径  
上传待执行文件，必须是一个zip 包  
待执行的python sql 或者scala,java 执行jar 都需要压缩为zip   

2. 参数  
执行参数  
**注意：** 不要添加submit，不需要指定执行执行者，不需要指定yarn 队列。

### demo
如上图所示

### demo资源
1. [zip 下载](https://share.weiyun.com/8f66f045d7b7fb7c71b075b303481f75)
2. 执行参数  
--class org.apache.spark.examples.streaming.NetworkWordCount /usr/hdp/2.2.0.0-2041/spark/examples/jars/spark-examples_2.10-2.1.0.jar localhost 9999
