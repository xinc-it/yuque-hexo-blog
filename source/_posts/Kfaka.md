---
title: Kfaka
date: '2025/4/11 21:17:06'
updated: '2025-07-06 11:11:07'
categories: 消息队列
tags:
  - 消息队列
  - kafka
---
# 认识 kafka
![](/images/abf0d7e3b275412c1635d40c286f960b.png)



## 基本概念
1. 生产者： 发送消息
2. 消费者：接收消息
3. broker：服务器
4. topic 主题： 消息的分类，消费者通过主题来订阅消息
5. partition 分区： 主题的子集，用来存储消息



## 多副本机制
![](/images/d9573524501c60e050e6a9a15e518eda.png)

目的：提升容灾能力

副本关系：一主多从

leader 副本： 负责独写消息

follwer 副本：同步 leader 中的消息

AR：所有副本  
ISR：与 leader 副本保持一定同步的副本（包括 leader 副本）

OSR ： 与 leader 副本之后过多的副本

LEO：<font style="color:rgb(6, 6, 7);">分区中下一条消息的位置</font>

HW：消费者中可以消费的消息位置最大值

![](/images/a581a7b16a7f4f33a8a47776c4118d63.png)

## 服务端参数


1. `zookeeper.connect` zookeeper 连接地址
2. `listeners` 客户端 broker 的连接地址信息 ，格式 :`protocol1://hostname1:port1,` 支持的协议：PLAINTEXT，SSL，SASL_SSL
3. `broker.id` kafka 服务器 ID
4.  `**log.dir**`<font style="color:rgb(6, 6, 7);">：用于指定单个日志文件存储的目录路径。</font>
5. `**log.dirs**`<font style="color:rgb(6, 6, 7);">：用于指定多个日志文件存储的目录路径，路径之间用逗号分隔，用于分散存储日志以提高性能和数据安全性。</font>
6. `<font style="color:rgb(6, 6, 7);">message.max.bytes</font>`<font style="color:rgb(6, 6, 7);"> 单个消息最大值</font>

<font style="color:rgb(6, 6, 7);"></font>

<font style="color:rgb(6, 6, 7);"></font>

<font style="color:rgb(6, 6, 7);"></font>

# <font style="color:rgb(6, 6, 7);">生产者</font>




## 客户端开发
### 必要参数配置
1. `bootstrap.servers` 指定 kafka 服务器地址格式为：`host1:port1,host2:port2`
2. `key.serializer`和 `value.serializer` 指定消息序列化器
3. 其他参数可参考`ProducerConfig`类

### 发送消息


```java
public class ProducerRecord<K, V> { 
    private final String topic; //主题
    private final Integer partition;//分区号
    private final Headers headers; //消息头部
    private final K key; //键
    private final V value; //值
    private final Long timestamp; //消息的时间戳
}
```





发送消息的三种模式

1. 同步
2. 异步
3. 发后即忘

#### 发后即忘（性能高，可靠性差）
**只发送消息，不管消息是否正确到达**

```java
try {
    //发送消息
    producer.send(record);
} catch (Exception e) {
    e.printStackTrace();
}
```

#### 同步
```java
try { 
    producer.send(record) .get(); 
} catch (ExecutionException I InterruptedException e) {
    e .printStackTrace()
}



try { 
    Future<RecordMetadata> future= producer.send(record); 
    //RecordMetadata对象里包含了消息的一些元数据信息，比如当前消息的主题、分区号、分区中的偏移量（offset〕、时间戳
    RecordMetadata metadata =future.get(); 
    System.out.println(metadata.topic() +” - " + metadata.partition() +”:”+ metadata.offset());
    } catch (ExecutionException || InterruptedException e){
    e.printStackTrace();
}
```



#### 异步


```java
producer.send(record, new Callback() {
    @Override
    public void onCompletion(RecordMetadata metadata, Exception exception) {
        if (exception != null) {
            exception.printStackTrace();
        } else {
            System.out.println(metadata.topic() + "-" + metadata.partition() + ":" + metadata.offset());
        }
    }
});
```



#### 可重试异常和不可重试异常
可重试：NetworkException、LeaderNotAvailableException、UnknownTopicOrPartitionException、NotEnoughReplicasException、NotCoordinatorExceptin

不可重试：RecordTooLargeException（消息太大）

可重试次数配置

```java
props.put(ProducerConfig. RETRIES_CONFIG, 10);
```







### 序列化(将消息转成字节)
可以自定义序列化器，

