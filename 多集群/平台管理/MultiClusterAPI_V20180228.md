<font color=red>**备注：10.0.0.3为TBDS的Portal地址**</font>

# 1.集群相关 #

## 1.1创建集群 ##

**参数说明**

- clusterPassword:添加的集群管理帐号登录密码
- clusterType:添加的集群类型(CDH或者TBDS)
- clusterUser:添加的集群管理帐号登录名
- description:添加的集群描述
- identification:添加的集群标识
- ownerIds:添加的集群属主id
- owners:添加的集群属主名
- serverIp:添加的集群登录IP
- serverPort:添加的集群登录IP的端口

<pre>
curl -H "Content-Type:application/json" -X POST --data '{
		
		"cluster": {
		"identification": "TESTORG",
		"clusterType": "CDH",
		"serverIp": "10.0.0.30",
		"serverPort": 7180,
		"ownerIds": "1",
		"owners": "admin",
		"clusterUser": "admin",
		"clusterPassword": "admin",
		"description": "xxx"
	}
}'  --header 'Authorization:XXXX'  http://10.0.0.3/api/multicluster/addNewCluster
</pre>



**请求返回结果:返回创建好的集群ID**

<pre>{
    "resultCode": "0",
    "message": null,
    "resultData": 5,
	"markinfo": ""
}</pre>

## 1.2获取集群列表 ##

<pre>curl -X GET  --header "Authorization:XXXX" http://10.0.0.3/api/multicluster/query</pre>

**返回结果:返回所有集群列表**

<pre>{
    "resultCode": "0",
    "message": null,
    "resultData": [{
        "id": 1,
        "identification": "default_cluster",
        "clusterName": "tdw",
        "clusterType": "TBDS",
        "serverIp": "10.0.0.3",
        "serverPort": 8088,
        "ownerIds": "1",
        "owners": "admin",
        "clusterUser": "admin",
        "clusterPassword": "admin",
        "status": "1",
        "description": null,
        "creator": "admin",
        "createTime": "2017-10-12 18:24:11",
        "updateTime": "2017-10-12 19:00:29"
    }]
}
</pre>

# 2.项目相关 #
## 2.1创建项目 ##

**参数说明**

- name:项目名称
- identification:项目标识
- remark:项目备注
- clusterId:项目关联的集群id
- managerIds:项目的责任人id，多个责任人逗号分隔

<pre>
curl -H "Content-Type:application/json" -X POST --data '{
    "businessName": "业务_001",
    "name": "测试项目",
    "identification": "project_tbds_001",
    "clusterId": 1,
    "remark": "test for api",
    "managerIds": "1"
}'  --header 'Authorization:XXXX'  http://10.0.0.3/api/project/addNewProject
</pre>

**返回结果：新建项目的ID**
<pre>
{
    "resultCode": "0",
    "message": "ok",
    "resultData": "4"
}
</pre>


## 2.2根据项目ID获取项目信息 ##
**参数说明**

- id:项目的ID

<pre>
curl -X GET --header 'Authorization:XXXX' http://10.0.0.3/api/project/findProjectById?id=4
</pre>
**返回结果**
<pre>
{
    "resultCode": "0",
    "message": null,
    "resultData": {
        "projectId": 4,
        "identification": "project_tbds_001",
        "projectName": "测试项目",
        "leagerNum": 1,
        "resourceName": null,
        "vcores": null,
        "memory": null,
        "hdfsSize": null,
        "createTime": 1508576881000,
        "managers": [{
            "id": 1,
            "userName": "admin",
            "realName": "admin",
            "wechat": null,
            "passWord": "",
            "roles": null,
            "org": null,
            "tel": null,
            "email": null,
            "creator": "systemInit",
            "createTime": null,
            "yn": "y",
            "remark": null
        }],
        "groups": [],
        "userGroups": null,
        "remark": "test for api",
        "org": null,
        "clusterName": "default_cluster"
    }
}
</pre>

