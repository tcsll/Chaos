

## 1.什么RabbitMq

RabbitMQ是一款开源的，Erlang编写的，基于AMQP协议的消息中间件



## 2.众多MQ的技术选型

|            | ActiveMQ                                                | RabbitMQ                                                     | RocketMQ                                                     | Kafka                                                        |
| :--------- | :------------------------------------------------------ | :----------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 单机吞吐量 | 比RabbitMQ低                                            | 2.6w/s（消息做持久化）                                       | 11.6w/s                                                      | 17.3w/s                                                      |
| 开发语言   | Java                                                    | Erlang                                                       | Java                                                         | Scala/Java                                                   |
| 主要维护者 | Apache                                                  | Mozilla/Spring                                               | Alibaba                                                      | Apache                                                       |
| 成熟度     | 成熟                                                    | 成熟                                                         | 开源版本不够成熟                                             | 比较成熟                                                     |
| 订阅形式   | 点对点(p2p)、广播（发布-订阅                            | 提供了4种：direct, topic ,Headers和fanout。fanout就是广播模式 | 基于topic/messageTag以及按照消息类型、属性进行正则匹配的发布订阅模式 | 基于topic以及按照topic进行正则匹配的发布订阅模式             |
| 持久化     | 支持少量堆积                                            | 支持少量堆积                                                 | 支持大量堆积                                                 | 支持大量堆积                                                 |
| 顺序消息   | 不支持                                                  | 不支持                                                       | 支持                                                         | 支持                                                         |
| 性能稳定性 | 好                                                      | 好                                                           | 一般                                                         | 较差                                                         |
| 集群方式   | 支持简单集群模式，比如’主-备’，对高级集群模式支持不好。 | 支持简单集群，'复制’模式，对高级集群模式支持不好。           | 常用 多对’Master-Slave’ 模式，开源版本需手动切换Slave变成Master | 天然的‘Leader-Slave’无状态集群，每台服务器既是Master也是Slave |
| 管理界面   | 一般                                                    | 较好                                                         | 一般                                                         | 无                                                           |

综上，各种对比之后，有如下建议：

一般的业务系统要引入 MQ，最早大家都用 ActiveMQ，但是现在确实大家用的不多了，没经过大规模吞吐量场景的验证，社区也不是很活跃，所以大家还是算了吧，我个人不推荐用这个了；

后来大家开始用 RabbitMQ，但是确实 erlang 语言阻止了大量的 Java 工程师去深入研究和掌控它，对公司而言，几乎处于不可控的状态，但是确实人家是开源的，比较稳定的支持，活跃度也高；

不过现在确实越来越多的公司会去用 RocketMQ，确实很不错，毕竟是阿里出品，但社区可能有突然黄掉的风险（目前 RocketMQ 已捐给 Apache，但 GitHub 上的活跃度其实不算高）对自己公司技术实力有绝对自信的，推荐用 RocketMQ，否则回去老老实实用 RabbitMQ 吧，人家有活跃的开源社区，绝对不会黄。

所以中小型公司，技术实力较为一般，技术挑战不是特别高，用 RabbitMQ 是不错的选择；大型公司，基础架构研发实力较强，用 RocketMQ 是很好的选择。

如果是大数据领域的实时计算、日志采集等场景，用 Kafka 是业内标准的，绝对没问题，社区活跃度很高，绝对不会黄，何况几乎是全世界这个领域的事实性规范。



## 3.RabbitMq的优缺点

优点上面已经说了，就是在特殊场景下有其对应的好处，解耦、异步、削峰。

缺点有以下几个：

系统可用性降低

本来系统运行好好的，现在你非要加入个消息队列进去，那消息队列挂了，你的系统不是呵呵了。因此，系统可用性会降低；

系统复杂度提高

加入了消息队列，要多考虑很多方面的问题，比如：一致性问题、如何保证消息不被重复消费、如何保证消息可靠性传输等。因此，需要考虑的东西更多，复杂性增大。

一致性问题

A 系统处理完了直接返回成功了，人都以为你这个请求就成功了；但是问题是，要是 BCD 三个系统那里，BD 两个系统写库成功了，结果 C 系统写库失败了，咋整？你这数据就不一致了。

所以消息队列实际是一种非常复杂的架构，你引入它有很多好处，但是也得针对它带来的坏处做各种额外的技术方案和架构来规避掉，做好之后，你会发现，妈呀，系统复杂度提升了一个数量级，也许是复杂了 10 倍。但是关键时刻，用，还是得用的。



# 4.RabbitMq的使用场景

1. 服务间异步通信

2. 顺序消费？

3. 定时任务？

4. 请求削峰

   ## RabbitMq怎么实现延迟队列
   
   应用场景
   
   三方支付，扫码支付调用上游的扫码接口，当扫码有效期过后去调用查询接口查询结果。实现方式：每当一笔扫码支付请求后，立即将此订单号放入延迟队列中（RabbitMQ），队列过期时间为二维码有效期，此队列没有设置消费者，过了有效期后消息会重新路由到指定的的队列，有消费者去执行订单查询。
   
   RabbitMQ本身没有直接支持延迟队列功能，但是可以通过以下特性模拟出延迟队列的功能。 但是我们可以通过RabbitMQ的两个特性来曲线实现延迟队列：Time To Live(TTL)   和   Dead Letter Exchanges（DLX）
   
   实现:队列超时时间+死信队列
   
   

# 5.RabbitMq的基本概念

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20180812231313584?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3ZpcHNob3BfZmluX2Rldg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

- Broker：消息队列服务的实体
- Exchange：交换机，按照消息的规则分发到不同的队列
- Queue：消息队列载体，每个消息都会被投入到一个或多个队列
- Binding：？他的作用就是将exchange和queue按照规则绑定起来
- RoukingKey：路由关键字，exchange根据这个关键字进行消息投递
- VHost：vhost 可以理解为虚拟 broker ，即 mini-RabbitMQ server。其内部均含有独立的 queue、exchange 和 binding 等，但最最重要的是，其拥有独立的权限系统，可以做到 vhost 范围的用户控制。当然，从 RabbitMQ 的全局角度，vhost 可以作为不同权限隔离的手段（一个典型的例子就是不同的应用可以跑在不同的 vhost 中）
- Produce：消息生产者，就是投递消息的程序
- Consumer：消息消费者，就是接受消息的程序
- Channel：消息通道，在每个客户端连接里，可建立多个channel，每个channel代表一个会话任务

由Exchange,Queue,RoutingKey 3个才能决定一个从exchange到queue的唯一线路

------

问题1.RabbitMQ 上的一个 queue 中存放的 message 是否有数量限制

```
可以认为是无限制，因为限制取决于机器的内存，但是消息过多会导致处理效率的下降
```



## 6.RabbitMq的工作模式

### 1.simple模式(简单模式)

1.消息产生消息，将消息放入队列

2.消息的消费者(consumer) 监听 消息队列,如果队列中有消息,就消费掉,消息被拿走后,自动从队列中删除(隐患 消息可能没有被消费者正确处理,已经从队列中消失了,造成消息的丢失，这里可以设置成手动的ack,但如果设置成手动ack，处理完后要及时发送ack消息给队列，否则会造成内存溢出)。

生产者

```java
public class Sender {
    private final static String QUEUE_NAME = "simple_queue";
 
    public static void main(String[] args) throws IOException {
        Connection connection = ConnectionUtil.getConnection();
        Channel channel = connection.createChannel();
        //声明队列
        /**
         * 队列名
         * 是否持久化
         * 是否排外  即只允许该channel访问该队列   一般等于true的话用于一个队列只能有一个消费者来消费的场景
         * 是否自动删除  消费完删除
         * 其他属性
         *
         */
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
 
        //消息内容
        /**
         * 交换机
         * 队列名
         * 其他属性  路由
         * 消息body
         */
        String message = "错的不是我，是这个世界~";
        channel.basicPublish("", QUEUE_NAME,null,message.getBytes());
        System.out.println("[x]Sent '"+message + "'");
        channel.close();
        connection.close();
    }
}
```

消费者

