# hippo管理

这一节会用一个名为test的topic详细展示套件用户admin（为方便展示，admin是管理员）使用hippo的过程。

## HippoUI

登录到套件主页面之后，选择数据接入，如下图。

![](/数据接入/Hippo管理/enter_step1.png)

然后进入如图1-1所示页面，

![](/数据接入/Hippo管理/enter_step2.png)
                       图1-1 套件主页面



然后选择hippo管理，计入hippo主页面 如图1-2所示。

![](/数据接入/Hippo管理/enter_step3.png)
                          图1-2 hippo主页面


如图1-2所示hippo主页面，包括左侧栏上展示的六个模块，依次是：接入列表、队列信息、集群管理、数据消费、监控、消费进展。

## 数据接入

数据接入之前需要先申请topic，topic审批通过之后，生产者才可以生产数据。通过图1-2右上角“接入申请”按钮可以进入topic的申请页面，如图1-3所示。

![](/数据接入/Hippo管理/data_input_step1.png)
                            图1-3 topic申请


topic申请需要填写一些信息，前7项都是必填项，最后一项“业务描述”用来描述业务，是可选项。如图1-3所示，必填项处已经给出了相应的提示和说明，以下是具体解释：

* topic名称：全局唯一的，不能重复；
* ip列表：用户生产topic的ip列表，多个ip之间使用分号分隔开，支持模糊匹配，申请时请添加所有有生产需求的ip；
* 队列模型：topic的队列模型，包括并发和顺序两种，并发可以提高数据消费的并发读，顺序则可以保证数据的有序，即消费的顺序和生产的顺序一致（必须是在单进程内单线程使用同步方式生产数据，如果多个进程同时生产数据则无法保证数据的顺序性）；
* 数据量：估算值，每天的生产总量；
* 峰值速率：估算值，生产数据的最大速率；
* 负责人：topic的负责人，即有权限生产此topic数据的所有用户，此处会给出tbds的用户列表，从中选取即可；
* 业务描述：辅助信息，用来描述topic。

填写完成之后，就可以提交申请，然后管理员可以在套件的“个人中心”看到这个接入申请并做审批。管理员在个人中心看到的信息如图1-4所示。

![](/数据接入/Hippo管理/data_input_step2.png)
            图1-4 管理员-个人中心-审批单-hippo


管理员可以看到申请类型是HIPPO\_PRODUCE，表明这是一个接入申请，相对应的申请类型还有HIPPO\_CONSUME，表明是消费申请；还有申请人、申请时间、申请原因、申请状态等信息，最后管理员可以点击 “详情”来查看这个申请的详情，并执行审批操作，如图1-5所示。

![](/数据接入/Hippo管理/data_input_step3.png)
                        图1-5 审批单详情



详情中可以看到申请时填写的信息，管理员审批时应该注意集群的选择（示例中只有一个BrokerGroup，实际场景可能会有多组），管理员可以选择BrokerGroup、指定该BrokerGroup上的新增队列数量。管理员在审批时应该根据该BrokerGroup的topic和queue数量做好资源的分配，最后执行通过或者驳回操作。


管理员审批通过一个topic之后，就可以在接入列表查看这个topic的信息，如图1-2所示。需要注意的是对于一个topic可以执行扩容或者调试操作，扩容操作仅针对于并发模型的topic，扩容界面和审批中的集群选择类似，需要勾选BrokerGroup并指定扩容的队列数量，扩容操作可以提高并发度，提升消费的速率，一般在消费滞后时使用；调试功能仅针对于顺序模型的topic，在一个顺序模型的topic生产了数据之后，通过调试功能可以查看生产数据的内容。在接入列表页面点击topic名称，可以看到topic的详细信息，如图1-6所示。

![](/数据接入/Hippo管理/data_input_step4.png)
                   图1-6 topic详细信息


详细信息中注意两点：首先是secretId和secretKey，secretId和secretKey在生产、消费时需要写到客户端配置中；sdk下载是客户端sdk的下载链接，目前只有java的sdk，客户端的使用说明参考“Hippo客户端使用”章节。

## 队列信息

队列信息按照 队列ID展示了所有topic的队列信息，也可以用来查看数据的生产状况。如图1-7所示。

