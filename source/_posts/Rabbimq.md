---
title: Rabbimq
date: '2025/5/11 21:17:06'
updated: '2025-07-06 11:11:30'
categories: 消息队列
tags:
  - 消息队列
  - Rabbimq
---
# 入门


## 概念介绍
<font style="color:transparent;">ge: " + new</font>

<font style="color:transparent;">#ing(body));  try {  TimeUnit.SECONDS.sleep(1);  } catch (InterruptedException e) {  e.printStackTrace();  }  channel.basicAck(envelope.getDeliveryTag(), false); </font>![](/images/416710ec4e3bfa699218c9158b79b252.png)



### 生产者和消费者
#### 生产者 （消息投递）
消息=消息体（业务数据）和标签（消息分类）



#### 消费者（接收消息）
订阅消息队列接收消息，消费队列中的消息（只有消息体）



### 队列


#### 消息传输过程
![](/images/ea713b39b201d4ea17418422bfddb743.png)

> 首先生产者将业务方数据进行可能的包装，之后封装成消息，发送（AMQP协议里这个动作对应的命令为Basic.Publish）到Broker中。消费者订阅并接收消息（AMQP协议里这个动作对应的命令为Basic.Consume或者Basic.Get），经过可能的解包处理得到原始的数据，之后再进行业务处理逻辑。这个业务处理逻辑并不一定需要和接收消息的逻辑使用同一个线程。消费者进程可以使用一个线程去接收消息，存入到内存中，比如使用Java中的BlockingQueue。业务处理逻辑使用另一个线程从内存中读取数据，这样可以将应用进一步解耦，提高整个应用的处理效率。
>



#### 队列
![](/images/0bcdbf262fa23dac81c35959340cb2fe.png)

**存储消息**

**不支持广播消费**： 队列可以一对 N 个消费者，但是消息会被平均分摊给多个消费者（轮询），而不是所有消费者





#### 交换器
![](/images/c161f7a5f050f60a12e2266d46fbda39.png)

将消息发送给指定的队列





#### bindingkey
将交换器和队列绑定

#### routingkey
指定消息的路由规则

#### 两者区别在大部分场景下可以相同，但是部分场景下不同


> 沿用本章开头的比喻，交换器相当于投递包裹的邮箱，RoutingKey相当于填写在包裹上的地址，BindingKey相当于包裹的目的地，当填写在包裹上的地址和实际想要投递的地址相匹配时，那么这个包裹就会被正确投递到目的地，最后这个目的地的“主人”——队列可以保留这个包裹。如果填写的地址出错，邮递员不能正确投递到目的地，包裹可能会回退给寄件人，也有可能被丢弃
>

> 在direct交换器类型下，RoutingKey和BindingKey需要完全匹配才能使用，所以上面代码中采用了此种写法会显得方便许多。但是在topic交换器类型下，RoutingKey和BindingKey之间需要做模糊匹配，两者并不是相同的。
>
> BindingKey其实也属于路由键中的一种，官方解释为：the routing key to use for the binding。可以翻译为：在绑定的时候使用的路由键。大多数时候，包括官方文档和RabbitMQ  Java  API中都把BindingKey和RoutingKey看作RoutingKey，为了避免混淆，可以这么理解： 在使用绑定的时候，其中需要的路由键是BindingKey。涉及的客户端方法如：channel.exchangeBind、channel.queueBind，对应的AMQP命令（详情参见2.2节）为Exchange.Bind、Queue.Bind。 在发送消息的时候，其中需要的路由键是RoutingKey。涉及的客户端方法如channel.basicPublish，对应的AMQP命令为Basic.Publish
>







#### 交换器类型
常用四种 ：direct 、fanout、topic、headers

1. fanout 发送给所有绑定改交换器的队列
2. direct 发送给 routingkey 和 bindingkey 完全匹配的队列
3. topic： 存在比 direct 更多样的匹配规则
4. headers：消息体中的 headers 属性匹配交换器和队列绑定时的指定键值对

![](/images/2b77ff253e32ab95116811f89091ca48.png)

#### topic 匹配规则
1. RoutingKey为一个点号“.”分隔的字符串（被点号“.”分隔开的每一段独立的字符串称为一个单词），如“com.rabbitmq.client”、“java.util.concurrent”、“com.hidden.client”； 
2. BindingKey和RoutingKey一样也是点号“.”分隔的字符串； 
3. BindingKey中可以存在两种特殊字符串“*”和“#”，用于做模糊匹配，其中“#”用于匹配一个单词，“*”用于匹配多规格单词（可以是零个）