## 2.3为项目增加项目角色 ##
**参数说明**

-  targetIds:用户ID，多个用户时以逗号分隔
-  roleIds:角色ID，多个角色时以逗号分隔

<pre>
curl -H "Content-Type:application/json" -X POST --data '[{
	"projectIds": "4",
	"targetIds": "1,2",
 	"roleIds": "4,3"
 }]' --header 'Authorization:XXXX'  http://10.0.0.3/api/project/addProjectLeagerRole
</pre>
**返回结果**
<pre>
{
    "resultCode": "0",
    "message": null,
    "resultData": null
}
</pre>
## 2.4修改项目成员的角色 ##
**参数说明**

-  targetIds:用户ID，多个用户时以逗号分隔
-  roleIds:角色ID，多个角色时以逗号分隔

<pre>
curl -H "Content-Type:application/json" -X POST --data '[{
    "projectIds": "4",
    "targetIds": "2",
    "roleIds": "3,2,4"
}]'  --header 'Authorization:XXXX' http://10.0.0.3/api/project/modifyProjectLeagerRole
</pre>
**返回结果**
<pre>
{
    "resultCode": "0",
    "message": null,
    "resultData": null
}
</pre>
## 2.5删除项目的用户角色 ##
**参数说明**

- projectId:项目id
- userIds:删除角色的用户id，多个以逗号分隔

<pre>
curl -X GET --header 'Authorization:XXXX' 'http://10.0.0.3/api/project/removeLeaguerFromProject?projectId=4&userIds=2'
</pre>
**返回结果**
<pre>
{
    "resultCode": "0",
    "message": null,
    "resultData": null
}
</pre>
## 2.6删除项目 ##
**参数说明**

-  ids:项目ID，多个项目以逗号分隔

<pre>
curl -X GET --header 'Authorization:XXXX' http://10.0.0.3/api/project/deleteProjects?ids=13
</pre>
**返回结果**
<pre>
{
    "resultCode": "0",
    "message": "ok",
    "resultData": null
}
</pre>
## 2.7设置项目环境变量 ##
**参数说明**

-   约定 kafka分区数量的常量名：NUM_PARTITION_KAFKA，在数据接入Kafka模块中，会判断该项目是否有kafka分区数量 的限制，若有，则创建的topic的分区数 不大于 NUM_PARTITION_KAFKA 的值。
-   projectFilesystemPath：设置文件系统的目录
-   activeTaskOperator：是否启用工作流的统一操作用户
-   operator：统一操作用户的用户名
-   operatorId：统一操作用户的id

<pre>
curl -H "Content-Type:application/json" -X POST --data '{
    "projectId": 16,
    "variables": {
        "NUM_PARTITION_KAFKA": "10"
    },
    "projectFilesystemPath": "/project/project_iden001",
    "activeTaskOperator": false,
    "operator": "",
    "operatorId": ""
}' --header 'Authorization:XXXX' http://10.0.0.3/api/project/updateProjectEnv
</pre>
**返回结果**
<pre>
{
    "resultCode": "0",
    "message": null,
    "resultData": true
}
</pre>
# 3.资源划分 #
## 3.1划分HDFSquota资源 ##
**参数说明**

- directory:划分HDFS的文件路径
- maxFileNum:最大文件数
- maxDiskSpace:最大占用磁盘空间
- enableAllocateResource:false—不进行真正划分资源；true—真正划分HDFSquota

<pre>
curl -H "Content-Type:application/json" -X POST --data '{
    "project": "4",
    "directory": "/project/project_tbds_001/",
    "maxFileNum": 71283,
    "maxDiskSpace": 301330651,
    "fileNumLimited": false,
    "diskSpaceLimited": true,
    "enableAllocateResource": false
}' --header 'Authorization:XXXX' http://10.0.0.3/api/resource/quotas/config
</pre>


