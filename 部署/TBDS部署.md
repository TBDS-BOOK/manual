## 前置条件
软件清单
* tbds-mirror-$**version**.tar.gz   (其中$version为TBDS版本号) <font color="red">\*</font>
* tbds-mirror-patch-$**version**.tar.gz
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

创建集群描述文件

	# cd /data
	# vim cluster.info
![](cluster.info.jpg)

* <font color="red">注：portal节点需要在cluster.info文件的第一行, 此处假设用户/密码为root/123456</font>

初始化集群并部署portal

    # sh tbds-bootstrap.sh init
    ...
    # sh tbds-install-portal.sh

部署成功后会显示portal的访问地址，接下来通过portal进行TBDS集群部署

## 部署TBDS集群
* 登陆portal(默认用户密码: **admin/123456**)

![](初次登陆.jpg)

* 输入License, 阅读用户协议，并开始

![](输入License.jpg)

* 选择需部署的服务

![](选择服务.jpg)

* 输入服务器列表

![](输入服务器列表.jpg)

* 服务主机分配(根据实际情况调整)

![](服务主机分配.jpg)

* 服务配置（多磁盘需要在此处调整各服务配置,如下示例）

![](服务配置.jpg)

* 开始部署

![](部署过程.jpg)





