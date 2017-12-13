# 背景

套件kafka基于社区的0.10.0.1版本开发。主要扩展了认证授权。
认证的修改包括：
1）增加自定义TBDS认证
2）修改社区认证SASL_PLAIN的校验逻辑：增加白名单，非集群内用户或白名单配置项之外的用户不能访问kafka


# 使用

## 1.无认证访问（低版本kafka访问方式）

大数据套件已经关闭无认证端。不支持无认证的访问



## 2.SASL_PLAIN认证访问

**命令行访问（console-consumer，console-producer）**

**生产示例：**

```
$KAFKA_HOME/bin/kafka-console-producer.sh --broker-list broker_ip:6668 --topic first_topic --producer.config ./config/client_sasl.properties
```

**消费示例：**

```
$KAFKA_HOME/bin/kafka-console-consumer.sh --bootstrap-server broker_ip:6668 --topic first_topic --new-consumer --consumer.config ./config/client_sasl.properties --from-beginning
```

其中，一定要用新版的消费API即加参数--new-consumer ，client_sasl.properties文件位置随意，内容必须包含如下配置：

```
security.protocol=SASL_PLAINTEXT

sasl.mechanism=PLAIN
```

**java客户端**

步骤1：用新消费者API去访问，在consumer的参数中多加两个认证相关参数，代码片段：

```
Properties props = new Properties();
props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, brokerList);
props.put(ConsumerConfig.GROUP_ID_CONFIG, group);
props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");
props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "1000");
props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, "30000");
props.put(security.protocol,"SASL_PLAIN");
props.put(sasl.mechanism, "PLAIN");
KafkaConsumer<string, string="string"> consumer = new KafkaConsumer<string,string>(props);
```

producer代码类似，也只需要给producer的参数加上红色部分的配置
步骤2：启动client java程序，加jvm参数：
```
-Djava.security.auth.login.config=/data/home/tbds/kafka/kafka010/config/kafka_client_jaas.conf
```
kafka_client_jaas.conf内容：

```
KafkaClient {
    org.apache.kafka.common.security.plain.PlainLoginModule required
                username="kafka"
                password="kafka@Tbds.com";
            };
```


username和password的具体值需要参考broker安装路径下的config/kafka_jaas.conf文件中的值，从JAAS的KafkaServer段中选择一个用户

** python客户端 **

python只支持pykafka访问，API很简单，示例片段：

    consumer =KafkaConsumer('monitor_metrics_topic',
                        group_id='metrics-group',
                        sasl_plain_username='{{sasl_plain_username}}',
                        sasl_plain_password='{{sasl_plain_password}}',
                        sasl_mechanism='{{sasl_mechanism}}',
                        security_protocol='{{security_protocol}}',
                        bootstrap_servers=[{{kafka_broker_list}}])

其中各个参数含义很直观。



## 3.TBDS认证访问

**命令行访问**

**生产示例：**

```
$KAFKA_HOME/bin/kafka-console-producer.sh --broker-list broker_ip:6667 --topic first_topic --producer.config ./config/client_sasl.properties
```

**消费示例：**

```
$KAFKA_HOME/bin/kafka-console-consumer.sh --bootstrap-server broker_ip:6667 --topic first_topic --new-consumer --consumer.config ./config/client_sasl.properties --from-beginning
```

需要注意的是端口，端口号跟SASL_PLAIN认证的端口是不一样的，即每种认证都有独立的端口。client_sasl.properties文件中的内容：
```
security.protocol=SASL_TBDS
sasl.mechanism=TBDS
sasl.tbds.secure.id=xxxx
sasl.tbds.secure.key=xxx
```

其中id和key是用户对应kafka模块的accesskey相关信息，记住一定是kafka模块的，然后一定是在portal里处于enabled状态（默认申请的是disabled，管理员控制使能）

**java客户端**

跟SASL_PLAIN类似，只是把client_sasl.properties中的内容作为key-val加到客户端类(KafkaConsumer,KafkaProducer)的构造函数properties参数中中.示例代码片段：

        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, brokerList);
        props.put(ConsumerConfig.GROUP_ID_CONFIG, group);
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");
        props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "1000");
        props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, "30000");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("security.protocol","SASL_TBDS");
        props.put("sasl.mechanism", "TBDS");
        props.put("sasl.tbds.secure.id","U6hXsHKVGwwBc0dZRBGPjaby1a3VCf0NfMnV");
        props.put("sasl.tbds.secure.key", "riglLTEAk2fBCWZdmjvhskKpsvXAn7rj");
        KafkaConsumer<string, string="string"> consumer = new KafkaConsumer<string,string>(props);

**python客户端**

不支持
