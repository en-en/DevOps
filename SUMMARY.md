# 目录

## 前言

* [序言](README.md)

## 云原生

* [云原生（Cloud Native）的定义](cloud-native/cloud-native-definition.md)
* [云原生的设计](cloud-native/cloud-native-philosophy.md)
* [Kubernetes 与云原生应用概览](cloud-native/kubernetes-and-cloud-native-app-overview.md)
* [部署运维环境方案调研结果](cloud-native/from-kubernetes-to-cloud-native.md)

## 部署实践

* [在 CentOS 上部署 Kubernetes 集群](practice/install-kubernetes-on-centos.md)
  * [创建 TLS 证书和秘钥](practice/create-tls-and-secret-key.md)
  * [创建 kubeconfig 文件](practice/create-kubeconfig.md)
  * [创建高可用 etcd 集群](practice/etcd-cluster-installation.md)
  * [安装 kubectl 命令行工具](practice/kubectl-installation.md)
  * [部署 master 节点](practice/master-installation.md)
  * [安装 flannel 网络插件](practice/flannel-installation.md)
  * [部署 node 节点](practice/node-installation.md)
  * [安装 kubedns 插件](practice/kubedns-addon-installation.md)
  * [安装 dashboard 插件](practice/dashboard-addon-installation.md)
  * [安装 EFK 插件](practice/efk-addon-installation.md)
* [服务发现与负载均衡](practice/service-discovery-and-loadbalancing.md)
  * [安装 Traefik ingress](practice/traefik-ingress-installation.md)
  * [安装 Nginx ingress](practice/nginx-ingress-installation.md)
* [存储管理](practice/storage.md)
  * [NFS](practice/nfs.md)
    * [利用 NFS 动态提供 Kubernetes 后端存储卷](practice/using-nfs-for-persistent-storage.md)
* [集群与应用监控](practice/monitoring.md)
  * [Prometheus](practice/prometheus.md)
    * [使用 Prometheus 监控 kubernetes 集群](practice/using-prometheus-to-monitor-kuberentes-cluster.md)
* [分布式跟踪](practice/distributed-tracing.md)
  * [OpenTracing](practice/opentracing.md)
* [持续集成与发布](practice/ci-cd.md)
  * [使用 Jenkins 进行持续集成与发布](practice/jenkins-ci-cd.md)
* [数据库高可用](database/info.md)
  * [mysql](database/mysql.md)
  * [postgre](database/postgre.md)
  * [redis](database/redis.md)
* [文件存储](filestore/filestoreinfo.md)

## 运维工作描述

* [运维环境](devops/environment.md)
* [运维职责](devops/duty.md)

## 云原生应用部署运维

* [基础手册](cloud-native/handbook.md)

