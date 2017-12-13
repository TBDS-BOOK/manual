Consumer:

```
/**
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements.  See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License.  You may obtain a copy of the License at
 *
 *    http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package com.tencent.kafka;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import java.util.Collections;
import java.util.Properties;
import java.util.Random;

public class KafkaConsumerDemo implements Runnable{
    private final KafkaConsumer<String, String> consumer;
    private final String topic;

    public KafkaConsumerDemo(String topic, String group,String brokerList, String secureId, String secureKey) {
        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, brokerList);
        props.put(ConsumerConfig.GROUP_ID_CONFIG, group);
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");
        props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "1000");
        props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, "30000");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");

        confForAuthentication(props, secureId, secureKey);

        consumer = new KafkaConsumer<String,String>(props);
        this.topic = topic;
    }

    private void confForAuthentication(Properties props,String secureId,String secureKey) {
      // 设置认证参数
      props.put(TbdsAuthenticationUtil.KAFKA_SECURITY_PROTOCOL, TbdsAuthenticationUtil.KAFKA_SECURITY_PROTOCOL_AVLUE);
      props.put(TbdsAuthenticationUtil.KAFKA_SASL_MECHANISM, TbdsAuthenticationUtil.KAFKA_SASL_MECHANISM_VALUE);
      props.put(TbdsAuthenticationUtil.KAFKA_SASL_TBDS_SECURE_ID,secureId);
      props.put(TbdsAuthenticationUtil.KAFKA_SASL_TBDS_SECURE_KEY,secureKey);
    }

    public void run() {
      while(true){
        doWork();

        try {
          Thread.sleep(1000);
        } catch (Exception e) {
        }
      }
    }

    public void doWork() {
        consumer.subscribe(Collections.singletonList(this.topic));
        ConsumerRecords<String, String> records = consumer.poll(1000);
        for (ConsumerRecord<String, String> record : records) {
            System.out.println("Received message: (" + record.key() + ", " + record.value() + ") at offset " + record.offset());
        }
    }

    public static void main(String[] args) {
      if (args == null || args.length != 5) {
        System.out.println("Usage: topic group brokerlist[host1:port,host2:port,host3:port] secureId secureKey");
        System.exit(1);
      }
      KafkaConsumerDemo kafkaConsumerDemo = new KafkaConsumerDemo(args[0], args[1],args[2],args[3],args[4]);
      Thread thread = new Thread(kafkaConsumerDemo);
      thread.start();
    }
}
```

Producer

