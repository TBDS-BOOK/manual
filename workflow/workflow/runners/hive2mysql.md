hive 导出到DB(mysql)

### 功能说明
hive 查询记录导入关系型数据库（mysql）

### 其他说明
hive 查询使用，任务第一责任人连接（portal前台登陆用户）

### 任务设置
#### 1. 基本信息  
参考 [基本信息设置](/workflow/workflow/runnerBasicInfo.md)  
#### 2. 调度  
参考 [调度设置](/workflow/workflow/runnerCycle.md)  

#### 3. 参数
参数配置如下图  
![hive2mysql](/workflow/workflow/images/hive2mysql.png)

1. 源服务器
2. 目标服务器
3. 源db名称
4. 源SQL
5. 源SQL查询结果字段数量
6. 分隔符
7. 目标表（mysql）
8. 目标表列名
9. 脏数据阀值
10. 数据源为空任务是否成功
11. 数据入库模式
12. 是否分区
13. beeline输出格式
14. 任务超时时间
15. 临时存储中间结果HDFS环境
16. 数据临时存放目录
17. 读并发度
18. 写并发度


### demo

### demo资源
