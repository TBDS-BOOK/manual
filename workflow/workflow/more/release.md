### V5.0
#### 功能新增
1. 【工作流】taskSchedule base,serivce 默认使用服务发现的metadata地址 。优先使用8080页面配置数据库（lhotse_opendb 名称不能变化）
2. 去掉原来metrics 统计信息，去掉stateRunner 统计，将调度信息，统计信息通过kafka写入hermes
3. 支持设置runnerNode 可执行runner 并发上限（任务类型）
4. 支持服务器刷新和查询 rpc 接口（提供client）
5. 提升loader 有效请求率  
 * 请求频率支持毫秒级别
 * 请求线程更加稳定
 * 下发实例，从db全量加载改为批量加载
6. 支持强制下发 (state = -1)
7. 优化log4j 性能，默认缓存8k
8. taskSchedule service 服务支持HA(portal 通过服务发现随机取一个)
9. 为了支持大规模集群，需要支持工作流任务实例从db切到hbase
 * 是否将任务实例入HBase 是可配置的，但是只能从mysql 切到HBase ，无法从HBase 切到mysql
 * 该功能涉及 taskSchedule service,taskSchedule base 模块
10. 导入导出任务 on spark
 * 支持多中HDFS文件格式（parquet,text,csv,orc,json）
 * 支持多中字段类型，避免字段类型转换过程中的精度问题
 * runner 更轻量，本地节点使用的cpu和内存更少，on spark 使用yarn 资源管理，资源利用更高

#### bugfix

### V4.0.3.1
