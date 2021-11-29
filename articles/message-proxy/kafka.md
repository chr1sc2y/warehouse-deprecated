# Kafka 入门

[TOC]

## 概述

Kafka 最初是为了解决 LinkedIn 数据管道问题应运而生的。它的设计目的是提供一个高性能的消息系统，可以处理多种数据类型，并能够实时提供纯净且结构化的用户活动数据和系统度量指标。

它不只是一个能够存储数据的系统（比如传统的关系型数据库、键值存储引擎、搜索引擎或缓存系统），还是一个持续变化和不断增长的流处理系统。现在 Kafka 已经被广泛地应用在社交网络的实时数据流处理当中，成为了下一代数据架构的基础。Kafka 经常会被拿来与现有的企业级消息系统、大数据系统（如 Hadoop）和数据集成 ETL 工具等技术作比较。

Kafka 有点像消息系统，允许发布和订阅消息流。从这点来看，它类似于ActiveMQ、RabbitMQ 或 IBM 的 MQSeries 等产品。Kafka 的特点在于它以集群的方式运行，可以**自由伸缩**，处理大量的应用程序；其次，Kafka 可以按照要求持久化数据，即提供了数据传递的保证——可复制、持久化，保留多长时间完全可以由你来决定。最后，流处理将数据处理的层次提升到了新高度；消息系统只会传递消息，而 Kafka 的流式处理能力让我们只用很少的代码就能够动态地处理派生流和数据集。



## 1 基础概念

#### 消息代理

在一个基于发布与订阅的消息系统中，数据消息的发送者不直接把消息发送给接收者，而是通过一个**消息代理 message broker** 传递消息，接收者订阅消息代理，并以特定的方式接收消息。Kafka 就是一个消息代理。

**消息代理 message broker** 是一种针对处理消息流而优化的数据库，它作为独立的中间服务运行，生产者和消费者作为客户端连接到消息代理服务，在使用消息代理的架构中主要有 3 种角色：

1. **生产者**将消息写入消息代理；生产者一般是异步架构的，当生产者发送消息时，它只会等待消息代理确认消息已经被缓存，而不等待消息被消费者处理
2. **消息代理**负责消息的存储，发送、重传等，一般会包含多个**消息队列 message queue**
3. **消费者**从消息代理接收消息并进行处理；消费者只依赖于消息代理，与生产者完全隔离

消息代理的优势主要有以下几点：

1. 实现异步处理，提升处理性能

   把消息处理流程使用消息代理异步化，不会阻塞生产者服务，生产者服务可以在得到处理结果之前继续执行，并提高其并发处理的能力。

2. 提高系统的可伸缩性

   生产者将大量消息推送到消息代理中，消息代理可以将这些消息分发给不同的消费者，使得多个消费者并行地处理消息，当消费者负载变化时，可以很容易地对消费者服务进行水平伸缩

3. 削峰填谷

   当生产者推送消息的速度比消费者处理消息的速度更快时，可以使用消息队列作为消息的缓冲，来削弱峰值流量，防止系统被短时间内的流量冲垮

4. 应用解耦

   使用消息代理后，生产者和消费者即可解耦，不再需要有任何联系，也不需要受对方的影响，只要保持使用一致的消息格式即可。

#### 消息和批次

Kafka 的数据单元被称为消息，消息类似于关系型数据库里的一个数据行或一条记录；消息由字节数组组成，当消息以一种可控的方式写入不同的分区时，会用到 key，kafka 会为 key 生成一个一致性散列值，然后使用散列值对主题分区数进行取模，为消息选取分区。这样可以保证具有相同 key 的消息总是被写到相同的分区上。

如果每一个消息都单独发送，会导致大量的网络开销。为了提高效率，消息会被分批次写入Kafka；**批次 batch** 就是一组消息，这些消息属于同一个主题和分区；批次数据在传输时会被压缩，这样可以提升数据的传输和存储能力；单个批次的消息数量越大，单位时间内处理的消息就越多，但单个批次的传输时间就越长，因此需要在时延和吞吐量之间作出权衡。

#### 主题和分区
Kafka 的消息通过**主题 topic** 进行分类，主题就好比关系型数据库的表，或者文件系统里的目录；同一个主题可以被分为若干个**分区 partition**，一个 partition 即一个提交日志，消息以追加的方式写入 partition，然后以先入先出的顺序读取。一个 topic 一般包含多个 partition，因此无法在整个 topic 的维度保证消息的顺序，只能保证消息在单个 partition 内的顺序。

我们通常用**流 stream** 来描述 Kafka 中的数据，在一个 topic 中，无论它有多少个partition，其中的数据都被视为一个 stream。

#### 生产者和消费者

Kafka 的客户端有两种基本类型：**生产者 producer**和**消费者 consumer**。

默认情况下，生产者生产的消息会被发布到一个特定的 topic 上。消息会被均衡地分布到这个 topic 的所有 partition 上，但我们也可以指定生产者的消息直接写到指定的 partition 上。

消费者会订阅一个或多个 topic，并按照消息生成的顺序依次读取它们。消费者通过检查消息的**偏移量 offset**来区
分已经读取过的消息，offset 是一种元数据，它会不断递增；在给定的 partition 中，每个消息的 offset 都是唯一的，offset 值会被保存在 Zookeeper 或 Kafka 上，这样即使消费者停止或重启后它的读取状态依然不会丢失。

在 Kafka 中，多个消费者可以组成一个**消费者群组 consumer group**，它们共同读取同一个 topic，group 会保证每个 partition 只能被一个消费者使用，并且这个群组的消费者对给定的消息只处理一次。

