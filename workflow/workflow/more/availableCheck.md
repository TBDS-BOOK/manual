如果确认工作流后台服务可用以及诊断工作流后台功能不可用的操作

前言  
用户发现工作流后台服务不肯用，通常是通过任务实例不生成，实例不下发或8080端口页面有告警来发现的  
这里重点说明，在8080端口显示taskScheduler 服务正常，但是实际可能不用的诊断操作。

**1.** 查看base进程是否存在  
ps -ef |grep base_go

**2.** 查看base线程是否都存在  
jstack -pid |grep "r-taskType"

**3.** 查看loader进程是否存在   
ps -ef|grep loader

**4.** 查看loader线程是否都存在  
jstack -pid 

**5.** 检查 base 调度信息是否有输出    
/usr/local/lhotse_base]$ tail -f log/base_schedule.log  
查看对应任务类型的处理  
/usr/local/lhotse_runners]$ tail -f log/lhotse_task_loader.log |grep "loader-87"

**6.**检查loader 请求信息是否有输出  
/usr/local/lhotse_runners]$ tail -f log/lhotse_task_loader.log  
判断是否有实例正在下发运行  
/usr/local/lhotse_runners]$ tail -f log/lhotse_task_loader.log |grep "state RUNNING"

##### 更多其他诊断操作
基于arthas 的诊断（更多arthas 参考 https://alibaba.github.io/arthas/index.html）  
base 卡主定位分析  

通过调用频率来确认线程是否卡死  
monitor -c 3 com.tencent.teg.dc.lhotse.base.socket.ServerWorkThread addRequest
monitor -c 3 com.tencent.teg.dc.lhotse.base.issuer.TaskIssuer addRequest
monitor -c 3 com.tencent.teg.dc.lhotse.base.issuer.TaskIssuer issueTask  

一览方法的输入和输出，可带条件  
watch com.tencent.teg.dc.lhotse.base.issuer.TaskIssuer addRequest "{target.requests.length}" -x 3 -n 1
watch com.tencent.teg.dc.lhotse.base.issuer.TaskIssuer addRequest "{params,target.requests}" -x 2
watch com.tencent.teg.dc.lhotse.base.issuer.TaskIssuer getTaskToIssue "{params,target.requests}" -x 2
watch com.tencent.teg.dc.lhotse.base.issuer.TaskIssuer issueTask "{params,target.requests}" -x 2
watch com.tencent.teg.dc.lhotse.message.MessageBroker receiveMessage "{params,returnObj}" -x 2 -n 2

查询方法调用栈  
  trace -j com.tencent.teg.dc.lhotse.message.MessageBroker sendMessage  'params.length==3' -n 2
  trace -j  com.tencent.teg.dc.lhotse.base.issuer.TaskIssuer getTaskToIssue 'returnObj==null' -n 2
  trace -j com.tencent.teg.dc.lhotse.base.issuer.TaskIssuer validateCheck 'returnObj==false' -n 1

 向上查询方法执行路径   
  stack com.tencent.teg.dc.lhotse.message.MessageBroker sendMessage  'params.length==3' -n 2
  stack  com.tencent.teg.dc.lhotse.base.issuer.TaskIssuer getTaskToIssue  'returnObj==null' -n 2