## AMQP 协议
协议三层 

1. module layer 
2. session layer
3. transport layer







### 生产者流转
![](/images/8840f7b4202b8729f019988778bb0589.png)

### 消费者流转
![](/images/6c08f544c88d86da619d4d39115653fd.png)



### 命令概览
![](/images/220a1d93470d18a50abf294bf39fb263.png)







# 客户端开发
## 连接 Rabbitmq
1. 通过 URI 的方式连接
2. 通过设定指定参数连接

```java
//连接方式1
factory.setHost(HOST);
factory.setPort(PORT);
factory.setUsername("root");
factory.setPassword("root123");
//连接方式2
factory.setUri("amqp://root:root123@192.168.150.101:5672");
```

### 易错点


1. 多线程共享 channel 是不安全的，应用程序应该为每一个线程开辟一个Channel。

> 某些情况下Channel的操作可以并发运行，但是在其他情况下会导致在网络上出现错误的通信帧交错，同时也会影响发送方确认（publisher confirm）机制的运行（，所以多线程间共享Channel实例是非线程安全的
>

2. 不推荐使用 isOpen 方法校验通道和连接状态





## 交换器和队列的使用
#### 交换器声明
##### exchangeDeclare
##### 参数解析
![](/images/6a7a1d9f0a0ec75781782be154a8d827.png)

自动删除的场景：有队列和交换器绑定和当前交换器绑定，绑定的队列和交换器与当前交换器都解绑了，才删除

##### exchangeDeclarePassive
它主要用来检测相应的交换器是否存在。如果存在则正常返回；如果不存在则抛出异常：404 channel exception，同时Channel也会被关闭。









#### 队列声明
##### queueDeclare
##### 参数解析
![](/images/ab749f933e5afa7bf897ebc841671602.png)





##### queueDeclarePassive
#### 交换器与队列绑定
##### queueBind


#### 交换器绑定
![](/images/2af32fe016e3e946179cab9dd1dcf6ad.png)

##### exchangeBind


#### 创建队列
1. 预先创建 ：如果充分了解队列的使用情况，可预先创建
2. 动态创建： 由业务程序中的生产者和消费者创建





## 发送消息
##### basicPublish(String exchange, String routingKey, boolean mandatory, boolean immediate, BasicProperties props, byte[] body)
##### 参数解析
> exchange：交换器的名称，指明消息需要发送到哪个交换器中。如果设置为空字符串，则消息会被发送到RabbitMQ默认的交换器中。
>
> routingKey：路由键，交换器根据路由键将消息存储到相应的队列之中。
>
> props：消息的基本属性集，其包含14个属性成员，分别有contentType、contentEncoding、headers(Map<String,Object>)、deliveryMode、priority、correlationId、replyTo、expiration、messageId、timestamp、type、userId、appId、clusterId。其中常用的几种都在上面的示例中进行了演示。
>
> byte[] body：消息体（payload），真正需要发送的消息
>
> mandatory：没有绑定队列时
>
> 和immediate的详细内容请参考4.1节。
>







## 消费消息
##### 推模式
###### basicConsume
> 通过实现Consumer接口或继承DefaultConsumer类来接收消息。使用消费者标签（consumerTag）区分同一Channel中的消费者。
>

###### 参数解析
> queue：队列的名称；
>
> autoAck：设置是否自动确认。建议设成false，即不自动确认；
>
> consumerTag：消费者标签，用来区分多个消费者；
>
> noLocal：设置为true则表示不能将同一个Connection中生产者发送的消息传送给这个Connection中的消费者；
>
> exclusive：设置是否排他；
>
> arguments：设置消费者的其他参数；
>
> callback：设置消费者的回调函数。用来处理RabbitMQ推送过来的消息，比如DefaultConsumer，使用时需要客户端重写（override）其中的方法
>



##### 拉模式
###### basicGet
```java
GetResponse response = channel.basicGet(QUEUE_NAME, false); 
System.out.println(new String(response.getBody())); 
channel.basicAck(response.getEnvelope().getDeliveryTag(),false);
```





