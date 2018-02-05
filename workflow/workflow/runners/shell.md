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

ruguo

### demo
执行一个简单的while 循环  
执行包路径：[zip 下载](http://dl.download.csdn.net/down11/20180205/441ee758c6d4e3f90395d998076900dd.zip?response-content-disposition=attachment%3Bfilename%3D%22testovertime.zip%22&OSSAccessKeyId=9q6nvzoJGowBj4q1&Expires=1517813646&Signature=aET4puXUfTutmA6VAcMzaVEWJyo%3D&user=wuyinxian&sourceid=10240523&sourcescore=2&isvip=0)  
shell执行命令： ./testovertime.sh  
