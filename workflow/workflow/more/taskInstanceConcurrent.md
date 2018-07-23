## 任务和实例的并发数调整
taskSchdule有多种粒度的并发控制：
可以从下面几个方便来说明:
1. 服务器配置的并发数控制
2. 某任务类型对应实例在某节点上并发数控制
3. 实例在集群上并发数控制


### 一、服务器配置的并发数
创建服务器时，可以指定服务器的并发度,该并发度表示当前集群允许同时使用该服务器的实例数总数上限。  

![](/workflow/workflow/images/serverConcurrent.png)

**其他：**  
对应lhotse_open.lb_server 表对应 parallelism 字段值。 

### 二、某任务类型对应实例数在某节点上并发数
该并发度没有开放给用户设置，默认值为1000.  
对应lhotse_open.lb_runner 表对应broker_parallelism 字段值
```
           type_id: 118
         broker_ip: 10.10.10.10
       broker_port: 10000
   executable_path: aaa
         in_charge: *u
            status: Y
      startup_time: NULL
    last_heartbeat: 2018-07-23 19:43:27
    priority_limit: 9
broker_parallelism: 1000
        is_upgrade: 0

```
上面的例子说明任务类型为118 的实例在10.10.10.10 节点运行总数上限是1000

三、任务对应的实例在集群上并发数控制
对应lhotse_open.lb_task_type 表。
```
            type_id: 37
 broker_parallelism: 10
   task_parallelism: 10
do_redo_parallelism: 10
```
该控制又可以细分为三个方面  
1. 在某一时刻,37任务类型对应的某任务在某个runner节点(看哪个节点请求了该实例)上允许运行的实例数上限    
对应broker_parallelism 字段值

2. 在某一时刻，37任务类型对应的某个任务在所有runner节点上允许运行的实例总数的上限  
对应task_parallelism 值  

3. 在某一个时刻，37任务类型对应的某个任务在所有runner节点上允许重跑的实例总数上限  
对应do_redo_parallelism 值
