执行shell脚本

### 功能说明
执行一个shell脚本，系统不会对shell进行语法检查。

### 其他说明
shell脚本执行者为任务第一个责任人（portal登录用户)

### 任务设置
#### 1. 基本信息  
参考 [基本信息设置](/workflow/workflow/runnerBasicInfo.md)  
#### 2. 调度  
参考 [调度设置](/workflow/workflow/runnerCycle.md)  

#### 3. 参数
参数设置如下图所示：
![shell 参数设置](/workflow/workflow/images/shell.jpg)
<br>
1. 执行包路径  
将需要执行的shell 文件，压缩为一个zip包，点击‘上传脚本’ 将zip 上传到tbds

2. shell 命令  
执行上传shell 文件的shell 命令。可以带运行参数

### demo
执行一个简单的while 循环  
执行包路径：[简单while循环](https://share.weiyun.com/f53b310ad6ffde819830d522a8dd51de)  
&nbsp;&nbsp;如果要测试 执行失败，可以下载[任务失败](https://share.weiyun.com/11c232afeb32c12ccbefadd9a520526f)  
shell执行命令：  
&nbsp;&nbsp;./testovertime.sh  
&nbsp;&nbsp;任务失败 使用./testerror.sh 
