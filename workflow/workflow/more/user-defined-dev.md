## 自定义Runner代码设计说明文档

### 1 .需要依赖的jar包
 自定义runner任务依赖 runner-common.jar 
``` 
<dependency>
    <groupid>tencent.lhotse.runner</groupid>
    <artifactid>runner-common</artifactid>
    <version>1.2.2-SNAPSHOT</version>
 </dependency>
``` 
如果是4.0.3.1 的版本我们依赖
```
<dependency>
    <groupId>com.tencent.teg.dc.runner</groupId>
    <artifactId>runner-common</artifactId>
    <version>0.0.1-SNAPSHOT</version>
 </dependency>
```

如果是5.0 以后的版本 需要依赖对应的版本  
比如tbds 安装的是5.0 版本
```
        <dependency>
            <groupId>com.tencent.tbds</groupId>
            <artifactId>runner-common</artifactId>
            <version>5.0</version>
            <scope>provided</scope>
        </dependency>
```
**5.0 以后版本需要注意**
1. 依赖的runner-common 建议加上provided .这样能减少打包文件大小。
2. 如果要引入runner-common 中没有的依赖jar,需要添加到跟主类同一层的目录下。

### 2 .继承的基类  
1. HDFS与传统数据库导入的runner可以继承HDFSToDBRunner或者DBToHdfsRunner，调用DBUtil类来调用JDBC执行DAO  
2. 普通的runner直接继承jar包中的AbstractCustomTaskTypeRunner基类就可以了。
       
### 3 .必须实现的抽象方法  
重载AbstractTaskRunner的execute()方法和kill（）方法。

### 4 .启动任务流程  
        
1. 新建main 函数 public static void main(String[] args) {}
    并在main 函数中实例化runner对象,将args[0] 作为实例化对象的传入参数。
2. 在main函数中调用对象startwork()方法，启动自定义runner。 


### 5 .获取参数的方法
1. 通过this.getTask()方法获取TaskRuntimeInfo对象。
 
2. 通过TaskRuntimeInfo对象，获取获取用户填写参数  
  
 2.1 获取实例和任务信息   
&nbsp;&nbsp;task.getId() 获取实例id  
&nbsp;&nbsp;task.getType() 获取任务类型  
&nbsp;&nbsp;task.getCurRunDate() 获取实例数据时间  
&nbsp;&nbsp;task.getNextRunDate() 获取实例下一个数据时间   
 
 2.2 获取用户在ui上填写的填写任务参数   
&nbsp;&nbsp;this.getProperties().get(key) //获取任务参数.  
 
 2.3 获取用户在ui上选择的源服务器和目标服务器信息  
获取方式如下
```
List<ServerRuntime> sourceServers = taskRuntime.getSourceServers();
List<ServerRuntime> targetServers = taskRuntime.getTargetServers();
```  
虽然返回值是一个list队列，但目前只支持一个源服务器和一个目标服务器  
服务器对象 ServerRuntime 结构如下,通过get*() 方法获取对应的值
```
private String tag; // 服务器标示符
private String type;// 服务器类型 支持hive,hdfs,ftp,postgreSQL,mysql,sql server,oracle
private String host;// 服务器地址(对应主机地址)
private int port;// 服务器连接端口(对应端口)
private String userName;// 连接用户名
private String password;// 连接密码
private String service;// 当前不用
private String userGroup;// 当前不用
private String version;// 当前不用
```

2.4 通过服务发现，获取服务地址  
getServerListConnectInfoByDiscovery(String serverName,String component,String protocol,String role)
比如  
```java
        List<ServiceConnectInfo> serveListrConnectInfo = getServerListConnectInfoByDiscovery(
                    "portal", "portal", "rpc", "MASTER");
```

### 6 .日志输出方法  
this.writeLocalLog(Level.INFO, "****");   
或直接System.out.println("******")   
    
### 7 .提交任务实例执行状态  
this.commitTask(state, runtimeId, desc);  

Ps:   
常用运行状态有：RUNNING（正在运行），KILLED（已经停止），SUCCESSFUL（成功），FAILED（失败）

### 8 .停止任务实例运行     
   1. 停止实例运行前，清理runner资源，并提交停止状态。  
   2. 我们实现了runner调度资源的清理方法， CommonUtils.killProcess(this.taskRuntime, this);  
    __建议开发者，使用统一的停止方法，如下：__  