```java
public class CompanySerializer implements Serializer<Company> {
    @Override
    public void configure(Map configs, boolean isKey) {
        // 配置方法的实现，如果有需要的话
    }

    @Override
    public byte[] serialize(String topic, Company data) {
        if (data == null) {
            return null;
        }
        byte[] name, address;
        try {
            if (data.getName() != null) {
                name = data.getName().getBytes("UTF-8");
            } else {
                name = new byte[0];
            }
            if (data.getAddress() != null) {
                address = data.getAddress().getBytes("UTF-8");
            } else {
                address = new byte[0];
            }
            ByteBuffer buffer = ByteBuffer.allocate(4 + 4 + name.length + address.length);
            buffer.putInt(name.length);
            buffer.put(name);
            buffer.putInt(address.length);
            buffer.put(address);
            return buffer.array();
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
            return new byte[0];
        }
    }

    @Override
    public void close() {
        // 关闭资源的实现，如果有需要的话
    }
}
```





### 分区器（消息分配分区）
#### 默认分区器 `DefaultPartitioner`
1. **<font style="color:rgb(6, 6, 7);">Key非空时</font>**<font style="color:rgb(6, 6, 7);">：使用MurmurHash2算法对Key进行哈希，根据哈希值分区</font>
2. **<font style="color:rgb(6, 6, 7);">Key为空时</font>**<font style="color:rgb(6, 6, 7);">：消息通过轮询方式分配到可用的分区，不保证消息顺序</font>

> + <font style="color:rgb(6, 6, 7);">当消息的Key不为null时，系统会计算出一个分区号，这个分区号可以是</font>**<font style="color:#DF2A3F;background-color:#FBDE28;">所有分区中</font>**<font style="color:rgb(6, 6, 7);">的任意一个。</font>
> + <font style="color:rgb(6, 6, 7);">当Key为null时，系统计算出的分区号仅限于</font>**<font style="color:#DF2A3F;background-color:#FBDE28;">可用分区</font>**<font style="color:rgb(6, 6, 7);">中的任意一个。</font>
>

#### 自定义分区器 `Partitioner`








### 拦截器
作用： 

1. 过滤不符合要求的消息
2. 修改消息内容
3. 回调逻辑前定制化需求（统计消息）

接口：`ProducerInterceptor`

`OnSend` 在序列化和分区器之前执行

`onAcknowledgement` 异步回调函数之前执行









## 原理分析
![](/images/0f3a4396e97c0c4d2343afe6991a4dd1.png)

### 整体结构 
1. **<font style="color:rgb(6, 6, 7);">消息创建</font>**<font style="color:rgb(6, 6, 7);">：</font>
    - <font style="color:rgb(6, 6, 7);">在主线程中，KafkaProducer创建消息。</font>
2. **<font style="color:rgb(6, 6, 7);">消息处理</font>**<font style="color:rgb(6, 6, 7);">：</font>
    - <font style="color:rgb(6, 6, 7);">创建的消息通过拦截器、序列化器和分区器进行处理。</font>
3. **<font style="color:rgb(6, 6, 7);">消息缓存</font>**<font style="color:rgb(6, 6, 7);">：</font>
    - <font style="color:rgb(6, 6, 7);">处理后的消息被缓存到RecordAccumulator（消息累加器或消息收集器）中。</font>
    - <font style="color:rgb(6, 6, 7);">RecordAccumulator中的缓存大小可以通过</font>`buffer.memory`<font style="color:rgb(6, 6, 7);">参数配置，默认为32MB。</font>
4. **<font style="color:rgb(6, 6, 7);">消息批次管理</font>**<font style="color:rgb(6, 6, 7);">：</font>
    - <font style="color:rgb(6, 6, 7);">消息被追加到RecordAccumulator的双端队列（Deque）中，每个分区都有一个双端队列。</font>
    - <font style="color:rgb(6, 6, 7);">双端队列中的内容是ProducerBatch的大小可以通过</font>`batch.size`<font style="color:rgb(6, 6, 7);">参数配置，默认为 16KB，即Deque<ProducerBatch>。</font>
5. **<font style="color:rgb(6, 6, 7);">内存管理</font>**<font style="color:rgb(6, 6, 7);">：</font>
    - <font style="color:rgb(6, 6, 7);">使用java.io.ByteBuffer和BufferPool管理消息内存，提高缓存的高效利用。</font>
6. **<font style="color:rgb(6, 6, 7);">消息发送</font>**<font style="color:rgb(6, 6, 7);">：</font>
    - <font style="color:rgb(6, 6, 7);">Sender线程从RecordAccumulator中获取消息，并将其转换为<Node, List<ProducerBatch>>的形式。</font>
    - <font style="color:rgb(6, 6, 7);">进一步封装成<Node, Request>的形式，准备发送到Kafka集群。</font>