```java
public class Receiver {
    private final static String QUEUE_NAME = "simple_queue";
 
    public static void main(String[] args) throws IOException, InterruptedException {
        Connection connection = ConnectionUtil.getConnection();
        Channel channel = connection.createChannel();
 
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        QueueingConsumer consumer = new QueueingConsumer(channel);
        channel.basicConsume(QUEUE_NAME, true, consumer);
 
        while(true){
            //该方法会阻塞
            QueueingConsumer.Delivery delivery = consumer.nextDelivery();
            String message = new String(delivery.getBody());
            System.out.println("[x] Received '"+message+"'");
        }
    }
}
```

### 2.work工作模式(资源的竞争)

1.消息产生者将消息放入队列，消费者可以有多个,消费者1,消费者2同时监听同一个队列,消息被消费。

2.C1 C2共同争抢当前的消息队列内容,谁先拿到谁负责消费消息(隐患：高并发情况下,默认会产生某一个消息被多个消费者共同使用,可以设置一个开关(syncronize) 保证一条消息只能被一个消费者使用)。

生产者

```java
public class Sender {
    private final  static String QUEUE_NAME = "queue_work";
 
    public static void main(String[] args) throws IOException, InterruptedException {
        Connection connection = ConnectionUtil.getConnection();
        Channel channel = connection.createChannel();
 
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        for(int i = 0; i < 100; i++){
            String message = "冬马小三" + i;
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
            System.out.println("[x] Sent '"+message + "'");
            Thread.sleep(i*10);
        }
 
        channel.close();
        connection.close();
    }
}
```

消费者1

```java
public class Receiver1 {
    private final static  String QUEUE_NAME = "queue_work";
 
    public static void main(String[] args) throws IOException, InterruptedException {
        Connection connection = ConnectionUtil.getConnection();
        Channel channel = connection.createChannel();
 
        channel.queueDeclare(QUEUE_NAME, false,false, false,null);
        //同一时刻服务器只会发送一条消息给消费者
        channel.basicQos(1);
 
        QueueingConsumer consumer = new QueueingConsumer(channel);
        //关于手工确认 待之后有时间研究下
        channel.basicConsume(QUEUE_NAME, false, consumer);
 
        while(true){
            QueueingConsumer.Delivery delivery = consumer.nextDelivery();
            String message = new String(delivery.getBody());
            System.out.println("[x] Received1 '"+message+"'");
            Thread.sleep(10);
            //返回确认状态
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        }
 
    }
}
```

消费者2

```java
public class Receiver2 {
    private final static  String QUEUE_NAME = "queue_work";
 
    public static void main(String[] args) throws IOException, InterruptedException {
        Connection connection = ConnectionUtil.getConnection();
        Channel channel = connection.createChannel();
 
        channel.queueDeclare(QUEUE_NAME, false,false, false,null);
        //同一时刻服务器只会发送一条消息给消费者
        channel.basicQos(1);
 
        QueueingConsumer consumer = new QueueingConsumer(channel);
        channel.basicConsume(QUEUE_NAME, false, consumer);
 
        while(true){
            QueueingConsumer.Delivery delivery = consumer.nextDelivery();
            String message = new String(delivery.getBody());
            System.out.println("[x] Received2 '"+message+"'");
            Thread.sleep(1000);
            //返回确认状态
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        }
 
    }
}
```

### 3.订阅模式(publish/subscribe)

1、每个消费者监听自己的队列；

2、生产者将消息发给broker，由交换机将消息转发到绑定此交换机的每个队列，每个绑定交换机的队列都将接收到消息。



### 4.路由模式(routing)

1.消息生产者将消息发送给交换机按照路由判断,路由是字符串(info) 当前产生的消息携带路由字符(对象的方法),交换机根据路由的key,只能匹配上路由key对应的消息队列,对应的消费者才能消费消息;

2.根据业务功能定义路由字符串

3.从系统的代码逻辑中获取对应的功能字符串,将消息任务扔到对应的队列中。

4.业务场景:error 通知;EXCEPTION;错误通知的功能;传统意义的错误通知;客户通知;利用key路由,可以将程序中的错误封装成消息传入到消息队列中,开发者可以自定义消费者,实时接收错误;

生产者

```java
public class Sender {
    private final static String EXCHANGE_NAME = "exchange_direct";
    private final static String EXCHANGE_TYPE = "direct";
 
    public static void main(String[] args) throws IOException {
        Connection connection = ConnectionUtil.getConnection();
        Channel channel = connection.createChannel();
 
        channel.exchangeDeclare(EXCHANGE_NAME,EXCHANGE_TYPE);
 
        String message = "那一定是蓝色";
        channel.basicPublish(EXCHANGE_NAME,"key2", null, message.getBytes());
        System.out.println("[x] Sent '"+message+"'");
 
        channel.close();
        connection.close();
    }
}
```

消费者1

```java
public class Receiver1 {
    private final  static  String QUEUE_NAME = "queue_routing";
    private final static String EXCHANGE_NAME = "exchange_direct";
 
    public static void main(String[] args) throws IOException, InterruptedException {
        // 获取到连接以及mq通道
        Connection connection = ConnectionUtil.getConnection();
        Channel channel = connection.createChannel();
 
        channel.queueDeclare(QUEUE_NAME, false,false,false,null);
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"key");
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"key2");
 
        channel.basicQos(1);
 
        QueueingConsumer consumer = new QueueingConsumer(channel);
        channel.basicConsume(QUEUE_NAME, false, consumer);
 
        while(true){
            QueueingConsumer.Delivery delivery = consumer.nextDelivery();
            String message = new String(delivery.getBody());
            System.out.println("[x] Received1 "+message);
            Thread.sleep(10);
 
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        }
    }
}
```

消费者2

```java
public class Receiver2 {
    private final  static  String QUEUE_NAME = "queue_routing2";
    private final static String EXCHANGE_NAME = "exchange_direct";
 
    public static void main(String[] args) throws IOException, InterruptedException {
        // 获取到连接以及mq通道
        Connection connection = ConnectionUtil.getConnection();
        Channel channel = connection.createChannel();
 
        channel.queueDeclare(QUEUE_NAME, false,false,false,null);
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"key2");
 
        channel.basicQos(1);
 
        QueueingConsumer consumer = new QueueingConsumer(channel);
        channel.basicConsume(QUEUE_NAME, false, consumer);
 
        while(true){
            QueueingConsumer.Delivery delivery = consumer.nextDelivery();
            String message = new String(delivery.getBody());
            System.out.println("[x] Received2 "+message);
            Thread.sleep(10);
 
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        }
 
 
    }
}
```

### 5.主题模式(topic)

1.星号井号代表通配符

2.星号代表多个单词,井号代表一个单词

3.路由功能添加模糊匹配

4.消息产生者产生消息,把消息交给交换机

5.交换机根据key的规则模糊匹配到对应的队列,由队列的监听消费者接收消息消费

（在我的理解看来就是routing查询的一种模糊匹配，就类似sql的模糊查询方式）

生产者

```java
package com.sakura.topic;
 
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.sakura.util.ConnectionUtil;
 
import java.io.IOException;
 
/**
 * Created by apple on 2018/9/4.
 */
public class Sender {
    private final static String EXCHANGE_NAME = "exchange_topic";
    private final static String EXCHANGE_TYPE = "topic";
 
    public static void main(String[] args) throws IOException {
        Connection connection = ConnectionUtil.getConnection();
        Channel channel = connection.createChannel();
 
        channel.exchangeDeclare(EXCHANGE_NAME, EXCHANGE_TYPE);
 
        //消息内容
        String message = "如果真爱有颜色";
        channel.basicPublish(EXCHANGE_NAME,"key.1",null,message.getBytes());
        System.out.println("[x] Sent '"+message+"'");
 
        //关通道 关连接
        channel.close();
        connection.close();
    }
}
```

消费者1

```java
public class Receiver1 {
    private final static String QUEUE_NAME = "queue_topic";
    private final static String EXCHANGE_NAME = "exchange_topic";
    private final static String EXCHANGE_TYPE = "topic";
 
    public static void main(String[] args) throws IOException, InterruptedException {
        Connection connection = ConnectionUtil.getConnection();
        Channel channel = connection.createChannel();
 
        channel.queueDeclare(QUEUE_NAME, false, false,false, null);
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "key.*");
 
        channel.basicQos(1);
 
        QueueingConsumer consumer = new QueueingConsumer(channel);
        channel.basicConsume(QUEUE_NAME, false, consumer);
 
        while(true){
            QueueingConsumer.Delivery delivery = consumer.nextDelivery();
            String message = new String(delivery.getBody());
            System.out.println("[x] Received1 '"+message + "'");
            Thread.sleep(10);
 
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        }
    }
}
```

