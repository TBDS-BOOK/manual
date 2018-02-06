提交自定义(strom)topology任务

### 功能说明
执行一个storm topology 任务。

### 其他说明
多集群版本支持提交到不同storm 集群，其他版本不支持。

### 任务设置
#### 1. 基本信息  
参考 [基本信息设置](/workflow/workflow/runnerBasicInfo.md)  
#### 2. 调度  
参考 [调度设置](/workflow/workflow/runnerCycle.md)  

#### 3. 参数
任务配置参考：  
![自定义topology](/workflow/workflow/images/custormTopology.png)

1. Topology名称  
提交到storm 集群topology名称.建议从参数中获取。

2. 程序包   
topology可执行jar,当前环境建议使用如下依赖
```
        <dependency>
            <groupId>org.apache.storm</groupId>
            <artifactId>storm-core</artifactId>
            <version>0.9.6</version>
            <scope>provided</scope>
        </dependency>
```

3. Topology类  
可执行jar 对应的函数入口。  

4. 参数  
根据提交的可执行jar 自行确定。

### demo
如上图所示.

### demo资源
随机的输出，[资源下载](https://share.weiyun.com/67ce256ae080d12396761489aad9e996)
