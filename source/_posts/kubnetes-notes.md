---
title: kubnetes_notes
date: 2018-09-29 10:06:11
tags:
- arch
---


整理一下k8s 相关的理解以及实践， 并理一下下一步深入的方向

<!-- more -->

# K8s 是做什么的， 我们的使用

## 是什么
微服务 image 编排。 image 编排维护成为了问题， mesos k8s。 k8s 比较流行一些。 

## 目前的好处
微服务， 容易扩容， 容易部署。 kubctl 一个命令就搞定了。 

每个服务都是无服务的， 另外一种情况是一个集群。 

## 不利的地方
维护的成本， 出了问题的解决。 日志的处理。 监控。 

## 项目中的使用。  
作为主要的业务应用。 每一个。 
- 安装的话， 手工也可以装。 但是我们用rancher 来安装的。 但是定制一些东西。 确实容易一些。 
- 我们现在12个节点， 最多20个节点的node。 微服务大约是在10个左右。 
- 前台是 	-ingress -harproxy 做的负载均衡。 

 比如我们coper的 来处理设备的， 我们放了三个。 每个verst 可以做全局锁。 对一个设备只能同时做这个一个事情。 三个是个集群， 有一个share data 的东西。 hazelcast 等。 

# 基本结构和原理

## 基本组件
包含哪些核心部分。 起什么作用。 

- pod， 
	pod 最小的运行单元。 里面更小的component是 container （一个pod 上一个docker image 运行。 ）， 每个 跑一个proces谁
	可以单独部署 ， 但是一般不那么用， 用deployment 来
	
- deploymnet
	相当于一个魔板， 以前用过replicaset 后来的版本其实不太用了。 功能有一些强化。 

- service
	答：为pod稳定地提供服务发现和负载均衡的能力。

- Ingress
	对外的 

- PV PVC 
	答：用于存储持久化数据，不同类型的Volume有不同的生命周期。

-  configmap
如何传变量。 - env

## - 基本原理
- 状态是保存在 etcd - raft。 -- 看过论文。 notes。 一个leader。 
		选举； 如何决定。 计数。 
- 用namespace 来分别命名空间。 我们kube system， 我们定义了cloud
- 可以用label 来制定哪些node 做哪些事情

- service
- 如何schedule 分配 这是挺重要的 
	schedule 是在master 上， 然后他要决定哪个node 来做这个事情。	
	It’s not responsible for actually running the pod – that’s the kubelet’s job. 
调度的过程分两步，首先是做筛选(predict)，其次是排序(priorities)。
预选：根据配置的Predicates Policies（默认为DefaultProvider中定义的default predicates policies集合）过滤掉那些不满足这些Policies的的Nodes，剩下的Nodes就作为优选的输入。

- service 的问题
	Pod的IP是在docker0网段动态分配的 会变所以service 
	Kubernetes service有以下三种类型：
    - ClusterIP:提供一个集群内部的虚拟IP以供Pod访问。
    - NodePort:在每个Node上打开一个端口以供外部访问。
    - LoadBalancer:通过外部的负载均衡器来访问。

	port和nodePort都是service的端口，前者暴露给集群内客户访问服务，后者暴露给集群外客户访问服务。从这两个端口到来的数据都需要经过反向代理kube-proxy流入后端pod的targetPod，从而到达pod上的容器内。

#  如何安装， 维护， 监控， 日志

- 安装
 -  salt pilar --》
 -  Grains是saltstack的组件，用于收集salt-minion在启动时候的信息，又称为静态信息。
 -  Pillar : 用于存储master分配给minion的数据信息。 top.sls jinjia 处理的

- 如何监控
	两个层面的。 
    第一个是node level- nagios
	第二个是自己的监控， 我们用pu。 

- 如何log
    fluentbit？如何安装以及工作的。  
    也是pod， 然后可以收集， 配置到elastic search 去的。 

- 维护
    加host。 k8s 自己会来做这个

- coreDNS 就是自己寻找的。 - 这个


# 遇到哪些问题 排查  

## 网络问题
-- 少。 
## 状态显示不对 
    有的时候状态不及时。 要等于下
	其实并不是。 不清楚。 你主要找image 下载了没有。 
	是不是log 有没问题。 
## 部署
    -- 就是deploy image。 registry的加密。 证书的处理。 

## 资源不足的问题。 
	- 不是太深刻， 但是发现多了就申请了新的node。 5-》11
	加host。 k8s 自己会来做这个

# TODO
1. service
clster ip coreDNS
2. 实践 关于命令的具体分析
3. 如何做scheduler的呢。 

