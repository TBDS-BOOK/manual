## 聚类算法
 * [KMeans](#1-kmeans)
 * [DBSCAN](#2-dbscan)

### 1. KMeans

* **算法说明**

  KMeans是一种常用的聚类算法，将无标签的数据聚成K个类。平台提供的KMeans算法实现了并行的k-means++的初始化算法。

- **输入**
  - 数据形式：[Dense或Libsvm](./tdw_ml_jarvis_dataformat.md#2-数据格式要求)
  - 格式：| 参与计算的features | 不参与计算的features |
  - 参与计算的features：可通过**算法参数**的**选择特征列**指定
  - 不参与计算的features：如果存在则保留在输出中，对于Libsvm格式的数据，也包括了label列，该label仅用于标识样本ID
- **输出**
  - 格式：| 不参与计算的features | cluster_id |
  - cluster_id：样本对应的簇Id

- **算法参数**
  - 选择特征列：从0开始计数，以类似“1-12,15”的方式选择特征，表示取特征在表中的1到12列，15列
  - 并行数：训练数据的分区数、spark的并行数
  - 抽样率：输入数据的采样率
  - 聚类个数K：期望将数据聚成K类
  - 聚类中心初始化方式：k-means|| 或者 random
  - 聚类重复次数：默认为1
  - 最大迭代次数：默认为20
  - 算法终止误差：聚类中心位置更新小于该值时，迭代终止，默认为1e-4

### 2. DBSCAN

* **算法说明**

  DBSCAN算法是一种基于密度的聚类算法。该算法可将具有足够高密度的区域划分为簇，并在具有噪声的数据中发现任意形状的簇。

  **注意**
  1. 鉴于目前对空间密度定义的限制，本算法仅对二维数据进行聚类。用户需要选择对应x与y轴的两个特征。
  2. 输入数据尽量只包含x与y轴对应的特征，其他不参与聚类的特征也可以包括，但为了提高运行速度，尽量避免引入过多不参与聚类的特征
  3. 所有识别出的噪音点的簇Id均为0，非噪音点的簇Id从1开始计数

- **输入**
  - 数据形式：[Dense](./tdw_ml_jarvis_dataformat.md#21-dense数据格式)
  - 格式：| 参与计算的features | 不参与计算的features |
  - 参与计算的features：可通过**算法参数**的**对应x轴的特征列**和**对应y轴的特征列**指定
  -  不参与计算的features：为了运行效率，尽量避免引入
- **输出**
  - 格式：| x轴对应特征 | y轴对应特征 | 不参与计算的features | 所属的簇Id |
  - 聚类的簇Id：噪音点的簇Id均为0，非噪音点的簇Id从1开始计数
- **算法参数**
  - 对应x轴的特征列：从0开始计数
  - 对应y轴的特征列：从0开始计数
  - 并行数：训练数据的分区数、spark的并行数
  - 抽样率：输入数据的采样率
  - 邻域半径：确定邻域的半径
  - 邻域内最少点数：在邻域内形成簇的最少点数，注意该点数包括某点本身
  - 分区内最大点数：每个分区内的最大点数，建议该值取小点，以提高并行度
