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
	![](./部署/cluster.info.jpg)



## 部署TBDS集群