消费者2

```java
public class Receiver2 {
    private final static String QUEUE_NAME = "queue_topic2";
    private final static String EXCHANGE_NAME = "exchange_topic";
    private final static String EXCHANGE_TYPE = "topic";
 
    public static void main(String[] args) throws IOException, InterruptedException {
        Connection connection = ConnectionUtil.getConnection();
        Channel channel = connection.createChannel();
 
        channel.queueDeclare(QUEUE_NAME, false, false,false, null);
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "*.*");
 
        channel.basicQos(1);
 
        QueueingConsumer consumer = new QueueingConsumer(channel);
        channel.basicConsume(QUEUE_NAME, false, consumer);
 
        while(true){
            QueueingConsumer.Delivery delivery = consumer.nextDelivery();
            String message = new String(delivery.getBody());
            System.out.println("[x] Received2 '"+message + "'");
            Thread.sleep(10);
 
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        }
    }
}
```

### 6.rpc模式



## 7.如何保证RabbitMQ消息的顺序性



## 8.消息如何分发及应答



### 1.消息分发

我理解的可能是多个消费者同时消费一个队列的时候，消息是如何分发到各个消费者的

众多文案表示：RabbitMQ是采用轮询机制（round-robin）将消息队列Queue中的消息依次发给不同的消费者

但还没有自己看过源码，所以暂且保留意见

### 2.消息应答

消息应答是保证消息被正确接受并处理

结合上面的分发图说明，默认情况下消费者C1接收到消息1无论是否正常接受和处理都会立即应答rabbit服务器，然后消息1就会从队列中被删除，假如C1突然出现异常状况导致消息1没有被处理完毕，那么消息1就处理失败了，也不会有其他消费者去处理消息1。事实上我们希望的是消息1如果没有被C1正确处理完毕，那么就发送给其他消费者处理，为了达到这个目的，只需要做两件事情，第一关闭rabbitMq的自动应答机制，第二消费者正确处理完消息后手动应答。

手动确认与自动确认的区别：

```java
   //第二个参数autoAck设置成false表示关闭自动应答
        channel.basicConsume(QUEUE_NAME, false, consumer);  
        while (true)
        {  
            QueueingConsumer.Delivery delivery = consumer.nextDelivery();
            String message = new String(delivery.getBody());
            System.out.println(hashCode + " [x] Received '" + message + "'");
            doWork(message);
            System.out.println(hashCode + " [x] Done");
            //手动应答，第二个参数multiple表示是否批量应答，很明显现在不是批量应答
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false );
        }  
```



### 3.消费者负载均衡

（prefetchCount代表consumer必须能够处理给定数量消息并正确发回回执之后，队列才会给consumer分发信息，否则不分发，可以通过prefetchCount限制consumer处理消息数量）
默认情况下rabitMq会把队列里面的消息立即发送到消费者，无论该消费者有多少消息没有应答，也就是说即使发现消费者来不及处理，新的消费者加入进来也没有办法处理已经堆积的消息，因为那些消息已经被发送给老消费者了。 
chanel.basicQos(prefetchCount) 
prefetchCount：会告诉RabbitMQ不要同时给一个消费者推送多于N个消息，即一旦有N个消息还没有ack，则该consumer将block掉，直到有消息ack。 
这样做的好处是，如果系统处于高峰期，消费者来不及处理，消息会堆积在队列中，新启动的消费者可以马上从队列中取到消息开始工作。

上图描述了这样一个过程：刚启动时消费者C1、C2、C3均为空闲状态，且它们的channel都设置了prefetch=1，队列中有消息1~N。下面按照各消费者收到消息的时间顺序说明整个过程。

C1收到消息1开始处理。 
C2收到消息2开始处理。 
C3收到消息3开始处理。 
C2处理完消息2并产生应答，可以看到图中消息2从队列中移除了。 
C2收到消息4开始处理，可以看到图中消息4被标记为已发送状态。 
C3处理完消息3并产生应答，可以看到图中消息3从队列中移除了。 
C3收到消息5开始处理，可以看到消息5被标记为已发送状态。 
上图反应的就是直到C3收到消息5并处理时的整个队列中所有消息的状态。

### 4.消息消费的推/拉模式



## 9.消息怎么路由

从概念上来说，消息路由必须有三部分：交换器、路由、绑定。生产者把消息发布到交换器上；绑定决定了消息如何从路由器路由到特定的队列；消息最终到达队列，并被消费者接收。

消息发布到交换器时，消息将拥有一个路由键（routing key），在消息创建时设定。
通过队列路由键，可以把队列绑定到交换器上。
消息到达交换器后，RabbitMQ会将消息的路由键与队列的路由键进行匹配（针对不同的交换器有不同的路由规则）。如果能够匹配到队列，则消息会投递到相应队列中；如果不能匹配到任何队列，消息将进入 “黑洞”。

### 1.常用的交换器(3种)：

dirct：如果路由键完全匹配，消息就被投递到相应的队列
fanout：如果交换器收到消息，将会广播到所有绑定的队列上
topic：可以使来自不同源头的消息能够到达同一个队列。 使用topic交换器时，可以使用通配符，比如：“*” 匹配特定位置的任意文本， “.” 把路由键分为了几部分，“#” 匹配所有规则等。特别注意：发往topic交换器的消息不能随意的设置选择键（routing_key），必须是由"."隔开的一系列的标识符组成。

### 2.不常用的交换器(1种)

headers:

不处理路由键。而是根据发送的消息内容中的headers属性进行匹配。在绑定Queue与Exchange时指定一组键值对；当消息发送到RabbitMQ时会取到该消息的headers与Exchange绑定时指定的键值对进行匹配；如果完全匹配则消息会路由到该队列，否则不会路由到该队列。headers属性是一个键值对，可以是Hashtable，键值对的值可以是任何类型。而fanout，direct，topic 的路由键都需要要字符串形式的。

匹配规则x-match有下列两种类型：

x-match = all ：表示所有的键值对都匹配才能接受到消息

x-match = any ：表示只要有键值对匹配就能接受到消息

### 3.交换机的声明方式





## 10.消息基于什么传输？

### 1.传输原理(AMQP)

由于 TCP 连接的创建和销毁开销较大，且并发数受系统资源限制，会造成性能瓶颈。RabbitMQ 使用信道的方式来传输数据。信道是建立在真实的 TCP 连接内的虚拟连接，且每条 TCP 连接上的信道数量没有限制。



### 2.RabbitMq消息队列运转过程

生产者生产过程

1.生产者连接到 RabbitMQ Broker 建立一个连接( Connection) ，开启 个信道 (Channel)

2.生产者声明一个交换器 ，并设置相关属性，比如交换机类型(direct/fanout/topic/header)、是否持久化等

3.生产者声明 个队列井设置相关属性，比如是否排他、是否持久化、是否自动删除等 

4.生产者通过路由键将交换器和队列绑定起来

5.生产者发送消息至 RabbitMQ Broker ，其中包含路由键、交换器等信息

6.相应的交换器根据接收到的路由键查找相匹配的队列 如果找到 ，则将从生产者发送过来的消息存入相应的队列中

7.如果没有找到 ，则根据生产者配置的属性选择丢弃还是回退给生产者

8.关闭信道

9.关闭连接



