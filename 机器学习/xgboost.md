## TDinsight里的xgboost介绍
	

- xgboost-yarn组件（基于社区版0.4，以Yarn作业形式运行在x86机型的集群上）
 
- XGBoost-yarn组件属于TDinsight中的机器学习组件，基于社区的xgboost0.4版本，可以实现Train、Predict、Dump三种功能。

     <div  align="center">
	   <img src="./manual/resource_5.png"/>   
	 </div>

- XGBoost-yarn组件包含“组件参数”和“资源参数”两种参数，下面分别举例进行说明。

  - 资源参数配置如下：
	  <div  align="center">
	    <img src="./manual/resource_4.png"/>   
	  </div>

  - 组件参数配置分以下三步完成：
  
     -  1. 编写xgboost参数配置文件，参考[xgboost官方的0.4版本](https://github.com/dmlc/xgboost/blob/v0.40/doc/parameter.md),Train阶段和Predict阶段的配置文件，举例如下：
	    
         <div  align="center">
		   <img src="./manual/train_3.png"/>   
		 </div>
	
         <div  align="center">
		   <img src="./manual/predict_3.png"/>   
		 </div>

     -  2. 点击“xgboost_configuration”右侧空白输入框，上传配置文件。当然，也可在线编辑脚本。
	
         <div  align="center">
		   <img src="./manual/resource.png"/>   
		 </div>


     -  3. Train阶段和Predict阶段中，涉及到IO相关的参数配置在外面输入框中

         <div  align="center">
		   <img src="./manual/train_1.png"/>   
		 </div>

	
         <div  align="center">
		   <img src="./manual/predict_1.png"/>   
		 </div>


- 常见问题和排查技巧 

     - LibSVM格式举例
       - 可以参考xgboost官方的demo中的数据文件
              - https://github.com/dmlc/xgboost/blob/master/demo/data/agaricus.txt.train
       - 内容中第一列为label(float型)，后面为一组feature_index:feature_value，空格分隔
           - 1 3:1 10:1 11:1 21:1 30:1 34:1 36:1 40:1 41:1 53:1 58:1 65:1 69:1 77:1 86:1 88:1 92:1 95:1 102:1 105:1 117:1 124:1
           - 0 3:1 10:1 20:1 21:1 23:1 34:1 36:1 39:1 41:1 53:1 56:1 65:1 69:1 77:1 86:1 88:1 92:1 95:1 102:1 106:1 116:1 120:1
           - 0 1:1 10:1 19:1 21:1 24:1 34:1 36:1 39:1 42:1 53:1 56:1 65:1 69:1 77:1 86:1 88:1 92:1 95:1 102:1 106:1 116:1 122:1
           - 1 3:1 9:1 19:1 21:1 30:1 34:1 36:1 40:1 42:1 53:1 58:1 65:1 69:1 77:1 86:1 88:1 92:1 95:1 102:1 105:1 117:1 124:1
           - 0 3:1 10:1 14:1 22:1 29:1 34:1 37:1 39:1 41:1 54:1 58:1 65:1 69:1 77:1 86:1 88:1 92:1 95:1 98:1 106:1 114:1 120:1

    - LibSVM常见错误

      - 特征值出现nan或LibSVM中特征值超过float的最大值和最小值分别为3.40282e+38（10 +38），1.17549e-38（10-38）
         - "AssertError:the bound variable must be max"
      - 逻辑回归中label列超过了[0,1]的取值范围
         - "label must be in [0,1] for logistic regression"
      - 输入数据集某些部分为空
         - "label set cannot be empty"
         - "NumCol:need column access"和"38870x0 matrix with 0 entries is loaded from hdfs:xxx"
         - "0x0 matrix with 0 entries is loaded from"


    - 用户查看报错详情的步骤（举例说明）
      - 1. 对报错的xgboost节点右键点击，查看spark控制台，打开stderr。
      - 2. 查看日志搜索如下：Task 0 failed on container_e05_1519815850090_0324_01_000004. See LOG at : http://XX-XX-XX-XX:8042/node/containerlogs/container_e05_1519815850090_0324_01_000004/yarn
      - 3. 假设访问TDInsight使用的地址为AA.AA.AA.AA，那么请将上述日志中的链接变换为：
         http://AA.AA.AA.A:8080/nodemanager/XX-XX-XX-XX/node/containerlogs/container_e05_1519815850090_0324_01_000004/yarn
      - 4. 在打开的链接里即可继续查看此worker上的详细日志

     
