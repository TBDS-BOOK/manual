## 一 大数据Accesskey认证

大数据套件的主要服务（如Hadoop，HBase，Kafka，Hippo等）采用Accesskey的方式进行认证。这里主要介绍其概念，生成和获取方式。

### 1.1 概念

Accesskey由一对字符串（secureId和secureKey）组成。服务认证时，需要这一对字符串才能完成，其中secureId对应着某一个用户，认证成功后，就以该用户的身份去操作相应的服务（如Hadoop,HBase,Kafka，Hippo），同时accesskey在生成时，和某一个服务对应，假如accesskey对应着HBase服务，那么他只能在HBase认证时才有效，而无法对HDFS服务进行认证。

### 1.2创建Accesskey

Accesskey在大数据套件portal界面由系统管理员（具有Administrator角色的用户）创建，其创建步骤如下：

  
1）使用系统管理员登录portal

![](/组件使用/accesskey/login.png)

2）进入平台管理  
![](/组件使用/accesskey/portal_main.png)

3）进入用户管理  
![](/组件使用/accesskey/user_manage_main.png)

4）进入密钥管理  
![](/组件使用/accesskey/portal_accesskey.png)

5）点击右上角 新建密钥  
![](/组件使用/accesskey/portal_new_acckey.png)

6）配置信息  


![](/组件使用/accesskey/new_acckey_config.png)

用户选择：  
  选择portal创建的用户，与预创建的Accesskey绑定，认证成功后，以该用户的身份进行操作对应的服务  
模块选择：  
  该Accesskey要认证的服务，选择那个服务，则只能认证那个服务，要认证其他的服务，需要创建对应服务的Accesskey。目前可选的服务为：ALL，元数据，全局工作流，HIPPO，KAFKA，HBASE，WHEREHOWS，HADOOP，其中有两个服务的accesskey有些特殊：

1）ALL可以认证所有的服务；

2）HADOOP可以认证HDFS和YARN服务。  
  
类型选择：ALL，当前没有实际含义，扩展字段，可以忽略。

  
7）点击确定，则生成accesskey。  
![](/组件使用/accesskey/gen_acckey.png)

注意：  
1）accesskey被创建后，默认是关闭的，只有在 开启 后，才能进行认证，否则会认证失败。  
![](/组件使用/accesskey/enable_acckey.png)

2）点击小眼睛可以展开accesskey  
![](/组件使用/accesskey/inspect_acckey.png)

### 1.3 获取Accesskey

1）系统管理员用户可以在Accesskey创建界面查看所有用户的accesskey  
2）普通用户，可以在个人中心参看自己相关的Accesskey，步骤如下：  
![](/组件使用/accesskey/portal_main.png)![](/组件使用/accesskey/use_acckey.png)

