Hive SQL脚本
============

步骤（1）新建工作流
-------------------

点击工作流列表右上角的“+”，在弹出的窗口中输入工作流名称“hive_sql_demo”，点击“确定”完成工作流的创建。

![](media/67361da9ef964af2775e8edb8ec47078.png)

![](media/c997336933eb70c73bdb364cce1d2b8f.png)

步骤（2）新建任务
-----------------

拖拽画布左上角“+”，在弹出的窗口中选中“HIVE SQL脚本”，点击“确定”完成任务的创建。

![](media/622096ffacb99c24b3232ce8480db615.png)

![](media/5a0917feab50ad592791bd8bc779abc5.png)

步骤（3）填写任务基本信息
-------------------------

右键点击任务框，点击“编辑”进入任务的编辑界面。

![](media/928ff774d3d001a32cef18973cd7392a.png)

在“基本信息”中填写任务名称“hive_sql_demo”。

![](media/1312fd40cffd668c3c586de5f46251c1.png)

步骤（4）填写任务调度设置
-------------------------

在“调度设置”中选择周期类型“一次性非周期”。

![](media/926fb587fcacfdde494a5d748751bca2.png)

步骤（5）填写任务参数设置
-------------------------

在“参数配置”中选择源服务器“hive_server”。点击“上传脚本”，在弹出的窗口中上传Hive
SQL脚本文件“hive_sql_demo.sql”。脚本文件内容：

show databases;

use hive_test;

select \* from mysql_user;

select count(\*) from mysql_user;

注意：

（1）务必确保任务的第一责任人有权限访问脚本文件中的Hive表；

（2）脚本中可以使用时间参数\${YYYYMMDDHH}，运行时会使用实例时间进行替换；

（3）如果脚本文件含有中文，务必确保文件是无BOM的UTF-8编码格式。

![](media/cc28f2dc0dd50d42ad43c0382d02bc51.png)

![](media/3cde27d743b182ede7ac0eb8badeddeb.png)

点击保存图标保存任务配置。

![](media/694be163554c5b5f02e29c13cdb07606.png)

步骤（6）任务审批
-----------------

返回工作流画布。右键点击任务框，点击“运行”，在弹出的窗口中勾选“审批通过后自动运行”，点击“确定”。

![](media/5e06d019c0f9e9b3e994069c2305291e.png)

![](media/85729ce3875b1d69cdfa9dcbe64942f1.png)

任务进入“审批中”状态，等待审批结果。

![](media/5645a54142c42b90f696de046a8a5659.png)

步骤（7）查看任务状态
---------------------

审批通过后，任务进入“运行”状态，右键任务框，点击“查看运行状态”，依次为显示“等待调度”和“运行中”。

![](media/8871baec453b6b4ff635149f04c80698.png)

![](media/e73a70bd31af5bdbcdde004bcf226bc5.png)

![](media/43b495dbe2f7ed2dd8132c39515643de.png)

步骤（8）验证任务是否成功
-------------------------

状态变为“成功”后，点击“查看”，可以看到执行的Hive SQL脚本和运行结果。

![](media/dda32cf9834ba00ea8c8a6ff2e35731e.png)

![](media/7a34bc8e7fdf82f3a7de586b05f899a6.png)

![](media/2e7ed7ac69a2619e9fdbb4b1b7ef37b4.png)

