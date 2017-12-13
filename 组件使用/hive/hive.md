## 一 Hive使用说明
Hive的使用与开源没有本质区别，需要注意的主要有两点：1）Hive打开了Ldap认证，连接Hive时，需要设置用户名和密码；2）Hive实现了HA，通过在zk注册多个hiveserver实例实现，多个hiveserver并行工作，工作时，通过zk路由到某个可用的hiveserver上，各hiveserver之间互相独立。
### 1.1 Hive认证
Hive认证通过打开hive自带的Ldap认证功能来实现，连接hive时的用户名和密码被传递给HiverServer, HiveServer去Ldap里面进行比较。认证所用的用户有两种,两种用户都可以在连接hive时进行认证：
1）用户在大数据套件界面（以下简称portal界面）创建的用户和密码
2）系统自带默认用户（包括：hive,httpfs,supersql,tbds_shellUser,root,olap,hbase,hdfs,yarn,mapred,ranger,amshcat,hue,tez,zookeeper,ambari-qa,hadoop,flume,tencent,jstorm,kafka,lhotse,hdfs,postgres,redis,wherehows,hermes），这些默认用户及其密码都保存在套件portal节点的Ldap里，各个用户的初始密码为：用户名+@Tbds.com, 如hive用户的密码为：hive@Tbds.com  hdfs 用户的密码为：hdfs@Tbds.com ，如果想更改这些用户的初始密码，可以通过ldap命令直接去ldap修改。 建议这些默认用户不暴露给最终用户使用，最终用户所使用的用户都是在portal页面创建的用户， 这里说明系统用户默认密码主要是为了方便套件上层开发的应用程序访问。
### 1.2使用Hive
1)连接hive时，有两种不同的方式，对应的连接字符串也有所不同：
直接连接到指定的hiveserver,如： jdbc:hive2://172.16.32.10:10000 其中，172.16.32.10 为hiveserver ip, 10000为hiveserver port
通过zk的方式连接hive,如：jdbc:hive2://172.16.32.14:2181,172.16.32.13:2181,172.16.32.8:2181/default;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2
其中，172.16.32.14:2181,172.16.32.13:2181,172.16.32.8:2181为zk ip和port ，使用时其他的字符串保持不变 
注意：建议通过zk的方式连接zk，可以保证hive的高可用性。
2）根据客户端的使用方式不同，有以下几种：
beeline方式连接，去安装有hive client的节点执行 beeline命令： !connect jdbc:hive2://172.16.32.10:10000 其中，链接字符串可以使用上节介绍的任意一种（推荐zk方式）
java客户端使用jdbc驱动连接，连接字符串可以任意一种，用户名和密码参见《Hive认证》章节，需要引用hive的pom依赖为
<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-jdbc</artifactId>
    <version>2.2.0-SNAPSHOT</version>
</dependency>
使用Thrift接口去连接Hive,使用方法和开源完全相同，传递用户名和密码时，参考《Hive认证》 章节即可
### 1.3 集群外部署Hive Client
方案1：
	1）通过yum源(提前配置好套件的yum源)安装套件版本的Hive到集群外机器：
		yum install  hive_*  hiveclient.x86_64  -y	  
	2）拷贝集群内任一hive客户端机器/etc/hive/conf目录的内容，替换集群外客户端对应路径:
	集群内客户端节点：
		cd  /etc/hive/conf
		tar -zcvf  hive.tar.gz  /etc/hive/conf/*
	集群外客户端节点：
		cd  /etc/hive/conf
		tar -zxvf  hvie.tar.gz
方案2：
	1.打包集群内任一hive客户端所在节点的/usr/hdp/2.2.0.0-2041/hive/，并拷贝到集群外目标客户端节点
	2.同方案1的步骤2
### 1.4客户端代码编译打包
1) 拷贝或部署套件提供的maven库到开发者可访问的本地仓库或远程仓库
2) 在客户端maven工程pom引入对应的套件版hive依赖，版本号为：2.2.0-SNAPSHOT
<dependency>
			<groupId>org.apache.hive</groupId>
			<artifactId>hive-jdbc</artifactId>
            <version>2.2.0-SNAPSHOT</version>
</dependency>
<dependency>
			<groupId>org.apache.hive</groupId>
			<artifactId>hive-common</artifactId>
			<version>2.2.0-SNAPSHOT</version>
</dependency>
