# k8s

视频教程 https://www.bilibili.com/video/BV1w4411y7Go?spm_id_from=333.337.search-card.all.click

下载地址：https://pan.baidu.com/s/18M4WCXsfD0jDeM6Jq1lMDw ，提取码：x1sg



## 学习目标

基础概念： 什么是pod, 有哪些控制器类型， k8s的网络通讯模式，怎么构建k8s集群

资源清单： 掌握资源清单的语法， 编写pod ， 掌握pod的生命周期

pod控制器： 掌握各种控制器的特点以及使用，定义方式

服务发现：掌握SVC原理以及构建方式

存储：configMap, volume, secret, pv, 掌握多种存储类型的特点，并且能够在不同环境中选择合适的存储方案

调度器： 掌握调度器原理，能够根据要求把pod定义到想要的节点上运行

集群安全机制：认证，鉴权，访问控制原理及其流程  这个很难也很重要

HELM: 类似yum管理器

运维: 源码修改， cicd构建， 修改kubeadm 达到证书可用期限为10年， 构建高可用k8s



比较好的博客

http://victorfengming.gitee.io/kubernetes/1_Kubernetes%E7%AE%80%E4%BB%8B/#kubernetes%E7%AE%80%E4%BB%8B

http://bbigsun.gitee.io/kubernetes-study/#/README

https://gitee.com/moxi159753/LearningNotes/tree/master/K8S



## k8s 组件

Borg系统是k8s的前生， k8s用go语言编写

Borg架构

![image-20220504212140648](k8s.assets/image-20220504212140648.png)

k8s架构

![image-20220504212118915](k8s.assets/image-20220504212118915.png)

高可用集群副本数据最好是 >= 3 奇数个
	
api server：所有服务访问统一入口
srontrollerManager：维持副本期望数目
scheduler：：负责介绍任务，选择合适的节点进行分配任务
ctcd：键值对数据库  储存K8S集群所有重要信息（持久化）
kubelet：直接跟容器引擎交互实现容器的生命周期管理
kube-proxy：负责写入规则至 IPTABLES、IPVS 实现服务映射访问的
CoreDNS：可以为集群中的SVC创建一个域名IP的对应关系解析
Dashboard：给 K8S 集群提供一个 B/S 结构访问体系
Ingress Controller：官方只能实现四层代理，INGRESS 可以实现七层代理
Federation：提供一个可以跨集群中心多K8S统一管理功能
Prometheus：提供K8S集群的监控能力
ELK：提供 K8S 集群日志统一分析介入平台



# 1. Pod