7. **<font style="color:rgb(6, 6, 7);">请求管理</font>**<font style="color:rgb(6, 6, 7);">：</font>
    - <font style="color:rgb(6, 6, 7);">发送的请求被缓存在InFlightRequests中，直到收到响应。</font>
    - <font style="color:rgb(6, 6, 7);">InFlightRequests限制每个连接最多缓存的请求数，默认为5。</font>
8. **<font style="color:rgb(6, 6, 7);">异常处理</font>**<font style="color:rgb(6, 6, 7);">：</font>
    - <font style="color:rgb(6, 6, 7);">如果生产者发送消息的速度超过发送到服务器的速度，send()方法调用会被阻塞或抛出异常，取决于</font>`max.block.ms`<font style="color:rgb(6, 6, 7);">参数的配置，默认为60秒。</font>
9. **<font style="color:rgb(6, 6, 7);">参数调整</font>**<font style="color:rgb(6, 6, 7);">：</font>
    - <font style="color:rgb(6, 6, 7);">可以通过调整</font>`batch.size`<font style="color:rgb(6, 6, 7);">和</font>`buffer.memory`<font style="color:rgb(6, 6, 7);">参数来优化性能和吞吐量。</font>
10. **<font style="color:rgb(6, 6, 7);">网络I/O与应用逻辑转换</font>**<font style="color:rgb(6, 6, 7);">：</font>
    - <font style="color:rgb(6, 6, 7);">生产者客户端与具体的broker节点建立连接，发送消息到broker节点。</font>
    - <font style="color:rgb(6, 6, 7);">需要将应用逻辑层面的消息分区映射到网络I/O层面的broker节点。</font>

<font style="color:rgb(6, 6, 7);"></font>

<font style="color:rgb(6, 6, 7);"></font>

<font style="color:rgb(6, 6, 7);"></font>

### <font style="color:rgb(6, 6, 7);">元数据更新</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">InFlightRequests可以获得leastLoadedNode，即所有Node中负载最小的。leastLoadedNode一般用于元数据请求、消费者组播协议等交互。</font>
+ <font style="color:rgba(0, 0, 0, 0.75);">当客户端中没有需要使用的元数据信息或超过</font>`<font style="color:rgb(199, 37, 78);background-color:rgb(249, 242, 244);">metadata.max.age.ms</font>`<font style="color:rgba(0, 0, 0, 0.75);">没有更新元数据时，就会引起元数据更新操作。</font>

<font style="color:rgba(0, 0, 0, 0.75);"></font>

### <font style="color:rgba(0, 0, 0, 0.75);">重要生产参数</font>
#### acks （确保消息发送到服务端是否丢失）
ack=1 leader 副本成功写入，收到服务端成功响应。如果成功发送服务端但是 leader 崩溃依然会丢失消息

ack=0 不等待服务端成功响应

ack=-1 或 all 等待所有 ISR 副本写入成功，收到服务端成功响应



#### max.request.size (生产者消息大小）
默认为 1mb，不建议改动，会影响到其他参数





#### retries 和 retry.backoff.ms
retries :生产者重试次数 默认未为 0

retrit.backoff.ms ：重试之间的时间间隔，默认 100ms





#### compression.type
指定消息压缩方式，默认值 none 不压缩

可以配置 gzip、snappy、lz4

好处： 减少网络传输量、降低网络 IO，提高整体性能

坏处：时延会加大





#### connections.max.idle.ms
关闭闲置连接的等待时间 540000 ms，





#### linger.ms
等待消息加入 ProducerBatch 的等待时间，默认是 0







#### receive.buffer.bytes
Socket 接收消息的缓冲区大小默认 32kb,

-1 使用操作系统默认值





#### send.buffer.bytes
Socket 发送消息的缓冲区大小默认 128kb,

-1 使用操作系统默认值





#### request.timeout.ms
等待响应最长时间 `30000ms`











# 消费者


## 消费者和消费者组
消费者组：用来订阅主题，<font style="color:rgb(6, 6, 7);">允许多个消费者实例共同消费一个主题的消息，而每个消费者实例消费该主题下的不同分区。</font>

<font style="color:rgb(6, 6, 7);">kafka 通过消费者组实现广播和点对点的通信方式</font>

1. 点对点： 多个消费者分配到同一个消费者组中， 每个消费者消耗指定的分区
2. 广播： 每个消费者放入一个单独的消费者，多个消费者组订阅同一个主题







## 客户端开发
消费步骤：

1. 配置客户端参数和创建对应实例
2. 订阅主题
3. 拉取消息并消费
4. 提交消费偏移
5. 关闭消费者实例

