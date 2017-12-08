**介绍开发一个wordcount 计算任务运行在tstorm上**   

### 1. 创建maven 项目
生成一个maven 项目。  
编译的java 版本 使用1.8
```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.1</version>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
        <encoding>UTF8</encoding>
    </configuration>
</plugin>
```

### 2. 引入依赖
Tstorm基于storm 0.9.6 版本，所以依赖的storm-core 建议使用0.9.6 
如果本地调试模块，依赖scope 必须选用 compile。如果提交到生产环境，依赖scope 使用provided。
```
<dependency>
    <groupId>org.apache.storm</groupId>
    <artifactId>storm-core</artifactId>
    <version>0.9.6</version>
    <scope>provided</scope>
    <!-- 如果本地调试模块 ，必须选用 compile，如果提交到生产环境使用provided-->
</dependency>
```
### 3. 编写topology
##### 3.1 编写入口类
topology 入口就是一个普通的java 程序。  
```
    public static void main(String[] args) throws Exception {
        TopologyBuilder builder = new TopologyBuilder();

        builder.setSpout("word", new WordSpout(), 10);
        builder.setBolt("exclaim1", new ExclamationBolt(), 3).shuffleGrouping("word");
        builder.setBolt("exclaim2", new ExclamationBolt(), 2).shuffleGrouping("exclaim1");

        Config conf = new Config();
        conf.setDebug(true);

        if (args != null && args.length > 0) {
            conf.setNumWorkers(3);// work 个数
            StormSubmitter.submitTopologyWithProgressBar(args[0], conf, builder.createTopology());
        } else {
            // 本地提交模式
            LocalCluster cluster = new LocalCluster();
            cluster.submitTopology("test", conf, builder.createTopology());
            TimeUnit.SECONDS.sleep(10);
            cluster.killTopology("test");
            cluster.shutdown();
        }
    }
```

##### 3.2 实现spout
spout 通常继承 BaseRichSpout或者实现IRichSpout接口
并重写 nextTuple 方法
```
    @Override
    public void nextTuple() {
        Utils.sleep(10000);
        final String[] words = new String[] {"nathan", "mike", "jackson", "golda", "bertels"};
        final Random rand = new Random();
        final String word = words[rand.nextInt(words.length)];
        LOG.info("emit " + word);
        _collector.emit(new Values(word));
    }
```

##### 3.3 实现blot
blot 通常是继承 BaseRichBolt，BaseBasicBolt 或实现 IRichBolt 接口  
并重写execute 方法

```
    @Override
    public void execute(Tuple tuple) {
        String word = tuple.getString(0);
        LOG.info("get word=" + word);
        _collector.emit(tuple, new Values(word + "!!!"));
        _collector.ack(tuple);
    }
```





更多开发实例参考：https://github.com/apache/storm/tree/v0.9.6/examples/storm-starter  
更多storm 接口参考：http://storm.apache.org/releases/0.9.6/index.html
### 4. 打包jar
在pom.xml 文件中添加打包插件
```
  <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-assembly-plugin</artifactId>
      <version>2.5.5</version>
      <configuration>
          <archive>
              <manifest>
                  <!--这里要替换成jar包main方法所在类 -->
                  <mainClass>com.tencent.dc.lz.ExclamationTopology</mainClass>
              </manifest>
          </archive>
          <descriptorRefs>
              <descriptorRef>jar-with-dependencies</descriptorRef>
          </descriptorRefs>
      </configuration>
      <executions>
          <execution>
              <id>make-assembly</id> <!-- this is used for inheritance merges -->
              <phase>package</phase> <!-- 指定在打包节点执行jar包合并操作 -->
              <goals>
                  <goal>single</goal>
              </goals>
          </execution>
      </executions>
  </plugin>
```
通过maven 打包命令,会将项目打包成一个可执行jar 。
### 5. 本地调试 
建议用户使用IntelliJ IDEA 进行本地调试，直接运行main 函数，会在本地嵌入运行一个storm 环境，方便用户调试核心逻辑。

### 6. 提交topology到集群
通过后台提交topology，需要切到nimbus 节点。 上传打包好的jar,切到系统用户jstorm。  
使用提交命令格式如下：   
/usr/local/jstorm/bin/storm jar topology.jar main.class  topologyName   
例如：
```
/usr/local/jstorm/bin/storm jar wordCountTstormDemo-0.0.1-SNAPSHOT-jar-with-dependencies.jar com.tencent.dc.lz.ExclamationTopology "nihao"
```
### 7. 查看任务和查看日志
通过后台命令（/usr/local/jstorm/bin/storm list） 查看提交topology 状态（使用jstorm 用户）。   
也可以使用storm ui 查看topology 运行状态和日志。访问地址为 (http://portalIP:8080/tstorm)  

### 8. 停止top
通过后台命令杀死对应的topology  
/usr/local/jstorm/bin/storm kill topologyName  

### 9. 更多  
更多后台操作命令使用： /usr/local/jstorm/bin/storm help   
提交topology也可以通过工作流中的storm任务。
