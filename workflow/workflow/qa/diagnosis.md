4031 ，411，50 版本，任务实例的诊断功能，相关消息来自hermes

调度信息的流程为： base 打印到本地 > flume 采集 > 发送kafka > hermes 消费。

  
定位问题  
hermes 查询 对应节点为 Hermes Server(HERMESOFFLINE_SERVER)

1. 查询表结构  
curl http://172.16.32.8:8080/config -d "cmd=desctables&tablename=lhotseScheduleStatistic"  
curl http://172.16.32.8:8080/config -d "cmd=desctables&tablename=lhotseSchedule"

2. 创建一个hermes 表  
curl http://172.16.32.8:8080/config -d 'tableinfo={"hermesFields":[{"fieldname":"instance_1","type":"date","indexed":true,"stored":true},{"fieldname":"instance_2","type":"text","indexed":true,"stored":true},{"fieldname":"instance_3","type":"text","indexed":true,"stored":true},{"fieldname":"instance_4","type":"text","indexed":true,"stored":true},{"fieldname":"instance_5","type":"text","indexed":true,"stored":true},{"fieldname":"instance_6","type":"text","indexed":true,"stored":true},{"fieldname":"instance_7","type":"text","indexed":true,"stored":true}],"topic":"lhotsediagnose","split":",","key":"instance_1","timefield":"instance_4","dateFormat":"YYYYMMDD","json":false,"tablename":"lhotsediagnose","range":7,"viewTable":false}'

3. 创建一个hermes topic  
curl http://172.16.32.8:8080/config -d 'kafkainfo={"topic": "lhotsediagnose","serverHost": "10.0.0.33:6668","group": "lhotsediagnose_gorup"}'

4. 查询hermes 已有的表  
curl http://10.0.0.45:8080/config?cmd=showtables
curl http://192.168.16.37:8080/config?cmd=showtables

5. 查询hermes 总信息(有数据才显示)  
curl http://10.0.0.45:8080/hermes?type=count

5. 查询表分区信息  
curl http://10.0.0.45:8080/hermes -d "type=datelist&projectname=lhotseSchedule"
curl http://172.16.32.8:8080/hermes -d "type=datelist&projectname=lhotseScheduleStatistic"

6. 记录总数  
curl http://172.16.32.8:8080/sql -d "sql=select count(*) from lhotseSchedule where thedate=20190222"

7. 浏览部分数据  
curl http://10.0.0.45:8080/sql -d "sql=select * from lhotseSchedule where thedate=20190304 limit 10"

8. 实例诊断（浏览器）  
curl http://172.16.32.8:8080/sql -d "sql=select * from lhotseSchedule where thedate=20190221 and instance_2=20190221155033872 and instance_3='2019-02-21 00:00:00' order by instance_1 desc limit 10"

如果上面的操作出现异常，定位方式如下：  
首先 确认是否为 kafka 问题
1. 添加认证信息：   
vi config/client_sasl.properties  
添加如下  
```
security.protocol=SASL_PLAINTEXT  
sasl.mechanism=PLAIN 
```
2. 消费数据demo  
bin/kafka-console-consumer.sh --bootstrap-server 10.0.0.33:6668 --topic lhotseSchedule --new-consumer --consumer.config config/client_sasl.properties --from-beginning

3. 生产数据demo  
bin/kafka-console-producer.sh --broker-list tbds-172-16-32-8:6668  --topic lhotseSchedule --producer.config config/client_sasl.properties