### 必要参数配置
+ `<font style="color:rgb(199, 37, 78);background-color:rgb(249, 242, 244);">bootstrap.servers</font>`<font style="color:rgba(0, 0, 0, 0.75);">：连接broker地址清单，</font>
+ `<font style="color:rgb(199, 37, 78);background-color:rgb(249, 242, 244);">group.id</font>`<font style="color:rgba(0, 0, 0, 0.75);">：消费组名称</font>
+ `<font style="color:rgb(199, 37, 78);background-color:rgb(249, 242, 244);">key.deserializer、</font>`<font style="color:rgba(0, 0, 0, 0.75);">value.deserializer`：反序列化器</font>

<font style="color:rgba(0, 0, 0, 0.75);"></font>

### <font style="color:rgba(0, 0, 0, 0.75);">订阅主题和分区</font>
#### 订阅主题
<font style="color:rgba(0, 0, 0, 0.75);">subscribe ： 订阅主题</font>

<font style="color:rgba(0, 0, 0, 0.75);">assign()：订阅指定主题分区</font>

<font style="color:rgba(0, 0, 0, 0.75);">partitionFor（）获取分区信息</font>

<font style="color:rgba(0, 0, 0, 0.75);">subscribe 和 assign 区别</font>

1. subscribe 具有消费者负载均衡的功能， 而 assign 不具备

#### <font style="color:rgba(0, 0, 0, 0.75);">取消订阅</font>
<font style="color:rgba(0, 0, 0, 0.75);">unsubscribe():取消订阅</font>

<font style="color:rgba(0, 0, 0, 0.75);">subscribe(new ArrayList<String>()); 取消订阅</font>

<font style="color:rgba(0, 0, 0, 0.75);">assign(newArrayL工st<TopicPartition>()) 取消订阅</font>

<font style="color:rgba(0, 0, 0, 0.75);"></font>

<font style="color:rgba(0, 0, 0, 0.75);"></font>

### 反序列化 `Deserializer`
不建议使用自定义的序列化和反序列化，会增加生产者和消费者的耦合度。







### 消息消费


public ConsumerRecords<K, V> poll(final Duration timeout)

> <font style="color:rgba(0, 0, 0, 0.75);">返回的是所订阅的主题(分区)上的一组消息，可设定timeout参数来控制阻塞时间</font>
>



public List<ConsumerRecord<K, V>records(TopicPartitionpartition)

> ConsumerRecords 消费指定分区的消息
>

ConsumerRecords.partitions（）

> 获取消息所在的分区信息
>



### 位移提交


消费位移： 消费者已经消费的消息在主题中的偏移量  

提交位移：下一次消费的偏移量



public OffsetAndMetadata comrnitted(TopicPartition partition)  获取提交位移

public long position(TopicPartition partition) 获取下一次偏移量



消费者自动提交

1. 消费者通过`enable.auto.commit`和`auto.commit.interval.ms` 参数值进行自动提交设定默认是 每 5 秒提交一次
2. 自动提交将消费位移提交给 kafka 中`_cousumer_offset` 主题中，方便下次消费时知道消费的位置



#### 消费者自动位移提交问题
1. 消息重复消费： 消息消费了一部分，但是出现异常导致，自动提交还没提交。下次消费时还是从上次的消费位移拉取的消
2. 消息丢失： 自动提交已经提交了，消息消费了一部分，但是出现了异常，导致消息没有消费完，下次拉取的消息从提交的位消费位移开始消费。

![](/images/dcef2581bd23d5b99bea8a3c3159b10f.png)





#### 手动同步提交 阻塞消费者线程
commitSync（） 自动提交 poll 拉取的消费位移



commitSync(final Map<TopicPartition,OffsetAndMetadata> offsets) 指定提交偏移量 



```java
//按分区粒度同步提交位移
for (TopicPartition partition : records.partitions()) {
    List<ConsumerRecord<String, String>> recordsPartition = records.records(partition);
    for (ConsumerRecord<String, String> record : recordsPartition) {
        System.out.printf("分区%d消费消息:%s\n",record.partition(),record.value());
    }
    //最后一次消费的偏移量
    long lastConsumedOffset = recordsPartition.get(recordsPartition.size() - 1).offset();
    System.out.printf("分区%d-lastConsumedOffset:%s\n",partition.partition(),lastConsumedOffset);
    consumer.commitSync(Collections.singletonMap(partition,new OffsetAndMetadata(lastConsumedOffset+1)));
}
```



**<font style="color:#DF2A3F;background-color:#FBDE28;">同步提交会一直提交，直到提交成功，除非出现不可恢复的错误</font>**<font style="color:#DF2A3F;background-color:#FBDE28;">，</font>

比如CommitFailedException、WakeupException、InterruptException、AuthenticationException、AuthorizationException等，我们可以将其捕获并做针对性的处理。

#### 手动异步提交 不会阻塞消费者线程
1. `commitAsync()`<font style="color:rgb(6, 6, 7);">：异步提交当前消费者的偏移量。不提供回调函数，因此无法知道提交操作是否成功。</font>
2. `commitAsync(OffsetCommitCallback callback)`<font style="color:rgb(6, 6, 7);">：异步提交当前消费者的偏移量，并提供一个回调函数</font>`OffsetCommitCallback`<font style="color:rgb(6, 6, 7);">，用于在提交操作完成时被调用，无论成功或失败。</font>
3. `commitAsync(final Map<TopicPartition, OffsetAndMetadata> offsets, OffsetCommitCallback callback)`<font style="color:rgb(6, 6, 7);">：异步提交指定的偏移量映射</font>`Map<TopicPartition, OffsetAndMetadata>`<font style="color:rgb(6, 6, 7);">，并提供一个回调函数</font>`OffsetCommitCallback`<font style="color:rgb(6, 6, 7);">，用于在提交操作完成时被调用，无论成功或失败。这允许更细粒度的控制，可以为不同的分区提交不同的偏移量。</font>

<font style="color:rgb(6, 6, 7);"></font>

<font style="color:rgb(6, 6, 7);"></font>

<font style="color:rgb(6, 6, 7);"></font>

#### <font style="color:rgb(6, 6, 7);">异步提交重复消费问题</font>
可以用异步+同步消费来解决异步提交失败的问题









### 控制或关闭消费
paused 暂停消费

pause 暂停指定分区消费

resume  恢复已暂停的分区







### 指定位移消费
`auto.offset.reset` ：消费者找不到消费位移时，通过 auto.offset.reset 的记录值读取

1. none 消费者找不到位移时，抛出`NoOffsetForPartitionException`
2. earliest 从起始处消费
3. latest 从即将要写入的一条消息消费

![](/images/4cb40f9d71b8988ffb4d5b725e41e505.png)

#### 指定分区的偏移量消费
`public void seek(TopicPartition partition,long offset)`

```java
        // 创建 KafkaConsumer 实例
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        // 订阅主题
        consumer.subscribe(Arrays.asList(topic));

        // 首次轮询，等待分区分配
        consumer.poll(Duration.ofMillis(10000)); // ①

        // 获取当前分配的分区
        Set<TopicPartition> assignment = consumer.assignment(); // ②

        // 为每个分区设置偏移量
        for (TopicPartition tp : assignment) {
            consumer.seek(tp, 10); // ③
        }

        // 循环消费消息
        while (true) {
            // 轮询获取消息
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
            // 消费记录
            // consume the record.
        }

