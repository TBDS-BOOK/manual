FTP导入HDFS

### 功能说明
将源FTP 对应目录数据导入到HDFS。

### 其他说明
数据落地HDFS目录拥有者为为任务第一个责任人（portal登录用户)

### 任务设置
#### 1. 基本信息  
参考 [基本信息设置](/workflow/workflow/runnerBasicInfo.md)  

#### 2. 调度  
参考 [调度设置](/workflow/workflow/runnerCycle.md)

#### 3. 参数
任务参数配置如下图所示：
![ftp2hdfs](/workflow/workflow/images/ftp.png)
__ps:__ 红框部分为需要关注参数  
1. 源服务器  
源目标服务器是ftp 服务器连接信息。[更多参数](/workflow/services/readme.md)  

2. 目标服务器  
目标服务器是hdfs 服务器连接信息。[更多参数](/workflow/services/readme.md)  

3. 源目录  
指定将ftp 服务器哪些目录数据导入hdfs。  
该参数不能直接到文件。请务必确保源目录用户有权限访问。  
支持[时间隐式参数](/workflow/more/implicitVariable.md)

4. 文件名正则表达式  
支持java 正则表达规则， .* 为任意文件，支持[时间隐式参数](/workflow/more/implicitVariable.md)  

5. 是否遍历子目录  
选择是，表示任务会遍历源目录下的子目录，遍历的深度由最大遍历层数决定。  
选择否，表示不会遍历源目录下的子目录，最大遍历层数 不会生效。  

6. 最大遍历层数  
只有将是否遍历子目录设置为 是，该设置才会生效。当前目录为初始值为1。  

7. 目标目录  
ftp 数据导入hdfs的目的路径。  
注意：  如果遍历子目录，系统会保留ftp 目录结构。

### demo
如上图所示