``` 
	public void kill() throws IOException {
		this.writeLocalLog(Level.INFO, " hello word had been kill ");
		boolean killResult = false;
		try {
			killResult = CommonUtils.killProcess(this.taskRuntime, this);
			if (killResult) {
				this.writeLocalLog(Level.SEVERE, "kill job succeed!");
				this.commitTask(LState.KILLED, "", "kill job succeed!");
			} else {
				this.writeLocalLog(Level.SEVERE, "kill job failed!");
				this.commitTask(LState.HANGED, "", "kill job failed!");
			}
		} catch (Exception e) {
			this.writeLocalLog(Level.SEVERE,
					"kill job failed:" + CommonUtils.stackTraceToString(e));
			this.commitTask(LState.HANGED, "", "kill job failed!");
		}
	}
``` 
### 9. HellowordRunner示例
下面是一个完整例子  

```java
import com.tencent.teg.dc.lhotse.newrunner.ServerRuntime;
import com.tencent.teg.dc.lhotse.proto.LhotseObject.LState;
import com.tencent.teg.dc.lhotse.runner.AbstractCustomTaskTypeRunner;
import com.tencent.teg.dc.lhotse.runner.util.*;
import org.apache.commons.collections.CollectionUtils;
import org.apache.commons.lang.StringUtils;
import java.io.IOException;
import java.util.List;
import java.util.logging.Level;

public class TaskRunner extends AbstractCustomTaskTypeRunner {

    public TaskRunner(String configFileName) {
        super(configFileName);
    }

    private boolean success = false;

    @Override
    public void execute() throws IOException {
        try {
            taskRuntime = this.getTask();
            String p1 = taskRuntime.getProperties().get("parameter1");
            this.writeLocalLog(Level.INFO, "get parameter 1 =" + p1);
            if(!"kill".equals(p1)){
                commitTaskAndLog(LState.FAILED, "", "parameter is kill");
            }

            List<ServerRuntime> sourceServerList = taskRuntime.getSourceServers();  // 源服务器列表
            if (CollectionUtils.isEmpty(sourceServerList) || sourceServerList.size() < 1) {//少于1台
                throw new Exception("服务器配置异常");
            }

            success = true;
        } catch (Exception e) {
            this.writeLocalLog(Level.SEVERE ,"执行HelloWord runner 出现异常");
            String st = CommonUtils.stackTraceToString(e);
            this.writeLocalLog(Level.SEVERE, "Exception stackTrace: " + st);
            commitTaskAndLog(LState.RUNNING, "", "Exception: " + e.getMessage());
            throw new IOException(e);
        }
        finally {
            if (!success) {
                commitTaskAndLog(LState.FAILED, "", "failed");
            } else {
                commitTaskAndLog(LState.SUCCESSFUL, "", "success execute");
            }
        }
    }
    @Override
    public void kill() throws IOException {
        this.writeLocalLog(Level.INFO, " hello word had been kill ");
        boolean killResult = false;
        try {
            killResult = CommonUtils.killProcess(this.taskRuntime, this);
            if (killResult) {
                this.writeLocalLog(Level.SEVERE, "kill job succeed!");
                this.commitTask(LState.KILLED, "", "kill job succeed!");
            } else {
                this.writeLocalLog(Level.SEVERE, "kill job failed!");
                this.commitTask(LState.HANGED, "", "kill job failed!");
            }
        } catch (Exception e) {
            this.writeLocalLog(Level.SEVERE,
                    "kill job failed:" + CommonUtils.stackTraceToString(e));
            this.commitTask(LState.HANGED, "", "kill job failed!");
        }
    }
    private void commitTaskAndLog(LState state, String runtimeId, String desc) {
        try {
            if (desc != null && desc.length() > 4000) {
                desc = StringUtils.substring(desc, 0, 4000);
            }
            this.commitTask(state, runtimeId, desc);
        }
        catch (Exception e) {
            String st = CommonUtils.stackTraceToString(e);
            this.writeLocalLog(Level.INFO, "Log_desc :" + desc);
            this.writeLocalLog(Level.SEVERE, "Commit task failed, StackTrace: " + st);
        }
    }

    public static void main(String[] args) {
        String configure = "";
        if(args == null ||args.length<1){
            System.out.println("需要配置文件作为输入参数");
            System.exit(2);
        }else{
            configure = args[0];
        }
        TaskRunner runner = new TaskRunner(configure);
        runner.startWork();
    }
}
```

### 十、新建自定义任务
登陆portal，进入如下页面   
![]()