## 消费确认和拒绝
##### 确认： 确保消息完全被消费者消费
#### basicAck
##### 拒绝：
###### basicReject (long deliveryTag, boolean requeue)：拒绝单条消息
###### 参数解析
+ `deliveryTag`<font style="color:rgb(6, 6, 7);"> </font><font style="color:rgb(6, 6, 7);">是消息的唯一编号，最大值为9223372036854775807。</font>
+ `requeue`<font style="color:rgb(6, 6, 7);"> </font><font style="color:rgb(6, 6, 7);">参数：</font>
    - `true`<font style="color:rgb(6, 6, 7);">：消息重新入队，可被其他消费者接收。</font>
    - `false`<font style="color:rgb(6, 6, 7);">：启用死信队列，检测被拒绝或者未送达的消息来追踪问题</font>

###### basicNack（long deliveryTag, boolean multiple, boolean requeue)：拒绝多条消息


###### 参数解析
+ `deliveryTag`<font style="color:rgb(6, 6, 7);"> </font><font style="color:rgb(6, 6, 7);">是消息的唯一编号，最大值为9223372036854775807。</font>
+ `requeue`<font style="color:rgb(6, 6, 7);"> </font><font style="color:rgb(6, 6, 7);">参数：</font>
    - `true`<font style="color:rgb(6, 6, 7);">：消息重新入队，可被其他消费者接收。</font>
    - `false`<font style="color:rgb(6, 6, 7);">：启用死信队列，检测被拒绝或者未送达的消息来追踪问题</font>
+ `multiple`<font style="color:rgb(6, 6, 7);"> </font><font style="color:rgb(6, 6, 7);">参数决定消息拒绝的范围：</font>
    - `false`<font style="color:rgb(6, 6, 7);">：仅拒绝当前消息（</font>`deliveryTag`<font style="color:rgb(6, 6, 7);">）。</font>
    - `true`<font style="color:rgb(6, 6, 7);">：拒绝当前消息及之前所有未确认的消息。</font>
+ <font style="color:rgb(6, 6, 7);">当 </font>`multiple`<font style="color:rgb(6, 6, 7);"> 为 </font>`false`<font style="color:rgb(6, 6, 7);"> 时，</font>`basicNack`<font style="color:rgb(6, 6, 7);"> 和 </font>`basicReject`<font style="color:rgb(6, 6, 7);"> 的效果相同。</font>









###### basicRecover
`requeue`<font style="color:rgb(6, 6, 7);"> 参数控制未确认消息的处理方式。</font>

<font style="color:rgb(6, 6, 7);">设置为 </font>`true`<font style="color:rgb(6, 6, 7);">：消息会重新入队，可能被其他消费者处理。</font>

<font style="color:rgb(6, 6, 7);">设置为 </font>`false`<font style="color:rgb(6, 6, 7);">：消息会重新分配给原来的消费者。</font>

<font style="color:rgb(6, 6, 7);">默认情况下，</font>`requeue`<font style="color:rgb(6, 6, 7);"> 相当于设置为 </font>`true`<font style="color:rgb(6, 6, 7);">，即消息会重新入队</font>

<font style="color:rgb(6, 6, 7);"></font>

<font style="color:rgb(6, 6, 7);"></font>

## <font style="color:rgb(6, 6, 7);">关闭连接</font>
关闭对象： Connnection，channel

生命周期

1. open：开启状态，对象可用
2. Closing：正在关闭
3. closed：已经关闭，对象内所有的对象已关闭



#### addShutdownListener：添加关闭监听器，对象正在关闭或已经关闭时会调用关闭监听器
#### removeShutdownListener：
#### getCloseReason：知道对象关闭原因










# Rabbitmq 进阶


## 生产者消息防丢失
### mandatory 参数
+ <font style="color:rgb(6, 6, 7);">如果设置为</font>`mandatory`<font style="color:rgb(6, 6, 7);">为</font>`true`<font style="color:rgb(6, 6, 7);">，且没有队列匹配，消息会返回给生产者。</font>
+ <font style="color:rgb(6, 6, 7);">如果设置为</font>`mandatory`<font style="color:rgb(6, 6, 7);">为</font>`false`<font style="color:rgb(6, 6, 7);">，消息则会被直接丢弃，生产者不会收到任何通知。</font>

#### 如何获取通知
channel.addReturnListener来添加ReturnListener监听器实现

```java
channel.basicPublish(EXCHANGE_NAME, "", true,MessageProperties.PERSISTENT_TEXT_PLAIN,"mandatory test".getBytes());  
channel.addReturnListener(new ReturnListener() {
    public void handleReturn(int replyCode, String replyText,String exchange, String routingKey,AMQP.BasicProperties basicProperties,byte[] body) throws IOException {         
        String message = new String(body);        
        System.out.println("Basic.Return返回的结果是："+message);     
    } });
```