```java
private final RabbitTemplate.ConfirmCallback confirmCallback = (correlationData, ack, s) -> {
        if (ack) {
            System.out.println(correlationData.getId());
        } else {
            log.error("ConfirmCallback消息发送失败: {}", s);
        }
    };

    private final RabbitTemplate.ReturnCallback returnCallback = (message, replyCode, replyText, exchange, routingKey)
            -> log.error("ReturnCallback消息发送失败: {}", new String(message.getBody(), StandardCharsets.UTF_8));


    public <T> void sendMsg(String exchangeName, String routingKeyName, T content) {
        // 设置每个消息都返回一个确认消息
        // 设为 true 时，交换器无法根据自身的类型和路由键找到一个符合条件的队列,mq会调用Basic.Return命令将消息返回给生产者
        // 默认为false，直接丢弃
        this.rabbitTemplate.setMandatory(true);
        // 消息确认机制
        this.rabbitTemplate.setConfirmCallback(confirmCallback);
        // 消息发送失败机制
        this.rabbitTemplate.setReturnCallback(returnCallback);
        // 异步发送消息
        CorrelationData correlationData = new CorrelationData();
        correlationData.setId("123");
        this.rabbitTemplate.convertAndSend(exchangeName, routingKeyName, content, correlationData);
    }
```



消费者接收过程

1.消费者连接到 RabbitMQ Broker 建立一个连接( Connection) ，开启 个信道 (Channel)

2.消费者向 RabbitMQ Broker 请求消费相应队列中的消息，可能会设置相应的回调函数， 以及做 些准备工作

3.等待 RabbitMQ Broker 回应并投递相应队列中的消息， 消费者接收消息

4.消费者确认(ack) 接收到的消息

5.RabbitMQ 从队列中删除相应己经被确认的消息

6.关闭信道

7.关闭连接

### 3.对象传输的实现



## 11.如何保证消息不被重复消费

同:如何保证消息消费时的幂等性

先说为什么会重复消费：正常情况下，消费者在消费消息的时候，消费完毕后，会发送一个确认消息给消息队列，消息队列就知道该消息被消费了，就会将该消息从消息队列中删除；

但是因为网络传输等等故障，确认信息没有传送到消息队列，导致消息队列不知道自己已经消费过该消息了，再次将消息分发给其他的消费者。

针对以上问题，一个解决思路是：保证消息的唯一性，就算是多次传输，不要让消息的多次消费带来影响；保证消息等幂性；

比如：在写入消息队列的数据做唯一标示，消费消息时，根据唯一标识判断是否消费过；

假设你有个系统，消费一条消息就往数据库里插入一条数据，要是你一个消息重复两次，你不就插入了两条，这数据不就错了？但是你要是消费到第二次的时候，自己判断一下是否已经消费过了，若是就直接扔了，这样不就保留了一条数据，从而保证了数据的正确性。

生产者

```java
public class MessageIdempotentProducer {
    private static final String EXCHANGE_NAME = "message_idempotent_exchange";
    private static final String ROUTE_KEY = "message.insert";
 
    @Autowired
    private RabbitTemplate rabbitTemplate;
 
    /**
     * 发送消息
     */
    public void sendMessage() {
        //创建消费对象，并指定全局唯一ID(这里使用UUID，也可以根据业务规则生成，只要保证全局唯一即可)
        MessageProperties messageProperties = new MessageProperties();
        //这里通过设置UUID为消息全局ID，当然也可以使用时间戳或者业务标识+UUID都可以，只要保证消息ID唯一即可
        messageProperties.setMessageId(UUID.randomUUID().toString());
        messageProperties.setContentType("text/plain");
        messageProperties.setContentEncoding("utf-8");
        Message message = new Message("hello,message idempotent!".getBytes(), messageProperties);
        rabbitTemplate.convertAndSend(EXCHANGE_NAME, ROUTE_KEY, message);
    }
}
```

消费者

```java
public class MessageIdempotentConsumer {
 
    private static final Logger logger = LoggerFactory.getLogger(MessageIdempotentConsumer.class);
 
    @Autowired
    private MessageIdempotentRepository messageIdempotentRepository;
 
    @RabbitHandler
    //org.springframework.amqp.AmqpException: No method found for class [B 这个异常，并且还无限循环抛出这个异常。
    //注意@RabbitListener位置，笔者踩坑，无限报上面的错，还有另外一种解决方案: 配置转换器
    @RabbitListener(queues = "message_idempotent_queue")
    @Transactional
    public void handler(Message message, Channel channel) throws IOException {
        /**
         * 发送消息之前，根据消息全局ID去数据库中查询该条消息是否已经消费过，如果已经消费过，则不再进行消费。
         */
 
        // 获取消息Id
        String messageId = message.getMessageProperties().getMessageId();
        if (StringUtils.isBlank(messageId)) {
            logger.info("获取消费ID为空！");
            return;
        }
        MessageIdempotent messageIdempotent = messageIdempotentRepository.findOne(messageId);
        //如果找不到，则进行消费此消息
        if (null == messageIdempotent) {
            //获取消费内容
            String msg = new String(message.getBody(), StandardCharsets.UTF_8);
            logger.info("-----获取生产者消息-----------------" + "messageId:" + messageId + ",消息内容:" + msg);
            //手动ACK
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
 
            //存入到表中，标识该消息已消费
            MessageIdempotent idempotent = new MessageIdempotent();
            idempotent.setMessageId(messageId);
            idempotent.setMessageContent(msg);
            messageIdempotentRepository.save(idempotent);
        } else {
            //如果根据消息ID(作为主键)查询出有已经消费过的消息，那么则不进行消费；
            logger.error("该消息已消费，无须重复消费！");
        }
    }
}
```



## 12.如何确保消息正确地发送至 RabbitMQ？ 

比如消息发给了一个不存在的队列

1. 生产者确认
2. 持久化
3. 手动Ack

### 1.生产者确认机制

发送方确认机制是指生产者将信道设置成confirm（确认）模式，一旦信道进入confirm模式，所有在该信道上面发布的消息都会被指派一个唯一的ID（从1开始），一旦消息被投递到RabbitMQ服务器之后，RabbitMQ就会发送一个确认（Basic.Ack）给生产者（包含消息的唯一ID），这就使得生产者知晓消息已经正确到达了目的地了。

如果RabbitMQ因为自身内部错误导致消息丢失，就会发送一条nack（Basic.Nack）命令，生产者应用程序同样可以在回调方法中处理该nack指令。

   开启confirm模式的方法：

   生产者通过调用channel的confirmSelect方法将channel设置为confirm模式，(注意一点，已经在transaction事务模式的channel是不能再设置成confirm模式的，即这两种模式是不能共存的)，如果没有设置no-wait标志的话，broker会返回confirm.select-ok表示同意发送者将当前channel信道设置为confirm模式(从目前RabbitMQ最新版本3.6来看，如果调用了channel.confirmSelect方法，默认情况下是直接将no-wait设置成false的，也就是默认情况下broker是必须回传confirm.select-ok的，而且我也没找到我们自己能够设置no-wait标志的方法)；

   生产者实现confiem模式有三种编程方式：

#### 1.1.普通confirm模式

channel.waitForConfirms() 普通发送方确认模式；

每发送一条消息，调用waitForConfirms()方法等待服务端confirm，这实际上是一种串行的confirm，每publish一条消息之后就等待服务端confirm，如果服务端返回false或者超时时间内未返回，客户端进行消息重传；

 测试： 首先从最简单的开始，仅仅将channel设置成confirm模式，并且生产者每发送一条消息就等待broker回应确认消息，至于确认消息是什么我们不去做任何处理，为了测试方便，此处生产者只发送了5条消息，实现代码如下：

```java
class Sender
{
	private ConnectionFactory factory;
	private int count;
	private String exchangeName;
	private String 	queueName;
	private String routingKey;
	private String bindingKey;
	
	public Sender(ConnectionFactory factory,int count,String exchangeName,String queueName,String routingKey,String bindingKey) {
		this.factory = factory;
		this.count = count;
		this.exchangeName = exchangeName;
		this.queueName = queueName;
		this.routingKey = routingKey;
		this.bindingKey = bindingKey;
	}
	
	public void run() {
		Channel channel = null;
		try {
			Connection connection = factory.newConnection();
			channel = connection.createChannel();
			//创建exchange
			channel.exchangeDeclare(exchangeName, "direct", true, false, null);
			//创建队列
			channel.queueDeclare(queueName, true, false, false, null);
			//绑定exchange和queue
			channel.queueBind(queueName, exchangeName, bindingKey);
			channel.confirmSelect();
			//发送持久化消息
			for(int i = 0;i < count;i++)
			{
				//第一个参数是exchangeName(默认情况下代理服务器端是存在一个""名字的exchange的,
				//因此如果不创建exchange的话我们可以直接将该参数设置成"",如果创建了exchange的话
				//我们需要将该参数设置成创建的exchange的名字),第二个参数是路由键
				channel.basicPublish(exchangeName, routingKey,MessageProperties.PERSISTENT_BASIC, ("第"+(i+1)+"条消息").getBytes());
				if(channel.waitForConfirms())
				{
					System.out.println("发送成功");
				}
			}
			final long start = System.currentTimeMillis();
			System.out.println("执行waitForConfirmsOrDie耗费时间: "+(System.currentTimeMillis()-start)+"ms");
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
```

