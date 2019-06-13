mongodb 常见问题和处理方式

1. 确认mongodb 功能是否可用
连接到mongo router   
命令：${mongodb}bin/mongo --host hostname --port 20000    
如果save 和 findOne 命令都ok，则表示mongodb 可用 
```java  
use admin  
db.auth("admin","联系系统管理员")  
sh.enableSharding("test")
sh.shardCollection("test.Log", { id: 1})
for(var i = 1; i <= 10; i++){  db.Log.save({id:i,"message":"message"+i});}
db.Log.save({id:1,"message":"service check "});
db.Log.stats()
db.Log.findOne()
```
2. mongodb 部署策略  
 + 2.1 数据量比较小，但是需要强可靠  
建议config 和 shard 不分片，但需要多个副本（机器隔离）  
如果条件允许 config 和 shard 都保持两个副本（最少一个副本）（机器隔离）  ，其中shard会占用磁盘   
 + 2.2 数据量比较大，而且需要强可靠  
    + 2.2.1 表比较多，数据比较少（频繁修改表）  
config 建议分片，每个分片至少一个副本，条件允许保证有两个副本（机器隔离）  
shard 不分片，至少一个副本，条件允许保证两个副本（机器隔离）  
    + 2.2.2 表比较少，数据量比较多  
config 不分片，至少一个副本，条件允许保证两个副本（机器隔离）  
shard 分片，每个分片至少一个副本，条件允许保证两个副本（机器隔离）  
    + 2.2.3 表比较多，数据量比较多  
config ,shard 都分片，每个分片至少一个副本，条件允许保证两个副本（机器隔离）  

3. mongodb config 宕机  
 + 3.1 config primary 服务宕机，且不能恢复  
    + 3.1.1 登陆其中一个可用的secondary config   
    + 3.1.2 查询集群config 配置  
    确认其他config 服务在集群配置中
    ```rs.status()  ```  
    + 3.1.3 强制移除宕机的 primary config（**请先联系系统管理员**）  
    ```mongodb
    rs.conf()
    new_config=rs.conf()
    new_config.members=[new_config.members[0]] # 保留正常的config,0为conf() 方法输出的数组下标，可用保留多个,如： new_config.members=[new_config.members[1],new_config.members[3]]
    rs.reconfig(new_config,{force:ture})
    ```    
    + 3.1.4 系统会在剩余的config 中选举出新的primary  
    注意：有config 宕机，移除不可用的config ，还需要添加config ，以使得config 每个分片至少一个副本，条件允许保证两个副本（机器隔离）  
 + 3.2. config secondary 服务宕机且不可恢复  
    + 3.2.1. 扩展一个新的config   
    8080页面扩展,并启动（忽略启动异常）（记得修改配置）  
    + 3.2.2. 将新扩展的config ，添加到mongo集群中  
    登陆primary config ,执行  
```
    rs.add( { host: "<hostnameNew>:<portNew>"} ) 
```
    + 3.2.3. 检查宕机的config 是否还在集群配置中   
    登陆primary config ,执行
    ```rs.status() ```  
    如果输出结果的members 中是否有对应的节点  
    如果有，则需要通过下面命令remove  
    ```
    rs.remove("<hostnameOld>:<portOld>")
    ```
    如果没有，则不需要其他操作  
ps： 如果所有config 都宕机且不能恢复，即使shard 没有宕机，mongodb 也是不可用的。需要恢复请联系管理员  

4. shard 操作  
 + 4.1 shard 状态查看  
 登陆router 并认证   
 执行```db.adminCommand( { listShards: 1 } )``` 输出shard 列表
 + 4.2 添加shard   
 8080页面扩展shard(记得修改配置)  
 登陆router 并认证  
 执行```sh.addShard("<replica_set>/<hostname><:port>")```添加新的shard  
 再执行```db.adminCommand( { listShards: 1 } )``` 输出shard 列表，确认shard已经添加
 + 4.3 移除shard  
 **请联系管理员** 
