# Hemres SQL API端口介绍
* 使用格式为：
http://hermesserver:port/sql?sql=$sqlquery
* 获取当前入库表情况：
http://hermesserver:port /tablemanager
* 获取表入库数据量：
http://hermesserver:port /hermes?type=count
* 获取线上数据时间日期：
http://hermesserver: port /hermes?type=datelist&projectname=name

# 实时数据接入接口
* Kafka数据接入

```
{
    "topic": "topic",    //  topic
    "serverHost": "10.241.88.165:9092",  // master host 接口
    "group": "groupname" 	//消费组名
}

```
* Hippo数据接入

```
{
    "controllerHost": "controllerHost",    //  controllerHost
    "group": "group",  // group
    "userName": "userName", 	//userName
    "secretId": "secretId", 	//secretId
    "secretKey": "secretKey" 	//secretKey
}
```

* 调用方式:
* GET方式：
```
http:// hermesserver: port /config?kafkainfo={"topic":"topic","serverHost":"10.241.88.165:9092","group":"tboss_g1"}
```
* POST方式：
```
curl http://hermesserver:port/config -d 'kafkainfo={"topic":"topic","serverHost":"10.241.88.165:9092","group":"tboss_g1"}'
```

# Kafka和Hippo建表配置

```
{
    "hermesFields": [ // 表结构所包含字段有哪些，字段顺序需要和入库顺序一致
        {
            "fieldname": "name",  // 字段名
            "type": "string",  //字段类型
            "indexed": true,   //是否建立索引
            "stored": false	//是否存储内容
        },
        {
            "fieldname": "age",
            "type": "int",
            "indexed": true,
            "stored": false
        },
    ],
    "topic": " topic ", //所属topic
    "split": ",",  // 字段分隔符
    "key": "name",  //主键，hermes主键可以有重复
"timefield": "system_default",  //时间分区字段，默认为系统当前时间
"dateFormat": "YYYYMMDD", //时间格式( 支持 YYYY,YYYYMM,YYYYMMDD,YYYYMMDDHH,NULL)
    "json": false, 	//接入数据是否为json格式
    "tablename": "person",  	// 表名
    "range": 3   // 数据保留周期
}
```

* 调用方式：
* GET方式：
```
http://hermesserver:port/config?tableinfo={"hermesFields":[{"fieldname":"name","type":"string","indexed":true,"stored":false},{"fieldname":"age","type":"long","indexed":true,"stored":false}],"topic":"tboss","split":"," ,"key":"name","timefield":"system_default","json":false,"range":3,"dateFormat":"YYYYMMDD","tablename":"person"}
```
* POST方式：
```
curl http://hermesserver:port/config -d ‘tableinfo={"hermesFields":[{"fieldname":"name","type":"string","indexed":true,"stored":false},{"fieldname":"age","type":"long","indexed":true,"stored":false}],"topic":"tboss","split":"," ,"key":"name","timefield":"system_default","json":false,"range":3,"dateFormat":"YYYYMMDD","tablename":"person"}’
```

# Hermes Sql 实例
|    SQL    |
| ---------- |
| Select count(*) from tablename where thedate=’20150706’ |
| select sum(money) from tablename where thedate in (’20150706’,’20150707’) |

# Hermes SQL 专有名词
|    名称    | 说明 |备注
| ---------- | --- |
| Hermespartion(thedate) | Hermespartion，数据查询日期指定，相当于date参数 | 注意日期用单引号扣起来，且查询语句中必须加上hermespartion条件
| hermespartion=’20150706’ or hermespartion in (’20150706’, ’20150707’) or thedate=’20150706’ | -- | -- |
| 注意：过滤条件可以进行and与or的组合（但最外层必须是and ）,例子如下：</br>hermespartion in (20140908) and hermestopic='qqface' and (ddwuid='1' or ddwuid='2' or (dwinserttime>=20140908202626 and dwinserttime<=20140908203026)) | -- | -- |

# Hermes SQL 条件查询
|    条件语句    | 说明备注
| ---------- | --- |
| in(…)集合查询|括号内包含查询条件的集合，（时间分区字段hermespartion，thedate，多日期查询必须使用in语句）|
| hermespartion in (20140908，20140910) and hermestopic='qqface' and (ddwuid='1' or ddwuid='2' or (dwinserttime>=20140908202626 and dwinserttime<=20140908203026)) | -- |
|>,<,>=,<=,[],{}区间查询|对于带有范围的过滤筛选推荐使用下列含有TO的查询，能提升查询效率,注意两种括号的区别，前者不包含边界值，但后者则包含|
| dwinserttime like '([20140916000000 TO 20140916115000] )' </br>dwinserttime like '({20140916000000 TO 20140916115000})' </br>实在不能避免可以考虑下面的写法，但是对于重复值很少的列，查询起来会比较慢</br>dwinserttime>=20140916115000 and dwinserttime <=20140916000000 |--|
| like模糊匹配 |szlogstr like 'iris *' 注意如何该列排重后的值得数量特别多，禁止将通配符写在最前面，否则查询性能很低</br>ddwuin like '6938433??' 如要要进行类似QQ号码的定长匹配，可以用?来代替*，*是匹配0~多个字符，而?则表示只匹配一个字符|
| 空字符串的匹配</br>select s1_syn,hermes_rawdata from hermeslink_90day where hermespartion='20150203' and hermestopic='qqface' and s1_syn like '({* TO @space@})'  limit 0,100</br>Null字符串的匹配</br>select s3_syn,hermes_rawdata from hermeslink_90day where hermespartion='20150203' and hermestopic='qqface' and s3_syn <>'*' limit 0,100|--|
| limit 0，N返回记录条数|默认返回20条查询到的结果记录，上限是返回 10000条记录|
| <> | 不等于查询如：</br>zlogstr<>'2100' and szlogstr<>'2534' and szlogstr<>'1002' </br>logstr<>’*’ 匹配空值|
| like ([* TO *]) | 查询所有非空值：eg: f120014 like '([* TO *])'|
| hermesall=1 | 是否全表扫描数据，不加此参数返回limit条结果之后不在扫描集群其他数据 |