### 备份交换器（第二种消息防丢失方案）
![](/images/cae1350e1117824cee1b2a84ffaa2105.png)

```java

Map<String, Object> args = new HashMap<String, Object>(); 
args.put("alternate-exchange", "myAe"); 
channel.exchangeDeclare("normalExchange", "direct", true, false, args);
channel.exchangeDeclare("myAe", "fanout", true, false, null); 
channel.queueDeclare("normalQueue", true, false, false, null); 
channel.queueBind("normalQueue", "normalExchange", "normalKey"); 
channel.queueDeclare("unroutedQueue", true, false, false, null
```

### 备份交换器的特殊情况
1. <font style="color:rgb(6, 6, 7);">如果备份交换器不存在，消息会丢失，但客户端和服务端不会有异常。</font>
2. <font style="color:rgb(6, 6, 7);">如果备份交换器没有绑定队列，消息同样会丢失，客户端和服务端不会有异常。</font>
3. <font style="color:rgb(6, 6, 7);">如果备份交换器没有匹配的队列，消息也会丢失，客户端和服务端不会有异常。</font>
4. <font style="color:rgb(6, 6, 7);">如果同时使用备份交换器和</font>`mandatory`<font style="color:rgb(6, 6, 7);">参数，</font>`mandatory`<font style="color:rgb(6, 6, 7);">参数会失效。</font>

<font style="color:rgb(6, 6, 7);"></font>

## 过期时间 TTL
### 消息过期时间
#### 两种设置方式
1. 队列属性设置消息过期时间

```java
Map<String, Object>  argss = new HashMap<String, Object>(); 
argss.put("x-message-ttl",6000); 
channel.queueDeclare(queueName, durable, exclusive, autoDelete, args);
```

2. 每条消息单独设置

```java
AMQP.BasicProperties.Builder builder = new AMQP.BasicProperties.Builder(); 
builder.deliveryMode(2);//持久化消息
builder.expiration("60000");
//设置TTL=60000ms 
AMQP.BasicProperties properties = builder.build(); 
channel.basicPublish(exchangeName,routingKey,mandatory,properties,"ttlTestMessage".getBytes())
```

**<font style="color:#DF2A3F;">同时设置取时间最小的 </font>**

超过 TTL 消息变成**<font style="background-color:#FBDE28;">死信</font>**

**<font style="background-color:#FBDE28;"></font>**

**<font style="background-color:#FBDE28;"></font>**





### 队列过期时间
<font style="color:rgb(6, 6, 7);">用 </font>`channel.queueDeclare`<font style="color:rgb(6, 6, 7);"> 方法时，可以通过设置 </font>`x-expires`<font style="color:rgb(6, 6, 7);"> 参数来指定队列在多久未使用后自动删除。这里的“未使用”指的是：</font>

1. <font style="color:rgb(6, 6, 7);">队列上没有消费者。</font>
2. <font style="color:rgb(6, 6, 7);">队列没有被重新声明。</font>
3. <font style="color:rgb(6, 6, 7);">在设定的时间内没有执行过 </font>`Basic.Get`<font style="color:rgb(6, 6, 7);"> 命令。</font>

```java
Map<String, Object> args = new HashMap<String, Object>();  
args.put("x-expires", 1800000);
channel.queueDeclare("myqueue", false, false, false, args);
```

> 设置队列里的TTL可以应用于类似RPC方式的回复队列，在RPC中，许多队列会被创建出来，但是却是未被使用的。
>
> RabbitMQ会确保在过期时间到达后将队列删除，但是不保障删除的动作有多及时。在RabbitMQ重启后，持久化的队列的过期时间会被重新计算。
>



## 死信交换器、死信队列、 DLX
死信的几种情况

1. 消息被拒绝，且 `requeue` 为 `false`
2. 消息过期
3. 队列达到最大长度

```java
channel.exchangeDeclare("exchange.dlx", "direct", true); 
channel.exchangeDeclare("exchange.normal", "fanout", true);
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-message-ttl", 10000); 
args.put("x-dead-letter-exchange", "exchange.dlx");
args.put("x-dead-letter-routing-key", "routingkey"); 
channel.queueDeclare("queue.normal", true, false, false, args); 
channel.queueBind("queue.normal", "exchange.normal", "");
channel.queueDeclare("queue.dlx", true, false, false, null);
channel.queueBind("queue.dlx", "exchange.dlx", "routingkey");
channel.basicPublish("exchange.normal", "rk",MessageProperties.PERSISTENT_TEXT_PLAIN, "dlx".getBytes());
```

