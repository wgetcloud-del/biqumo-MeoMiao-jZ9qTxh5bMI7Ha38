
[★消息队列16篇](https://github.com "★消息队列16篇")


# 1 认识RocketMQ


RocketMQ是一款基于Java开发的分布式消息中间件，它以其高性能、高可靠性、高实时性以及分布式特性而广受好评。


它支持事务消息、顺序消息、批量消息、定时消息、消息回溯等。互联网场景中经常使用RocketMQ进行消息路由、订阅发布、异步解耦、流量削减峰等操作，来缓解系统的压力。


![image](https://img2024.cnblogs.com/blog/167509/202411/167509-20241116103307426-92083918.png "点击查看大图")


# 2 三种常见的消息中间件比较




| 特性 | RabbitMQ | RocketMQ | Kafka |
| --- | --- | --- | --- |
| 开发语言 | erlang | java | scala |
| 单机吞吐量 | 1w\+ | 10w\+ | 10w\+ |
| 时效性 | us级 | ms级 | ms级以内 |
| 可用性 | 高(主从架构) | 非常高(分布式架构) | 非常高(分布式架构) |
| 消息可靠性 | 基本不丢 | 参数化配置和持久化：基本不丢 | 参数化配置和持久化：基本不丢 |
| 功能特性 | 基于erlang开发，所以并发能力很强，性能极其好，延时很低;管理界面较丰富 | MQ功能比较完备，扩展性佳 | 只支持主要的MQ功能，像一些消息查询，消息回溯等功能没有提供，毕竟是为大数据准备的，在大数据领域应用广。 |
| 生态 | 开源、稳定、社区活跃度高 | 阿里开源，交给Apache，社区活跃度低 | Apache开发，开源、高吞吐量、社区活跃度高 |


**技术选型决策参考：**
(1\)中小型软件，建议选RabbitMQ， 中小型软件数据量没那么大，选消息中间件，应首选功能比较完备的，所以kafka排除。
不考虑rocketmq的原因是，rocketmq是阿里出品，如果阿里放弃维护rocketmq，中小型公司一般抽不出人来进行rocketmq的定制化开发，因此不推荐。
(2\)大型软件公司，根据具体使用在rocketMq和kafka之间二选一。一方面，大型软件公司，具备足够的资金搭建分布式环境，也具备足够大的数据量。
针对rocketMQ,大型软件公司也可以抽出人手对rocketMQ进行定制化开发，毕竟有能力改JAVA源码的人，还是相当多的。
至于kafka，根据业务场景选择，如果有日志采集功能，肯定是首选kafka了。具体该选哪个，看使用场景。引入MQ之后，必然导致系统可用性降低，复杂性增大。


# 3 消息中间件使用场景


**1\. 解耦：** 比如说系统A会交给系统B去处理一些事情，但是A不想直接跟B有关联，避免耦合太强，就可以通过在A，B中间加入消息队列，A将要任务的事情交给消息队列 ,B订阅消息队列来执行任务。



> 这种场景很常见，比如A是订单系统，B是库存系统，可以通过消息队列把削减库存的工作交予B系统去处理。如果A系统同时想让B、C、D...多个系统处理问题的时候，这种优势就更加明显了。


![image](https://img2022.cnblogs.com/blog/167509/202204/167509-20220403151445587-1247122330.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/688 "点击查看大图")


**2\. 有序性：** 先进先出原理，先来先处理，比如一个系统处理某件事需要很长一段时间，但是在处理这件事情时候，有其他人也发出了请求，可以把请求放在消息队里，一个一个来处理。



> 对数据的顺序性和一致性有强需求的业务，比如同一张银行卡同时被多个入口使用，需要保证入账出账的顺序性，避免出现数据不一致。


![image](https://img2022.cnblogs.com/blog/167509/202204/167509-20220403152229902-463184452.png "点击查看大图")


**3\. 消息路由/数据分发：** 按照不同的规则，将队列中消息发送到不同的其他队列中



> 通过消息队列将不同染色的请求发送到不同的服务去操作。这样达成了流量按照业务拆分的目的。


![image](https://img2024.cnblogs.com/blog/167509/202411/167509-20241116101758794-1044922588.png "点击查看大图")


4、**异步处理：** 处理一项任务的时候，有3个步骤A、B、C，需要先完成A操作, 然后做B、C 操作。任务执行成功与否强依赖A的结果，但不依赖B、C 的结果。
如果我们使用串行的执行方式，那处理任务的周期就会变长，系统的整体吞吐能力也会降低（在同一个系统中做异步其实也是比较大的开销），所以使用消息队列是比较好的办法。



> 登录操作就是典型的场景：A：执行登录并得到结果、B：记录登录日志、C：将用户信息和Token写入缓存。 执行完A就可以从登录页跳到首页了，B、C让服务慢慢去消化，不阻塞当前操作。


![image](https://img2022.cnblogs.com/blog/167509/202204/167509-20220403152938036-1318862151.png "点击查看大图")


5、**削峰：** 将峰值期间的操作削减，比如A同学的整个操作流程包含12个步骤，后续的11个步骤是不需要强关注结果的数据，可以放在消息队列中。


# 4 MQ的概念、结构和原理


## 4\.1 构成说明


![image](https://img2024.cnblogs.com/blog/167509/202411/167509-20241116104914310-2106367613.png "点击查看大图")


RocketMQ主要有四大核心组成部分：NameServer、Broker、Producer以及Consumer四部分。这些角色通常以集群的方式存在，RocketMQ 基于纯Java开发，具有高吞吐量、高可用性、适合大规模分布式系统应用的特点。


* RockerMQ想要启动，首先需要启动 NameServer，再启动 Brober 主机
* Broker 会向 NameServer 注册对应的路由和服务
* Producer会进行路的发现，向NameServer请求Broker路由信息，进行消息的发送
* Consumer要连通NameServer，获取到相关的路由信息，方便我们进行消息的订阅
* Broker 主要负责消息的存储，不管是生产消息还是订阅消息，消息的来源都是 Broker，
* 消息的发送(Producer)只会发到主节点，然后Broker会进行消息的同步，同步到从节点，作为消费者(Consumer)也只会优先从Master节点，获取消息，进行消费
* Broker主节点不可用或者非常繁忙，会选择从节点进行消费


**1\. Producer：**
负责生产消息，一般由业务系统负责。生产者通过调用API将消息发送到指定的Topic（主题）中。


**2\. Broker：**
消息存储中心，负责接收来自Producer的消息并存储，同时Consumer也从这里取得消息。Broker还存储与消息相关的元数据，包括消费者组、消费进度偏移量、队列信息等。每个Broker可以存储多个Topic的消息，每个Topic的消息也可以分片存储于不同的Broker。在实际部署中，Broker对应一台服务器，并分为Master与Slave两种类型，Master负责读写，Slave只负责读，以此实现数据的备份和负载均衡。


**3\. Consumer：**
负责消费消息，一般由后台系统负责。消费者通过订阅Topic来获取消息，并根据业务逻辑进行处理。消费者可以以集群消费或广播消费的方式消费消息。


**4\. NameServer：**
充当路由消息的提供者，负责保存Broker的元数据信息，并供Producer和Consumer查询。NameServer被设计成几乎无状态的，可以横向扩展，节点之间无通信。


## 4\.2 基础概念


![image](https://img2024.cnblogs.com/blog/167509/202411/167509-20241116105332122-1882746020.png "点击查看大图")


### 4\.2\.1 Group（分组）


在RocketMQ中，Group功能是其核心特性之一。Group主要分为发送端Group和消费端Group。


* 发送端Group：允许将多个发送者（Producer）组织成一个发送者组。通过发送端Group，可以实现对消息的批量处理和负载均衡，提高系统性能和可靠性。
* 消费端Group：RocketMQ允许消费者同时订阅多个主题（Topic），这些消费者可以被组织成一个或多个消费组。消费组内的消费者可以共同分担消息的消费任务，实现负载均衡和容错。当某个消费者出现故障时，其他消费者可以接管其消费任务，确保消息被及时处理。同时，消费组还支持消息过滤和顺序消费等高级特性。


### 4\.2\.2 Topic（主题）


用来区分消息的种类，表示一类消息的逻辑名字，消息的逻辑管理单位，无论生产还是消费消息，都需要执行Topic。比如一个Topic专门用于用户订单消息发送，一个Topic专门用于扣减或增加积分的。


* 一个发送者可以发送消息给一个或者多个Topic
* 一个消息接受者可以订阅一个或多个Topic消息


### 4\.2\.3 Message Queue（消息队列）


MessageQueue是RocketMQ中用于存储和传输消息的数据结构。每个MessageQueue都有一个唯一标识符，由Topic名称和队列编号组成。MessageQueue具有以下特点：


* 唯一标识：确保每个MessageQueue都可以被唯一地识别和定位。
* 消息顺序性：对于同一个MessageQueue中的消息，RocketMQ保证其消费的顺序性，即先进先出（FIFO）。
* 负载均衡：RocketMQ通过动态调整消息分配策略，将消息均匀地分布到所有的MessageQueue中，实现负载均衡，避免某个MessageQueue过载而其他MessageQueue空闲的情况。
* 高可用性：RocketMQ支持将多个Broker节点组成集群，每个MessageQueue可以在不同的Broker节点上进行主从复制，提供高可用性和数据冗余。即使某个Broker节点出现故障，其他节点也可以继续提供服务，确保消息的可靠传输。


### 4\.2\.4 Tag（标签）


Tag是RocketMQ中用于对消息进行分类和过滤的标记。生产者可以在发送消息时指定Tag，消费者可以根据Tag来过滤和订阅消息。Tag的使用可以使得消息的管理和消费更加灵活和高效。例如，一个电商系统可能会根据商品的类别（如服装、电子产品等）来设置不同的Tag，消费者可以根据这些Tag来订阅和处理特定类别的消息。


### 4\.2\.5 Offset（偏移量）


Offset在RocketMQ中用于标识消费者在消息队列中的位置。每个消费者都会维护一个Offset，以便知道下一次从哪里开始消费。Offset的使用可以确保消息不丢失、避免消息重复消费以及支持消息的顺序消费。


* Message queue 是无限长的数组。一条消息进来下标就会涨 1，而这个数组的下标就是 offset。
* 确保消息不丢失：通过持久化Offset，即使在消费者宕机后重启，也能从上次消费的位置继续消费，保证消息至少被消费一次。
* 避免消息重复消费：消费者在成功处理消息后更新Offset，确保每条消息只被消费一次。
* 支持消息顺序消费：对于顺序消息，通过维护Offset可以保证消息的顺序消费。


# 5 程序实现


## 5\.1 Java中引入和实现RocketMQ


1. **引入依赖**


在Java项目中，通常通过Maven或Gradle等构建工具来引入RocketMQ的客户端依赖。以Maven为例，可以在`pom.xml`文件中添加以下依赖：



```
<dependency>
    <groupId>org.apache.rocketmqgroupId>
    <artifactId>rocketmq-clientartifactId>
    <version>最新版本号（5.3.1）version>
dependency>

```
2. **实现生产者**


在Java中，通过创建`DefaultMQProducer`对象来实现消息的生产者。生产者需要设置NameServer地址，并调用`start()`方法初始化。然后，可以创建`Message`对象并设置主题、标签和消息内容，最后调用`send()`方法发送消息。



```

public class Producer {
    public static void main(String[] args) throws Exception {
        // 创建一个消息生产者，并设置消息生产者组
        DefaultMQProducer producer = new DefaultMQProducer("your_producer_group");
        // 指定NameServer地址
        producer.setNamesrvAddr("your_nameserver_address");
        // 初始化Producer
        producer.start();

        // 创建消息对象，并设置主题、标签和消息内容
        Message msg = new Message("your_topic", "your_tag", "Hello RocketMQ".getBytes());
        // 发送消息并获取发送结果
        SendResult sendResult = producer.send(msg);
        System.out.printf("%s%n", sendResult);

        // 关闭生产者实例
        producer.shutdown();
    }
}

```
3. **实现消费者**


在Java中，通过创建`DefaultMQPushConsumer`对象来实现消息的消费者。消费者需要设置NameServer地址和消费组名称，并调用`subscribe()`方法订阅主题和标签。然后，注册消息监听器来处理接收到的消息。



```

public class Consumer {
    public static void main(String[] args) throws Exception {
        // 创建一个消息消费者，并设置消息消费者组
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("your_consumer_group");
        // 指定NameServer地址
        consumer.setNamesrvAddr("your_nameserver_address");
        // 订阅指定Topic下的所有消息
        consumer.subscribe("your_topic", "*");

        // 注册消息监听器
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List msgs, ConsumeConcurrentlyContext context) {
                for (MessageExt msg : msgs) {
                    System.out.printf("接收到消息: %s%n", new String(msg.getBody()));
                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });

        // 启动消费者实例
        consumer.start();
    }
}

```


# 6 总结


本文只是了解下RocketMQ的基本原理和实现，在后面的章节中，我们会带来RocketMQ的完整解读。


 本博客参考[FlowerCloud机场](https://yunbeijia.com)。转载请注明出处！
