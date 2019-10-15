rpm 下载地址  
http://100.76.22.204/tbds-mirror-4.1.1.0/tbds/lhotse/  
不同版本，可能对应的目录不一样（如4040 ：tbds-mirror-4.0.4.0）  

### 操作步骤：
#### 0. 确认yum 源地址
#### 1. 更新集群所使用yum 源
#### 2. 更新yum 源索引
在路径 /data/rpm/tbds 目录执行  
yum createrepo .
#### 3. 停止taskScheduler 服务（通过8080 页面停止，并确认停止正常）
#### 4. 升级
如果已经安装的环境，rpm 带有 RC 标示，则为git 版本
##### 4.1 svn 跨 git 升级
svn 跨 git 升级(需要到每个服务对应节点后台执行) 先卸载，再升级
** 4.1 svn 跨 git 升级(需要到每个服务对应节点后台执行) 先卸载，再升级**

###### 4.1.1 yum remove lhotse-XXX;

###### 4.1.2 需要确保对应服务相关目录被清理（service 为 /usr/local/lhotse_service，base 为/usr/local/lhotse_base，runner 为/usr/local/lhotse_runners），如果有残余目录(log或logs 目录不影响，而且应该保留)，手动清理即可。

###### 4.1.3 yum clean all

###### 4.1.4 yum install -y lhotse-XXX

##### 4.2 git 版本升级(需要到每个服务对应节点后台执行)
直接升级(yum clean all;yum update -y lhotse-XXX)
