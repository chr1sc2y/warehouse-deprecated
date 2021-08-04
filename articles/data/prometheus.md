# Prometheus

Prometheus 是一套开源的数据采集与监控框架，可以做后台服务器的监控告警，此处用来采集被测服务的性能指标数据，包括CPU占用比率、内存消耗、网络IO等。Promethus框架如下图所示，

## 1 数据模型

Prometheus 存储的是[时序数据](https://en.wikipedia.org/wiki/Time_series), 即按照相同时序(相同的名字和标签)，以时间维度存储连续的数据的集合。

### 时序索引

时序(time series) 是由名字(Metric)，以及一组 key/value 标签定义的，具有相同的名字以及标签属于相同时序。

时序的名字由 ASCII 字符，数字，下划线，以及冒号组成，它必须满足正则表达式 `[a-zA-Z_:][a-zA-Z0-9_:]*`, 其名字应该具有语义化，一般表示一个可以度量的指标，例如: `http_requests_total`, 可以表示 http 请求的总数。

时序的标签可以使 Prometheus 的数据更加丰富，能够区分具体不同的实例，例如 `http_requests_total{method="POST"}` 可以表示所有 http 中的 POST 请求。

标签名称由 ASCII 字符，数字，以及下划线组成， 其中 `__` 开头属于 Prometheus 保留，标签的值可以是任何 Unicode 字符，支持中文。



### 时序类型

1. Counter

Counter 表示收集的数据只增不减，常用于记录服务请求总量、错误总数等，它会在程序重启的时候会被重设为 0。

例如 Prometheus server 中 `http_requests_total`, 表示 Prometheus 处理的 http 请求总数，我们可以使用 `delta`, 很容易得到任意区间数据的增量，这个会在 PromQL 一节中细讲。

2. Gauge

Gauge 表示搜集的数据是一个瞬时的值，与时间没有关系，可以任意变高变低，往往可以用来记录内存使用率、磁盘使用率等。

例如 Prometheus server 中 `go_goroutines`, 表示 Prometheus 当前 goroutines 的数量。

3. Histogram

Histogram 由 `<basename>_bucket{le="<upper inclusive bound>"}`，`<basename>_bucket{le="+Inf"}`, `<basename>_sum`，`<basename>_count` 组成，主要用于表示一段时间范围内对数据进行采样（通常是请求持续时间或响应大小），并能够对其指定区间以及总数进行统计，通常它采集的数据展示为直方图。

例如 Prometheus server 中 `prometheus_local_storage_series_chunks_persisted`, 表示 Prometheus 中每个时序需要存储的 chunks 数量，我们可以用它计算待持久化的数据的分位数。

4. Summary

Summary 和 Histogram 类似，由 `<basename>{quantile="<φ>"}`，`<basename>_sum`，`<basename>_count` 组成，主要用于表示一段时间内数据采样结果（通常是请求持续时间或响应大小），它直接存储了 quantile 数据，而不是根据统计区间计算出来的。

例如 Prometheus server 中 `prometheus_target_interval_length_seconds`。



## 2 数据采集

Prometheus server 的数据采集基于 Pull 模型，其工作流程大致为 Prometheus server 会定期地从 exporter 上通过 HTTP 接口获取 metrics 结构的数据，并进行存储，如果 Prometheus Server 与 Exporter 不能够直接进行通信，那么我们可以将 exporter 上的数据推送到 Pushgateway 上，并让 Prometheus Server 从 PushGateway 上获取数据。

### 术语

#### Exporter

所有向 Prometheus server 提供数据的程序都可以被称为 exporter，Prometheus server 会周期性地从 exporter 提供的HTTP 服务 URL 拉取数据，我们只需要在 Prometheus server 的配置文件 /etc/prometheus/prometheus.yml 中添加一个target，并重启服务即可定位到 exporter 并拉取数据。

#### Instance

任意一个独立的数据源 target 都可以被称为 Instance 实例，它是用来提供数据的最小单位。

#### Job

包含相同类型的实例的集合叫做 Job，例如在 k8s 集群上被复制出的可弹性伸缩的一组 pod 中的相同进程。



### Pushgateway

由于 Prometheus server 采用 pull 模式获取数据，如果 Prometheus server 和 exporter 由于不在一个子网环境或由于防火墙原因导致两者无法直接通信，或在我们需要将多个 exporter 的数据汇总到一起时，我们可以主动将数据推送到 Pushgateway 上，间接地进行数据采集；但它的弊端也很明显，那就是单点故障，如果唯一的 Pushgateway 出现不可用的情况，那么所有的数据都无法被 Prometheus server 获取。

#### 安装

Pushgateway 不需要任何配置，安装后即可使用。

##### docker 安装

```
docker pull prom/pushgateway

docker run -d -p 9091:9091 prom/pushgateway
```

#### 配置

安装好 Pushgateway 之后，需要在 Prometheus server 的配置中添加 Pushgateway：

```shell
docker exec -it --user root f257794e5e3d sh
```



#### 使用

##### 数据持久化

默认情况下，Pushgateway 将所有数据存储在内存中，因此一旦 Pushagateway 服务因为故障而停止运行，那么所有尚未被 Prometheus server 获取的数据都将会丢失，为此可以在启动 Pushgateway 服务时通过指定 `persistence.file` 参数将数据持久化：

```javascript
pushgateway --persistence.file="/tmp/pushgateway_persist"
```

默认情况下，文件每五分钟持久化写入一次，我们可以修改 `persistence.interval` 参数来进行调整。

#### API



##### 删除

删除特定 job 和 instance 的所有数据：

```shell
curl -X DELETE http://pushgateway.example.org:9091/metrics/job/some_job/instance/some_instance
```

删除特定 job ，且 instance="" 下的所有数据，注意这不会删除其他 instance 的数据：

```shell
curl -X DELETE http://pushgateway.example.org:9091/metrics/job/some_job
```



## 3 PromQL

PromQL (Prometheus Query Language) 是专门用于查询 Prometheus 时序数据的 [DSL](https://en.wikipedia.org/wiki/Domain-specific_language) 语言，在数据可视化，告警，以及使用 Grafana 查询 Prometheus 数据时都会用到它。

PromQL 是Promethus内置的查询工具，Grafana通过PromQL 的 API 查询采集的数据并展示在界面上。

### 聚合操作

PromQL 包含了内置的聚合操作符，可以将使用瞬时表达式返回的瞬时向量进行聚合并形成一个新的时间序列。

- `sum` 求和
- `min` 最小值
- `max` 最大值
- `avg` 平均值
- `stddev` 标准差
- `stdvar` 标准方差
- `count` 计数
- `count_values` 对value进行计数
- `bottomk` 后 n 条时序
- `topk` 前n条时序
- `quantile` 分位数

### 范围查询

类似于 `http_requests_total` 的表达式在返回时间序列查询结果时只会包含该时间序列中最新的一个样本值，这样的返回结果称为**瞬时向量**，而相应的这样的表达式称为**瞬时向量表达式**；如果想要查询一段时间范围内的样本数据则需要使用**区间向量表达式**，通过区间向量表达式查询到的结果称为**区间向量**，它与前者的区别在于需要指定查询时间的范围，时间范围通过 `[]` 进行定义，例如 `http_requests_total{}[1m]` 表示最近 1 分钟之内的所有 http 请求的样本数据。



## 4 告警

### 告警规则

示例：

```yaml
groups:
- name: example
  rules:
  - alert: HighErrorRate
    expr: job:request_latency_seconds:mean5m{job="myjob"} > 0.5
    for: 10m
    labels:
      severity: page
    annotations:
      summary: High request latency
      description: description info
```

告警规则主要包含以下部分：

- groups：一般将一组相关的规则定义在同一个 group 下，每个 group 有一个共同的 name，其中可以定义多条 rules
- alert：告警规则的名称。
- expr：基于PromQL表达式告警触发条件，用于计算是否有时间序列满足该条件。
- for：评估等待时间，可选参数。用于表示只有当触发条件持续一段时间后才发送告警。在等待期间新产生告警的状态为pending。
- labels：自定义标签，允许用户指定要附加到告警上的一组附加标签。
- annotations：用于指定一组附加信息，比如用于描述告警详细信息的文字等，annotations的内容在告警产生时会一同作为参数发送到Alertmanager。



## 5 常见问题

- expected integer as timestamp, got ""
  - 去掉数据行末尾的空格
