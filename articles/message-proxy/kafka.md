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

![consumer group](https://raw.githubusercontent.com/ZintrulCre/warehouse/master/resources/message-proxy/consumer group.png)

#### broker 和集群

一个独立的 Kafka 服务器被称为 broker；broker 会独立接收来自生产者的消息，为消息设置 offset，并将消息保存到磁盘，并处理消费者读取分区的请求，返回已经保存到磁盘上的消息；单个broker 最多可以处理数千个 partition 以及每秒百万级的消息量。

broker 是 kafka **集群 cluster** 的组成部分，每个集群都有一个 broker 充当集群控制器，负责管理工作，包括将 partition 分配给 broker，以及监控其他 broker。在集群中，一个 partition 从属于一个 broker，这个 broker 被称为 partition 的**首领 leader**。一个 partition 可能被分配给多个 broker，这个时候会发生分区**复制 replica**，这种复制机制为分区提供了消息冗余，如果有一个 broker 失效，其他 broker 可以接管领导权；同时，相关的消费者和生产者都要重新连接到新的首领。

![replica](https://raw.githubusercontent.com/ZintrulCre/warehouse/master/resources/message-proxy/replica.png)

Kafka broker **消息保留**的默认策略是保留一段时间或保留消息达到一定大小的字节数，当消息的规模达到这些上限时，旧消息就会过期并被删除，每个 topic 可以配置自己的保留策略。基于这个机制，Kafka 允许消费者非实时地从磁盘读取消息。

#### 多集群

随着 Kafka 部署数量的增加，基于数据类型分离，安全需求隔离和多数据中心容灾等原因，建议使用多集群方案，但 Kafka 的消息复制机制只能在单个集群里进行，不能在多个集群之间进行。为此 Kafka 提供了一个叫作 MirrorMaker 的工具， 可以用它来实现集群间的消息复制。



## 2 生产者

一个应用程序在很多情况下需要往 Kafka 写入消息：记录用户活动（用于统计和分析）、记录度量指标、保存日志消息、与其他应用程序进行异步通信、缓存即将写入到数据库的数据等等。多样的使用场景意味着多样的需求，每种消息的重要性，时延和吞吐量各不相同。例如在信用卡事务处理系统里，消息丢失或重复是不被允许的，可以接受的延迟在 500 ms 左右，同时对吞吐量要求较高——我们希望每秒钟可以处理一百万个或更多的消息。保存网站的点击信息则是另一种场景，这时我们允许丢失少量的消息丢失或重复，延迟也可以高一些，只要不影响用户体验即可。

下图展示了从生产者向 Kafka 发送消息的主要步骤。

![producer](https://raw.githubusercontent.com/ZintrulCre/warehouse/master/resources/message-proxy/producer.png)

生产者发送消息的第一步是创建一个 ProducerRecord 对象；ProducerRecord 必须包含 topic 和要发送的 value，同时还可以指定 key 或 partition。在发送 ProducerRecord 前，**序列化器 serializer** 会把 key 和 value 序列化成字节数组。

接下来，数据被传给**分区器 partitioner**，如果在 ProducerRecord 对象里指定了分区，那么会直接使用这个分区；如果没有指定分区，那么分区器会根据 ProducerRecord 对象的 key 来选择一个分区。选择好分区以后，生产者就知道该往哪个 topic 和 partition 发送这条消息了。

紧接着，这条记录被添加到一个 batch 里，这个 batch 里的所有消息会被发送到相同的 topic 和 partition 上。生产者会使用一个独立的线程把整个 batch 发送到对应的 broker 上。

Kafka 服务器在收到这个 batch 后会返回一个响应。如果消息成功写入Kafka，就返回一个 RecordMetaData 对象，它包含了消息发送到的 topic 和 partition，以及记录在分区里的 offset。如果写入失败，则会返回一个错误。生产者在收到错误之后会尝试重新发送消息，几次之后如果仍然失败，就返回错误信息。

### 2.1 分区器

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

#### 序列化器

创建一个生产者对象时必须指定序列化器，

















































### 消息代理优势

1. 实现异步处理，提升处理性能

把消息处理流程使用消息代理异步化，不会阻塞生产者服务，生产者服务可以在得到处理结果之前继续执行，并提高其并发处理的能力。

2. 提高系统的可伸缩性

生产者将大量消息推送到消息代理中，消息代理可以将这些消息分发给不同的消费者，使得多个消费者并行地处理消息，当消费者负载变化时，可以很容易地对消费者服务进行水平伸缩

3. 削峰填谷

当生产者推送消息的速度比消费者处理消息的速度更快时，可以使用消息队列作为消息的缓冲，来削弱峰值流量，防止系统被短时间内的流量冲垮

4. 应用解耦

使用消息代理后，生产者和消费者即可解耦，不再需要有任何联系，也不需要受对方的影响，只要保持使用一致的消息格式即可



