```

如果 poll 设置为 0 时，消费者无法进行指定偏移量消费

> <font style="color:rgb(6, 6, 7);">在 Kafka 中，消费者通过一个叫做消费者组（Consumer Group）的机制来协调多个消费者实例之间的消息消费。当消费者加入到一个消费者组时，Kafka 会负责将主题的分区分配给组内的消费者。这个过程通常发生在消费者第一次调用 </font>`poll`<font style="color:rgb(6, 6, 7);"> 方法时。</font>
>
> <font style="color:rgb(6, 6, 7);">如果 </font>`poll`<font style="color:rgb(6, 6, 7);"> 方法的参数设置为 </font>`0`<font style="color:rgb(6, 6, 7);">，那么 </font>`poll`<font style="color:rgb(6, 6, 7);"> 方法会立即返回，不会给 Kafka 服务器足够的时间来完成分区分配的逻辑。因为分区分配是一个异步过程，它需要一些时间来完成。如果 </font>`poll`<font style="color:rgb(6, 6, 7);"> 方法立即返回，那么分区分配可能还没有开始或者完成，导致消费者没有被分配任何分区。</font>
>



#### 从分区末尾开始消费消息
1. 获取消息末尾的位置

`public Map<TopicPartition,Long>endoffsets(Collection<TopicPartition>partitions)`

`public Map<TopicPartition,Long>endoffsets(Collection<TopicPartition>partitions,Duration timeout)`

2. 从分区末尾消费消息

`public void seekToEnd(Collection<TopicPartition>partitions)`





#### 从分区开头消费消息
1. 获取消息开始的位置

`public Map<TopicPartition,Long>beginningoffsets( Collection<TopicPartition>partitions) `

`public Map<TopicPartition,Long>beginningoffsets Collection<TopicPartition>partitions, Duration timeout)`

2. 从分区开头消费消息

`public void seekToBeginning(Collection<TopicPartition>partitions)`





#### 指定时间开始消费
`public Map<TopicPartition,offsetAndTimestamp>offsetsForTimes Map<TopicPartition,Long>timestampsToSearch) `

`public Map<TopicPartition,offsetAndTimestamp>offsetsForTimes Map<TopicPartition,Long>timestampsToSearch, Duration timeout)`





#### 消费位移存储到 DB 中
```java
consumer.subscribe (Arrays.asList (topic));
//省略pol1()方法及assignment的逻辑
for(TopicPartition tp:assignment){
    long offset=getoffsetFromDB(tp);//从DB中读取消费位移
    consumer.seek(tp,offset);
}
while(true){
    ConsumerRecords<String,String>records
    consumer.poll(Duration.ofMillis(1000));
    for (TopicPartition partition records.partitions ())
    List<ConsumerRecord<String,String>>partitionRecords
    records.records (partition);
    for (ConsumerRecord<String,String>record partitionRecords){
        //process the record.
        long lastConsumedoffset partitionRecords
        get(partitionRecords.size()-1).offset ()
        //将消费位移存储在DB中
        storeoffsetToDB(partition,lastConsumedoffset+1);
    }
}
```





### 再均衡 `ConsumerRebalanceListener`


```java
  consumer.subscribe(Collections.singleton(TOPIC), new ConsumerRebalanceListener() {
            //再均衡开始之前调用逻辑
            @Override
            public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
                //第一种应用，再均衡之前提交所有位移
                System.out.println("消费者再均衡之前调用onPartitionsRevoked");
                consumer.commitSync();
                //第二种应用将位移存储到数据库中，在均衡之后取出重新消费
                // store offset  in db
            }

            @Override
            public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
                for (TopicPartition partition : partitions) {
                    //从数据库中取出对应分区的偏移量
//                    offsets.put(partition,getOffsetFromDB(partition));
                }
            }
        });