然后 确认是否为 hermes 问题  
1. 是否有表 查 hdfs  
hadoop fs -ls /user/hermes/tbds/hermesconf  
结果如： 
``` 
Found 3 items  
drwxrwxr-x   - hermes hdfs          0 2019-03-04 17:20 /user/hermes/tbds/hermesconf/lhotseSchedule
drwxrwxr-x   - hermes hdfs          0 2019-03-04 17:20 /user/hermes/tbds/hermesconf/lhotseScheduleStatistic
drwxrwxr-x   - hermes hdfs          0 2019-03-04 17:11 /user/hermes/tbds/hermesconf/ranger_audits
```
2. 是否有数据 查 hdfs  
hadoop fs -ls /user/hermes/tbds/index  
结果如:  
```
Found 2 items
drwxrwxr-x   - hermes hdfs          0 2019-03-04 17:25 /user/hermes/tbds/index/lhotseSchedule
drwxrwxr-x   - hermes hdfs          0 2019-03-04 17:26 /user/hermes/tbds/index/lhotseScheduleStatistic
```
3. 查询sql的日志在（如命令：curl http://10.0.0.45:8080/hermes?type=count）  
HERMESOFFLINE_SERVER 节点的 /usr/hdp/2.2.0.0-2041/hermes/logs/server.log

4. 数据处理（有表，但是没有数据，需要查看work 节点的日志）  
/usr/hdp/2.2.0.0-2041/hermes/logs/worker.log 


跟 flume 调试   
1. 配置确认  
/var/lib/ambari-agent/cache/common-services/FLUME/1.4.0.2.0/configuration/flume-conf.xml  
有scheduler_statistic 关键字
2. 确认配置生效  
/var/log/flume/flume-agent.log
有关键字  lhotse_base_scheduler_statistic_src

诊断序列梳理  
按照判断的顺序，id 号从小到大
```
DEPENDENCY_STATUS_CHECK("检查任务状态",2,"任务不是运行状态"),
DEPENDENCY_RETRIES_CHECK("重试次数判断",4,"任务实例重试次数已经大于或等于设置的最大重试次数"),
DEPENDENCY_STARTUP_CHECK("执行时间窗口判断",6,"任务设置的开始时间异常(>1440)"),
DEPENDENCY_PRIORITY_CHECK("优先级合规判断",8,"任务优先级设置异常,需要满足 0到9之间"),
DEPENDENCY_DATATIME_CHECK("数据时间合规判断",10,"实例初始化数据时间失败(或为空)"),

DEPENDENCY_NOBRIDGE_PARENTINSTANCES_CHECK("无边依赖条件下,基于父实例运行成功的执行条件判断",12,"任务配置了自身依赖,但前周期的实例没有全部运行成功"),
DEPENDENCY_BRIDGE_PARENT_INSTANCES_CHECK("有边依赖条件下,基于前周期实例运行成功的执行条件判断",14,"有边依赖,任务配置了自身依赖;前周期的实例没有全部运行成功"),
DEPENDENCY_BRIDGE_CHECK("边状态判断",16,"依赖不是可用状态(对应标记为Y表示可用)"),
DEPENDENCY_ParentTaskExist_CHECK("基于边关系的父任务存在（基于边关系）判断",20,"边关系表中对应的父任务ID丢失"),
DEPENDENCY_ParentTaskExistCache_CHECK("基于边关系的父任务存在（基于内存）判断",22,"父任务没有加载到内存"),
DEPENDENCY_ParentInstance_CHECK("若父任务为一次性任务,其实例存在判断",24,"父任务为一次性任务,对应实例还没有生成"),
DEPENDENCY_ParentInstanceSuccess_CHECK("若父任务为一次性任务,其实例运行成功的执行条件判断",26,"父任务为一次性任务,对应实例还没有运行成功"),
DEPENDENCY_ParentTaskDataTimeSuccess_CHECK("若父任务为周期性任务,其数据时间合规判断",27,"依据边依赖找不到父任务对应的时间周期"),
DEPENDENCY_ParentInstanceA_CHECK("若父任务为周期性任务,其实例存在判断",28,"对应时间区间内父任务实例数为0"),
DEPENDENCY_ParentInstanceIntact_CHECK("若父任务为周期性任务,其实例数目完整判断",29,"对应时间区间内父任务实例数不全"),
DEPENDENCY_ParentInstanceLegal_CHECK("若父任务为周期性任务,其实例合法性判断",30,"其实例在其任务开始时间之前(父任务)"),
DEPENDENCY_AParentInstanceSuccess_CHECK("若父任务为周期性任务,其实例运行成功的执行条件判断",31,"父任务对应实例没有全部运行成功"),
DEPENDENCY_Bridge_Status_CHECK("若父任务为周期性任务,边依赖属性是否可用判断",32,"对应边依赖属性不可用（dependency!=1）"),
DEPENDENCY("依赖判断通过",38),

ISSUE_BrokerLimit_check("该runner节点运行的某任务类型实例数是否超过设置的上限(跟节点相关)判断",40,"在runner节点运行的某任务类型实例数等于设置的上限(跟节点相关)"),
ISSUE_TaskLimit_check("该任务对应的正在运行的实例数是否超过设置的上限(任务类型)判断",42,"非并行任务对应的实例，一个节点最多运行一个实例"),
ISSUE_TaskLimitC1_check("该任务对应的正在运行的实例数是否超过设置的上限(任务类型)判断",44,"该任务对应的正在运行的实例数等于设置最大可并行实例数(任务类型)"),
ISSUE_RedoTaskLimit_check("重跑任务实例数是否超过设置的上限判断",46,"重跑状态下的任务其对应的实例数超过设置的上限(任务类型相关)"),

ISSUE_Source_Server_Check("其使用的源服务器存在性（基于内存）判断",47,"源服务器信息没有加载到内存"),
ISSUE_Target_Server_Check("其使用的目标服务器存在性（基于内存）判断",48,"目标服务器信息没有加载到内存"),
ISSUE_Server_Concurrent_Check("正在使用该的服务器实例数是否超过服务器允许的上限判断",49,"使用了该服务器的任务，其下正在运行的实例数等于该服务器允许的上限"),


ISSUE_BrokerAvailable_check("请求实例节点可用性判断",50,"非任务指定运行的runner节点在请求该实例（多次出现可能是指定的runner节点异常）"),
ISSUE_BrokerPriority_check("请求实例节点优先级合规判断",52,"该实例运行优先级不支持在请求该实例的runner节点运行(实例所属任务优先级太低)"),
ISSUE_TaskDataTime_check("实例数据时间合规判断",54,"任务实例还不到运行时间"),
ISSUE_TaskDataTimeC1_check("实例开始时间合规判断",56,"任务实例还不到运行时间(开始时间不满足)"),
ISSUE_TaskTryLimit_check("实例重试次数合规判断",58,"尝试运行次数大于或等于设置的允许尝试运行次数上限"),
ISSUE_InstanceExeTime_check("实例执行时间合规判断",60,"任务实例不可在结束时间之后运行"),
ISSUE_InstanceRedo_check("实例是否刚被重跑判断",67,"任务实例刚被用户重跑"),

INITIAL("生成实例",1),
ISSUE("下发实例",70),
RUNNING("正在运行",75),
FAILED("执行失败",80),
SUCCESS("执行成功",85);
```

