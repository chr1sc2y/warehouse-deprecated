# Kubernetes

Kubernetes 是一个容器编排系统，简称 k8s，它在概念上主要分为资源对象和控制对象两种，资源对象有容器、应用、配置、网络、存储等，控制对象是用于管理资源对象而抽象出来的控制层，。

## 1 资源对象

### 1.1 容器

容器是最小的隔离单位，类似于一台虚拟机。

### 1.2 Pod

Pod 是 kubernetes 中部署的最小单位，也是集群中运行的进程，它用于封装容器、存储、网络 IP、运行策略登，kuberentes 直接管理 Pod 而不是容器。

Pod 被创建后会被 Kubernetes 调度到集群的 node 上，直到这个 Pod 进程被终止或 node 发生故障。

#### 1.2.1 管理容器

通常来说每个 Pod 用于封装单个容器，除非有多个耦合并且需要共享存储和网络的容器，这些容器共同成为一个service 单位，需要使用另一个 sidecar 容器来更新这些文件。

每个 Pod 都会被分配一个唯一的 IP，Pod 中的所有容器共享存储和存储网络空间（包括 IP 和 port），内部容器在与外部通信时需要分配网络资源（例如使用宿主机的端口映射）。

#### 1.2.2 Pod 中的容器

## 2 控制对象

### 2.1 ReplicaSet

在进行服务的部署时，一般会部署多个副本，以提高承载承能力，并防止单点故障影响服务的可用性。

ReplicaSet 就是多个副本的集合，它是一个控制器，当其中一个副本因为异常而被终止后，控制器会启动新的 pod 来满足副本数的需求。

一般我们通过 Deployment 来控制 ReplicaSet。

### 2.2 Deployment

Deployment 也是一个控制器，通过配置文件我们可以很方便地控制 pod 的部署、更新、回滚、扩展、收缩等等。下面是示例配置文件：

```yaml
```



获取配置：kubectl get deploy

### 2.3 Service

### 2.4 Namespace

Namespace 可以将不同的资源对象隔离开，不同的 Namespace 之间的资源是不共享的。







## 3 资源管理

#### 访问集群

为了访问集群，我们需要拥有访问它的凭证，并通过修改环境变量，修改默认配置参数，添加执行参数等方式使用凭证：

```shell
# 1
echo "export KUBECONFIG_SAVED=config" > ~/.bashrc
source ~/.bashrc

# 2
kubectl --kubeconfig=config
```





### Context

kubeconfig 文件中的 context 用于对参数分组，它包含 cluster, namespace, user 三个参数：

```yaml
contexts:
- context:
    cluster: c
    user: u
  name: n
```

对 context 的所有操作都在 `kubectl config` 下。

### 2. Namespace

- 获取集群中现有的 namespace `kubectl get ns`
- 集群中默认有 `default` 和 `kube-system` 两个 namespace
- 使用 `-n` 指定操作的 namespace，默认在 `default` 下执行，为整个集群提供服务的应用一般部署在 `kube-system` 下，例如在安装 kubernetes 集群时部署的 `kubedns`、`heapseter`、`EFK` 等
- 并不是所有的资源对象都有 namespace，`node` 和 `persistentVolume` 就不属于任何 namespace

#### 创建

1. 通过 yaml 文件：

   ```yaml
   # namespace.yaml
   apiVersion: v1
   kind: Namespace
   metadata:
       name: ns
   ```

   

   ```shell
   kubectl create -f namespace.yaml
   ```

2. 直接创建

   ```shell
   kubectl create namespace foo
   ```

#### 删除



