![consumer-group-1](https://raw.githubusercontent.com/ZintrulCre/warehouse/master/resources/message-proxy/consumer-group-1.png)

#### broker 和集群

一个独立的 Kafka 服务器被称为 broker；broker 会独立接收来自生产者的消息，为消息设置 offset，并将消息保存到磁盘，并处理消费者读取分区的请求，返回已经保存到磁盘上的消息；单个broker 最多可以处理数千个 partition 以及每秒百万级的消息量。

broker 是 kafka **集群 cluster** 的组成部分，每个集群都有一个 broker 充当集群控制器，负责管理工作，包括将 partition 分配给 broker，以及监控其他 broker。在集群中，一个 partition 从属于一个 broker，这个 broker 被称为 partition 的**首领 leader**。一个 partition 可能被分配给多个 broker，这个时候会发生分区**复制 replica**，这种复制机制为分区提供了消息冗余，如果有一个 broker 失效，其他 broker 可以接管领导权；同时，相关的消费者和生产者都要重新连接到新的首领。

![replica](https://raw.githubusercontent.com/ZintrulCre/warehouse/master/resources/message-proxy/replica.png)

Kafka broker **消息保留**的默认策略是保留一段时间或保留消息达到一定大小的字节数，当消息的规模达到这些上限时，旧消息就会过期并被删除，每个 topic 可以配置自己的保留策略。基于这个机制，Kafka 允许消费者非实时地从磁盘读取消息。

#### 多集群

随着 Kafka 部署数量的增加，基于数据类型分离，安全需求隔离和多数据中心容灾等原因，建议使用多集群方案，但 Kafka 的消息复制机制只能在单个集群里进行，不能在多个集群之间进行。为此 Kafka 提供了一个叫作 MirrorMaker 的工具， 可以用它来实现集群间的消息复制。

#### Zookeeper

ZooKeeper 是一个分布式协调框架，负责管理和协调 Kafka 集群的元数据，包括当前正在运行的 Broker，存在于集群中的 topic，每一个 topic 中的 partition，以及 partition 中的 leader 和 follower。

Kafka 使用 Zookeeper 来维护集群成员的信息，每个 broker 都有一个唯一的**标识符 id**，标识符可以通过配置文件指定，也可以自动生成。在 broker 启动的时候，它会将自己的 id 注册到 Zookeeper 上，Kafka 的其他组件组件通过订阅 Zookeeper 服务器的 /brokers/ids 路径来获取 broker 的信息和变化。当一个 broker 因为故障与 Zookeeper 断开连接时，Zookeeper 会将这个 broker 在的临时节点移除。所有监听了 broker 列表的其他组件会被通知该 broker 已经离线。虽然临时节点已经消失，不过它的 id 会继续存在于其他数据结构中，在完全关闭这个 broker 之后，如果使用相同的 id 启动另一个全新的 broker，它会立即加入集群，并拥有与这个 broker 相同的 topic 和 partition。

## 2 生产者

一个应用程序在很多情况下需要往 Kafka 写入消息：记录用户活动（用于统计和分析）、记录度量指标、保存日志消息、与其他应用程序进行异步通信、缓存即将写入到数据库的数据等等。多样的使用场景意味着多样的需求，每种消息的重要性，时延和吞吐量各不相同。例如在信用卡事务处理系统里，消息丢失或重复是不被允许的，可以接受的延迟在 500 ms 左右，同时对吞吐量要求较高——我们希望每秒钟可以处理一百万个或更多的消息。保存网站的点击信息则是另一种场景，这时我们允许丢失少量的消息丢失或重复，延迟也可以高一些，只要不影响用户体验即可。

下图展示了从生产者向 Kafka 发送消息的主要步骤。

![producer](https://raw.githubusercontent.com/ZintrulCre/warehouse/master/resources/message-proxy/producer.png)

生产者发送消息的第一步是创建一个 `ProducerRecord` 对象；`ProducerRecord` 必须包含 topic 和要发送的 value，同时还可以指定 key 或 partition。在发送 `ProducerRecord` 前，**序列化器 serializer** 会把 key 和 value 序列化成字节数组。

接下来，数据被传给**分区器 partitioner**，如果在 `ProducerRecord` 对象里指定了分区，那么会直接使用这个分区；如果没有指定分区，那么分区器会根据 `ProducerRecord` 对象的 key 来选择一个分区。选择好分区以后，生产者就知道该往哪个 topic 和 partition 发送这条消息了。

紧接着，这条记录被添加到一个 batch 里，这个 batch 里的所有消息会被发送到相同的 topic 和 partition 上。生产者会使用一个独立的线程把整个 batch 发送到对应的 broker 上。

Kafka 服务器在收到这个 batch 后会返回一个响应。如果消息成功写入Kafka，就返回一个 RecordMetaData 对象，它包含了消息发送到的 topic 和 partition，以及记录在分区里的 offset。如果写入失败，则会返回一个错误。生产者在收到错误之后会尝试重新发送消息，几次之后如果仍然失败，就返回错误信息。



### 2.1 序列化器

key 和 value 的序列化器在 `KafkaProducer` 的构造函数中由不同的 Serializer 初始化，如果传入的 Serializer 为空，则使用配置中的序列化器名字，利用反射机制来创建对象：

```java
public class KafkaProducer<K, V> implements Producer<K, V> {

    KafkaProducer(ProducerConfig config,
                  Serializer<K> keySerializer,
                  Serializer<V> valueSerializer,
                  ProducerMetadata metadata,
                  KafkaClient kafkaClient,
                  ProducerInterceptors<K, V> interceptors,
                  Time time) {
        // ...
        
            if (keySerializer == null) {
                this.keySerializer = config.getConfiguredInstance(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
                                                                                         Serializer.class);
                this.keySerializer.configure(config.originals(Collections.singletonMap(ProducerConfig.CLIENT_ID_CONFIG, clientId)), true);
            } else {
                config.ignore(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG);
                this.keySerializer = keySerializer;
            }
            if (valueSerializer == null) {
                this.valueSerializer = config.getConfiguredInstance(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,
                                                                                           Serializer.class);
                this.valueSerializer.configure(config.originals(Collections.singletonMap(ProducerConfig.CLIENT_ID_CONFIG, clientId)), false);
            } else {
                config.ignore(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG);
                this.valueSerializer = valueSerializer;
            }
        
        // ...
    }

}
```

以 `StringSerializer` 为例，它实现了 `Serializer` 接口，其中的 `serialize` 函数用来实现对数据的序列化：

```java
/**
 *  String encoding defaults to UTF8 and can be customized by setting the property key.serializer.encoding,
 *  value.serializer.encoding or serializer.encoding. The first two take precedence over the last.
 */
public class StringSerializer implements Serializer<String> {
    private String encoding = StandardCharsets.UTF_8.name();

    @Override
    public void configure(Map<String, ?> configs, boolean isKey) {
        String propertyName = isKey ? "key.serializer.encoding" : "value.serializer.encoding";
        Object encodingValue = configs.get(propertyName);
        if (encodingValue == null)
            encodingValue = configs.get("serializer.encoding");
        if (encodingValue instanceof String)
            encoding = (String) encodingValue;
    }

    @Override
    public byte[] serialize(String topic, String data) {
        try {
            if (data == null)
                return null;
            else
                return data.getBytes(encoding);
        } catch (UnsupportedEncodingException e) {
            throw new SerializationException("Error when serializing string to byte[] due to unsupported encoding " + encoding);
        }
    }
}
```

### 2.2 分区器

生产者 `KafkaProducer` 发送消息前，会调用 `partition` 接口来获取接收消息的分区，如果在 `record` 中指定了 partition，那么会直接使用这个 partition，否则会调用分区器 `partitioner` 的 `partition` 接口来获取。

```java
public class KafkaProducer<K, V> implements Producer<K, V> {
    
    private int partition(ProducerRecord<K, V> record, byte[] serializedKey, byte[] serializedValue, Cluster cluster) {
        Integer partition = record.partition();
        return partition != null ?
                partition :
                partitioner.partition(
                        record.topic(), record.key(), serializedKey, record.value(), serializedValue, cluster);
    }
    
}
```

`KafkaProducer` 使用的默认分区器是 `DefaultPartitioner`，它实现了 `Partitioner` 接口，其中的 `partition` 方法用来实现分配 partition 的逻辑：

```java
public class DefaultPartitioner implements Partitioner {

    private final ConcurrentMap<String, AtomicInteger> topicCounterMap = new ConcurrentHashMap<>();

    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
        int numPartitions = partitions.size();
        if (keyBytes == null) {
            int nextValue = nextValue(topic);
            List<PartitionInfo> availablePartitions = cluster.availablePartitionsForTopic(topic);
            if (availablePartitions.size() > 0) {
                int part = Utils.toPositive(nextValue) % availablePartitions.size();
                return availablePartitions.get(part).partition();
            } else {
                return Utils.toPositive(nextValue) % numPartitions;
            }
        } else {
            return Utils.toPositive(Utils.murmur2(keyBytes)) % numPartitions;
        }
    }

    private int nextValue(String topic) {
        AtomicInteger counter = topicCounterMap.get(topic);
        if (null == counter) {
            counter = new AtomicInteger(ThreadLocalRandom.current().nextInt());
            AtomicInteger currentCounter = topicCounterMap.putIfAbsent(topic, counter);
            if (currentCounter != null) {
                counter = currentCounter;
            }
        }
        return counter.getAndIncrement();
    }
    
}
```

这段代码主要分为两部分：

- 当 key 为空时，调用 `nextValue` 获取一个自增值，其中 `topicCounterMap` 为每一个 topic 维护一个计数值，每次调用 `nextValue` 时递增并对 nextValue 取余来获取 partition
- 当 key 不为空时，使用 murmur2 哈希算法对 key 进行 hash 操作，获取 Partition

可以看到只有当指定 key 的时候，消息才会被分配到同一个 partition 中，从而保证被有序地消费。

Kafka 2.4.0 引入了新的粘性分区缓存类 `StickyPartitionCache`；它会在 key 为空时，从缓存中获取分区数：

```java
public class DefaultPartitioner implements Partitioner {

    private final StickyPartitionCache stickyPartitionCache = new StickyPartitionCache();

    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster, int numPartitions) {
        if (keyBytes == null) {
            return stickyPartitionCache.partition(topic, cluster);
        }
        // hash the keyBytes to choose a partition
        return Utils.toPositive(Utils.murmur2(keyBytes)) % numPartitions;
    }
}


/**
 * An internal class that implements a cache used for sticky partitioning behavior. The cache tracks the current sticky
 * partition for any given topic. This class should not be used externally. 
 */
public class StickyPartitionCache {
    private final ConcurrentMap<String, Integer> indexCache;
    public StickyPartitionCache() {
        this.indexCache = new ConcurrentHashMap<>();
    }

    public int partition(String topic, Cluster cluster) {
        Integer part = indexCache.get(topic);
        if (part == null) {
            return nextPartition(topic, cluster, -1);
        }
        return part;
    }

    public int nextPartition(String topic, Cluster cluster, int prevPartition) {
        List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
        Integer oldPart = indexCache.get(topic);
        Integer newPart = oldPart;
        // Check that the current sticky partition for the topic is either not set or that the partition that 
        // triggered the new batch matches the sticky partition that needs to be changed.
        if (oldPart == null || oldPart == prevPartition) {
            List<PartitionInfo> availablePartitions = cluster.availablePartitionsForTopic(topic);
            if (availablePartitions.size() < 1) {
                Integer random = Utils.toPositive(ThreadLocalRandom.current().nextInt());
                newPart = random % partitions.size();
            } else if (availablePartitions.size() == 1) {
                newPart = availablePartitions.get(0).partition();
            } else {
                while (newPart == null || newPart.equals(oldPart)) {
                    int random = Utils.toPositive(ThreadLocalRandom.current().nextInt());
                    newPart = availablePartitions.get(random % availablePartitions.size()).partition();
                }
            }
            // Only change the sticky partition if it is null or prevPartition matches the current sticky partition.
            if (oldPart == null) {
                indexCache.putIfAbsent(topic, newPart);
            } else {
                indexCache.replace(topic, prevPartition, newPart);
            }
            return indexCache.get(topic);
        }
        return indexCache.get(topic);
    }

}
```

`StickyPartitionCache` 类会维护一个从 topic 到 partition 的缓存，其本质也是轮询。

在老版本的分区方式中，同一个 topic 中没有指定 partition 和 key 的消息会被轮询到不同的 partition 中，这样不同的 partition 中就会产生很多的 batch，从而导致更多的网络请求。而 StickyPartitionCache 保证了这样的消息可以被投递到同一个 partition 中，构成同一个 batch 发送给 Kafka 服务器，从而提高 Kafka 的吞吐量。

## 3 消费者

Kafka 的消费者服务经常会做一些高延迟的 IO 操作，比如把数据写到磁盘，数据库或 HDFS，或者使用数据进行比较耗时的计算。在这些情况下，单个消费者无法跟上数据生成的速度，所以只能通过横向伸缩的方式，增加更多的消费者，让它们分担负载，从而提高吞吐量。

就像多个生产者可以同时向相同的 topic 写入消息一样，我们也可以使用多个消费者订阅并从同一个 topic 读取消息，对消息进行分流。Kafka 中的消费者从属于消费者群组，一个群组订阅一个 topic，每个消费者则接收其中一部分 partition 的消息。

![consumer-group-2](https://raw.githubusercontent.com/ZintrulCre/warehouse/master/resources/message-proxy/consumer-group-2.png)

### 3.1 消费过程

Kafka 中客户端的消费是基于 pull 模式的，这样做的优势在于，不同消费者客户端的消费能力和消费策略不同，它们可以调整 pull 的频率以适配自己的 IO 能力；如果使用 push 模式则有可能因为推送速度过快而造成消费者客户端崩溃。

消费消息是一个不断**轮询 poll** 的过程，消费者所要做的就是不断地重复调用 `poll` ，并等待 Kafka 服务器返回其订阅的 partition 上的一组消息。

```java
public class KafkaConsumerDemo {
    
    public static void main(String[] args) {
        Properties props = initConfig();
        KafkaConsumer<String, String> consumer = new KafkaConsumer(props);
        consumer.subscribe(Arrays.asList(TOPIC));
        try {
            while (IS_RUNNING.get()) {
                ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
                for (ConsumerRecord<String, String> record : records) {
                    System.out.println("topic=" + record.topic() + ", partition=" + record.partition() + ", offset=" + record.offset());
                    System.out.println("key=" + record.key() + ", value=" + record.value());
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            consumer.close();
        }
    }
    
}
```

`poll` 方法返回一个记录列表 `ConsumerRecords`，其中每条记录都包含了记录所属 topic 和 partition 的信息，记录所在 partition 的 offset，以及记录的 key-value。同时 `poll` 方法有一个**超时 timeout** 参数，无论有没有可用的数据，`poll` 的执行时间一旦超过 timeout 就会立刻返回。

在消费者客户端退出之前应该调用 `close` 方法来将其关闭，并立即触发一次 group 的再平衡，而不是等待 group 协调器来发现这个客户端心跳超时并认定其已不再能提供服务。

### 3.2 偏移量

消费者对象 `KafkaConsumerRunner` 运行时，通过先订阅 `subscribe` 然后轮询 `poll` 来向 Kafka 服务器请求数据，每次调用 `poll` 方法后，服务器会返回 partition 中还没有被消费者读取过的记录，消费者可以追踪当前 partition 中的偏移量，而更新 partition 当前偏移量的操作叫作**提交 submit**。

如果消费者一直处于运行状态，那么偏移量就没有意义；而如果消费者发生崩溃或者有新的消费者加入 group，group 会触发和完成再平衡机制，每个消费者会被分配到新的 partition，为了能够继续之前的工作，消费者需要读取当前 partition 中最后一次提交的偏移量，然后从偏移量所在位置继续处理消息。最后一次提交的偏移量的位置如果在最后实际处理的消息之后或之前，分别会造成消息丢失和消息重复的问题，因此提交的方式显得尤为重要。

如果将消费者处理后的记录与偏移量都作为一个原子操作或事务都提交到数据库上，那么我们甚至可以不依赖于 Kafka 服务器提供的 offset，而是在消费者启动或分配到新分区后，直接调用 `seek` 方法，在 Kafka 服务器中查找这个特定 offset 位置的消息。

#### 自动提交

最简单的提交方式是让消费者自动提交偏移量，这种情况下，每过 5s（默认情况下），消费者会自动把从 `poll` 方法接收到的最大偏移量进行提交。

使用自动提交是存在隐患的，假设在设置的自动提交间隔之内，group 发生了再平衡，那么再平衡完成之后消费者获取到的偏移量则会落后于实际应该处理的消息位置，部分消息会被重复处理。虽然可以通过缩短提交间隔来更频繁地提交偏移量，减少重复处理消息的数量，但这样的情况是无法避免的。

#### 同步提交

通过调用 `KafkaConsumer.commitSync`，可以对偏移量进行同步提交，不传递任何参数时默认提交的最大偏移量。

```java
public class KafkaConsumer<K, V> implements Consumer<K, V> {

    public void commitSync(final Map<TopicPartition, OffsetAndMetadata> offsets, final Duration timeout) {
        acquireAndEnsureOpen(); // Acquire the light lock and ensure that the consumer hasn't been closed.
        long commitStart = time.nanoseconds();
        try {
            maybeThrowInvalidGroupIdException();
            offsets.forEach(this::updateLastSeenEpochIfNewer);
            if (!coordinator.commitOffsetsSync(new HashMap<>(offsets), time.timer(timeout))) {
                throw new TimeoutException("Timeout of " + timeout.toMillis() + "ms expired before successfully " +
                        "committing offsets " + offsets);
            }
        } finally {
            kafkaConsumerMetrics.recordCommitSync(time.nanoseconds() - commitStart);
            release();
        }
    }
    
    private void acquireAndEnsureOpen() {
        acquire();
        if (this.closed) {
            release();
            throw new IllegalStateException("This consumer has already been closed.");
        }
    }
    
}
```

`acquireAndEnsureOpen` 会尝试获取锁，并确保当前客户端尚未关闭，但实际上 `KafkaConsumer` 并不支持被并发访问，所以在被多个线程调用时会简单地抛出状态错误的异常。

```java
public final class ConsumerCoordinator extends AbstractCoordinator {
    
    public boolean commitOffsetsSync(Map<TopicPartition, OffsetAndMetadata> offsets, Timer timer) {
        invokeCompletedOffsetCommitCallbacks();

        if (offsets.isEmpty())
            return true;

        do {
            if (coordinatorUnknown() && !ensureCoordinatorReady(timer)) {
                return false;
            }

            RequestFuture<Void> future = sendOffsetCommitRequest(offsets);
            client.poll(future, timer);

            // We may have had in-flight offset commits when the synchronous commit began. If so, ensure that
            // the corresponding callbacks are invoked prior to returning in order to preserve the order that
            // the offset commits were applied.
            invokeCompletedOffsetCommitCallbacks();

            if (future.succeeded()) {
                if (interceptors != null)
                    interceptors.onCommit(offsets);
                return true;
            }

            if (future.failed() && !future.isRetriable())
                throw future.exception();

            timer.sleep(rebalanceConfig.retryBackoffMs);
        } while (timer.notExpired());

        return false;
    }
    
}
```

`ConsumerCoordinator.commitOffsetsSync` 先通过 `sendOffsetCommitRequest` 获取到发送 request 后的 `future` 对象，然后再使用 `poll(future)` 阻塞地等待其执行完成；如果同步提交失败，但尚未超时，`` 会在 `` 函数中尝试重新提交，这可以最大程度地保证数据提交成功，但同时也会降低程序的吞吐量。

#### 异步提交

和同步提交相反，异步提交在执行的时候不会阻塞消费者线程，这样可以使得消费者客户端的性能和吞吐量得到一定的提升。

```java
public class KafkaConsumer<K, V> implements Consumer<K, V> {

    public void commitAsync(final Map<TopicPartition, OffsetAndMetadata> offsets, OffsetCommitCallback callback) {
        acquireAndEnsureOpen();
        try {
            maybeThrowInvalidGroupIdException();
            log.debug("Committing offsets: {}", offsets);
            offsets.forEach(this::updateLastSeenEpochIfNewer);
            coordinator.commitOffsetsAsync(new HashMap<>(offsets), callback);
        } finally {
            release();
        }
    }
    
}
```

相较于同步提交 `commitSync`，异步提交 `commitAsync` 接口的唯一区别是在参数里增加了一个回调函数。

```java
public final class ConsumerCoordinator extends AbstractCoordinator {
    
	private void doCommitOffsetsAsync(final Map<TopicPartition, OffsetAndMetadata> offsets, final OffsetCommitCallback callback) {
        this.subscriptions.needRefreshCommits();
        RequestFuture<Void> future = sendOffsetCommitRequest(offsets);
        final OffsetCommitCallback cb = callback == null ? defaultOffsetCommitCallback : callback;
        future.addListener(new RequestFutureListener<Void>() {
            @Override
            public void onSuccess(Void value) {
                if (interceptors != null)
                    interceptors.onCommit(offsets);

                completedOffsetCommits.add(new OffsetCommitCompletion(cb, offsets, null));
            }

            @Override
            public void onFailure(RuntimeException e) {
                Exception commitException = e;

                if (e instanceof RetriableException)
                    commitException = RetriableCommitFailedException.withUnderlyingMessage(e.getMessage());

                completedOffsetCommits.add(new OffsetCommitCompletion(cb, offsets, commitException));
            }
        });
    }
    
}
```

类似地，`ConsumerCoordinator.doCommitOffsetsAsync` 会先通过 `sendOffsetCommitRequest` 获取到 `future` 对象，但只为其添加好监听事件就返回了。

### 3.3 再平衡

再平衡指当 partition 的所属权从一个消费者转移到另一个消费者的行为，它为 group 的高可用性和伸缩性提供了保证；一般有三种时机会触发再平衡：

- group 中新增或移除了部分消费者，，对应的 partition 需要被分配给 group 内的其他消费者；
- group 订阅的 topic 发生了变化，例如 group 利用正则表达式订阅了 "test.*" 的 topic，在某个时间集群内新增了一个符合正则表达式的 topic，那么该 topic 的所有 partition 也会被分配给当前的 group；
- group 订阅的 topic 中新增了 partition。

在再平衡发生的期间，group 内的消费者无法读取消息，即整个 group 会变得**不可用**；并且当一个 partition 被重新分配给另一个消费者时，消费者的**状态会丢失**，即其偏移量有可能尚未提交。因此一般情况下应该尽量避免不必要的再平衡的发生。

#### RangeAssignor

RangeAssignor 是默认的，也是最简单的再平衡策略：

```java
public class RangeAssignor extends AbstractPartitionAssignor {
    public static final String RANGE_ASSIGNOR_NAME = "range";
    
    @Override
    public Map<String, List<TopicPartition>> assign(Map<String, Integer> partitionsPerTopic,
                                                    Map<String, Subscription> subscriptions) {
        Map<String, List<MemberInfo>> consumersPerTopic = consumersPerTopic(subscriptions);

        Map<String, List<TopicPartition>> assignment = new HashMap<>();
        for (String memberId : subscriptions.keySet())
            assignment.put(memberId, new ArrayList<>());

        for (Map.Entry<String, List<MemberInfo>> topicEntry : consumersPerTopic.entrySet()) {
            String topic = topicEntry.getKey();
            List<MemberInfo> consumersForTopic = topicEntry.getValue();

            // 每个 topic 的 partition 数量
            Integer numPartitionsForTopic = partitionsPerTopic.get(topic);
            if (numPartitionsForTopic == null)
                continue;

            Collections.sort(consumersForTopic);

            // 每个消费者都会被分配到的 partition 数量
            int numPartitionsPerConsumer = numPartitionsForTopic / consumersForTopic.size();
            // 部分消费者会被分配到的额外 partition
            int consumersWithExtraPartition = numPartitionsForTopic % consumersForTopic.size();

            List<TopicPartition> partitions = AbstractPartitionAssignor.partitions(topic, numPartitionsForTopic);
            for (int i = 0, n = consumersForTopic.size(); i < n; i++) {
                int start = numPartitionsPerConsumer * i + Math.min(i, consumersWithExtraPartition);
                int length = numPartitionsPerConsumer + (i + 1 > consumersWithExtraPartition ? 0 : 1);
                assignment.get(consumersForTopic.get(i).memberId).addAll(partitions.subList(start, start + length));
            }
        }
        return assignment;
    }
}
```

可以看到 `RangeAssignor.assign` 函数中会计算 partition 数量 `numPartitionsForTopic` 与消费者数量 `consumersForTopic` 的整除结果 `numPartitionsPerConsumer` 和取余结果 `consumersWithExtraPartition`，并将连续的 partition 分配给同一个消费者。

#### RoundRobinAssignor

RoundRobinAssignor 策略的原理是将 group 内所有消费者以及所有 partition 按照字典序排序，然后通过轮询的方式逐个地进行分配：

```java

public class RoundRobinAssignor extends AbstractPartitionAssignor {
    public static final String ROUNDROBIN_ASSIGNOR_NAME = "roundrobin";

    @Override
    public Map<String, List<TopicPartition>> assign(Map<String, Integer> partitionsPerTopic,
                                                    Map<String, Subscription> subscriptions) {
        Map<String, List<TopicPartition>> assignment = new HashMap<>();
        List<MemberInfo> memberInfoList = new ArrayList<>();
        for (Map.Entry<String, Subscription> memberSubscription : subscriptions.entrySet()) {
            assignment.put(memberSubscription.getKey(), new ArrayList<>());
            memberInfoList.add(new MemberInfo(memberSubscription.getKey(),
                                              memberSubscription.getValue().groupInstanceId()));
        }

        // 环形链表，存储所有的消费者
        CircularIterator<MemberInfo> assigner = new CircularIterator<>(Utils.sorted(memberInfoList));

        // 获取 group 当前订阅的所有 topic 的 partition
        for (TopicPartition partition : allPartitionsSorted(partitionsPerTopic, subscriptions)) {
            final String topic = partition.topic();
            while (!subscriptions.get(assigner.peek().memberId).topics().contains(topic))
                assigner.next();
            assignment.get(assigner.next().memberId).add(partition);
        }
        return assignment;
    }

}
```

以上两种方式都对所有的 partition 进行了全量重新分配。

#### StickyAss的ignor

StickyAssignor 策略的目的有两个：

1. partition 的分配要尽可能的均匀，分配给各个消费者的 partition 数量最多只相差一个；
2. partition 的分配尽可能的与再分配之前的保持相同；

当两者冲突的时候，第一个目标优先于第二个目标。

```java
public abstract class AbstractStickyAssignor extends AbstractPartitionAssignor {
    
        public Map<String, List<TopicPartition>> assign(Map<String, Integer> partitionsPerTopic,
                                                    Map<String, Subscription> subscriptions) {
        Map<String, List<TopicPartition>> consumerToOwnedPartitions = new HashMap<>();
        Set<TopicPartition> partitionsWithMultiplePreviousOwners = new HashSet<>();
        if (allSubscriptionsEqual(partitionsPerTopic.keySet(), subscriptions, consumerToOwnedPartitions, partitionsWithMultiplePreviousOwners)) {
            log.debug("Detected that all consumers were subscribed to same set of topics, invoking the "
                          + "optimized assignment algorithm");
            partitionsTransferringOwnership = new HashMap<>();
            return constrainedAssign(partitionsPerTopic, consumerToOwnedPartitions, partitionsWithMultiplePreviousOwners);
        } else {
            log.debug("Detected that not all consumers were subscribed to same set of topics, falling back to the "
                          + "general case assignment algorithm");
            partitionsTransferringOwnership = null;
            return generalAssign(partitionsPerTopic, subscriptions, consumerToOwnedPartitions);
        }
    }
    
}
```

`AbstractStickyAssignor` 虚基类的 `assign` 函数中将再平衡分为了两种情况，分别是 group 里的消费者都订阅了相同的 topic，以及订阅的 topic 不尽相同。

对于前者，需要获取上次的分配情况，并删除已经失效的 partition，这样就获取到了一份可用的，与之前尽可能相同的预分配名单，但是可能分配不均，因此进入 `constrainedAssign` 函数来进行进一步的公平分配：

```java
public abstract class AbstractStickyAssignor extends AbstractPartitionAssignor {
    
    private Map<String, List<TopicPartition>> constrainedAssign(Map<String, Integer> partitionsPerTopic,
                                                                Map<String, List<TopicPartition>> consumerToOwnedPartitions,
                                                                Set<TopicPartition> partitionsWithMultiplePreviousOwners) {
        // ...
        
        // 消费者数量
        int numberOfConsumers = consumerToOwnedPartitions.size();
        // 分区数量
        int totalPartitionsCount = partitionsPerTopic.values().stream().reduce(0, Integer::sum);

        // 每个消费者分配到的 partition 最小数量
        int minQuota = (int) Math.floor(((double) totalPartitionsCount) / numberOfConsumers);
        // 每个消费者分配到的 partition 最大数量
        int maxQuota = (int) Math.ceil(((double) totalPartitionsCount) / numberOfConsumers);
        
        // 会获得额外 partition 的消费者数量
        int expectedNumMembersWithOverMinQuotaPartitions = totalPartitionsCount % numberOfConsumers;
        
        // ...

        // 验证预分配的数量与 minQuota 和 maxQuota 的关系，将超过的 maxQuota 的部分移除并记录，分配给低于 minQuota 的消费者
        // 注意这一步只能保证能够将所有超过 maxQuota 的部分取出
        for (Map.Entry<String, List<TopicPartition>> consumerEntry : consumerToOwnedPartitions.entrySet()) {
            List<TopicPartition> ownedPartitions = consumerEntry.getValue();

            // 对于预分配时未达到 minQuota 的消费者，将多余的分配给它们
            if (ownedPartitions.size() < minQuota) {
                if (ownedPartitions.size() > 0) {
                    consumerAssignment.addAll(ownedPartitions);
                    assignedPartitions.addAll(ownedPartitions);
                }
                unfilledMembersWithUnderMinQuotaPartitions.add(consumer);
            } else if (ownedPartitions.size() >= maxQuota && currentNumMembersWithOverMinQuotaPartitions < expectedNumMembersWithOverMinQuotaPartitions) { // 对于预分配时超过 maxQuota 的消费者，将多余的移除并保存下来
                currentNumMembersWithOverMinQuotaPartitions++;
                if (currentNumMembersWithOverMinQuotaPartitions == expectedNumMembersWithOverMinQuotaPartitions) {
                    unfilledMembersWithExactlyMinQuotaPartitions.clear();
                }
                List<TopicPartition> maxQuotaPartitions = ownedPartitions.subList(0, maxQuota);
                consumerAssignment.addAll(maxQuotaPartitions);
                assignedPartitions.addAll(maxQuotaPartitions);
                allRevokedPartitions.addAll(ownedPartitions.subList(maxQuota, ownedPartitions.size()));
            } else { // 对于预分配时恰好有 minQuota 的消费者，如果有剩余未分配的 partition，则分配一个给它们
                List<TopicPartition> minQuotaPartitions = ownedPartitions.subList(0, minQuota);
                consumerAssignment.addAll(minQuotaPartitions);
                assignedPartitions.addAll(minQuotaPartitions);
                allRevokedPartitions.addAll(ownedPartitions.subList(minQuota, ownedPartitions.size()));
                // 如果没有分配，则记录下来
                if (currentNumMembersWithOverMinQuotaPartitions < expectedNumMembersWithOverMinQuotaPartitions) {
                    unfilledMembersWithExactlyMinQuotaPartitions.add(consumer);
                }
            }
        }
        
        // 使用轮询的方式，再将多余的 quota 分配给未达到 minQuota 的消费者
        Iterator<String> unfilledConsumerIter = unfilledMembersWithUnderMinQuotaPartitions.iterator();
        for (TopicPartition unassignedPartition : unassignedPartitions) {
            // ...
            
            unfilledConsumerIter = unfilledMembersWithUnderMinQuotaPartitions.iterator();
            consumer = unfilledConsumerIter.next();
            
            int currentAssignedCount = consumerAssignment.size();
            if (currentAssignedCount == minQuota) {
                unfilledConsumerIter.remove();
                unfilledMembersWithExactlyMinQuotaPartitions.add(consumer);
            }
            
            // ...
        }
        
        // ...
    }
    
}
```

后者的实现与前者类似，有兴趣的朋友可以自行研究。

## 4 服务器

### 4.1 复制和副本

**复制 replication** 是 Kafka 服务器架构具备数据冗余，高伸缩性，故障自动转移功能的核心机制，它能够保证当个别节点失效时，Kafka 服务器仍然能够正常提供服务。Kafka 服务器中的 topic 被分为若干个 partition，每个 partition 有 n 个**副本 replica**，n 是 topic 的**复制因子 replication factor**。

replica 分为两种类型，一种是**首领 leader**，每个 partition 有且仅有一个 leader，为了保证数据的一致性，所有来自生产者和消费者的请求都会到达 leader；另一种是**跟随者 follower**，它的主要功能是进行数据的备份，partition 中除了 leader 以外的所有 replica 都是 follower，follower 不会处理任何请求，它们只会向 leader 发送 pull 请求（类似于消费者客户端），进行消息的复制，并始终保持与 leader 一致的状态，如果 leader 发生了失效，其中的一个 follower 会被提升为新的 leader。

对于客户端而言，follower 没有任何意义，它们既不像 MySQL 那样能够帮助 leader 分担承载读请求，也不能实现将就近读取来改善数据局部性，其原因有两点：第一是为了实现 RyW (read your write) / WfR (write follow read)，即每个写操作都依赖于上一个读操作，避免因多个节点写顺序不同导致的 dirty read，在**粘滞会话 sticky sessions** 的层面解决了一致性问题；第二是为了实现 monotonic read，即客户端在读到一个 value 之后，后续的所有读操作都要读到这个 value 或其之后更新。

#### 数据一致性

一般来说，有两种方案可以保证数据的强一致性：**主备复制 primary-backup replication** 和 **分布式复制 quorum-based replication**。

分布式复制一般采用 paxos 和 raft 等**共识算法 consensus algorithm** 实现，其特点是在有 2n + 1个节点的情况下，最多可以容忍 n 个节点失败，它的优点在于延迟更低（因为只需要部分节点写入成功）。

而主备复制需要等待所有的节点写入成功，在有 n 个节点的情况下，最多可以容忍 n-1 节点失败；它的优点在于可以容忍更多的节点失败（只要有一个节点存活就可以继续工作），并且最少只要有两个节点存在就可以提供服务，而前者需要有至少三个节点。

Kafka 使用了经典的主备复制方式来实现集群之间的日志复制，每个 replica 都在磁盘上维护了一个日志，他们会将接收到的日志按按顺序添加到日志中。当生产者将消息 push 到某个 partition 时，该消息首先会被转发到该 partition 的 leader 上，leader 将其追加到其磁盘日志中，同时 partition 上的其他所有 follower 不断地向 leader 请求消息，只有当有足够多的 follower 成功处理完消息后，leader 才会认为消息已经处理完成；但 leader 如果总是等待所有 follower 处理完消息，势必会提高系统的延迟，降低服务的可用性。

#### ISR

为了解决上面问题，Kafka引入了 **ISR In-Sync Replica** 的概念。partition 中的所有 replica 统称为**AR Assigned Repllicas**，所有与 leader 保持一定程度同步的组成了 **ISR In-Sync Replicas**，ISR 是 AR 的子集，ISR 包含了 leader。leader 接收到生产者发送的消息后，follower 才能从 leader 上拉取消息进行同步，同步期间内 follower 相对于 leader 会有一定程度的滞后，所谓的“一定程度”指可以忍受的滞后时间，这个滞后时间可以通过修改配置进行调整。相较于 leader 滞后过多的副本会组成**OSR Out-Sync Relipcas**，AR = ISR + OSR。正常情况下，所有的 follower 都应该与 leader 保持一定程度的同步，即AR = ISR, OSR = Ø。

Leader 负责维护和跟踪所有 follower 的滞后状态，当 follower 落后太多或者失效时，leader 会将其从 ISR 集合中剔除；如果 OSR 集合中的 follower 追上了 leader，leader 则会将其加入 ISR 集合。只有 ISR 集合中的 follower 才有资格被选举为leader。但如果当 ISR 集合变为空集，即 leader 失效，且没有其他任何一个 follower 与 leader 之前的状态一致时，可以通过修改配置开启 **Unclean Leader Election**，使得 OSR 集合中的 follower 可以成为新的 leader。这样可能会造成数据的重复消费，但好处是它使得 partition 至少有一个 leader 并可以继续提供服务，从而提高了高可用性。在这个问题上，Kafka 赋予了开发者选择 CAP 理论中一致性（Consistency）和可用性（Availability）的权利。

#### partition 分配

在创建 topic 时，Kafka 服务器首先会决定如何在 broker 间分配 replica，分配有三个目标：

1. 尽可能在 broker 间平均地分布 replica
2. 确保每个 replica 分布在不同的 broker 上
3. 如果为 broker 指定了机架信息，那么尽可能把每个 replica 分配到不同机架的 broker 上，保证在一个机架不可用的情况下不会导致整个 partition 不可用

如下是分配的流程：

```scala
object AdminUtils extends Logging {
    
  def assignReplicasToBrokers(brokerMetadatas: Iterable[BrokerMetadata],
                              nPartitions: Int,
                              replicationFactor: Int,
                              fixedStartIndex: Int = -1,
                              startPartitionId: Int = -1): Map[Int, Seq[Int]] = {
    if (nPartitions <= 0)
      throw new InvalidPartitionsException("Number of partitions must be larger than 0.")
    if (replicationFactor <= 0)
      throw new InvalidReplicationFactorException("Replication factor must be larger than 0.")
    if (replicationFactor > brokerMetadatas.size)
      throw new InvalidReplicationFactorException(s"Replication factor: $replicationFactor larger than available brokers: ${brokerMetadatas.size}.")
    // 没有指定机架
    if (brokerMetadatas.forall(_.rack.isEmpty))
      assignReplicasToBrokersRackUnaware(nPartitions, replicationFactor, brokerMetadatas.map(_.id), fixedStartIndex,
        startPartitionId)
    else {
      if (brokerMetadatas.exists(_.rack.isEmpty))
        throw new AdminOperationException("Not all brokers have rack information for replica rack aware assignment.")
      // 指定机架
      assignReplicasToBrokersRackAware(nPartitions, replicationFactor, brokerMetadatas, fixedStartIndex,
        startPartitionId)
    }
  }

  // 没有指定机架
  private def assignReplicasToBrokersRackUnaware(nPartitions: Int,
                                                 replicationFactor: Int,
                                                 brokerList: Iterable[Int],
                                                 fixedStartIndex: Int,
                                                 startPartitionId: Int): Map[Int, Seq[Int]] = {
    val ret = mutable.Map[Int, Seq[Int]]()
    val brokerArray = brokerList.toArray
    // 随机选取一个 broker 作为起始位置 startIndex
    val startIndex = if (fixedStartIndex >= 0) fixedStartIndex else rand.nextInt(brokerArray.length)
    // 当前当前 partition 的 id 设置为 0
    var currentPartitionId = math.max(0, startPartitionId)
    // 随机选取 broker 的位移
    var nextReplicaShift = if (fixedStartIndex >= 0) fixedStartIndex else rand.nextInt(brokerArray.length)
    for (_ <- 0 until nPartitions) {
      // 在遍历过 broker 数目个 partition 后将位移加一
      if (currentPartitionId > 0 && (currentPartitionId % brokerArray.length == 0))
        nextReplicaShift += 1
      // 当前 partition 的 id 加上起始位置，对 brokersize 取余得到第一个副本的位置
      val firstReplicaIndex = (currentPartitionId + startIndex) % brokerArray.length
      val replicaBuffer = mutable.ArrayBuffer(brokerArray(firstReplicaIndex))
      for (j <- 0 until replicationFactor - 1)
        // 计算出每个副本的位置
        replicaBuffer += brokerArray(replicaIndex(firstReplicaIndex, nextReplicaShift, j, brokerArray.length))
      ret.put(currentPartitionId, replicaBuffer)
      //分区 id 加一
      currentPartitionId += 1
    }
    ret
  }

}
```

举个例子，假设现在有 6 个broker，我们打算创建一个包含 10 个 partition 的topic，replication factor = 3，那么总共应该有 30 个 replica。为了实现这个目标，我们随机选择一个 broker（假设是 4），然后使用轮询的方式给每个 broker 分配 partition 来确定 leader 的位置，于是 leader0 会在 broker 4 上，leader 1 在 broker 5 上，leader 2 在 broker 0 上，以此类推。然后从 leader 开始，依次分配 follower，既然 leader 0 在 broker 4 上，那么它的 follower 就会依次分布在 broker 5, broker 0；leader 1 在 broker 5 上，那么它的 follower 就会依次分布在 broker 0, broker 1 上。如果配置了机架信息，那么就会按照交替机架的方式来选择 broker。

### 4.2 消息处理

Kafka 服务器采用 Reactor 模式处理消息。Reactor 模式是**事件驱动**架构的一种实现方式，适用于处理多个客户端**并发**向服务器端发起请求的场景，多个客户端会发送请求给 Reactor，Reactor 的请求分发线程 Acceptor 将不同的请求分发给多个工作线程 worker thread 处理。

![reactor](https://raw.githubusercontent.com/ZintrulCre/warehouse/master/resources/message-proxy/reactor.png)

broker 上有一个SocketServer 组件，类似于 Reactor 模式中的 Dispatcher，包括对应的 Acceptor 线程和工作线程池，在 Kafka 中叫做网络线程池，默认值为 3 个。Acceptor 线程采用**轮询**的方式将流量公平地分发给所有网络线程，避免了请求处理的倾斜，有利于实现较为公平的请求处理调度。

![process-request](https://raw.githubusercontent.com/ZintrulCre/warehouse/master/resources/message-proxy/process-request.png)

网络线程获取到请求后，并不会立即处理，而是将请求放入到共享请求队列中；随后，IO 线程池（默认有 8 个）负责从共享请求队列中取出请求，并执行真正的处理，如果是生产者发送的 push request，则将消息写入到底层的磁盘日志中，如果是消费者发送的 pull request，则从磁盘或页缓存中读取消息。IO 线程处理完请求后，会将生成的响应发送到网络线程池的响应队列中，然后由对应的网络线程将 response 返回给客户端；其中请求队列是所有网络线程共享的，而响应队列是每个网络线程专属的。在 IO 线程池和相应队列之间还有一个 Purgatory 组件，用于缓存延时请求。

### 4.3 物理存储

Kafka 的基本存储单元是 partition，partition 无法在多个 broker 间再进行细分，也无法在同一个 broker 的多个磁盘上再进行细分。因此，partition 的大小受到单个挂载点可用空间的限制（一个挂载点由单个磁盘或多个磁盘组成，如果配置了JBOD 属于单个磁盘，RAID 属于多个磁盘）。在配置 Kafka 的时候，我们可以指定用于存储 partition 的目录。

















日志压缩是Kafka 的一个高级特性，因为有了这
个特性，Kafka 可以用来长时间地保存数据。





























































