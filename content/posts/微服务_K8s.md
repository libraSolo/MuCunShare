---
date : 2025-02-28T16:21:57+08:00
draft : false
title : '微服务_K8S'
author : '木村凉太'
categories : ['微服务']
hiddenFromHomePage : true 
---

# K8s 基础
## 介绍
Kubernetes（简称 K8S）是一个开源的容器编排平台，用于自动化部署、扩展和管理容器化应用。
### 功能
1. 服务发现和负载均衡
    * 可以通过 DNS 名称或 IP 地址暴露容器的访问方式。
    * 可以在同组容器内分发负载以实现负载均衡。
2. 自愈功能
    * 重启已经停机的容器
    * 替换、kill 那些不满足自定义健康检查条件的容器
    * 在容器就绪之前，避免调用者发现该容器
3. 储存编排
    * 挂载本地或云存储（如 AWS EBS、NFS）到容器，支持持久化数据。
4. 自动发布和回滚
    * 可以在 K8s 中声明您期望应用程序容器应该达到的状态，K8s 将以合适的速率调整容器的实际状态，并逐步达到最终期望的结果。
5. 密钥及配置管理
    * 储存敏感信息，支持动态更新。

## 核心组件
### Master组件
1. master 组件负责集群中的全局决策
2. master 组件探测并响应集群事件

#### API Server(kube-apiserver)
* 作用：集群的“大脑”，提供 K8s API 的入口。
* 功能：
  1. 处理所有 REST 请求。
  2. 验证请求合法性，协调其他组件的数据交互。
* 特点：无状态设计，支持水平扩展。

#### ETCD
* 作用：分布式键值数据库，存储集群的所有状态数据（如 Pod、Service、节点信息）。
* 功能：
  1. 保证数据强一致性和高可用。
  2. 通过 API Server 处理数据，避免直接操作。

#### Scheduler (kube-scheduler)
* 作用： 决定 Pod 运行在哪个节点。
* 调度策略：
  1. 资源需求（CPU、内存）。
  2. 亲和性与反亲和性的约定。
  3. 节点状态（如磁盘压力、网络延迟）。
  4. 数据本地化要求
  5. 工作负载间的相互作用

#### Controller Manager (kube-controller-manager)
* 作用：运行一系列控制器，确保集群状态与期望一致。
* 核心控制器：
  1. **Node Controller**：监控节点状态（如宕机时标记为不可用）。
  2. **Deployment Controller**：管理 Deployment 的副本数和滚动更新。
  3. **Service Controller**：为 Service 创建负载均衡器（云厂商适配）。
  4. **Endpoint Controller**：维护 Service 与 Pod 的映射关系。

#### Cloud Controller Manager (cloud-controller-manager)
**K8s 1.6 引入特性，alpha 阶段。**
* 作用：对接云厂商特定功能（如 AWS、Azure 的负载均衡器、存储卷）。
* 适用场景：在公有云环境中替代部分 Controller Manager 功能。

### Node 组件
Node 组件运行在每一个节点上（包括 master 节点和 worker 节点），负责维护运行中的 Pod 并提供 Kubernetes 运行时环境。

#### kubelet
* 作用：节点上的“代理”，负责管理本节点的 Pod 生命周期。
* 功能：
  1. 接收 API Server 下发的 Pod 定义，启动/停止容器。
  2. 定期上报节点状态（资源状态，Pod 健康状态）。

#### kube-proxy
* 作用：维护节点的网络规则（如 iptables/IPVS），实现 Service 的负载均衡。
* 功能：
  1. 将 Server 的虚拟 IP 映射到后端 Pod。
  2. 支持流量转发和会话保持。

#### Container Runtime(容器运行时)
* 作用：实际运行容器的底层引擎。
* 常见实现：
  1. Docker(已逐步被代替)
  2. containerd(轻量级，K8S 推荐)
  3. CRI-O

### Addons 组件(插件)
Addons 使用 Kubernetes 资源（DaemonSet、Deployment等）实现集群的功能特性。由于他们提供集群级别的功能特性，addons使用到的Kubernetes资源都放置在 kube-system 名称空间下。

#### DNS
* 作用：为集群提供 DNS 服务，解析 Service 和 Pod 的域名。

#### Web UI(Dashboard)
* 作用：Web 管理界面，可视化查看和管理集群资源。

#### Kuboard
* 作用：Kuboard 是一款基于Kubernetes的微服务管理界面，相较于 Dashboard，Kuboard 强调：
  1. 无需手动修改 Yaml 文件。
  2. 上下文相关监控。