更多统计  
1. 今天生成的实例数目  
curl http://172.16.32.8:8080/sql -d "sql=select count(*) from lhotseSchedule where thedate=20190222 and instance_2='1'"

2. 今天通过依赖判断的实例数目  
http://10.215.128.43:8080/sql?sql=select count(*) from schedulerMsgTable where thedate=20181205 and instance_2='38'

3. 今天下发的实例实例数目  
http://10.215.128.43:8080/sql?sql=select count(*) from schedulerMsgTable where thedate=20181205 and instance_2='99'

半小时统计曲线  
1. 今天生成的实例数目  
http://10.215.128.43:8080/sql?sql=select count(*) from schedulerMsgTable where thedate=20181206 and instance_2=1 and instance_1<'2018-12-06 17:00:00' and instance_1>'2018-12-06 16:30:00'

2. 今天通过依赖判断的实例数目  
http://10.215.128.43:8080/sql?sql=select count(*) from schedulerMsgTable where thedate=20181206 and instance_2=30 and instance_1<'2018-12-06 17:00:00' and instance_1>'2018-12-06 16:30:00'

3. 今天下发的实例实例数目  
http://10.215.128.43:8080/sql?sql=select count(*) from schedulerMsgTable where thedate=20181206 and instance_2=99 and instance_1<'2018-12-07 17:30:00' and instance_1>'2018-12-07 17:00:00'