# Hermes 支持的SQL操作
|    Sql 操作    | 说明 | 备注
| ---------- | --- | --- |
|count统计操作|对表进行count统计|注意下count(*)与count(age)的区别，前者不管列的值是否是null，计数都会加1，而且因为不需要读取字段的值，性能很高，后者需要判断改字段的值是否为null，如果是null则不会进行累加计数，所以大部分场景都推荐使用count(*)|
|select count(*),count(ddwuid) from sngsearch03 where hermespartion=20140921 limit 0,100|--|--|
|avg 取平均值|对表字段进行取平均值操作|--|
|sum求和|字段求和|--|
|min取最小值|字段取最小值|--|
|max取最大值|字段取最大值|--|
|select sum(acnt),average(acnt),max(acnt),min(acnt) from sngsearch03 where hermespartion= 20140921 and hermestopic='qqface' limit 0,100|--|--|
|group by分类操作|分类汇总，支持对多列进行group by|--|
|order by排序操作|对查询的结果进行排序|排序的列目前只能是单列|
|select x1,count(*),sum(acnt) from sngsearch03 where hermespartion=20140921 and hermestopic='qqface'  group by x1 order by x1 limit 0,100|--|--|

# Hermes数据导出功能
|    接口    | 说明
| ---------- | --- |
|Sql查询语句|select count(*) from tablename where thedate=20171025|
|导出参数|hermeskv='projectname: tablename '    //导出表名</br>hermeskv='yyyymmdd: 20171025’      //导出时间分区</br>hermeskv='higo.export.userpackageid:export_test1'   //导出包id，唯一标识不可重复</br>hermeskv='higo.export.fields:qq,uin'    //导出字段, 导出多个字段用逗号分割|
|自定义导出参数（可选）|hermeskv=' hermes.export.kafka.broker:10.241.88.165:9092'   // kafka broker 地址端口</br>hermeskv=' hermes.export.kafka.topic:export_topic'    // 导出kafka topic名|
|http://hermesserver:port/sql?sql=select count(*) from tablename where thedate=20171025 and hermeskv='projectname:tablename' and hermeskv='yyyymmdd:20171025' and hermeskv='higo.export.userpackageid:export_test1' and hermeskv='higo.export.fields:qq,uin'|--|

* 导出任务状态进度查询

|    接口    | 说明
| ---------- | --- |
|查询接口|http:// hermesserver:port /exportstatus|
|参数|projectname=t_webitil_490   //导出任务时所指定的projectname</br>user.data.package.userpackageIds=kaynewu213  //导出任务id|
|删除参数|cmd=delete  //如果需要终止或者删除某个导出任务可加上该参数|
</br>
返回值：
</br>
```
｛
    "projectName": "t_webitil_490",
    "yyyymmdd": "20171024",
    "userpackageId": "kaynewu213",
    "status": "OK",   //导出结果 OK，ERROR
    "hadConvert": false,
    "currentStatusDesc": " ",
    "process": 1,         //到出进度
    "totalCount": 1572864,    // 导出数据量
    "lastModifyTime": 1508921963363,
    "exportUrl": "http://10.240.130.36:8089/kaynewu213.txt;",  //导出文件下载地址
    "userPackageProgressStatus": "SUCCESS",  //当前导出任务状态
    "share": false
}
```
userPackageProgressStatus：
```
INIT,//初始化
EXPORTING, // 正在导出
SUCCESS, // 成功
ERROR, // 错误
FAIL;// 失败
```
示例：
```
http://10.240.130.35:8080/exportstatus?projectname=t_webitil_490&user.data.package.userpackageIds=kaynewu213
```

* 自定义导出kafka topic 导出数据格式（业务直接从kafka消费出来的数据格式）
```
{
    "exportid": "export_test43",   //导出包 id，标识一次用户导出任务
    "uuid": "bb5e84f9-a21a-49c1-bf0a-3d4dda98ab45",  //uuid 标识符
    "type": "HEAD",   // 到处消息类型分为3种；HEAD（头部，total字段标识某个数据分片导出总量） DATA（消息体） END（尾部，标识某个数据分片已导出完成）
    "total": 3,  // 某个数据分片导出总量
    "size": 3,  // 携带导出数据个数
    "data": [ // 导出数据列表
        {
            "pid": "17268",
            "ftime": "2016-10-09 10:32:25"
        },
        {
            "pid": "17268",
            "ftime": "2016-10-09 11:35:39"
        },
        {
            "pid": "17268",
            "ftime": "2016-10-09 11:35:39"
        }
    ]
}
```
