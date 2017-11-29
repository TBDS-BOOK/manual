## 前置条件
软件清单
* tbds-mirror-$**version**.tar.gz   (其中$version为TBDS版本号) <font color="red">\*</font>
* tbds-mirror-patches-$**version**.tar.gz
* tbds-bootstrap.sh <font color="red">\*</font>
* tbds-install-portal.sh <font color="red">\*</font>
* tbds-cli.sh

License
* 请联系腾讯大数据商务或供应商

系统
* 部署TBDS需要root用户
* 请提前格式化并挂载好数据盘


## 部署Portal节点

> 确保上述软件清单软件已经在 **/data **目录

修改 **tbds-bootstrap.sh**

    ROOTPWD="123456"    # 主机root密码，用于portal部署和机器初始化
    DATAETH=eth0        # 用于内网数据传输的网卡，多网卡机型必须填写

修改 **tbds-install-portal.sh**

    DATAETH=eth0        # 用于内网数据传输的网卡，多网卡机型必须填写

创建 **cluster.info** 集群描述文件

	# cd /data
	# vim cluster.info

![](cluster.info.jpg)

* <font color="red">注：portal节点需要在cluster.info文件的第一行, 此处假设root用户密码为123456</font>

初始化集群并部署portal

    # sh tbds-bootstrap.sh init
    ...
    # tar xzf tbds-mirror* -C /data
    ...
    # sh tbds-install-portal.sh
    ...
    # sh tbds-bootstrap.sh postinit

部署成功后会显示portal的访问地址，接下来通过portal进行TBDS集群部署

## 部署TBDS集群
* 登陆portal(默认用户密码: **admin/123456**)

![](初次登陆.jpg)

* 输入License, 阅读用户协议，并开始

![](输入License.jpg)

* 选择需部署的服务

![](选择服务.jpg)

* 输入服务器列表(portal节点需要在第一行)

![](输入服务器列表.jpg)

* 服务主机分配(根据实际情况调整)

![](服务主机分配.jpg)

* 服务配置（**多磁盘**需要在此处调整各服务配置,如下示例）

![](服务配置.jpg)

* 开始部署

![](部署过程.jpg)

集群部署完成大概需要30分钟，完成后请注意<font color="red">修改admin用户的密码</font>