![img](https://img-blog.csdn.net/20170117115735204)

 可以看到上面生产者通过Confirm.Select将当前Channel信道设置成confirm模式，broker代理服务器收到之后回传Confirm.Select-Ok同一将当前Channel设置成confirm模式，此外看到返回5条Basic.Ack消息；

#### 1.2.批量confirm模式

channel.waitForConfirmsOrDie() 批量确认模式；

每发送一批消息之后，调用waitForConfirms()方法，等待服务端confirm，这种批量确认的模式极大的提高了confirm效率，但是如果一旦出现confirm返回false或者超时的情况，客户端需要将这一批次的消息全部重发，这会带来明显的重复消息，如果这种情况频繁发生的话，效率也会不升反降；

测试1：这种模式生产者不是每发送一条就等待broker确认，而是发送一批，实现代码见下

```java
class Sender
{
	private ConnectionFactory factory;
	private int count;
	private String exchangeName;
	private String 	queueName;
	private String routingKey;
	private String bindingKey;
	
	public Sender(ConnectionFactory factory,int count,String exchangeName,String queueName,String routingKey,String bindingKey) {
		this.factory = factory;
		this.count = count;
		this.exchangeName = exchangeName;
		this.queueName = queueName;
		this.routingKey = routingKey;
		this.bindingKey = bindingKey;
	}
	
	public void run() {
		Channel channel = null;
		try {
			Connection connection = factory.newConnection();
			channel = connection.createChannel();
			//创建exchange
			channel.exchangeDeclare(exchangeName, "direct", true, false, null);
			//创建队列
			channel.queueDeclare(queueName, true, false, false, null);
			//绑定exchange和queue
			channel.queueBind(queueName, exchangeName, bindingKey);
			channel.confirmSelect();
			//发送持久化消息
			for(int i = 0;i < count;i++)
			{
				//第一个参数是exchangeName(默认情况下代理服务器端是存在一个""名字的exchange的,
				//因此如果不创建exchange的话我们可以直接将该参数设置成"",如果创建了exchange的话
				//我们需要将该参数设置成创建的exchange的名字),第二个参数是路由键
				channel.basicPublish(exchangeName, routingKey,MessageProperties.PERSISTENT_BASIC, ("第"+(i+1)+"条消息").getBytes());
			}
			long start = System.currentTimeMillis();
			channel.waitForConfirmsOrDie();
			System.out.println("执行waitForConfirmsOrDie耗费时间: "+(System.currentTimeMillis()-start)+"ms");
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
```

       第50行调用channel.confirmSelect将当前channel信道设置成confirm模式，接着在第57行通过for循环发送了100条消息，第60行调用了channel的waitForConfirmsOrDie，从waitForConfirmsOrDie方法的注释上可以看出，该方法会等到最后一条消息得到确认或者得到nack才会结束，也就是说在waitForConfirmsOrDie处会造成当前程序的阻塞，以测试1程序发送100条消息为例，阻塞时间是135ms，我们再来看看对测试1的抓包情况：
![img](https://img-blog.csdn.net/20170115194209052)

 从红色箭头的标号1出可以看到：首先是24向74发送了Confirm.Select消息表示请求将当前信道设置为confirm模式，接着74向24回送了Confirm.Select-Ok消息表示同意将信道设置成confirm模式，从红色标号2处NoWait字段的值为false也印证了我们如果直接调用Channel信道的confirmSelect()方法的话，实际上默认是开启broker回传Confirm.Select-Ok确认消息的；  

   接下来我们看看broker回传给客户端的确认消息数据包是什么样子的呢？同样通过抓包看看结果：

![img](https://img-blog.csdn.net/20170115195051904)

你会发现，在上面测试1中我们通过for循环发送了100条消息，但是在抓包的时候我们仅仅看到有两个Basic.Ack确认消息回传回来，原因在于上面截图的标号3处，你会发现Multiple域的值是True的，之前我们已经讲过broker可以设置Multiple域表示broker已经收到当前确认消息的Delivery-Tag域之前标号的消息，以上面截图为例的话表示broker告诉发送者编号4之前的消息已经全部收到了，从这点我们看出broker端默认情况下是进行批量回复的，并不是针对每条消息都发送一条ack消息；

 测试2：  测试1我们仅仅是测试发送者能够收到broker的确认消息以及知道了broker对消息默认是采用批量回复方式的，那么在程序中我们该怎么获取到broker回传回来的确认消息呢，假如我们有时候需要在收到确认消息之后做一些提示性操作该怎么办呢？测试1中，我们采用的是Channel信道的waitForConfirmsOrDie等待broker端回传回ack确认消息的，但我们没法拿到这个ack消息进行后期操作，要想拿到ack消息的话，我们可以给当前Channel信道绑定监听器，具体来说就是调用Channel信道的addConfirmListener方法进行设置，Channel信道在收到broker的ack消息之后会回调设置在该信道监听器上的handleAck方法，在收到nack消息之后会回调设置在该信道监听器上的handleNack方法。

```java
class Sender
{
	private ConnectionFactory factory;
	private int count;
	private String exchangeName;
	private String 	queueName;
	private String routingKey;
	private String bindingKey;
	
	public Sender(ConnectionFactory factory,int count,String exchangeName,String queueName,String routingKey,String bindingKey) {
		this.factory = factory;
		this.count = count;
		this.exchangeName = exchangeName;
		this.queueName = queueName;
		this.routingKey = routingKey;
		this.bindingKey = bindingKey;
	}
	
	public void run() {
		Channel channel = null;
		try {
			Connection connection = factory.newConnection();
			channel = connection.createChannel();
			//创建exchange
			channel.exchangeDeclare(exchangeName, "direct", true, false, null);
			//创建队列
			channel.queueDeclare(queueName, true, false, false, null);
			//绑定exchange和queue
			channel.queueBind(queueName, exchangeName, bindingKey);
			channel.confirmSelect();
			//发送持久化消息
			for(int i = 0;i < count;i++)
			{
				//第一个参数是exchangeName(默认情况下代理服务器端是存在一个""名字的exchange的,
				//因此如果不创建exchange的话我们可以直接将该参数设置成"",如果创建了exchange的话
				//我们需要将该参数设置成创建的exchange的名字),第二个参数是路由键
				channel.basicPublish(exchangeName, routingKey,MessageProperties.PERSISTENT_BASIC, ("第"+(i+1)+"条消息").getBytes());
			}
			long start = System.currentTimeMillis();
			channel.addConfirmListener(new ConfirmListener() {
				
				@Override
				public void handleNack(long deliveryTag, boolean multiple) throws IOException {
					System.out.println("nack: deliveryTag = "+deliveryTag+" multiple: "+multiple);
				}
				
				@Override
				public void handleAck(long deliveryTag, boolean multiple) throws IOException {
					System.out.println("ack: deliveryTag = "+deliveryTag+" multiple: "+multiple);
				}
			});
			System.out.println("执行waitForConfirmsOrDie耗费时间: "+(System.currentTimeMillis()-start)+"ms");
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
```

我们调用了Channel信道的addConfirmListener设置了监听器，并且在监听器的handleAck和handleNack方法中打印了信息，运行程序查看输出：

![img](https://img-blog.csdn.net/20170115202644244)

  可以看到，虽然我们还是发送了100条消息，同样我们并没有收到100个ack消息 ，只收到两个ack消息，并且这两个ack消息的multiple域都为true，这点和测试1是相同的，你多次运行程序会发现每次发送回来的ack消息中的deliveryTag域的值并不是一样的，说明broker端批量回传给发送者的ack消息并不是以固定的批量大小回传的

 也就是我们通过信道Channel的waitForConfirmsOrDie方法或者为信道设置监听器都可以保证发送者收到broker回传的ack或者nack消息，那么这两种方式有什么区别呢？从测试一的第61行代码以及测试2的第72行代码处你就能找到答案啦，测试1中调用waitForConfirmsOrDie方法发送100条消息并且全部收到确认需要135ms，测试2中通过监听器的方式仅仅需要1ms，说明调用waitForConfirmsOrDie会造成程序的阻塞，通过监听器并不会造成程序的阻塞，下一篇博客我会试着从RabbitMQ的源码层面来分析这两种方式造成这种区别的原因啦？？？--《这个太屌了 我回头好好研究下》

#### 1.3异步监听confim模式

channel.addConfirmListener() 异步监听发送方确认模式

```java
public class Producer {

    /** 队列名称 */
    private static final String QUEUE_NAME = "test_queue";

    public static void main(String[] args) throws IOException, TimeoutException {
        /** 1.获取连接 */
        Connection newConnection = MQConnectionUtils.newConnection();
        /** 2.创建通道 */
        Channel channel = newConnection.createChannel();
        /** 3.创建队列声明 */
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        /** 4.开启发送方确认模式 */
        channel.confirmSelect();
        for (int i = 0; i < 10; i++) {
            String message = "我是生产者生成的消息：" + i;
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes("UTF-8"));
        }
        /** 5.发送消息 异步监听确认和未确认的消息 */
        channel.addConfirmListener(new ConfirmListener() {
            @Override
            public void handleNack(long deliveryTag, boolean multiple) throws IOException {
                System.out.println("未确认消息，标识：" + deliveryTag);
            }
            @Override
            public void handleAck(long deliveryTag, boolean multiple) throws IOException {
                System.out.println(String.format("已确认消息，标识：%d，多个消息：%b", deliveryTag, multiple));
            }
        });
        /** 6.关闭通道、连接 */
        /** channel.close();*/
        /** newConnection.close();*/
    }
}
```



### 2.事务机制

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200621002012494.png" alt="image-20200621002012494" style="zoom:80%;" />

RabbitMQ中与事务机制有关的方法有三个：txSelect(), txCommit()以及txRollback(), txSelect用于将当前channel设置成transaction模式，txCommit用于提交事务，txRollback用于回滚事务，在通过txSelect开启事务之后，我们便可以发布消息给broker代理服务器了，如果txCommit提交成功了，则消息一定到达了broker了，如果在txCommit执行之前broker异常崩溃或者由于其他原因抛出异常，这个时候我们便可以捕获异常通过txRollback回滚事务了。

```java
1.channel.txSelect();
2.channel.basicPublish(ConfirmConfig.exchangeName, ConfirmConfig.routingKey, MessageProperties.PERSISTENT_TEXT_PLAIN, ConfirmConfig.msg_10B.getBytes());
3.channel.txCommit();
```

无事务

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20170217163058261?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzI1NjgxNg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

有事务

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20170217163039667?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzI1NjgxNg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



发现事务的多了四个步骤：(这里的Tx.Commit与Tx.Commit-Ok之间的时间间隔294ms，由此可见事务还是很耗时的）

1. client发送Tx.Select
2. broker发送Tx.Select-Ok(之后publish)
3. client发送Tx.Commit
4. broker发送Tx.Commit-Ok

下面我们来看下事务回滚是什么样子的。关键代码如下：

```java
try {
    channel.txSelect();
    channel.basicPublish(exchange, routingKey, MessageProperties.PERSISTENT_TEXT_PLAIN, msg.getBytes());
    int result = 1 / 0;
    channel.txCommit();
} catch (Exception e) {
    e.printStackTrace();
    channel.txRollback();
}
```

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20170217163110083?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzI1NjgxNg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

代码中先是发送了消息至broker中但是这时候发生了异常，之后在捕获异常的过程中进行事务回滚。

### 3.生产者确认机制与事务机制的对比

事务机制在一条消息发送之后会使发送端阻塞，以等待RabbitMQ的回应，之后才能继续发送下一条消息。

相比之下，发送方确认机制最大的好处在于它是异步的，一旦发布一条消息。生产者应用程序就可以在等信道返回确认的同时继续发送下一条消息，当消息最终得到确认后，生产者应用程序便可以通过回调方法来处理该确认消息。



## 13.如何确保消息接收方消费了消息？

接收方确认机制

消费者接收每一条消息后都必须进行确认（消息接收和消息确认是两个不同操作）。只有消费者确认了消息，RabbitMQ 才能安全地把消息从队列中删除。

这里并没有用到超时机制，RabbitMQ 仅通过 Consumer 的连接中断来确认是否需要重新发送消息。也就是说，只要连接不中断，RabbitMQ 给了 Consumer 足够长的时间来处理消息。保证数据的最终一致性；

下面罗列几种特殊情况

如果消费者接收到消息，在确认之前断开了连接或取消订阅，RabbitMQ 会认为消息没有被分发，然后重新分发给下一个订阅的消费者。（可能存在消息重复消费的隐患，需要去重）
如果消费者接收到消息却没有确认消息，连接也未断开，则 RabbitMQ 认为该消费者繁忙，将不会给该消费者分发更多的消息。





## 14.如何保证RabbitMQ消息的可靠传输？

消息不可靠的情况可能是消息丢失，劫持等原因；

丢失又分为：生产者丢失消息、消息列表丢失消息、消费者丢失消息；

### 1.生产者丢失消息

从生产者弄丢数据这个角度来看，RabbitMQ提供transaction和confirm模式来确保生产者不丢消息；

transaction机制就是说：发送消息前，开启事务（channel.txSelect()）,然后发送消息，如果发送过程中出现什么异常，事务就会回滚（channel.txRollback()）,如果发送成功则提交事务（channel.txCommit()）。然而，这种方式有个缺点：吞吐量下降；

confirm模式用的居多：一旦channel进入confirm模式，所有在该信道上发布的消息都将会被指派一个唯一的ID（从1开始），一旦消息被投递到所有匹配的队列之后；

rabbitMQ就会发送一个ACK给生产者（包含消息的唯一ID），这就使得生产者知道消息已经正确到达目的队列了；

如果rabbitMQ没能处理该消息，则会发送一个Nack消息给你，你可以进行重试操作。

### 2.消息队列丢数据

消息持久化

处理消息队列丢数据的情况，一般是开启持久化磁盘的配置。

这个持久化配置可以和confirm机制配合使用，你可以在消息持久化磁盘后，再给生产者发送一个Ack信号。

这样，如果消息持久化磁盘之前，rabbitMQ阵亡了，那么生产者收不到Ack信号，生产者会自动重发。

那么如何持久化呢？

这里顺便说一下吧，其实也很容易，就下面两步

将queue的持久化标识durable设置为true,则代表是一个持久的队列
发送消息的时候将deliveryMode=2
这样设置以后，即使rabbitMQ挂了，重启后也能恢复数据

### 3.消费者丢失消息

消费者丢数据一般是因为采用了自动确认消息模式，改为手动确认消息即可！

消费者在收到消息之后，处理消息之前，会自动回复RabbitMQ已收到消息；

如果这时处理消息失败，就会丢失该消息；

解决方案：处理消息成功后，手动回复确认消息。


## 15. message 持久化机制

在RabbitMQ中，如果遇到RabbitMQ服务停止或者挂掉，那么我们的消息将会出现丢失的情况，为了在RabbitMQ服务重启的情况下，不丢失消息，我们可以将Exchange（交换机）、Queue（队列）与Message（消息）都设置为可持久化的（durable）。这样的话，能够保证绝大部分的消息不会被丢失，但是还有有一些小概率会发生消息丢失的情况。下面通过一个简单的示例总结在RabbitMQ中如何进行消息持久化。

### 1.交换机持久化

声明交换机Exchange的时候设置 durable=true

```java
//public TopicExchange(String name, boolean durable, boolean autoDelete)
public TopicExchange exchange() {
        return new TopicExchange(EXCHANGE_NAME,true,false);
    }
```

### 2.队列持久化

声明队列Queue的时候设置 durable=true

```java
//public Queue(String name, boolean durable)
@Bean
    public Queue queue() {
        //durable：是否将队列持久化 true表示需要持久化 false表示不需要持久化
        return new Queue(QUEUE_NAME, true);
    }
```

### 3.消息持久化

发送消息的时候设置消息的 deliveryMode = 2

```java
//使用convertAndSend方式发送消息，消息默认就是持久化的
//Delivery mode: 是否持久化，1 - Non-persistent，2 - Persistent
new MessageProperties() --> DEFAULT_DELIVERY_MODE = MessageDeliveryMode.PERSISTENT --> deliveryMode = 2;
```

### 4.测试案例

这里我们声明两种队列，一个是持久化的，一个是非持久化的，我们测试在RabbitMQ服务重启的情况下，未被消费的消息是否还存在，消费者重启之后能否重新消费之前的消息。在RabbitMQ中，通过管理员方式（必须是管理员方式，否则可能会报错）运行CMD命令行执行下面的命令

```shell
net stop RabbitMQ && net start RabbitMQ
```

#### 4.1RabbitMQ配置类

RabbitMQ配置信息，绑定交换器、队列、路由键设置

```java
/**
 * @Description: RabbitMQ配置信息，绑定交换器、队列、路由键设置
 * <p>
 * 如果我们希望即使在RabbitMQ服务重启的情况下，也不会丢失消息，可以将交换机、队列、消息都进行持久化，这样可以保证绝大部分情况下消息不会丢失。
 * 但还是会有小概率发生消息丢失的情况（比如RabbitMQ服务器已经接收到生产者的消息，但还没来得及持久化该消息时RabbitMQ服务器就断电了），
 * 如果我们需要对这种小概率事件也要管理起来，那么我们要用到事务。(transaction/confirm机制)
 *
 * <p>
 * 说明：
 * 1. 队列持久化：需要在声明队列的时候设置durable=true，如果只对队列进行持久化，那么mq重启之后队列里面的消息不会保存
 * 如果需要队列里面的消息也保存下来，那么还需要对消息进行持久化；
 * <p>
 * 2. 消息持久化：设置消息的deliveryMode = 2，消费者重启之后还能够继续消费持久化之后的消息；
 * 使用convertAndSend方式发送消息，消息默认就是持久化的，下面是源码：
 * new MessageProperties() --> DEFAULT_DELIVERY_MODE = MessageDeliveryMode.PERSISTENT --> deliveryMode = 2;
 * <p>
 * 3.重启mq: CMD命令行下执行 net stop RabbitMQ && net start RabbitMQ
 */
public class RabbitMQConfig {
    private static final String DURABLE_QUEUE_NAME = "durable_queue_name";
    private static final String DURABLE_EXCHANGE_NAME = "durable_exchange_name";
    private static final String ROUTING_KEY = "user.#";
 
    private static final String QUEUE_NAME = "not_durable_queue_name";
    private static final String EXCHANGE_NAME = "not_durable_exchange_name";
 
    @Bean
    public Queue durableQueue() {
//        public Queue(String name) {
//            this(name, true, false, false);
//        }
//        public Queue(String name, boolean durable, boolean exclusive, boolean autoDelete)
        //不指定durable的话默认好像也是true
 
        //public Queue(String name, boolean durable)
        //durable：是否将队列持久化 true表示需要持久化 false表示不需要持久化
        return new Queue(DURABLE_QUEUE_NAME, true);
    }
 
    @Bean
    public TopicExchange durableExchange() {
//        public AbstractExchange(String name) {
//            this(name, true, false);
//        }
//        public AbstractExchange(String name, boolean durable, boolean autoDelete) {
//            this(name, durable, autoDelete, (Map)null);
//        }
        //声明交换机的时候默认也是持久化的
        return new TopicExchange(DURABLE_EXCHANGE_NAME);
    }
 
    @Bean
    public Binding durableBinding() {
        //如果exchange和queue都是持久化的，那么它们之间的binding也是持久化的。如果exchange和queue两者之间有一个持久化，一个非持久化，就不允许建立绑定
        return BindingBuilder.bind(durableQueue()).to(durableExchange()).with(ROUTING_KEY);
    }
 
 
    @Bean
    public Queue queue() {
        //public Queue(String name, boolean durable)
        //durable：是否将队列持久化 true表示需要持久化 false表示不需要持久化
        return new Queue(QUEUE_NAME, false);
    }
 
    @Bean
    public TopicExchange exchange() {
        return new TopicExchange(EXCHANGE_NAME,true,false);
    }
 
    @Bean
    public Binding binding() {
        return BindingBuilder.bind(queue()).to(exchange()).with(ROUTING_KEY);
    }
 
}
```

#### 4.2生产者

```java
public class Producer {
    private static final Logger logger = LoggerFactory.getLogger(Producer.class);
    private static final String DURABLE_EXCHANGE_NAME = "durable_exchange_name";
    private static final String ROUTING_KEY = "user.add";
 
    @Autowired
    private RabbitTemplate rabbitTemplate;
 
    public void sendMessage() {
        for (int i = 1; i <= 3; i++) {
            String message = "消息" + i;
            logger.info("【生产者】发送消息：" + message);
            rabbitTemplate.convertAndSend(DURABLE_EXCHANGE_NAME, ROUTING_KEY, message, new CorrelationData(UUID.randomUUID().toString()));
        }
    }
 
}
```

使用convertAndSend方式发送消息，消息默认就是持久化的

------

convertAndSend(…)：使用此方法，交换机会马上把所有的信息都交给所有的消费者，消费者再自行处理，不会因为消费者处理慢而阻塞线程。

convertSendAndReceive(…)：可以同步消费者。使用此方法，当确认了所有的消费者都接收成功之后，才触发另一个convertSendAndReceive(…)，也就是才会接收下一条消息。RPC调用方式。

------

#### 4.3消费者

```java
public class Consumer {
    private static final Logger logger = LoggerFactory.getLogger(Consumer.class);
 
    @RabbitListener(queues = "durable_queue_name")
    public void receiveMessage(String msg, Message message, Channel channel) {
        try {
            logger.info("【Consumer  receiveMessage】接收到消息为:[{}]", msg);
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
        } catch (Exception e) {
            logger.info("【Consumer  receiveMessage】接收消息后的处理发生异常", e);
        }
    }
 
}
```

#### 4.4运行结果

为了测试在MQ重启之后，消费者能够消费持久化之后的消息，这里可以先将消费者监听队列暂时注释掉，让生产者发送三条消息，但是没有消费者去消费，这样MQ重启之后，队列中还是存在三条消息。

![img](https://img-blog.csdnimg.cn/20190706090224813.png)

通过控制台，可见只剩下一个持久化的队列**durable_queue_name，并且队列里面的消息还存在，另外一个**not_durable_queue_name**已经丢失，说明在该队列上的消息也已经丢失，同理，交换机也类似。

![img](https://img-blog.csdnimg.cn/20190706090051679.png)

这时候放开之前注释的消费者代码块，重启项目，通过控制台可以看到消费者成功消费了之前持久化的三条消息，由此证明了在MQ重启之后，消费者可以继续消费之前的消息

![img](https://img-blog.csdnimg.cn/20190706090611257.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dlaXhpYW9odWFp,size_16,color_FFFFFF,t_70)

以上就是关于在RabbitMQ中如何实现消息和队列的持久化，虽然不能说百分之百保证消息不会丢失，但是能够保证绝大部分不会丢失。在实际项目中，通常需要对消息进行持久化，因为不可能保证服务器永远不会出现down机情况。

#### 5.为什么不应该对所有的 message 都使用持久化机制

首先，必然导致性能的下降，因为写磁盘比写 RAM 慢的多，message 的吞吐量可能有 10 倍的差距。

其次，message 的持久化机制用在 RabbitMQ 的内置 cluster 方案时会出现“坑爹”问题。矛盾点在于，若 message 设置了 persistent 属性，但 queue 未设置 durable 属性，那么当该 queue 的 owner node 出现异常后，在未重建该 queue 前，发往该 queue 的 message 将被 blackholed ；若 message 设置了 persistent 属性，同时 queue 也设置了 durable 属性，那么当 queue 的 owner node 异常且无法重启的情况下，则该 queue 无法在其他 node 上重建，只能等待其 owner node 重启后，才能恢复该 queue 的使用，而在这段时间内发送给该 queue 的 message 将被 blackholed 。

所以，是否要对 message 进行持久化，需要综合考虑性能需要，以及可能遇到的问题。若想达到 100,000 条/秒以上的消息吞吐量（单 RabbitMQ 服务器），则要么使用其他的方式来确保 message 的可靠 delivery ，要么使用非常快速的存储系统以支持全持久化（例如使用 SSD）。另外一种处理原则是：仅对关键消息作持久化处理（根据业务重要程度），且应该保证关键消息的量不会导致性能瓶颈。


## 16.如何保证高可用的？RabbitMQ 的集群

RabbitMQ 是比较有代表性的，因为是基于主从（非分布式）做高可用性的，我们就以 RabbitMQ 为例子讲解第一种 MQ 的高可用性怎么实现。RabbitMQ 有三种模式：单机模式、普通集群模式、镜像集群模式。

单机模式，就是 Demo 级别的，一般就是你本地启动了玩玩儿的?，没人生产用单机模式

普通集群模式，意思就是在多台机器上启动多个 RabbitMQ 实例，每个机器启动一个。你创建的 queue，只会放在一个 RabbitMQ 实例上，但是每个实例都同步 queue 的元数据（元数据可以认为是 queue 的一些配置信息，通过元数据，可以找到 queue 所在实例）。你消费的时候，实际上如果连接到了另外一个实例，那么那个实例会从 queue 所在实例上拉取数据过来。这方案主要是提高吞吐量的，就是说让集群中多个节点来服务某个 queue 的读写操作。

镜像集群模式：这种模式，才是所谓的 RabbitMQ 的高可用模式。跟普通集群模式不一样的是，在镜像集群模式下，你创建的 queue，无论元数据还是 queue 里的消息都会存在于多个实例上，就是说，每个 RabbitMQ 节点都有这个 queue 的一个完整镜像，包含 queue 的全部数据的意思。然后每次你写消息到 queue 的时候，都会自动把消息同步到多个实例的 queue 上。RabbitMQ 有很好的管理控制台，就是在后台新增一个策略，这个策略是镜像集群模式的策略，指定的时候是可以要求数据同步到所有节点的，也可以要求同步到指定数量的节点，再次创建 queue 的时候，应用这个策略，就会自动将数据同步到其他的节点上去了。这样的话，好处在于，你任何一个机器宕机了，没事儿，其它机器（节点）还包含了这个 queue 的完整数据，别的 consumer 都可以到其它节点上去消费数据。坏处在于，第一，这个性能开销也太大了吧，消息需要同步到所有机器上，导致网络带宽压力和消耗很重！RabbitMQ 一个 queue 的数据都是放在一个节点里的，镜像集群下，也是每个节点都放这个 queue 的完整数据。


## 17.生产问题

或者： 消息队列满了以后该怎么处理？  或者： 几百万消息持续积压几小时，怎么解决？

### 1.面试官心理分析

你看这问法，其实本质针对的场景，都是说，可能你的消费端出了问题，不消费了；或者消费的速度极其慢。接着就坑爹了，可能你的消息队列集群的磁盘都快写满了，都没人消费，这个时候怎么办？或者是这整个就积压了几个小时，你这个时候怎么办？或者是你积压的时间太长了，导致比如 RabbitMQ 设置了消息过期时间后就没了怎么办？

所以就这事儿，其实线上挺常见的，一般不出，一出就是大 case。一般常见于，举个例子，消费端每次消费之后要写 mysql，结果 mysql 挂了，消费端 hang 那儿了，不动了；或者是消费端出了个什么岔子，导致消费速度极其慢。

### 2.面试题剖析

关于这个事儿，我们一个一个来梳理吧，先假设一个场景，我们现在消费端出故障了，然后大量消息在 mq 里积压，现在出事故了，慌了。

大量消息在 mq 里积压了几个小时了还没解决

几千万条数据在 MQ 里积压了七八个小时，从下午 4 点多，积压到了晚上 11 点多。这个是我们真实遇到过的一个场景，确实是线上故障了，这个时候要不然就是修复 consumer 的问题，让它恢复消费速度，然后傻傻的等待几个小时消费完毕。这个肯定不能在面试的时候说吧。

一个消费者一秒是 1000 条，一秒 3 个消费者是 3000 条，一分钟就是 18 万条。所以如果你积压了几百万到上千万的数据，即使消费者恢复了，也需要大概 1 小时的时间才能恢复过来。

一般这个时候，只能临时紧急扩容了，具体操作步骤和思路如下：

先修复 consumer 的问题，确保其恢复消费速度，然后将现有 consumer 都停掉。

新建一个 topic，partition 是原来的 10 倍，临时建立好原先 10 倍的 queue 数量。

然后写一个临时的分发数据的 consumer 程序，这个程序部署上去消费积压的数据，**消费之后不做耗时的处理**，直接均匀轮询写入临时建立好的 10 倍数量的 queue。

接着临时征用 10 倍的机器来部署 consumer，每一批 consumer 消费一个临时 queue 的数据。这种做法相当于是临时将 queue 资源和 consumer 资源扩大 10 倍，以正常的 10 倍速度来消费数据。

等快速消费完积压数据之后，**得恢复原先部署的架构**，**重新**用原先的 consumer 机器来消费消息。

### 3.mq中的消息过期失效了

假设你用的是 RabbitMQ，RabbtiMQ 是可以设置过期时间的，也就是 TTL。如果消息在 queue 中积压超过一定的时间就会被 RabbitMQ 给清理掉，这个数据就没了。那这就是第二个坑了。这就不是说数据会大量积压在 mq 里，而是**大量的数据会直接搞丢**。

这个情况下，就不是说要增加 consumer 消费积压的消息，因为实际上没啥积压，而是丢了大量的消息。我们可以采取一个方案，就是**批量重导**，这个我们之前线上也有类似的场景干过。就是大量积压的时候，我们当时就直接丢弃数据了，然后等过了高峰期以后，比如大家一起喝咖啡熬夜到晚上12点以后，用户都睡觉了。这个时候我们就开始写程序，将丢失的那批数据，写个临时程序，一点一点的查出来，然后重新灌入 mq 里面去，把白天丢的数据给他补回来。也只能是这样了。

假设 1 万个订单积压在 mq 里面，没有处理，其中 1000 个订单都丢了，你只能手动写程序把那 1000 个订单给查出来，手动发到 mq 里去再补一次。

### 4.mq 都快写满了

如果消息积压在 mq 里，你很长时间都没有处理掉，此时导致 mq 都快写满了，咋办？这个还有别的办法吗？没有，谁让你第一个方案执行的太慢了，你临时写程序，接入数据来消费，**消费一个丢弃一个，都不要了**，快速消费掉所有的消息。然后走第二个方案，到了晚上再补数据吧。

### 5.死信队列和延迟队列的使用

### 6.排他队列



## 18.设计MQ思路

比如说这个消息队列系统，我们从以下几个角度来考虑一下：

首先这个 mq 得支持可伸缩性吧，就是需要的时候快速扩容，就可以增加吞吐量和容量，那怎么搞？设计个分布式的系统呗，参照一下 kafka 的设计理念，broker -> topic -> partition，每个 partition 放一个机器，就存一部分数据。如果现在资源不够了，简单啊，给 topic 增加 partition，然后做数据迁移，增加机器，不就可以存放更多数据，提供更高的吞吐量了？

其次你得考虑一下这个 mq 的数据要不要落地磁盘吧？那肯定要了，落磁盘才能保证别进程挂了数据就丢了。那落磁盘的时候怎么落啊？顺序写，这样就没有磁盘随机读写的寻址开销，磁盘顺序读写的性能是很高的，这就是 kafka 的思路。

其次你考虑一下你的 mq 的可用性啊？这个事儿，具体参考之前可用性那个环节讲解的 kafka 的高可用保障机制。多副本 -> leader & follower -> broker 挂了重新选举 leader 即可对外服务。

能不能支持数据 0 丢失啊？可以的，参考我们之前说的那个 kafka 数据零丢失方案。


## 19.RabbitMQ元数据

什么是元数据？元数据分为哪些类型？包括哪些内容？与 cluster 相关的元数据有哪些？元数据是如何保存的？元数据在 cluster 中是如何分布的？



