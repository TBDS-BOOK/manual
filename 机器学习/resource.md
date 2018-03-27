## 资源申请策略


#### Spark和PySpark资源申请策略
- **基本说明:**   
假如每个executor分配1个core,1个core跑1个task, 1个task对应1个partition, 建议1个partition的输入数据在200-300M左右, 1个task申请2-3GB内存;  
按线性关系，假如每个executor分配2个core, 那么每个executor同时可跑2个task, 每个executo需r分配4-6GB内存;  

- **举例:**   
输入数据为400MB，那就可以有以下两种资源申请：  
  方式一：配置num_executor(1) && driver_memory(2GB) && executor_cores(2) && executor_memory(4GB)   
  此时申请总资源数:   
  CPU： num_executors*executor_cores + 1 (driver)= 3 core;  
  内存： num_executor*executor_memory + driver_memory=6GB;  
  
&emsp;&emsp;方式二：配置num_executor(2) && driver_memory(2GB) && executor_cores(1) && executor_memory(2GB)  
&emsp;&emsp;此时申请总资源数同上述是一样的:  
&emsp;&emsp;CPU： num_executors*executor_cores + 1 (driver)= 3 core；  
&emsp;&emsp;内存：num_executor*executor_memory + driver_memory=6GB;  

- **注意:**   
1)driver内存视分区数而定，分区数多就给大些，分区数少就少给些，分区数在1000以下的，建议driver内存不超过4GB，通常2-4GB可以满足大多数场景;  
2)尽量不要在driver端collect过多数据,避免内存不够.其他值可根据自己的数据量级别适当线性调整申请资源大小.

#### xgBoost资源申请策略
xgBoost资源预估无做到如Spark那样精细，但用户在配置时可以进行参考如下的一些建议：    

-  根据每个资源组可申请到的资源总量(core数和内存数)来选择合适的资源  
- xgboost会等到用户申请的worker全部到位后才开始运行，所以如果申请的worker数过大会导致程序卡住；  
  如果stderr中没有长时间没有出现“ All of xxx nodes getting started”，可以判断为资源申请失败，请调小nworker值重新运行  
- 建议nworker数小于10，cores小于4，memory_mb小于10000，具体情况需要根据1和2进行增减