**返回结果:resultData是HDFS划分对象quota的id**
<pre>
{
    "resultCode": "0",
    "message": null,
    "resultData": "885c3dbb-ce59-4f4c-b29a-e614c06d78bc"
}
</pre>
## 3.2划分YarnQUEUE资源 ##
**参数说明**
    
-  "name":项目名称
-  "project":项目ID
-  "weight":权重
-  "minVCore":最小VCore数
-  "maxVCore":最大VCore数
-  "minMemory":最小内存数
-  "maxMemory":最大内存数
-  "maxAppNum":最大应用数
-  "maxAppMasterRatio":最大APPMasterRatio
-  "enableAllocateResource":是否正式实施还是只做项目记录，true为实际操作，false仅作项目记录
<pre>
curl -H "Content-Type:application/json" -X POST --data '{
    "name": "project_tbds_001",
    "project": "4",
    "weight": 1,
    "minVCore": 0,
    "maxVCore": 30,
    "minMemory": 0,
    "maxMemory": 819,
    "maxAppNum": 100,
    "maxAppMasterRatio": 0.5,
    "enableAllocateResource": false
}' --header 'Authorization:XXXX'  http://10.0.0.3/api/resource/queues/config
</pre>
**返回结果:resultData为划分的YarnQueue的资源id**
<pre>
{
    "resultCode": "0",
    "message": null,
    "resultData": "1d185977-eae2-4e86-98cc-58714f76e13a"
}
</pre>

对HDFS,Yarn的更新操作，接口相同，需要传入对应的id即可
<pre>
post /api/resource/quotas/config  # 更新HDFS quotas
{
    "project": "16",
    "directory": "/project/hello_test1111/",
    "maxDiskSpace": 1615981445120,
    "fileNumLimited": false,
    "diskSpaceLimited": true,
    "id": "c259d654-6006-49f8-84ef-f53e9d0b72d9"  # quotas_id
}


post /api/resource/queues/config # 更新yarn queue
{
    "name": "queue_res_name",
    "project": "16",
    "weight": 1,
    "minVCore": 0,
    "maxVCore": 32,
    "minMemory": 0,
    "maxMemory": 65536,
    "maxAppNum": 100,
    "maxAppMasterRatio": 0.5,
    "id": "42b61eb2-c140-4223-859a-31a80d775067"  # queue_id
}
</pre>
## 3.3删除HDFSquota资源 ##
**参数说明**

-  quota_id:项目的HDFSquota对象的ID
<pre>
curl -X DELETE --header 'Authorization:XXXX' http://10.0.0.3/api/resource/quotas/config/{quota_id}
</pre>
**返回结果**
<pre>
{
    "resultCode": "0",
    "message": null,
    "resultData": true
}
</pre>
## 3.4删除YarnQUEUE资源 ##
**参数说明**

-  queue_id:YarnQueue的资源id
<pre>
curl -X DELETE --header 'Authorization:XXXX' http://10.0.0.3/api/resource/queues/config/{queue_id}
</pre>
**返回结果**
<pre>
{
    "resultCode": "0",
    "message": null,
    "resultData": true
}
</pre>
## 3.5根据项目ID获取quotas信息 ##
**参数说明**

-  projectIds:项目ID
<pre>
curl -X GET --header 'Authorization:XXXX' http://10.0.0.3/api/resource/quotas/project-config?projectIds={projectId}
</pre>
**返回结果**
<pre>
{
    "resultCode": "0",
    "message": null,
    "resultData": [{
        "requestId": 0,
        "id": "78abb749-1716-4d3c-bef4-a9cbf5f39ae0",
        "project": "19",
        "projectName": null,
        "directory": "/project/project_test_001/",
        "fileNumLimited": false,
        "maxFileNum": 7137672,
        "diskSpaceLimited": true,
        "maxDiskSpace": 2074388627456,
        "timestamp": "2017-11-09 16:48:36",
        "usedFileNum": 0,
        "usedDiskSpace": 0,
        "enableAllocateResource": true
    }]
}
</pre>
## 3.6根据项目ID获取queue信息 ##
**参数说明**