![](/数据接入/Hippo管理/queue_info1.png)
                     图1-7 队列信息


队列信息包括：

* Id：唯一值，用来表示queue的编号
* store名称：queue的存储单元名称
* 开始编辑时间：第一条数据的写入时间
* 最后编辑时间：最后一条数据的写入时间
* Store容量：存储单元的容量
* Offset区间：存储单元中数据的偏移量区间
* 支持写：存储单元是否支持写入

## 集群管理

集群管理展示了集群中的BrokerGroup信息，如图1-8所示。

![](/数据接入/Hippo管理/queue_info2.png)
                图1-8 集群信息


集群信息包括：

* 集群名称：BrokerGroup名称
* Ip列表：BrokerGroup中各个Broker的Ip
* Topic数量：BrokerGroup上Topic的数量
* Queue数量：BrokerGroup上Queue的数量
* 上报状态：由BrokerGroup上报的BrokerGroup的读写状态
* 控制状态：由Controller设置的BrokerGroup的读写状态，管理员通过停止读/写和恢复读/写操作来改变集群的控制状态
* 操作：停止读/写，恢复读/写，主备切换

关于集群的操作，读写控制包括停止或者恢复读/写，停止读/写之后，BrokerGroup对于客户端来讲是不可读/写的；恢复读/写之后，BrokerGroup恢复读/写功能。主备切换用来切换BrokerGroup集群中Broker主备状态，如非必要，也不推荐使用。

## 数据消费

数据消费页面如图1-9所示。

![](/数据接入/Hippo管理/data_comsume_step1.png)
                图1-9数据消费


消费数据之前需要先提交消费申请，消费申请需要填写一些内容，如图1-10所示。

![](/数据接入/Hippo管理/data_comsume_step2.png)
                图1-10 新增消费


其中，前四项是必填项，“数据用途说明”是选填项。必填项包括：

* Topic名称：要申请消费的topic名称，必须是已经存在的topic
* 消费组：消费组的名称，唯一
* 数据用途：数据用途描述
* IP列表：消费客户端Ip地址列表，申请时请填入所有有消费需求的ip提交申请之后，topic的负责人可以在个人中心看到审批单（和接入申请的审批单类似），如图1-11所示。

![](/数据接入/Hippo管理/data_comsume_step3.png)
      图1-11 topic负责人个人中心-消费审批单


点击详情可以看到消费申请的详细信息，其详情如图1-12所示。

![](/数据接入/Hippo管理/data_comsume_step4.png)
           图1-12 消费申请审批单


Topic的负责人根据申请内容，决定是否通过审批。审批通过的消费申请，可以在数据消费页面看到其内容，如图1-9所示。可以看到对于审批通过的消费组有个“负载均衡查看”操作，此操作可以查看正在消费的客户端的队列分配情况。

## 监控

监控页面是开放给用户用来监控消费是否跟上生产的监控工具，如图1-13所示。

![](/数据接入/Hippo管理/moniter_step1.png)

           图1-13 消费生产进度监控


页面给出topic和消费组的候选，选择topic和对应的消费组之后，就可以看到其在每个queue上的生产、消费信息，如图1-14所示。

![](/数据接入/Hippo管理/moniter_step2.png)
            图1-14 生产消费监控结果


## 消费进展

消费进展展示了每个消费组的消费情况。如图1-15所示。

![](/数据接入/Hippo管理/comsume_progress_step1.png)
    图1-15 消费组消费进展


由图中可以看到，group01消费了topic名称为test的数据，具体各个项目含义如下：

* Topic名称：topic的名称
* QueueId：队列queue的id
* 消费者业务ID：消费者编号
* 已提交offset：消费者已提交的消费offset
* 已拉取至offset：消费者已从服务端拉取的offset
* 提交超时时间：提交的超时时间
* 已拉取未提交数：已经从服务端拉取但是还没有执行提交的数量
* 创建时间：消费的起始时间
* 修改时间：最后一次消费的时间

