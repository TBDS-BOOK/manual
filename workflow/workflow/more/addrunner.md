**概要**  
在tbds集群以外的节点安装runner服务，需要用到该文档做参考。 

安装步骤包括：  
1. 添加yum源  
2. 添加hdfs 用户  
3. 安装lhotse runner服务  
4. 安装httpd服务  
5. 修改httpd配置  
6. 修改lhotse runner配置  
7. 修改lhotse base设置（该步骤以下，只针对升级系统适用）  
8. 更新metadb  
9. 重启lhotse base  
10. 重启lhotse runner  
11. 更新lhotse service  

### 一、添加yum 源
假设需要安装runner服务的节点是10.0.0.1。  
__ps:__ 同时请确保安装前已经在yum 源更新了lhotse-runners*.rpm   
1. 进入10.0.0.1 节点，并切到root 用户
2. 编辑 /etc/yum.repos.d/TBDS.repo文件
3. 在打开的文件中添加如下内容：  
```
[TBDS]  
name=TBDS  
baseurl=http://10.151.136.14/tbds-mirror-4.0.3.1-docker/  
path=/  
enabled=1  
gpgcheck=0
```

4. 更新本地缓存  
yum clean all

### 二、添加用户
runner服务 依赖hdfs 用户,并需要将hdfs 用户添加到sudo 组
1. 添加hdfs  
```
useradd hdfs
```

2. 添加hdfs到sudo 组  
编辑 /etc/sudoers 文件 ，在最后一行添加  
```
hdfs ALL=(ALL) NOPASSWD:ALL
```

### 三、安装runner服务
root 用户操作  
yum install -y lhotse-runners

### 四、安装httpd服务
root 用户操作  
yum install -y httpd

### 五、修改httpd配置  
**使用hdfs用户做如下操作**  
1. 确认文件 /etc/httpd/conf/httpd.conf 包括  
ps:如果不包括，添加该行  
```
IncludeOptional conf.d/*.conf
```

2. 编辑文件 /etc/httpd/conf.d/runner.conf（不存在新建）  
插入如下内容  
```
Listen lhotse-runner 配置是 cgi.port 属性值    
ScriptAlias /runner/ /usr/local/lhotse_runners/getlog/  
```
例子如下：  
```
Listen 56986    
ScriptAlias /runner/ /usr/local/lhotse_runners/getlog/ 
``` 
3. 修改httpd 项目目录权限  
```
chown -R hdfs:hadoop /etc/httpd/logs/;  
chown -R hdfs:hadoop /usr/sbin/httpd;  
chown -R hdfs:hadoop /run/httpd  
```

### 六、修改runner配置  
**使用hdfs用户做如下操作**  
切到 /usr/local/lhotse_runners/cfg 目录，编辑如下文件
1. discover.properties   
将discovery.service.zk.conn.str 属性修改为tbds 集群中 zk host name list  
其他属性不变<br>
<br>**例子如下：**   
>discovery.service.zk.conn.str=tbds-10-151-140-233:2181,tbds-10-254-100-141:2181,tbds-10-254-99-28:2181   
discovery.service.client.fixedtime.sync.interval.millis=60000
discovery.exception.feedback.blacklist.policy.checktime.seconds=60
discovery.exception.feedback.blacklist.policy.checkcount.threshoud=3
discovery.service.zk.conn.timeout.millis=1200000
discovery.service.zk.session.timeout.millis=600000
discovery.service.zk.conn.str=tbds-10-151-140-233:2181,tbds-10-254-100-141:2181,tbds-10-254-99-28:2181
discovery.cluster.name=tdw
  
2. lhotse_base.properties  
编辑该文件 修改如下内容：<br>  
```
base_ip = taskscheduler hostname  
base_port = taskscheduler 对应 lhotse.base.port 属性值  
ftp_user=登陆用户，默认是 hdfs  
ftp_password= 密码默认是 ftp@Tbds.com  
ftp_path=默认是根路径 /  
doAndRedo_url=默认值http://lhotse-yh-webCgi.tencent-distribute.com:8081/LService/RedoAndDoState  
type.list=37,38,39,66,67,68,69,70,72,75,92,98,99,100,104,105,106,118,119,120  
discovery.service.zk.conn.str=tbds 集群 zk host:port 集合  
discovery.cluster.name=默认值 tdw  
```
<br>**例子如下：**  
>base_ip = tbds-10-254-99-18  
base_port = 9930  
ftp_user=hdfs  
ftp_password=ftp@Tbds.com  
ftp_path=/  
doAndRedo_url=http://lhotse-yh-webCgi.tencent-distribute.com:8081/LService/RedoAndDoState  
type.list=37,38,39,66,67,68,69,70,72,75,92,98,99,100,104,105,106,118,119,120  
discovery.service.zk.conn.str=tbds-10-151-140-233:2181,tbds-10-254-100-141:2181,tbds-10-254-99-28:2181  
discovery.cluster.name=tdw  