```





### 消费者拦截器 `Consumerlnterceptor`
#### onConsume
时机：** poll 返回之前**调用，

作用：**修改消息内容和过滤部分消息**，

发生异常会记录日志，但是不会会向上传递

####  onCommit
作用：记录跟踪提交的位移信息

时机：提交位移之后调用

#### close()
时机：客户端关闭时调用







#### 消息 TTL 的设置


```java
    //设置TTL时间
    private static  final  long EXPIRE_INTERVAL=1000*10;

    @Override
    public ConsumerRecords onConsume(ConsumerRecords records) {
        //获取消息分区
        Set<TopicPartition> partitions = records.partitions();
        Map<TopicPartition,List<ConsumerRecord<String,String>>> newRecords = new HashMap<>();

        for (TopicPartition partition : partitions) {
            //获取每个分区的消息
            List<ConsumerRecord<String,String>> tpRecords = records.records(partition);

            List<ConsumerRecord<String,String>> newTpRecords = new ArrayList<>();
            for (ConsumerRecord tpRecord : tpRecords) {
                if (System.currentTimeMillis()-tpRecord.timestamp()<EXPIRE_INTERVAL) {
                    newTpRecords.add(tpRecord);
                }else{
                    System.out.println(tpRecord.value()+"已经过期了");
                }
            }

            newRecords.put(partition,newTpRecords);
        }
        ConsumerRecords<String, String> consumerRecords = new ConsumerRecords<>(newRecords);
        return consumerRecords;
    }

```







### 多线程实现
`acquire`  判断当前 Consumer 是否只有一个线程操作

```java
private final AtomicLong currentThread=new AtomicLong(NO CURRENT THREAD);//KafkaConsumer中的成员变量
private void acquire(){
    long threadId Thread.currentThread().getId();
acc
    if (threadId !currentThread.get()&
        currentThread.compareAndSet (NO CURRENT THREAD,threadId))
        throw new ConcurrentModificationException
    ("KafkaConsumer is not safe for multi-threaded access");
    refcount.incrementAndGet ()
}
```





`release` 释放当前 Conumser 的线程

```java
private void release(){
    if (refcount.decrementAndGet()==0)
        currentThread.set (NO CURRENT THREAD);
}
```

#### 第一种多线程实现 多线程消费者
![](/images/82d0ede2ee9ed3da55a86727a9e60bdd.png)



创建多个消费者线程进行消费



```java
public class MultiThreadConumser1 {
    public static final String  BROKDER_LIST="192.168.150.101:9092";
    public static final String TOPIC="topic-demo1";
    public static final String GROUP_ID="group.demo";
    private static final AtomicBoolean isRunning=new AtomicBoolean(true);
    public static final    Properties properties=new Properties();

