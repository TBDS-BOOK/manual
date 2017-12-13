## 一．访问认证

套件版本在安全访问层面上开发了Tbds认证，相对于社区版本提供的Simple认证和Kerberos认证，TBDS认证在保证安全的基础下，使用上更加灵活简洁。TBDS认证访问hadoop只需要提供用户认证信息：认证ID，用户名，认证密钥。

```
    hadoop_security_authentication_tbds_secureid
    hadoop_security_authentication_tbds_username
    hadoop_security_authentication_tbds_securekey
```

** 获取认证信息 **

admin用户可以在portal创建所有用户的securekey密钥。普通用户需要securekey密钥则向管理员申请。

** 客户端访问Hadoop **

客户端访问Hadoop只需要两步：

_ jar包替换_** **

将客户端相关hadoop jar包替换成套件hadoop ja包，目前是2.7.2-TBDS-4.0.3.1版本。

** **_设置环境环境变量_

设置环境变量三种方式  
Shell访问  
　　环境变量，给客户端程序加3个环境变量，如:

```
　　export hadoop_security_authentication_tbds_secureid=F3QdVfxbQkNHVkn1OzLA3yK3In0bL6HgX
　　export hadoop_security_authentication_tbds_username=hdfs
　　export hadoop_security_authentication_tbds_securekey=o8AnGFYQ2lIB0AJ78TIeoJ0Uu1nkph12
```

配置文件

配置文件，放到客户端程序的hadoop配置文件中，如classpath下的core-site.xml或hdfs-site.xml：

```
　　<property>
　　　　<name>hadoop_security_authentication_tbds_secureid</name>
　　　　<value>F3QdVfxbQkNHVkn1OzLA3yK3In0bL6HgXPmK</value>
　　</property>
　　<property>
　　　　<name>hadoop_security_authentication_tbds_username</name>
　　　　<value>hdfs</value>
　　</property>
　　<property>
　　　　<name>hadoop_security_authentication_tbds_securekey</name>
　　　　<value>o8AnGFYQ2lIB0AJ78TIeoJ0Uu1nkph12</value>
　　</property>
```

代码设置

代码设置，在hadoop client代码里加入代码片段

```
conf.set("hadoop_security_authentication_tbds_secureid","F3QdVfxbQkNHVkn1OzLA3yK3In0bL6Hg");
　　conf.set("hadoop_security_authentication_tbds_username","hdfs");
　　conf.set("hadoop_security_authentication_tbds_securekey","o8AnGFYQ2lIB0AJ78TIeoJ0Uu1nkph12");
```

### 1.3 Webhdfs或HttpFs访问

同客户端访问hadoop原理一样，同样需要导入一些认证信息，但有web访问需要做额外的处理，我们需要手动的生成签名，客户端访问不用只是我们已经在代码里面做好这份工作。

#### 1.3.1 生成签名

生成签名，通过（secureId，currentTimeStamp，randomInt，secureKey）---&gt;signature。其中currentTimeStamp是当前时间戳，randomInt是一个随机数，Signature是我们要的输出

java示例：

```
private String generateSignature(String secureId, long timestamp, int randomValue, String secureKey){
　　Base64 base64 = new Base64();
　　byte[] baseStr = base64.encode(HmacUtils.hmacSha1(secureKey, secureId + timestamp + randomValue));
　　
　　String result = "";
　　try {
　　　　result = URLEncoder.encode(new String(baseStr), "UTF-8");
　　} catch (UnsupportedEncodingException e) {
　　　　LOG.error("Failed to encode.", e);
　　}
　　
　　return result;
}
```

Python示例：

```
　　def generateSignature(secureid,securekey):
　　from hashlib import sha1
　　import hmac
　　import base64
　　import urllib, urllib2
　　import time
　　import random
　　timestamp=str(long(time.time()*1000))
　　nonce=random.randint(0,2147483647)
　　seed="{0}{1}{2}".format(secureid,timestamp,nonce)
　　my_sign = hmac.new(securekey, seed, sha1).digest()
　　my_sign = base64.b64encode(my_sign)
　　signature = urllib.quote_plus(str(my_sign))
　　return "{0} {1} {2} {3}".format(secureid,timestamp,nonce,signature)
```

#### 1.3.2 web访问

header名称：tbds-auth  
header取值：secureId+" " + curTime + " " + random + " " + signature  
例子：

```
curl -i --header "tbds-auth:PxKf1JVUDHzP0ItYSzeMIaq6HSy7STURQbPO 1506411029889 639 xfVforphUwk6MRlGN7gvPmMSRME%3D"
-X GET http://tbds-10-254-100-139:14000/webhdfs/v1/user/abc?op=LISTSTATUS&user.name=httpfs
```

# 二．权限控制

Tbds统一权限控制，hdfs和yarn所有的目录访问和资源权限都有Tbds统一权限控制。用户如果需要对某些目录和资源访问需要像管理员申请权限。  
　　权限管理界面在Portal 运维中心-访问管理-访问策略 页面。  
　　![](/组件使用/hadoop/ranger_hdfs.png)  
　　  
　　  
　　对于hdfs，我们会细分到每个目录对于哪些组哪些用户有哪些使用使用权限，比如读写等：  
　　![](/组件使用/hadoop/ranger_hdfs_detail.png)  
　　  
　　  
　　  
　　对于yarn，我们细分到每个资源队列对于那些组哪些用户有哪些权限，不如提交app等。  
![](/组件使用/hadoop/ranger_yarn_detail.png)

![](/组件使用/hadoop/ranger_yarn_detail.png)

# 三．集群外客户端部署

### 3.1 客户端安装

（1）在任一安装hadoop客户端的集群内节点，打包hadoop的安装路径：/usr/hdp/2.2.0.0-2041/hadoop /，并拷贝到集群外目标客户端安装节点

（2）如果有套件的yum源，在集群外目标安装节点使用yum install安装

（3）如果有hadoop安装包，在集群外目标安装节点使用rpm -ivh安装

### 3.2 配置文件

在任一安装hadoop客户端的集群内节点，打包hadoop的配置路径： /etc/hadoop/conf，并拷贝到集群外目标客户端安装节点对应路径。

### 3.3 编译打包

套件的hadoop是基于社区二次开发，命名规则采用"社区版本号-TBDS-套件版本号"的方式命名.例：我们现在基于社区1.2.1版本的hbase进行开发，套件版本是4.0.3.3，则我们打出的hbase jar版本为1.2.1-TBDS-4.0.3.3，完整的yarn client maven jar文件名为：hadoop-yarn-client-1.2.1-TBDS-4.0.3.3.jar

#### 3.3.1基于套件提供的maven库开发

（1）拷贝或部署套件提供的maven库到开发者可访问的本地仓库或远程仓库  
　　（2）在客户端maven工程pom引入对应的套件版hadoop依赖，以套件4.0.3.3版本为例，需要在pom中加入的依赖片段（其他版本依次类推）：

```
　　 <dependency>
　　     <groupId>org.apache.hbase</groupId>
　　     <artifactId> hadoop-yarn-client</artifactId>
　　     <version>1.2.1-TBDS-4.0.3.3</version>
　　 </dependency>
```

### 3.4 运行

运行客户端代码与社区方式无区别