---
**该文档接下来的部分只对在原集群上升级系统适用，对于新安装的环境不适用**  

*ps:*  tdbank  给江苏消防的环境，需要执行下面的步骤。  

### 七、 修改base 设置  
1. 添加sql server 导入hive 任务类型    
登陆portal(或者8080端口)修改lhots-runner 配置type.list属性，在其中添加106 任务类型,并保存。    
![添加新任务类型](https://picabstract-preview-ftn.weiyun.com:8443/ftn_pic_abs_v2/0317fd959cc95c7d20198aa2443b77a1c15a11fa3c77eb0b24bf9675441ed24e5a2d6d6c7459f64aa6df4572e589786b?pictype=scale&from=30113&version=2.0.0.2&uin=821244074&fname=1508908165%281%29.jpg&size=1024) 

### 八、 更新metadb  
1. 连接metadb  
通过portal(或8080端口查询)METADATA master 节点ip  
mysql -hmetaDataMasterIP -ulhotse -plhotse@Tbds.com lhotse_open;  

2. 插入表数据  
a. 插入记录到lb_task_type表  
```sql  
replace into `lb_task_type` VALUES
(106,'SQL SERVER导入HIVE','数据接入','admin','2013-12-18 16:01:23',0,10,720,10,48,15,5,10,'sqlServer','hive','<parameters><parameter><name>hadoopPath_GAIA_AD</name><value>/usr/hdp/current/hadoop-client/bin/hadoop</value></parameter><parameter><name>plcProgram</name><value>/usr/bin/beeline</value></parameter><parameter><name>normal_hadoop</name><value>/usr/hdp/current/hadoop-client/bin/hadoop</value></parameter><parameter><name>im_hadoop</name><value>/usr/hdp/current/hadoop-client/bin/hadoop</value></parameter><parameter><name>ha_hadoop</name><value>/usr/hdp/current/hadoop-client/bin/hadoop</value></parameter><parameter><name>cdh3_hadoop</name><value>/usr/hdp/current/hadoop-client/bin/hadoop</value></parameter><parameter><name>externalTableDb</name><value>tdw_inter_db</value></parameter><parameter><name>removeDataFileRetry</name><value>10</value></parameter><parameter><name>hadoopPath_cdh3</name><value>/usr/bin/hadoop</value></parameter><parameter><name>dx_temp_path</name><value>/usr/local/lhotse_runners/DX/tmp/</value></parameter><parameter><name>dx_home</name><value>/usr/local/lhotse_runners/DX/</value></parameter><parameter><name>dx_template_db_to_hdfs</name><value>/usr/local/lhotse_runners/DX/jobs/DB2HDFSTemplate.xml</value></parameter></parameters>',0,'priority;tries;cycleunit;redo;do;rundate;createtime',2,'SqlServerToHiveRunner.jar',1,5000,1);
```
b. 插入记录到lb_task_type_ext表  
```sql  
replace into `lb_task_type_ext` VALUES
(106,'sql','Select SQL',1,1,'支持通配符，例：select * from qqshow_${1, 99} where p_date=${YYYYMMDD}; ',11,'textarea','string','','',1,0,0,1,'',1,''),
(106, 'databaseName', '目标DB名', 1, 1, 'DB名称', 12, 'text-single', 'string', NULL, NULL, '1', '0', '0', '1', NULL, '0', NULL),
(106, 'tableName', '目标表名', 1, 1, '入库数据表名', 13, 'text-single', 'string', NULL, NULL, '1', '0', '0', '1', NULL, '0', NULL),
(106,'charset.encoding','导出文件编码',1,1,'UTF-8或GBK',14,'text-single','string','UTF-8','',1,0,0,1,'',1,''),
(106,'separator','分隔符',1,1,'字段间分隔符',5,'select-option','string','9','9,124,44,34,42,32,59',1,0,0,1,'TAB键,竖线(|),逗号,双引号(\"),星号(*),空格,分号(;)',1,''),
(106, 'loadMode', '数据入库模式', 1, 1, 'APPEND可能出现数据重复，TRUNCATE可能出现整表删除（正常是分区级别操作）', 6, 'select-option', 'string', 'TRUNCATE', 'TRUNCATE,APPEND', '1', '0', '0', '1', 'TRUNCATE,APPEND', '0', NULL),
(106, 'partitionType', '分区格式', 1, 1, '分区格式', 7, 'select-option', 'string', '', 'N,P_${YYYYMM},P_${YYYYMMDD},P_${YYYYMMDDHH}', '0', '0', '0', '1', '无,P_${YYYYMM},P_${YYYYMMDD},P_${YYYYMMDDHH}', '0', NULL),
(106,'output_prefix','beeline输出格式',1,1,'目前可知的是有Output:和outputColumnNames:两种类型',84,'text-single','string','Output:','',0,1,0,1,'',0,'1-基本属性'),
(106, 'special_para', 'TDW参数', 1, 1, '可以支持多个参数，分号分隔;', 8, 'text-single', 'string', 'set hive.exec.parallel = true;', '', '0', '1', '0', '1', '', '0', NULL),
(106, 'sourceColumnNames', '源文件列名', 1, 1, '源文件的栏位名称，以逗号分割（结尾不能是逗号）,必须保证列数和文件内容一致（创建临时表所用表列名）。例如column1,column2,column3<br>注：不允许输入空格，源文件栏位名称只能由大小写字符、数字和下划线组成', 81, 'text-single', 'string', NULL, NULL, '1', '0', '0', '1', NULL, '0', NULL),
(106, 'targetColumnNames', '字段映射关系', 1, 1, '表的列名,英文逗号分隔,表示的列的内容顺序需和DB列字段保持一致。决定从临时表往目的表里写的字段顺序。日期和常量需要用中括号包起来，例如：[${YYYYMMDD}], [\'test\'], column_1, column_2, column_3',82, 'text-single', 'string', NULL, NULL, '1', '0', '0', '1', NULL, '0', NULL),
(106, 'task.main.timeout', '任务超时（分钟）', 1, 1, '任务超时，时间单位是分钟。当任务执行时间超过这段时间后，任务会被终止。',83, 'select-option', 'integer', '300', '10,30,60,150,300,480', '0', '0', '0', '1', '10,30,60,150,300,480', '0', '10,30,60,150,300,480'),
(106,'errorThreshold','脏数据阈值',1,1,'',84,'text-single','integer','10000','',1,0,0,1,'',1,''),
(106,'ignore.empty.datasource','数据源为空是否允许成功',1,1,'',85,'select-option','string','true','true,false',1,0,0,1,'是,否',1,''),
(106, 'failedOnZeroWrited', '入库为空时任务处理', '1', '1', '无源文件或入库记录为0时,可以指定任务为成功或失败', 86, 'select-option', 'string', '0', '0,1', '0', '0', '0', '1', '成功,失败', '0', NULL),
(106,'tmp_hdfs_host','临时存储中间结果HDFS环境',1,1,'格式host:port 或者cluster name',87,'text-single','string','hdfsCluster','',1,0,0,1,'',1,'1-基本属性'),
(106,'tmp_hdfs_targetPath','数据临时存放目录',1,1,'默认目录为/user/{username}/tmpsh/{taskid}/{datatime}',88,'text-single','string','','',0,0,0,1,'',1,'1-基本属性'),
(106,'readConcurrency','读并发度',1,1,'',95,'text-single','integer','1','',1,0,0,1,'',1,'1-基本属性'),
(106,'writeConcurrency','写并发度',1,1,'',96,'text-single','integer','4','',1,0,0,1,'',1,'1-基本属性'),
(106, 'distcp.update.number', 'Map个数', 1, 1, 'distcp命令map 个数设置', 98, 'text-single', 'integer', '30', '', '0', '0', '0', '1', '', '0', '');
update lb_task_type_ext set regex='{"tab":"1-基本属性","tip":"need to fulltill rex","reg":".*"}' where type_id=106 and prop_name = 'charset.encoding';
update lb_task_type_ext set regex='{"tab":"1-基本属性","tip":"必须是整数","reg":"^[0-9]*$"}' where type_id=106 and prop_name = 'errorThreshold';
update lb_task_type_ext set regex='{"tab":"1-基本属性","tip":"need to fulltill rex","reg":".*"}' where type_id=106 and prop_name = 'ignore.empty.datasource';
update lb_task_type_ext set regex='{"tab":"1-基本属性","tip":"必须是整数","reg":"^[0-9]*$"}' where type_id=106 and prop_name = 'readConcurrency';
update lb_task_type_ext set regex='{"tab":"1-基本属性","tip":"need to fulltill rex","reg":".*"}' where type_id=106 and prop_name = 'separator';
update lb_task_type_ext set regex='{"tab":"1-基本属性","tip":"need to fulltill rex","reg":".*"}' where type_id=106 and prop_name = 'sql';
update lb_task_type_ext set regex='{"tab":"1-基本属性","tip":"必须是整数","reg":"^[0-9]*$"}' where type_id=106 and prop_name = 'writeConcurrency';
update lb_task_type_ext set regex='{"tab":"1-基本属性","tip":"need to fulltill rex","reg":".*"}' where type_id= 106 and prop_name = 'databaseName';
update lb_task_type_ext set regex='{"tab":"1-基本属性","tip":"need to fulltill rex","reg":".*"}' where type_id= 106 and prop_name = 'distcp.update.number';
update lb_task_type_ext set regex='{"tab":"1-基本属性","tip":"need to fulltill rex","reg":".*"}' where type_id= 106 and prop_name = 'failedOnZeroWrited';
update lb_task_type_ext set regex='{"tab":"1-基本属性","tip":"need to fulltill rex","reg":".*"}' where type_id= 106 and prop_name = 'loadMode';
update lb_task_type_ext set regex='{"tab":"1-基本属性","tip":"need to fulltill rex","reg":".*"}' where type_id= 106 and prop_name = 'partitionType';
update lb_task_type_ext set regex='{"tab":"1-基本属性","tip":"need to fulltill rex","reg":".*"}' where type_id= 106 and prop_name = 'sourceColumnNames';
update lb_task_type_ext set regex='{"tab":"1-基本属性","tip":"need to fulltill rex","reg":".*"}' where type_id= 106 and prop_name = 'special_para';
update lb_task_type_ext set regex='{"tab":"1-基本属性","tip":"need to fulltill rex","reg":".*"}' where type_id= 106 and prop_name = 'tableName';
update lb_task_type_ext set regex='{"tab":"1-基本属性","tip":"need to fulltill rex","reg":".*"}' where type_id= 106 and prop_name = 'tmp_hdfs_host';
update lb_task_type_ext set regex='{"tab":"1-基本属性","tip":"need to fulltill rex","reg":".*"}' where type_id= 106 and prop_name = 'tmp_hdfs_targetPath';
update lb_task_type_ext set regex='{"tab":"2-高级属性","tip":"need to fulltill rex","reg":".*"}' where type_id= 106 and prop_name = 'output_prefix';
update lb_task_type_ext set regex='{"tab":"1-基本属性","tip":"need to fulltill rex","reg":".*"}' where type_id= 106 and prop_name = 'targetColumnNames';
update lb_task_type_ext set regex='{"tab":"1-基本属性","tip":"必须是整数","reg":"^[0-9]*$"}' where type_id= 106 and prop_name = 'task.main.timeout';
```

### 九、 重启base
在portal （或者8080端口）重启taskscheduler  

### 十、 启动tbds集群以外节点的runner服务  
**以下操作使用 hdfs 用户**
1. 启动runner 服务  
切到/usr/local/lhotse_runners 目录  
启动runner服务  ./start_jar.sh  
2. 启动httpd 服务  
执行/usr/sbin/httpd  
3. 确认httpd，runner启动  
执行命令：netstat -pan|grep httpd  有相关输出,确认httpd 启动ok.  
执行命令： jps ,有TaskRunnerLoader 相关进程信息，确认runner启动ok.    

### 十一、 更改lhotse service  
**以下操作使用 lhotse 用户**  
1. 切到lhotse serices 节点  
<br>
2. 更新并重启lhotse sercice  
a. 更新/usr/local/lhotse_service/webapps/LService/WEB-INF/classes/com/tencent/lhotse/config/AddServerConfig.class 文件  
b. 更新/usr/local/lhotse_service/webapps/LService/WEB-INF/classes/com/tencent/lhotse/servlet1/AddServerServlet.class 文件  
c. 更新/usr/local/lhotse_service/webapps/LService/WEB-INF/classes/com/tencent/lhotse/service1/AddServerService.class文件  
d. 重启lhotse serices  
    1. 切到/usr/local/lhotse_service 目录
    2. 停止lhotse server 
    ```shell
     bin/shutdown.sh 
    ```
    3. 启动lhotse server 
    ```shell
     bin/bin/startup.sh 
    ```
