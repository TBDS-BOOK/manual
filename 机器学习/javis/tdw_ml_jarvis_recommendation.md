## 推荐算法
  * [Collaborative Filtering](#1-collaborative-filtering)

### 1. Collaborative Filtering

* **算法说明**

  协同过滤是经典的基于邻域的推荐算法，平台上的协同过滤算法是通过ALS的矩阵分解优化求解的。

* **训练节点**

  * 输入
    * 数据形式：[Dense或Libsvm](./tdw_ml_jarvis_dataformat.md#2-数据格式要求)
    * 格式：\| User \| Item \| Rating \|
    * User：user所在列，通过**算法参数**中的User列指定
    * Item：item所在列，通过**算法参数**中的Item列指定
    * Rating：评分所在列的列号，通过**算法参数**中的Rating列指定
  * 输出
    * PMML格式的模型
    * 模型格式：
    * **userPath：\|id\|features\|**
      * id：用户Id
      * features：用户的特征向量
    * **itemPath：\|id\|features\|**
      * id：商品Id
      * features：商品的特征向量
  * 算法参数
    * User列：user所在列，从0开始计数
    * Item列：item所在列的列号，从0开始计数
    * Rating列：评分所在列的列号，从0开始计数
    * 隐变量数：矩阵变换中隐变量的个数，推荐值：10-200
    * 正则系数：正则项系数，推荐值：0.01
    * 最大迭代次数：最大迭代次数，推荐值：10-20
    * 反馈方式：隐性反馈与显性反馈方式
    * 是否使用非负的限制：是否对最小二乘法使用非负的限制
    * 偏好行为强度的基准：专门针对隐性反馈的参数，决定了偏好行为强度的基准
    * 并行数：训练数据的分区数、spark的并行数
    * 抽样率：输入数据的采样率

* **预测节点**
  * 输入
    * 格式：\| User \| Item \|
    * 说明：以空格连接各字段
    * User：同训练节点
    * Item：同训练节点
  * 输出
    * 格式：\| 原始数据 \| ratings \|
    * 说明：以空格连接各字段
    * ratings:预测的评分