# Hippo客户端使用
Hippo客户端的使用分为两部分，生产者和消费者，生产者申请topic，然后生产该topic的数据；消费者申请消费，然后消费改topic的数据。生产、消费都要依赖sdk，下载方式前文已有描述。
无论是生产者还是消费者，在生产或者消费数据之前都需要一些必要的配置，主要是包括5项：
* UserName: 套件用户名
* SecretId:套件用户的secretId，生产者可以在topic详情处获取，消费者可以咨询套件管理员
* SecretKey:套件用户的secretKey，生产者可以在topic详情处获取，消费者可以咨询套件管理员
* controllerIpList：Hippo controller的ip地址列表，如果是多台controller，那么每个controllerIp之间使用逗号分隔
* groupName:生产者和消费者都必须设定一个组名，消费者会根据组名进行负载均衡

## 生产者
生产者的配置是在ProducerConfig对象中完成的，示例如下：

```
String controllerIpList = "10.151.139.83:8066,10.151.139.83:8066，10.151.139.83:8066";
String producerGroupName = "music_producer_group";
ProducerConfig producerConf = new ProducerConfig(controllerIpList, producerGroupName);
producerConf.setSecretId("jSUAxMgVUHu4CaiCEJCc74ZBteVR7Gl2ouqE");
producerConf.setSecretKey("6ZfejyHmoWG7aIVL7SSxjFL0VDk72xGD");
producerConf.setUserName("admin");
```

生产数据之前需要进行topic的发布，即publish，topic发布之后，就可以进行数据的生产了，生产者有两种生产数据的方式，同步方式和异步方式，用户只需选择一种即可。
发送是的数据是封装在Message对象中的，Message可以携带一些属性信息，通过Message的addHeader()来实现。
同步方式发送数据会产生SendResult对象作为发送结果，根据这个对象可以判断是否发送成功，并执行后续操作；异步方式发送需要一个实现了SendCallback接口的对象来处理回调，在SendCallback中处理发送结果。
附件中有Producer使用的示例可以参考。

## 消费者
消费者消费数据时也需要和生产者一样需要执行一些配置，
```
String controllerIpList = "10.151.139.83:8066,10.151.139.83:8066，10.151.139.83:8066";
String consumerGroupName = "music_consumer_group";
ConsumerConfig consumerConf = new ConsumerConfig(controllerIpList, consumerGroupName);
consumerConf.setSecretId("igeU2BeH0BJFmstP8uikuAiRvpuEoHFN16hR");
consumerConf.setSecretKey("wdR8rihoNt0fhUKcV2ibBRPCbvM8J7Io");
consumerConf.setUserName("admin");
consumerConf.setConfirmTimeout(10000, TimeUnit.MILLISECONDS);
```
消费者消费数据之后需要进行消费的确认，这就涉及到消费数据处理的时机，一般推荐消费到的数据处理完成之后再调用确认，这样可以避免数据丢失。由于数据处理需要一定的时间，因此可能会需要配置确认的超时时间，超时时间应该大于数据处理时间加客户端和服务端网络通信时间之和。上面代码最后一行就是确认超时时间的设置方法。
配置完成之后，就可以拉取数据，拉取数据时要指定topic，batchSize和timeout时间，这里batchSize是指每次拉取的数据条数，timeout是客户端选取队列的最大等待时间。
数据消费会产生一个MessageResult作为结果，根据这个结果来判断消费是否成功。
消费的各个步骤在附件中示例可以参考。


# 附件
## 生产者示例代码

