## 手动切换base所在节点
在有一些版本中，比如4031 以前的版本（包括4031早起版本），是没有提供base 高可用的，所以遇见base所在节点机器挂掉，需要手动切换base节点。  

## 前提：  
数据库对应的主备同步正常。

## 1. 选择新节点安装base
a. base 对内存要求比较高，建议选择内存比较大，内存使用率不是高的节点。  
b. base 需要连接metadata 数据库，尽量选用无其他服务连接metadata 的节点。

## 2. 安装base
通过portal 页面新增base 组件。

## 3. 后续工作
a. 启动base ,确认base 启动正常（等两分钟确认base 组件状态正常，或直接查看base 日志）  
b. 重启lhotse service  
c. 重启所有 lhotse runner  
