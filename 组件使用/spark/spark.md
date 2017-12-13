## 1.TBDS认证

由于套件增加了自己的认证体系，在执行bin/spark-submit之前，需要先export认证相关的环境变量，

```
　　export hadoop_security_authentication_tbds_username=hdfs
　　export hadoop_security_authentication_tbds_secureid=F3QdVfxbQkNHVkn1OzLA3yK3In0bL6HgX
　　export hadoop_security_authentication_tbds_securekey=o8AnGFYQ2lIB0AJ78TIeoJ0Uu1nkph12
```

admin用户可以在portal创建所有用户的securekey密钥。普通用户需要securekey密钥则向管理员申请。详情请见认证相关文档。

执行上述export语句之后，就可正常提交spark任务。

## 2.Spark Streaming 消费 Kafka

由于访问套件Kafka也需要进行认证，因此这个地方也与开源版本不同。首先需要在自己的Spark Streaming App里面将maven依赖由社区版本替换成套件版本，

```
　　    <groupId>org.apache.spark</groupId>
　　    <artifactId>spark-streaming-kafka-0-10_2.11</artifactId>
　　    <version>2.1.0-TBDS-4.0.4</version>
```

Kafka认证支持两种方式，接下来分别说明。

### 2.1 TBDS认证访问

需要在Kafka参数里面增加参数，

```
　　     "security.protocol" -> "SASL_TBDS",
　　     "sasl.mechanism" -> "TBDS",
　　     “sasl.tbds.secure.id” -> “F3QdVfxbQkNHVkn1OzLA3yK3In0bL6HgX”,
　　     “sasl.tbds.secure.key” -> “o8AnGFYQ2lIB0AJ78TIeoJ0Uu1nkph12”
```

其中认证相关的密钥信息同上述。

### 2.2 SASL\_PLAIN认证访问

需要在Kafka参数里面增加参数，

```
　　      "security.protocol" -> "SASL_PLAINTEXT",
　　      "sasl.mechanism" -> "PLAIN"
```

完整示例代码如下，

    　　package org.apache.spark.examples.streaming

    　　import org.apache.kafka.common.serialization.StringDeserializer

    　　import org.apache.spark.SparkConf
    　　import org.apache.spark.streaming._
    　　import org.apache.spark.streaming.kafka010._
    　　import org.apache.spark.streaming.kafka010.ConsumerStrategies.Subscribe
    　　import org.apache.spark.streaming.kafka010.LocationStrategies.PreferConsistent

    　　object DirectKafka010WordCount {
    　　  def main(args: Array[String]) {
    　　    if (args.length < 2) {
    　　      System.err.println(s"""
    　　        |Usage: DirectKafka010WordCount <brokers> <topics>
    　　        |  <brokers> is a list of one or more Kafka brokers
    　　        |  <topics> is a list of one or more kafka topics to consume from
    　　        |
    　　        """.stripMargin)
    　　      System.exit(1)
    　　    }

    　　    StreamingExamples.setStreamingLogLevels()

    　　    val Array(brokers, topics) = args

    　　    // Create context with 2 second batch interval
    　　    val sparkConf = new SparkConf().setAppName("DirectKafka010WordCount")
    　　    val ssc = new StreamingContext(sparkConf, Seconds(2))

    　　    // Create direct kafka stream with brokers and topics
    　　    val kafkaParams = Map[String, Object](
    　　      "bootstrap.servers" -> brokers,
    　　      "key.deserializer" -> classOf[StringDeserializer],
    　　      "value.deserializer" -> classOf[StringDeserializer],
    　　      "group.id" -> "tbds_spark_streaming_group",
    　　      "security.protocol" -> "SASL_PLAINTEXT",
    　　      "sasl.mechanism" -> "PLAIN"
    　　    )
    　　    val messages = KafkaUtils.createDirectStream[String, String](
    　　      ssc,
    　　      PreferConsistent,
    　　      Subscribe[String, String](topics.split(","), kafkaParams)
    　　    )

    　　    // Get the lines, split them into words, count the words and print
    　　    val lines = messages.map(record => record.value)
    　　    val words = lines.flatMap(_.split(" "))
    　　    val wordCounts = words.map(x => (x, 1L)).reduceByKey(_ + _)
    　　    wordCounts.print()

    　　    // Start the computation
    　　    ssc.start()
    　　    ssc.awaitTermination()
    　　  }
    　　}
    　　// scalastyle:on println
    ```　　

    并且，在执行bin/spark-submit的时候需要添加以下两个参数，

    　　--driver-java-options -Djava.security.auth.login.config=/etc/hadoop/conf/kafka_client_for_ranger_yarn_jaas.conf
    　　--conf "spark.executor.extraJavaOptions=-Djava.security.auth.login.config=/etc/hadoop/conf/kafka_client_for_ranger_yarn_jaas.conf"

这样才能保证Spark的driver和executor都能访问到认证所需要的jaas配置文件。

## 3.集群外客户端部署

（1）在任一安装spark客户端的集群内节点，打包spark的安装路径：  
     /usr/hdp/2.2.0.0-2041/spark/，并拷贝到集群外目标客户端安装节点  
（2）如果有套件的yum源，在集群外目标安装节点使用yum install安装  
（3）如果有spark的rpm包，在集群外目标安装节点使用rpm -ivh安装