```
/**
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements.  See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License.  You may obtain a copy of the License at
 *
 *    http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package com.tencent.kafka;

import org.apache.kafka.clients.producer.Callback;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;



//import com.tencent.tbds.sdk.core.util.ServicesUtil;
//import com.tencent.tbds.sdk.core.util.TbdsAuthenticationUtil;
import java.util.Properties;
import java.util.Random;

public class KafkaProducerDemo extends Thread {
    public static long count=0l;
    public static Object lock = 0 ;
    public static long startTime =System.currentTimeMillis();
    public static long lastKilloTime = System.currentTimeMillis();
    private final KafkaProducer<Long, String> producer;
    private final String topic;
    private final Boolean isAsync;
    private static int messageCount = 1000;
    public KafkaProducerDemo(String topic, Boolean isAsync,String brokerList, String secureId, String secureKey) {
        Properties props = new Properties();
        props.put("client.id", "DemoProducer");
        props.put("key.serializer", "org.apache.kafka.common.serialization.LongSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("bootstrap.servers", brokerList);
        props.put("batch.num.messages",5000);//每个批次的消息数量
        props.put("queue.buffering.max.messages", 300000);//缓存最多缓存的消息数量
        props.put("linger.ms", "1000");
        props.put("block.on.buffer.full", true);//缓存满之后阻塞producer
        props.put("queue.buffering.max.ms", 1000000);//缓存最多缓存的时间
        props.put("batch.size", 500000);//批次发送大小，字节
        props.put("buffer.memory", 7000000);//缓存大小（字节）
        props.put("metadata.fetch.timeout.ms", 12000);//首次获取metadata信息的超时时间
        props.put("metadata.max.age.ms", 1000000);//metadata过期的时间
        props.put("request.timeout.ms", 1000000);//请求超时时间，异步发送时某个批次在缓冲区中存在时间超过这个时间就会报metadata超时

        confForAuthentication(props, secureId, secureKey);

        producer = new KafkaProducer<Long, String>(props);
        this.topic = topic;
        this.isAsync = isAsync;
    }

    private void confForAuthentication(Properties props,String secureId,String secureKey) {
    //测试TBDS
        props.put(TbdsAuthenticationUtil.KAFKA_SECURITY_PROTOCOL, TbdsAuthenticationUtil.KAFKA_SECURITY_PROTOCOL_AVLUE);
        props.put(TbdsAuthenticationUtil.KAFKA_SASL_MECHANISM, TbdsAuthenticationUtil.KAFKA_SASL_MECHANISM_VALUE);
        props.put(TbdsAuthenticationUtil.KAFKA_SASL_TBDS_SECURE_ID,secureId);
        props.put(TbdsAuthenticationUtil.KAFKA_SASL_TBDS_SECURE_KEY,secureKey);

        //测试PLAIN
//        props.put(TbdsAuthenticationUtil.KAFKA_SECURITY_PROTOCOL,"SASL_PLAINTEXT");
//        props.put(TbdsAuthenticationUtil.KAFKA_SASL_MECHANISM, "PLAIN");

        //测试PLAINTEXT
        //props.put(TbdsAuthenticationUtil.KAFKA_SECURITY_PROTOCOL,"PLAINTEXT");
    }

    public void run() {
        long messageNo = 1;
        while (messageNo<KafkaProducerDemo.messageCount) {
//            String messageStr = java.util.UUID.randomUUID().toString() + "--" + messageNo;
            String messageStr = "{\"metrics\":[{\"timestamp\":0,\"type\":\"Long\",\"metricname\":\"rpcdetailed.rpcdetailed.IOExceptionNumOps\",\"appid\":\"journalnode\",\"hostname\":\"tbds-10-254-97-135\",\"starttime\":1512531389656,\"metrics\":{\"1512531389656\":0.0,\"1512531399657\":0.0,\"1512531409656\":0.0,\"1512531419656\":0.0,\"1512531429657\":0.0,\"1512531439656\":0.0,\"1512531449656\":0.0}},{\"timestamp\":0,\"type\":\"Double\",\"metricname\":\"rpcdetailed.rpcdetailed.IOExceptionAvgTime\",\"appid\":\"journalnode\",\"hostname\":\"tbds-10-254-97-135\",\"starttime\":1512531389656,\"metrics\":{\"1512531389656\":0.0,\"1512531399657\":0.0,\"1512531409656\":0.0,\"1512531419656\":0.0,\"1512531429657\":0.0,\"1512531439656\":0.0,\"1512531449656\":0.0}}]}";
            long startTime = System.currentTimeMillis();
            if (isAsync) { // Send asynchronously
                producer.send(new ProducerRecord<Long, String>(topic,
                    messageNo,
                    messageStr), new DemoCallBack(startTime, messageNo, messageStr));
            } else { // Send synchronously
                try {
                    producer.send(new ProducerRecord<Long, String>(topic,
                        messageNo,
                        messageStr)).get();
                    System.out.println("thread "+Thread.currentThread().getName() +" Sent message: (" + messageNo + ", " + messageStr + ")");
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            ++messageNo;
        }
    }

    public static void main(String[] args) {
      if (args == null || args.length != 6) {
        System.out.println("Usage: topic brokerlist[host1:port,host2:port,host3:port] secureId secureKey messageCount producerInstance");
        System.exit(1);
      }

      KafkaProducerDemo.messageCount = Integer.parseInt( args[4] );
      int producerCount = Integer.parseInt( args[5] );
      for( int i = 0 ; i < producerCount ; i ++ ) {
          KafkaProducerDemo kafkaProducerDemo = new KafkaProducerDemo(args[0], true, args[1], args[2], args[3]);
          kafkaProducerDemo.start();
      }
    }
}

class DemoCallBack implements Callback {
    private final long startTime;
    private final long key;
    private final String message;

    public DemoCallBack(long startTime, long key, String message) {
        this.startTime = startTime;
        this.key = key;
        this.message = message;
    }

    /**
     * A callback method the user can implement to provide asynchronous handling of request completion. This method will
     * be called when the record sent to the server has been acknowledged. Exactly one of the arguments will be
     * non-null.
     *
     * @param metadata  The metadata for the record that was sent (i.e. the partition and offset). Null if an error
     *                  occurred.
     * @param exception The exception thrown during processing of this record. Null if no error occurred.
     */
    public void onCompletion(RecordMetadata metadata, Exception exception) {
        long elapsedTime = System.currentTimeMillis() - startTime;
        if (metadata != null) {

            synchronized (KafkaProducerDemo.lock){

                KafkaProducerDemo.count ++;

                if( KafkaProducerDemo.count % 10000 == 0 ){

                    long curTime = System.currentTimeMillis();

                    System.out.println( " produced count: "+KafkaProducerDemo.count+"  speed avg : " +
                            (KafkaProducerDemo.count*1000 / ( curTime - KafkaProducerDemo.startTime) ) +"   "+
                            (10000000/(1+curTime - KafkaProducerDemo.lastKilloTime)) );
                    KafkaProducerDemo.lastKilloTime = curTime;

                 if( KafkaProducerDemo.count % 100000 == 0 )
                 System.out.println(
                    Thread.currentThread().getName() + " send message(" + key + ", " + message + ") sent to partition(" + metadata.partition() +
                    "), " +
                    "offset(" + metadata.offset() + ") in " + elapsedTime + " ms");
                }
            }

        } else {
            exception.printStackTrace();
        }
    }
}
```



