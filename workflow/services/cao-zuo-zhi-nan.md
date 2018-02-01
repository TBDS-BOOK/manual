### 服务器配置操作
点击服务器配置右上角按钮【新建配置】创建新增服务器配置。
![](/assets/4.png)

####1、MYSQL
选择服务器类型:mysql，进行MYSQL类型的服务器配置，配置完成后，可以点击【连接测试】按钮，校验服务器配置是否有效。【连接测试】会根据所填的服务器信息去进行连接的测试，判断连接是否正常。

参数说明：
- 服务器类型：选择服务器的类型，目前支持8种：mysql, postgres, oracle, sqlserver, hive, ftp, hdfs, mr-group；
- 服务器标识：设置服务器的唯一标识，创建后不支持修改；
- 服务器说明：服务器的备注信息，便于管理、识别；
- 责任人：服务器只能被责任人进行修改；
- 主机地址：MYSQL的连接IP地址，如：10.0.0.1；
- 端口：MYSQL的连接端口，默认：3306
- database名称：服务器配置所指向的MYSQL database；
- 数据库用户名：连接MYSQL并进行后续相应任务操作的用户名；
- 数据库密码：连接MYSQL的用户所对应的密码；
- 服务器并发度：并行执行的数量，默认是10；

![](/assets/3.png)


####2、POSTGRES
选择服务器类型:postgre，进行POSTGRES类型的服务器配置。

- 主机地址：POSTGRES的连接IP地址，如：10.0.0.1；
- 端口：POSTGRES的连接端口，默认：5432
- database名称：服务器配置所指向的POSTGRES database；
- 数据库用户名：连接POSTGRES并进行后续相应任务操作的用户名；
- 数据库密码：连接POSTGRES的用户所对应的密码；

![](/assets/5.png)


####3、ORACLE
选择服务器类型:oracle，进行ORACLE类型的服务器配置。
- 主机地址：ORACLE的连接IP地址，如：10.0.0.1；
- 端口：ORACLE的连接端口，默认：1521；
- database名称：ORACLE数据库的实例名，常用：orcl；
- 数据库用户名：连接ORACLE并进行后续相应任务操作的用户名；
- 数据库密码：连接ORACLE的用户所对应的密码；

![](/assets/oracle.png)


####4、SQLSERVER
选择服务器类型:sqlserver，进行SQLSERVER类型的服务器配置。
- 服务器版本：SQLSERVER的版本：MSSQL_2008：SQLSERVER 2008版本；
- 主机地址：SQLSERVER的连接IP地址，如：10.0.0.1；
- 端口：SQLSERVER的连接端口，默认：1433；
- database名称：SQLSERVER连接的database
- 数据库用户名：连接SQLSERVER并进行后续相应任务操作的用户名；
- 数据库密码：连接SQLSERVER的用户所对应的密码；

![](/assets/sever.png)

####5、FTP
选择服务器类型:ftp，进行FTP类型的服务器配置。
- 主机地址：FTP服务的连接IP地址，如：10.0.0.1；
- 端口：FTP的连接端口，默认：2222；
- ftp用户名：连接FTP用户
- ftp用户密码：连接FTP的用户所对应的密码；

**备注：** 选择创建FTP服务器配置时，默认会加载套件内安装的FTP服务的配置信息，如果新增非套件内的FTP服务器，则修改对应的连接信息即可。

![](/assets/ftp.png)

####6、HIVE
选择服务器类型:sqlserver，进行SQLSERVER类型的服务器配置。





####7、HDFS



####8、MR_GROUP









