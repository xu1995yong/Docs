# RocketMQ

## 消息队列功能介绍
1. 应用解耦
2. 流量削峰
3. 消息分发

## 核心概念
1.	Producer
生产者将消息发送给brokers。RocketMQ提供多种发送范例：同步，异步和单向。
2.	Producer Group
Producer Group是具有相同角色的生产者的组合。如果原始生产者在事务之后崩溃，则brokers可以联系同一Producer Group的不同生产者实例以提交或回滚事务
3.	Consumer
消费者从brokers获取消息并将其提供给应用程序。从用户应用的角度来看，提供了两种类型的消费者：
	-	PullConsumer ：读取操作中的大部分功能由使用者自主控制
	-	PushConsumer：由系统控制读取操作，收到消息后自动调用处理方法来处理消息，自动保存offset，而且新加入PushConsumer会自动做负载均衡。
4.	Consumer Group
Consumer Group是具有相同角色的消费者的组合。Consumer Group使消息消费方面实现负载平衡和容错目标非常容易。 ** 警告：Consumer Group的使用者实例必须具有完全相同的主题订阅。**
5.	Topic
Topic是Producer传递消息和Consumer提取消息的类别。Topic与Producer和Consumer的关系非常松散。
	-	一个Topic可能有零个，一个或多个Producer向它发送消息；相反，Producer可以发送不同Topic的消息。
	-	从Consumer的角度来看，Topic可以由零个，一个或多个Consumer群体订阅。类似地，Consumer Group可以订阅一个或多个Topic，只要该组的实例保持其订阅一致即可。
6.	Message
Message是要传递的信息。Message必须有一个主题，可以将其解释为您要发送给的邮件地址。Message还可以具有可选标记和额外的键-值对。
7.	Message Queue
8.	Tag
Tag也即子主题，为用户提供了额外的灵活性。对于Tag，来自同一业务模块的具有不同目的的消息可以具有相同的主题和不同的标记。标签有助于保持代码的清晰和连贯，而标签也可以方便RocketMQ提供的查询系统。
9.	Broker
Broker是RocketMQ系统的主要组成部分。它接收从生产者发送的消息，存储它们并准备处理来自消费者的拉取请求。它还存储与消息相关的元数据，包括消费者组，消耗进度偏移和主题/队列信息。
10.	Name Server
11.	Message Model
	-	Clustering：同一个ConsumerGroup里的每个Consumer只消费所订阅消息的一部分内容，同一个ConsumerGroup里所有的Consumer消费的内容合起来才是所订阅Topic内容的整体，从而达到负载均衡的目的。
	-	Broadcasting：同一个ConsumerGroup里的每个Consumer都能消费到所订阅Topic的全部消息，也就是一个消息会被多次分发。
12.	Message Order
当使用DefaultMQPushConsumer时，可能决定按顺序或并发使用消息。
	- Orderly：按顺序使用消息意味着消息的使用顺序与生产者为每个消息队列发送的顺序相同。如果您正在处理全局顺序是必需的方案，请确保您使用的主题只有一个消息队列。警告：如果指定了有序消耗，则消息消耗的最大并发数是消费者组订阅的消息队列数。 
	- Concurrently：** 警告：此模式不再保证消息顺序. **

## 生产者

## 消费者

### PushConsumer的原理
** 
PushConsumer中通过“长轮询”方式达到Push效果，长轮询方式既有Pull的优点，又有Push方式的实时性。
**
为什么使用长轮询方式：    
  1. 单纯的Push方式的弊端：
	- 加大Server端的工作量，进而影响Server的性能。
	- Client端的处理能力各不相同，Client的状态不受Server控制，如果Client不能及时处理Server推送的消息，就会造成各种问题。	
  2. 单纯的Pull方式的弊端：
	  - 循环拉取消息的间隔不好设定，间隔太短就容易处在“忙等”状态，浪费资源；时间间隔太长，又无法及时处理消息。
	  
### 长轮询方式源码解读
1. Client端
2. Broker端


## Example
1. Add Dependency    

	
	<dependency>
   ​     <groupId>org.apache.rocketmq</groupId>
   ​     <artifactId>rocketmq-client</artifactId>
   ​     <version>4.3.0</version>
    </dependency>