```
import com.google.common.base.Charsets;
import com.tencent.hippo.Message;
import com.tencent.hippo.client.producer.ProducerConfig;
import com.tencent.hippo.client.producer.SendCallback;
import com.tencent.hippo.client.producer.SendResult;
import com.tencent.hippo.client.producer.SimpleMessageProducer;
import com.tencent.hippo.common.HippoConstants;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.TimeUnit;


public class Producer {
    private static final Logger LOG = LoggerFactory.getLogger(Producer.class);

    public static void main(String[] args) {

       String controllerIpList = "10.151.139.83:8066,10.151.139.83:8066，10.151.139.83:8066";
       String producerGroupName = "music_producer_group";

        /**
         * 初始化producer配置信息
         * @param controllerIpList  所属集群地址列表,根据实际情况填写
         * @param producerGroupName 生产组名称，一般建议生产同一类消息的所有进程使用相同的组名
         */
        ProducerConfig producerConf = new ProducerConfig(controllerIpList, producerGroupName);
        producerConf.setSecretId("jSUAxMgVUHu4CaiCEJCc74ZBteVR7Gl2ouqE");
        producerConf.setSecretKey("6ZfejyHmoWG7aIVL7SSxjFL0VDk72xGD");
        producerConf.setUserName("admin");

        try {
            /**
             * 根据producerConf创建一个producer实例
             * producer全局共享，只需要实例化一次
             */
            SimpleMessageProducer producer = new SimpleMessageProducer(producerConf);

            /**
             * 发布相应的主题（告知服务端即将生产哪些主题的消息）
             */
            String[] topics = {"country_music", "rock_music"};
            for (String topic : topics) {
                producer.publish(topic);
            }

            /**
             * 构建一条country_music主题的消息
             */
            String countryMusicTopic = "country_music";
            byte[] messageBody = "country music message body".getBytes(Charsets.UTF_8);
            Message message = new Message(countryMusicTopic, messageBody);

            /**
             * 消息发送方式分为两种:同步（当前线程会阻塞直到结果返回为止），异步（当前线程不会阻塞，响应结果将以回调的方式进行反馈），可根据需要选择其中一种方式进行发送
             *
             * 同步方式: 调用producer同步方法发送消息
             */
            SendResult sendResult = producer.sendMessage(message);
            if (sendResult.isSuccess()) {
                LOG.info("Send message success.");
                /**
                 * TODO write your code
                 */
            } else {
                LOG.info("Send message fail,error message:" + sendResult.getErrorMessage() + ",error code:" + sendResult.getCode());
                /**
                 * TODO write your code
                 */
            }

            /**
             * 异步方式：调用producer异步方法发送消息
             * @param callback  实现SendCallback接口并构建相应的callback实例，响应返回时将自动调用callback实例内的方法
             */
            MusicMsgSendCallback callback = new MusicMsgSendCallback(message);
            long timeout = 5000;
            producer.sendMessage(message, callback, timeout, TimeUnit.MILLISECONDS);


            /**
             * 构建多条rock_music主题的消息
             * 示列中通过;号分隔多个消息体（实际可结合业务特点进行定制，此处没有唯一的标准，只需消费者消费时按照约定的格式进行解析即可）
             */
            String rockMusicTopic = "rock_music";
            byte[] mutiMessageBody = "rock music message body1;rock music message body2;rock music message body3".getBytes(Charsets.UTF_8);
            Message messagePack = new Message(rockMusicTopic, mutiMessageBody);
            /**
             * 消息公共属性
             *     系统属性（属性名称固定） timestamp 消息时间戳、h_cnt 消息包中包含的记录条数,服务端会根据系统属性生成指标
             *     自定义属性（属性名称自定义）
             */
            String timeStamp = String.valueOf(System.currentTimeMillis());
            //系统属性
            messagePack.addHeader(HippoConstants.TIMESTAMP, timeStamp);
            messagePack.addHeader(HippoConstants.MSG_CNT, String.valueOf(3));
            //自定义属性
            messagePack.addHeader("custom_attr", "custom_attr_value");
            SendResult result = producer.sendMessage(messagePack);
            if (result.isSuccess()) {
                LOG.info("Send message success.");
            } else {
                LOG.info("Send message fail,error message:" + result.getErrorMessage() + ",error code:" + result.getCode());
            }
        } catch (Exception e) {
            LOG.error("producer failed.", e);
        }
    }

    static class MusicMsgSendCallback implements SendCallback {
        private Message message;

        MusicMsgSendCallback(Message message) {
            this.message = message;
        }

        /**
         * 消息发送成功将会回调onMessageSent方法
         *
         * @param result
         */
        public void onMessageSent(SendResult result) {
            if (result.isSuccess()) {
                LOG.info("Send message success.");
                /**
                 * TODO write your code
                 */
            } else {
                LOG.info("Send message fail,error message:" + result.getErrorMessage() + ",error code:" + result.getCode());
            }
        }

        /**
         * 消息发送失败或者网络超时，将会回调onException方法
         *
         * @param e
         */
        public void onException(Throwable e) {
            LOG.info("Send message fail,error message:" + e.getClass().getSimpleName() + "#" + e.getMessage());
            /**
             * TODO write your code
             */
        }
    }

}
```

## 消费者示例代码

