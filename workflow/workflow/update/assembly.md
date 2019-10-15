工作流集成代码，更新通用操作  
兼容4031,411，50+  
通常开发会提供一个全量的集成代码压缩包，下面称 lhotse.zip  
### 操作方式步骤：
#### 1. 替换
用lhotse.zip 替换 tbds-server 节点  
/var/lib/tbds-server/resources/stacks/TDP/2.2/services/LHOTSE 目录  

#### 2. 格式化
并使用dos2unix package/files/*.sh ;chmod +x -R ./格式化

#### 3. 重启
tbds-server节点执行 tbds-server restart

#### 4. 补充（不用执行）
服务节点 存储的集成代码路径
base（/var/lib/ambari-agent/cache/stacks/TDP/2.2/services/LHOTSE）,  
service（/var/lib/ambari-agent/cache/stacks/TDP/2.2/services/LHOTSE）,  
runner (/var/lib/ambari-agent/cache/stacks/TDP/2.2/services/LHOTSE）,  
tbds-server（/var/lib/tbds-server/resources/stacks/TDP/2.2/services/LHOTSE ）