![](/images/d06f84c51e92fd3f2763c805df740e48.png)







## 延迟队列


存储对象：延迟消息，不想立即拿到，等待特定时间，消费者才能拿到消息消费

使用场景

1. 订单系统 30 分钟的支付等待时间
2. 手机远程遥控智能设备定时执行

实现方式：TTL+死信队列

![](/images/895340edf2d5debef025bcadb6a3d162.png)





## 优先级队列
优先级设置

```java
//队列优先级设置
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-max-priority", 10); 
channel.queueDeclare("queue.priority", true, false, false, args);



//消息优先级设置 ，最大为队列优先级，最小为0
AMQP.BasicProperties.Builder builder = new AMQP.BasicProperties.Builder();
builder.priority(5); 
AMQP.BasicProperties properties = builder.build(); 
channel.basicPublish("exchange_priority","rk_priority",properties,("messages").getBytes());
```

#### 前提： 队列中存在多条信息








## RPC 实现
![](/images/4f1d62644160948171bac83e5d03aee5.png)









## 持久化
### 交换器持久化 ：重启导致交换器配置没了，但是消息不影响
### 队列持久化： 重启队列没了，队列里的消息也没了
### 消息持久化：能保证重启后消息还存在，前提是队列也存在




### 重点
1. 消费者开启了自动确认，自动确认后服务宕机
2. 消息持久化写入磁盘时宕机，可以采用镜像队列提高可靠性





## 生产者确认：生产者无法知道消息正确给交换器
### 事务机制  消耗性能大
![](/images/d3239713bf959f5eb038002c93cd34d5.png)

![](/images/a3bb46db9131046aa4f752c206584006.png)

```java
channel.txSelect();    
for(int i=0;i<LOOP_TIMES;i++) {       
    try {            
        channel.basicPublish("exchange", "routingKey", null, ("messages" + i).getBytes()); 
        channel.txCommit();
    } catch (IOException e) {             
        e.printStackTrace();             
        channel.txRollback();         
    } }
```



### 发送方确认
![](/images/756216e4c91d5c343b01d547b72e25ff.png)

```java
try {
    channel.confirmSelect(); // 将信道置为publisher confirm模式
    // 之后正常发送消息
    channel.basicPublish("exchange", "routingKey", null, "publisher confirm test".getBytes());
    //等待消息确认
    if (!channel.waitForConfirms()) {
        System.out.println("send message failed");
        // do something else....
    }
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

```java
channel.confirmSelect();
channel.addConfirmListener(new ConfirmListener() {
    public void handleAck(long deliveryTag, boolean multiple) throws IOException {
        System.out.println("Ack, SeqNo: " + deliveryTag + ", multiple: " + multiple);
        if (multiple) {
            confirmSet.headSet(deliveryTag - 1).clear();
        } else {
            confirmSet.remove(deliveryTag);
        }
    }

    public void handleNack(long deliveryTag, boolean multiple) throws IOException {
        System.out.println("Nack, SeqNo: " + deliveryTag + ", multiple: " + multiple);
        if (multiple) {
            confirmSet.headSet(deliveryTag - 1).clear();
        } else {
            confirmSet.remove(deliveryTag);
        }
        // 注意这里需要添加处理消息重发的场景
    }
});

// 下面是演示一直发送消息的场景
while (true) {
    long nextSeqNo = channel.getNextPublishSeqNo();
    channel.basicPublish(ConfirmConfig.exchangeName, ConfirmConfig.routingKey,
                         MessageProperties.PERSISTENT_TEXT_PLAIN,
                         ConfirmConfig.msg_10B.getBytes());
    confirmSet.add(nextSeqNo);
}
```

### 二者不能同时使用
## 消费端要点




### 消息分发




![](/images/3cfdabab36e94e38b7567c13dcf6cc9d.png)

```java
Channel channel = ...; 
Consumer consumer1 = ...; 
Consumer consumer2 = ...; 
channel.basicQos(3, false); // Per consumer limit 
channel.basicQos(5, true); // Per channel limit
channel.basicConsume("queue1", false, consumer1); 
channel.basicConsume("queue2", false, consumer2);
```





### 消息顺序性
### 消息存在乱序情况
1. 使用事务机制的回滚，导致补偿发送
2. 发送确认机制的补偿发送
3. 不同超时时间消息的延迟队列
4. 设置消息优先级

### 解决方法 ： 业务方在消息体中添加全局有序 ID




