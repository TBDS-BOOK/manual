FTP导入HDFS
-----------

### 步骤（1）：新建工作流任务

将图中的“+”控件拖拽到画布中，搜索spark任务，选择spark任务，确认。

![](media/ee5da55d3111dc4e2e60268a7bd81b41.png)

### 步骤（2）：填写基本信息

在画布中找到新建任务，双击编辑

![](media/c706dcf4374d3589f6220de318272f23.png)

![](media/f1487dc58ff49ce79cfc2975097e5cd6.png)

### 步骤（3）：填写调度设置

![](media/28aaceb5a1d7131e6fc733a534f3a970.png)

步骤（4）：填写参数配置

![](media/569670575379ef369b4fb2f105d5da35.png)

1.  源服务器  
    源服务器是ftp
    服务器连接信息。[更多参数](https://tbds-book.gitbooks.io/tbds/workflow/services/readme.html)

2.  目标服务器  
    目标服务器是hdfs
    服务器连接信息。[更多参数](https://tbds-book.gitbooks.io/tbds/workflow/services/readme.html)

3.  源目录  
    指定将ftp 服务器哪些目录数据导入hdfs。  
    该参数不能直接到文件。请务必确保源目录用户有权限访问。  
    支持[时间隐式参数](https://tbds-book.gitbooks.io/tbds/workflow/workflow/more/implicitVariable.html)

4.  文件名正则表达式  
    支持java 正则表达规则， .\*
    为任意文件，支持[时间隐式参数](https://tbds-book.gitbooks.io/tbds/workflow/workflow/more/implicitVariable.html)

5.  是否遍历子目录  
    选择是，表示任务会遍历源目录下的子目录，遍历的深度由最大遍历层数决定。  
    选择否，表示不会遍历源目录下的子目录，最大遍历层数 不会生效。

6.  最大遍历层数  
    只有将是否遍历子目录设置为 是，该设置才会生效。当前目录为初始值为1。

7.  目标目录  
    ftp 数据导入hdfs的目的路径。  
    注意： 如果遍历子目录，系统会保留ftp 子目录结构。

#### 补充：编辑服务器配置

详情请查看<https://tbds-book.gitbooks.io/tbds/workflow/services/operation.html>

FTP:

选择服务器类型:ftp，进行FTP类型的服务器配置。

-   主机地址：FTP服务的连接IP地址，如：10.0.0.1；

-   端口：FTP的连接端口，默认：2222；

-   ftp用户名：连接FTP用户

-   ftp用户密码：连接FTP的用户所对应的密码；

**备注：** 选择创建FTP服务器配置时，默认会加载套件内安装的FTP服务的配置信息，如果新增非套件内的FTP服务器，则修改对应的连接信息即可。

![https://tbds-book.gitbooks.io/tbds/workflow/services/pictures/hive.png](media/3fa9801c3d616a8cd93ee4a8c90460a1.png)

HDFS

选择服务器类型:HDFS，进行HDFS类型的服务器配置。

-   namenode主机地址：默认获取集群内HDFS的namenode主机IP，若namenode为HA，则获取配置的文件空间，集群内默认是：hdfs://hdfsCluster

**备注：**

1.  选择创建HDFS服务器配置时，默认会加载套件内安装的HDFS服务的配置信息；

2.  服务器配置目前仅支持套件内的HDFS集群，不支持外部的HDFS集群；

![https://tbds-book.gitbooks.io/tbds/workflow/services/pictures/hdfs.png](media/317ac74a0be8a3a0989005d6e73b8896.png)

### 步骤（4）：保存和运行任务

![](media/5174651d95d224ecaa7194cb4cf5f30d.png)

### 步骤（5）：查看任务状态

![](media/79bcd92b2ca3cb72e71694a064c88d2b.png)

![](media/0e6cba1919415731984a5db504f4f22a.png)

![](media/cf62c291458b0f7822c67c7be1990a8e.png)

步骤（6）：在文件管理中查看结果

在运维中心-\>文件管理处，进入到所在的目录，查看结果。

![](media/f6898d0d8c1b489508ca1ee2ee71d26c.png)