2.1 Send Messages Synchronously

	public class SyncProducer {
	    public static void main(String[] args) throws Exception {
	        DefaultMQProducer producer = new  DefaultMQProducer("please_rename_unique_group_name");
	        producer.setNamesrvAddr("localhost:9876");
	        producer.start();
	        Message msg = new Message("TopicTest","TagA" ,
	               "Hello RocketMQ".getBytes(RemotingHelper.DEFAULT_CHARSET));
	            SendResult sendResult = producer.send(msg);
	            System.out.printf("%s%n", sendResult);
	        }
	        producer.shutdown();
	    }
	}

2.2 Send Messages Asynchronously

	public class AsyncProducer {
	    public static void main(String[] args) throws Exception {
	        DefaultMQProducer producer = new DefaultMQProducer("please_rename_unique_group_name"); 
	        producer.setNamesrvAddr("localhost:9876");
	        producer.start();
	        producer.setRetryTimesWhenSendAsyncFailed(0);
	        Message msg = new Message("TopicTest","TagA","OrderID188","Hello world".getBytes(RemotingHelper.DEFAULT_CHARSET));
	        producer.send(msg, new SendCallback() {
	            @Override
	            public void onSuccess(SendResult sendResult) {
	                 System.out.printf(sendResult);
	            }
	            @Override
	            public void onException(Throwable e) {
	                 e.printStackTrace();
	            }
	        });
	        producer.shutdown();
	    }
	}

3.1 Consume Messages

```java
//需要设置三个参数：1.Consumer的GroupName，2.NameServer的地址和端口号，3.要订阅的Topic
public class Consumer {
    public static void main(String[] args) throws Exception{
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("please_rename_unique_group_name");
        consumer.setNamesrvAddr("localhost:9876");
        consumer.subscribe("TopicTest", "*");
        consumer.registerMessageListener(new MessageListenerConcurrently() {

            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,
                                                            ConsumeConcurrentlyContext context) {
                System.out.printf( msgs);
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        consumer.start();
        System.out.printf("Consumer Started.%n");
    }
}
```

## 有序消息

### 1. 全局消息有序
若要保证全局消息有序，需要：
- 发送端：不能异步发送，异步发送在发送失败的情况下，就没办法保证消息顺序。比如你连续发了1，2，3。 过了一会，返回结果1失败，2, 3成功。你把1再重新发送1遍，这个时候顺序就乱掉了。
- 存储端：消息不能分区。也就是1个topic，只能有1个队列。如果你有多个队列，那同1个topic的消息，会分散到多个分区里面，自然不能保证顺序。
- 接收端：不能并行消费，也即不能开多线程或者多个客户端消费同1个队列。


### 2. 局部消息有序
发送端：只能选择同步发送（异步发送在发送失败时无法保证消息顺序），并且把同一个业务id（orderId）的消息发送到同一个Message Queue。这样每个Message Queue中的消息是有序的。

	for (int i = 0; i < 100; i++) {
			Integer orderId = i % 10;
			Message msg = new Message("OrderedMessage", "*",String.valueOf(i).getBytes(RemotingHelper.DEFAULT_CHARSET));
			SendResult sendResult = producer.send(msg, new MessageQueueSelector() {
				public MessageQueue select(List<MessageQueue> mqs, Message msg, Object arg) //arg就是传入的orderId {
					Integer id = (Integer) arg;
					int index = id % mqs.size();
					return mqs.get(index);
				}
			}, orderId);
	}

消费端：使用MessageListenerOrderly类保证同一个Message Queue读取的消息不被并发处理。在MessageListenerOrderly的实现中，为每个Consumer Queue加个锁，消费消息前，都需要先获得这个消息对应的Consumer Queue上的锁。这样保证了同一时间，同一个Consumer Queue的消息只被一个线程消费，而不同Consumer Queue上的消息可以并发处理。

```java
	consumer.registerMessageListener(new MessageListenerOrderly() {
		public ConsumeOrderlyStatus consumeMessage(List<MessageExt> msgs, ConsumeOrderlyContext context) {
			for (MessageExt msg : msgs) {
				if (msg.getQueueId() == 0)
					System.out.println(new String(msg.getBody()));
			}
			System.out.printf(Thread.currentThread().getName() + " Receive New Messages:" + msgs + "%n");

			return ConsumeOrderlyStatus.ROLLBACK;
		}
	});
```