在某些情况下，我们建立的任务需要依赖其他项目或者同一项目下，其他工作流的任务。  
为了处理这种跨工作流依赖的情况，tbds 提出跨工作流解决方案。

假设有工作流 w1中一个任务 t1,需要之定义一个父任务（在工作流 w2 中任务t2）
  
操作步骤  
1. 选择对应的工作流w2  
打开工作流w1 ,添加任务(新增方式-选取已有任务)   
通过项目名或者工作流名称搜索任务所在工作流  
![](/workflow/workflow/images/gap2.png)  

2. 选择对应的任务t2  
通过任务名称搜索需要添加依赖的任务t2    
![](/workflow/workflow/images/gap3.png)
点击确认，则会出现一个虚线框标示的任务 t2 (虚拟任务) ,具体如下图所示 
![](/workflow/workflow/images/gap4.png)

3. 建立依赖  
通过连线建立t1和t2 之间依赖关系
![](/workflow/workflow/images/gap5.png)

4. 确认父子关系  
确认步骤如下：  
4.1 查看运行状态  
![](/workflow/workflow/images/gap7.png)  
4.2 查看更多实例  
![](/workflow/workflow/images/gap8.png)  
4.3 查看父子任务   
![](/workflow/workflow/images/gap9.png)  
4.4 查看父实例  
![](/workflow/workflow/images/gap10.png)  
4.5 查看子实例
![](/workflow/workflow/images/gap11.png)  

5. 查看虚拟任务对应实例  
![](/workflow/workflow/images/gap6.png)  