```
import com.google.common.base.Charsets;
import com.tencent.hippo.Message;
import com.tencent.hippo.client.MessageResult;
import com.tencent.hippo.client.consumer.ConsumerConfig;
import com.tencent.hippo.client.consumer.PullMessageConsumer;
import com.tencent.hippo.common.HippoConstants;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.List;
import java.util.concurrent.TimeUnit;

/**
 * Created by mayozhang on 2017/4/12.
 */
public class Consumer {
    private static final Logger LOG = LoggerFactory.getLogger(Consumer.class);

    public static void main(String[] args) {


        /**
         * 初始化consumer配置信息
         * @param controllerIpList  所属集群地址列表,根据实际情况填写
         * @param consumerGroupName 消费组名称,消费相同topic的进程使用相同的消费组名
         */
        String controllerIpList = "10.151.139.83:8066,10.151.139.83:8066，10.151.139.83:8066";
        String consumerGroupName = "music_consumer_group";
        ConsumerConfig consumerConf = new ConsumerConfig(controllerIpList, consumerGroupName);
        consumerConf.setSecretId("igeU2BeH0BJFmstP8uikuAiRvpuEoHFN16hR");
        consumerConf.setSecretKey("wdR8rihoNt0fhUKcV2ibBRPCbvM8J7Io");
        consumerConf.setUserName("admin");
        /**
         * 确认超时时间，如果在超过当前时间内服务端还没收到对当前批次的确认,那么当前批次的消息不再支持确认提交
         * 当前拉取的信息存在被重复拉取的可能
         */
        consumerConf.setConfirmTimeout(10000, TimeUnit.MILLISECONDS);

        try {
            /**
             * 根据consumerConf创建一个consumer实例
             * consumer全局共享，只需要实例化一次
             */
            PullMessageConsumer consumer = new PullMessageConsumer(consumerConf);
            String topic = "rock_music";    //消费的主题
            int batchSize = 100;             //一次最大拉取条数，如服务端消息数不足batchSize条时则返回条数将小于batchSize值
            int timeout = 5000;              //单位 ms
            MessageResult result = consumer.receiveMessages(topic, batchSize, timeout);
            if (result.isSuccess()) {
                for (Message message : (List<Message>) result.getValue()) {
                    try {
                        /**
                         * 获取指定的属性：通过生产者自定义的属性名称可以获取响应的属性值
                         * 获取所有属性值：message.getAttribute();
                         */
                        String customAttrValue = message.getHeader("custom_attr");
                        LOG.info("message attr :{} ", message.getAttribute());
                        /**解码方式需要与生产者保持一致*/
                        String messageBody = new String(message.getData(), Charsets.UTF_8);
                        LOG.info("message customAttrValue :{} ", customAttrValue);
                        LOG.info("message messageBody :{} ", messageBody);
                        //TODO  write your consume logic code
                    } catch (Exception e) {
                        LOG.info("consume message error", e);
                    }
                }
                /**
                 * 处理完消息后，需要对当前拉取批次的消息进行成功确认
                 * 确认是针对当前拉取批次的所有消息，不支持针对其中某条消息的确认
                 * 要么全部确认成功，要么超时导致全部确认失败（确认失败当前批次的消息会被消费者再次拉取到）
                 */
                boolean confirmResult = result.getContext().confirm();
                if (!confirmResult) {
                    /**
                     *   TODO 可能因为上述消息处理过程(consume logic code)耗时超出了consumerConf.setConfirmTimeout(10000, TimeUnit.MILLISECONDS)设置的时间
                     *        可根据具体的业务耗时调整setConfirmTimeout时间,避免消息处理完后进行确认超时导致消费重复的情况
                     */
                    LOG.info("consume confirm failed");
                }
            } else if (result.getCode() == HippoConstants.NO_MORE_MESSAGE) {
                /**
                 * 该返回码表示服务端已经没有更多消息，当前消费者可以睡眠一段时间再去拉取
                 */
                LOG.info("No more message.");
                Thread.currentThread().sleep(100);
            } else {
                LOG.info("Receive message failed! code:{},msg:{}", result.getCode(), result.getErrorMsg());
            }

            /**
             * 进程退出前调用shutdonw方法
             */
            consumer.shutdown();
        } catch (Exception e) {
            LOG.error("consumer failed.", e);
        }
    }
}
```
