## 定位问题常用的操作

#### 1. 查询安装runner组件安装的节点
![](../images/operate1.png)
#### 2. 查看任务信息所在数据库

#### 3. 查看任务实例运行节点
![](../images/operate2.png)
#### 4. 通过portal日志定位问题
portal 节点执行  tailf /usr/local/tbds-portal/log/application.log  
页面触发对应操作，获取后台异常日志

更多操作：  
清理日志  
echo "" >  /usr/local/tbds-portal/log/application.log  
日志重定向  
tailf /usr/local/tbds-portal/log/application.log|tee /tmp/portal.log

#### 5. 通过service 日志定位问题
切到LHOTSE_SERVICE 节点执行  
tailf /usr/local/lhotse_service/logs/lhotse_service.log  
触发相应操作，获取后台异常日志

更多操作：  
清理日志  
echo "" >  /usr/local/lhotse_service/logs/lhotse_service.log  
日志重定向  
tailf /usr/local/lhotse_service/logs/lhotse_service.log  |tee /tmp/tmp.log