-  projectId:项目ID
<pre>
curl -X GET --header 'Authorization:XXXX' http://10.0.0.3/api/resource/queues/project-config?projectIds={projectId}
</pre>
**返回结果**
<pre>
{
    "resultCode": "0",
    "message": null,
    "resultData": [{
        "id": "700b0378-72a9-40a2-915a-f212f1eab06d",
        "name": "project_test_001",
        "project": "19",
        "weight": 1.0,
        "org": null,
        "percent": 0.0,
        "minVCore": 0,
        "maxVCore": 32,
        "minMemory": 0,
        "maxMemory": 65536,
        "maxAppNum": 100,
        "maxAppMasterRatio": 0.5,
        "requestId": 0,
        "timestamp": "2017-11-09 16:48:43",
        "projectName": null,
        "enableAllocateResource": true
    }]
}
</pre>
## 3.7设置项目申请资源的状态 ##
**参数说明**

-  service:对应申请资源的服务，支持服务：HDFS、HIVE
-  status:申请资源的状态，默认的状态是 1（允许申请资源），申请成功后，航信调用该接口，传入状态0（不允许申请资源），避免项目重复申请资源。

<pre>
curl -H "Content-Type:application/json" -X POST --data '{
    "projectId": 41,
    "userId": 1,
    "service": "HDFS",
    "status": "0"
}'  --header 'Authorization:XXXX'  http://10.0.0.3/api/project/updateProjectApplyStatus
</pre>
**返回结果**
<pre>
{
    "resultCode": "0",
    "message": null,
    "resultData": true
}
</pre>
# 4.用户体系 #
## 4.1获取全部有效用户 ##
**参数说明**

-  pageIndex:分页索引
-  pageSize:分页大小
<pre>
curl -X GET --header 'Authorization:XXXX' http://10.0.0.3/api/users/findUsers?pageSize=1000&pageIndex=0
</pre>
**返回结果**
<pre>
{
    "resultCode": "0",
    "message": "success",
    "resultData": {
        "content": [{
            "id": 5,
            "userName": "user_001",
            "realName": "某某某",
            "wechat": "",
            "passWord": "",
            "roles": "",
            "org": null,
            "tel": "",
            "email": "user_03@qq.com",
            "creator": "admin",
            "createTime": 1508211660000,
            "yn": "y",
            "remark": ""
        },
        {
            "id": 1,
            "userName": "admin",
            "realName": "admin",
            "wechat": null,
            "passWord": "",
            "roles": "",
            "org": null,
            "tel": null,
            "email": null,
            "creator": "systemInit",
            "createTime": null,
            "yn": "y",
            "remark": null
        }],
        "last": true,
        "totalElements": 5,
        "totalPages": 1,
        "firstPage": true,
        "lastPage": true,
        "first": true,
        "sort": null,
        "numberOfElements": 5,
        "size": 1000,
        "number": 0
    }
}
</pre>
## 4.2创建用户 ##
**参数说明**

-  realName:用户姓名(必填)
-  userName:登录名(必填)
-  email:邮箱地址(必填)
-  password:密码(必填)
-  tel:电话(选填)
-  wechat:微信(选填)
-  remark:用户描述(选填)
<pre>
curl -H "Content-Type:application/json" -X POST --data '{
    "realName": "某某某hello",
    "userName": "user_007",
    "email": "user_03@qq.com",
    "passWord": "user_02123456",
    "tel": "",
    "wechat": "",
    "remark": ""
}' --header 'Authorization:XXXX' http://10.0.0.3/api/users/addNewUser
</pre>
**返回结果:新用户的用户ID**
<pre>
{
    "resultCode": "0",
    "message": "success",
    "resultData": "6"
}
</pre>
## 4.3根据用户ID查询用户信息 ##
**参数说明**