    static {
        //开启自动提交
        properties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG,true);
        //设置服务器地址
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG,BROKDER_LIST);
        //设置反序列化器
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        //设置消费者组
        properties.put(ConsumerConfig.GROUP_ID_CONFIG,GROUP_ID);
    }
    public static void main(String[] args) {
        int currentThreadNum=4;
        for (int i = 0; i < currentThreadNum; i++) {
            new KafkaConcumerrThread(properties,TOPIC).run();
        }

    }


    //设置消费者多线程处理逻辑
    private static class  KafkaConcumerrThread extends Thread{

        private  final KafkaConsumer<String,String> consumer;
        
        public  KafkaConcumerrThread (Properties properties,String topic){
            this.consumer=new KafkaConsumer<String, String>(properties);
            consumer.subscribe(Collections.singleton(topic));
        }

        @Override
        public void run() {
            try {
                while (true){
                    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
                    for (ConsumerRecord<String, String> record : records) {
                        System.out.println(Thread.currentThread().getName()+"----处理消息----"+record.value());
                    }
                }
            }catch (Exception e){
                e.printStackTrace();
            }finally {
                consumer.close();
            }


        }
    }
}
```



**缺点**

1. 线程数不能大于分区数

> <font style="color:rgb(6, 6, 7);">多线程消费者线程数与分区数的关系主要基于以下几点：</font>
>
> 1. **<font style="color:rgb(6, 6, 7);">负载均衡</font>**<font style="color:rgb(6, 6, 7);">：每个消费者线程可以负责消费一个或多个分区的数据。如果消费者线程数等于分区数，那么理论上可以为每个分区分配一个消费者线程，这样可以确保负载均匀分布，每个线程只处理一个分区的消息，从而实现最大的并行度。</font>
> 2. **<font style="color:rgb(6, 6, 7);">性能优化</font>**<font style="color:rgb(6, 6, 7);">：如果消费者线程数多于分区数，那么一些线程将没有分区可以消费，这会导致资源浪费，因为这些线程将处于空闲状态。同时，如果一个线程消费多个分区，那么它需要在这些分区之间进行上下文切换，这可能会增加延迟并降低效率。</font>
> 3. **<font style="color:rgb(6, 6, 7);">避免竞争</font>**<font style="color:rgb(6, 6, 7);">：如果消费者线程数多于分区数，那么多个线程可能会尝试消费同一个分区的消息，这会导致不必要的竞争和冲突，因为Kafka的分区是设计为由单个消费者线程消费的。</font>
> 4. **<font style="color:rgb(6, 6, 7);">消费者组的分区策略</font>**<font style="color:rgb(6, 6, 7);">：在Kafka中，同一个消费者组内的消费者会根据分区数和消费者数自动进行分区分配。如果消费者数多于分区数，那么一些消费者将无法获得任何分区，从而无法消费消息。</font>
> 5. **<font style="color:rgb(6, 6, 7);">避免过度消费</font>**<font style="color:rgb(6, 6, 7);">：如果消费者线程数多于分区数，可能会导致某些消息被重复消费，因为多个线程可能会尝试消费同一个分区的消息，这违反了Kafka分区的设计理念。</font>
>

2. 多个线程建立了多个连接数，线程数量大时系统开销过大





**优点**

1. 保证每个分区的消息被顺序消费

#### 第二种实现 一个消费者多个线程处理消费逻辑
![](/images/913ad08a67fa660ebc2d918f3a284906.png)

缺点：

1. 不能报证顺序消费
2. 且位移容易丢失

```java
package com.xinc.consumer;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.serialization.StringDeserializer;

import java.time.Duration;
import java.util.Collections;
import java.util.Properties;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicBoolean;

/**
 * @version: 1.0.0
 * @Author: caoxin1
 * @description:
 * @date: 2024-11-10 7:57
 */
public class MultiThreadConsuemr2 {

    public static final String BROKDER_LIST = "192.168.150.101:9092";
    public static final String TOPIC = "topic-demo1";
    public static final String GROUP_ID = "group.demo";
    private static final AtomicBoolean isRunning = new AtomicBoolean(true);

