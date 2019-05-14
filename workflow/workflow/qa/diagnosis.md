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
1. 添加认证信息： vi config/client_sasl.properties
添加如下
security.protocol=SASL_PLAINTEXT
sasl.mechanism=PLAIN

2. 消费数据demo  
bin/kafka-console-consumer.sh --bootstrap-server 10.0.0.33:6668 --topic lhotseSchedule --new-consumer --consumer.config config/client_sasl.properties --from-beginning

3. 生产数据demo  
bin/kafka-console-producer.sh --broker-list tbds-172-16-32-8:6668  --topic lhotseSchedule --producer.config config/client_sasl.properties

然后 确认是否为 hermes 问题  
1. 是否有表 查 hdfs
hadoop fs -ls /user/hermes/tbds/hermesconf
结果如：
Found 3 items
drwxrwxr-x   - hermes hdfs          0 2019-03-04 17:20 /user/hermes/tbds/hermesconf/lhotseSchedule
drwxrwxr-x   - hermes hdfs          0 2019-03-04 17:20 /user/hermes/tbds/hermesconf/lhotseScheduleStatistic
drwxrwxr-x   - hermes hdfs          0 2019-03-04 17:11 /user/hermes/tbds/hermesconf/ranger_audits

2. 是否有数据 查 hdfs  
hadoop fs -ls /user/hermes/tbds/index
结果如
Found 2 items
drwxrwxr-x   - hermes hdfs          0 2019-03-04 17:25 /user/hermes/tbds/index/lhotseSchedule
drwxrwxr-x   - hermes hdfs          0 2019-03-04 17:26 /user/hermes/tbds/index/lhotseScheduleStatistic

3. 查询sql的日志在（如命令：curl http://10.0.0.45:8080/hermes?type=count）  
HERMESOFFLINE_SERVER 节点的 /usr/hdp/2.2.0.0-2041/hermes/logs/server.log

4. 数据处理（有表，但是没有数据，需要查看work 节点的日志）  
/usr/hdp/2.2.0.0-2041/hermes/logs/worker.log 


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