-  userId:用户ID
<pre>
curl -X GET --header 'Authorization:XXXX' http://10.0.0.3/api/users/findUserById?userId=6
</pre>
**返回结果**
<pre>
{
    "resultCode": "0",
    "message": "success",
    "resultData": {
        "id": 6,
        "userName": "user_007",
        "realName": "某某某hello",
        "wechat": "",
        "passWord": "",
        "roles": null,
        "org": null,
        "tel": "",
        "email": "user_03@qq.com",
        "creator": "1",
        "createTime": 1508578396000,
        "yn": "y",
        "remark": ""
    }
}
</pre>
## 4.4删除用户 ##
**参数说明**

-  userIds:用户ID，多个ID以逗号分隔 
<pre>
curl -X GET --header 'Authorization:XXXX' http://10.0.0.3/api/users/deleteUsers?userIds=5
</pre>
**返回结果**
<pre>
{
    "resultCode": "0",
    "message": "success",
    "resultData": null
}
</pre>
# 5.数据管理 #
## 5.1创建库表管理的数据库 ##
**参数说明**
-  dbName:数据库名称
-  dbDesc:数据库描述
-  dbType:数据库类型，支持Hive,HBase
-  projectIdent:数据库所属项目的项目标识
-  projectName:数据库所属项目的项目名称
-  projectId:数据库所属项目的项目ID
-  owner:数据库的负责人用户名
-  ownerId:数据库的负责人用户ID
-  enableAllocateResource:是否到对应类型数据库真实操作，true为实际操作，false为只在库表管理操作相关记录
<pre>
curl -H "Content-Type:application/json" -X POST --data '{
    "dbName": "hello_cdh_manager",
    "dbDesc": "dsdsdsds",
    "dbType": "hive",
    "projectIdent": "project_001",
    "projectName": "project_001",
    "projectId": 1,
    "owner": "admin",
    "ownerId": "1",
    "enableAllocateResource": false
}' --header 'Authorization:XXXX' http://10.0.0.3/api/tbdsmeta/database/create
</pre>
**返回结果**
<pre>
{
    "resultData": "create db success: hello_cdh_manager",
    "resultCode": "0",
    "message": null
}
</pre>
## 5.2删除手动创建的数据库 ##
**参数说明**

-  objectId:数据库对象ID
-  enableAllocateResource:是否到对应类型数据库真实操作，true为实际操作，false为只在库表管理操作相关记录
<pre>
curl -H "Content-Type:application/json" -X POST --data '{
    "objectId": "811600b6-4b43-4bf4-8299-12b15541dcbe",
    "enableAllocateResource": false
}' --header 'Authorization:XXXX'  http://10.0.0.3/api/tbdsmeta/database/delete
</pre>
**返回结果**
<pre>
{
    "resultData": "delete database success ",
    "resultCode": "0",
    "message": null
}
</pre>
# 6.多集群相关 #
## 6.1获取公共服务配置信息 ##
**参数说明**
<pre>
curl -X GET  --header "Authorization:XXXX" http://10.0.0.3/api/multiserver/query
</pre>
**返回结果**
<pre>
{
	"resultCode": "0",
	"message": null,
	"resultData": [{
		"id": 28,
		"serverName": "ORACLE_ALL",
		"serverType": "ORACLE",
		"multiHired": "0",
		"creator": "wliang",
		"status": "1",
		"properties": "{\"host\":\"10.0.0.0\",\"parallelism\":\"10\",\"password\":\"******\",\"port\":\"****\",\"service\":\"****\",\"user_name\":\"*****\"}",
		"description": "",
		"clusterId": null,
		"projectId": null,
		"createTime": "2018-01-03 15:07:50",
		"updateTime": "2018-01-03 16:29:32",
		"jsonProperties": {
			"password": "******",
			"port": "****",
			"service": "****",
			"user_name": "****",
			"parallelism": "10",
			"host": "10.0.0.0"
		},
		"enableChangeMultiHired": false,
		"projectName": null
	}],
	"markInfo": "xxx"
}
</pre>