    private static Properties initConfig() {
        Properties properties = new Properties();
        //开启自动提交
        properties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, true);
        //设置服务器地址
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, BROKDER_LIST);
        //设置反序列化器
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        //设置消费者组
        properties.put(ConsumerConfig.GROUP_ID_CONFIG, GROUP_ID);
        return properties;
    }

    public static void main(String[] args) {
        Properties properties = initConfig();
        int currentThreadNum = 4;
        for (int i = 0; i < currentThreadNum; i++) {
            new KafkaConcumerrThread(properties, TOPIC, Runtime.getRuntime().availableProcessors()).start();
        }

    }


    //设置消费者多线程处理逻辑
    private static class KafkaConcumerrThread extends Thread {

        private KafkaConsumer<String, String> consumer;
        private ExecutorService executorService;

        public KafkaConcumerrThread(Properties properties, String topic, int threadNumber) {
            this.consumer = new KafkaConsumer<String, String>(properties);
            consumer.subscribe(Collections.singleton(topic));
            executorService = Executors.newFixedThreadPool(threadNumber);
        }

        @Override
        public void run() {
            try {
                while (true) {
                    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(10));
                    executorService.submit(new RecoredHandler(records));

                }
            } catch (Exception e) {
                e.printStackTrace();
            }finally {
                consumer.close();
            }
        }



    }

    private static class RecoredHandler extends Thread {
        private ConsumerRecords<String, String> records;

        public RecoredHandler(ConsumerRecords record) {
            this.records = record;
        }

        @Override
        public void run() {
            for (ConsumerRecord record : records) {
                System.out.println(Thread.currentThread().getName() + "-----处理消息-----" + record.value());
            }

        }
    }
}

```

有实现方案但是目前还不知道咋实现[参考链接](https://github.com/MarshWinter/kafka-multithread-consume)



![](/images/2c988aac7c9deb1f0ed7b1631ab4858b.png)

### 重要参数
+ `<font style="color:rgb(199, 37, 78);background-color:rgb(249, 242, 244);">fetch.min.bytes</font>`<font style="color:rgba(0, 0, 0, 0.75);">：一次请求能拉取的最小数据量（默认1b）</font>
+ `<font style="color:rgb(199, 37, 78);background-color:rgb(249, 242, 244);">fetch.max.bytes</font>`<font style="color:rgba(0, 0, 0, 0.75);">：一次请求能拉取的最大数据量（默认52428800b，50m）</font>
+ `<font style="color:rgb(199, 37, 78);background-color:rgb(249, 242, 244);">fetch.max.wait.ms</font>`<font style="color:rgba(0, 0, 0, 0.75);">：与min.bytes有关，指定kafka拉取时的等待时间（默认500ms）</font>
+ `<font style="color:rgb(199, 37, 78);background-color:rgb(249, 242, 244);">max.partition.fetch.bytes</font>`<font style="color:rgba(0, 0, 0, 0.75);">：从每个分区里返回Consumer的最大数据量（默认1048576b，1m）</font>
+ `<font style="color:rgb(199, 37, 78);background-color:rgb(249, 242, 244);">max.poll.records</font>`<font style="color:rgba(0, 0, 0, 0.75);">：一次请求拉取的最大消息数（默认500）</font>
+ `<font style="color:rgb(199, 37, 78);background-color:rgb(249, 242, 244);">connections.max.idle.ms</font>`<font style="color:rgba(0, 0, 0, 0.75);">：多久后关闭闲置连接，默认（540000，9分钟）</font>
+ `<font style="color:rgb(199, 37, 78);background-color:rgb(249, 242, 244);">receive.buffer.bytes</font>`<font style="color:rgba(0, 0, 0, 0.75);">：Socket接收消息缓冲区的大小（默认65536，64k）</font>
+ `<font style="color:rgb(199, 37, 78);background-color:rgb(249, 242, 244);">send.buffer.bytes</font>`<font style="color:rgba(0, 0, 0, 0.75);">：Socket发送消息缓冲区的大小（默认131072，128k）</font>
+ `<font style="color:rgb(199, 37, 78);background-color:rgb(249, 242, 244);">request.timeout.ms</font>`<font style="color:rgba(0, 0, 0, 0.75);">：Consumer等待请求响应的最长时间（默认30000ms）</font>
+ `<font style="color:rgb(199, 37, 78);background-color:rgb(249, 242, 244);">metadata.max.age.ms</font>`<font style="color:rgba(0, 0, 0, 0.75);">：元数据过期时间（默认30000，5分钟）</font>
+ `<font style="color:rgb(199, 37, 78);background-color:rgb(249, 242, 244);">reconnect.backoff.ms</font>`<font style="color:rgba(0, 0, 0, 0.75);">：尝试重新连接指定主机前的等待时间（默认50ms）</font>
+ `<font style="color:rgb(199, 37, 78);background-color:rgb(249, 242, 244);">retry.backoff.ms</font>`<font style="color:rgba(0, 0, 0, 0.75);">：尝试重新发送失败请求到指定主题分区的等待时间（默认100ms）</font>
+ `<font style="color:rgb(199, 37, 78);background-color:rgb(249, 242, 244);">isolation.level</font>`<font style="color:rgba(0, 0, 0, 0.75);">：消费者的事务隔离级别</font>

