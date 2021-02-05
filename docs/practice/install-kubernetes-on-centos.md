# 在CentOS上部署kubernetes集群

> 基于kubenetes1.8版本编写的


# 集群硬件要求


| 主机名称 | 操作系统 | 系统配置 | 备注
| :--- | --- |  --- | --- | --- |
| k8s-master1 | CentOS-7 | 8核16g |  |
| k8s-master2 | CentOS-7 | 8核16g |  |
| k8s-master3 | CentOS-7 | 8核16g |  |
| k8s-node1 | CentOS-7 | 8核16g |  |
| k8s-node2 | CentOS-7 | 8核16g |  |
| commonNode | CentOS-7 | 8核16g | 提供镜像仓库，nuget私有包，gitlab  |

## 集群详情

+ OS：CentOS Linux release 7.4.1708 (Core) 3.10.0-693.11.6.el7.x86_64
+ Kubernetes 1.8.3（最低的版本要求是1.6）
+ Docker：建议使用 Docker CE，**请勿使用 docker-1.13.1-84.git07f3374.el7.centos.x86_64 版本**，
+ Etcd 3.1.5
+ Flannel 0.7.1 vxlan或者host-gw 网络
+ TLS 认证通信 (所有组件，如 etcd、kubernetes master 和 node)
+ RBAC 授权
+ kubelet TLS BootStrapping
+ kubedns、dashboard、heapster(influxdb、grafana)、EFK(elasticsearch、fluentd、kibana) 集群插件
+ 私有docker镜像仓库[harbor](https://github.com/vmware/harbor)

## 步骤介绍

1. [创建 TLS 证书和秘钥](create-tls-and-secret-key.md)
2. [创建kubeconfig 文件](create-kubeconfig.md)
3. [创建高可用etcd集群](etcd-cluster-installation.md)
4. [安装kubectl命令行工具](kubectl-installation.md)
5. [部署master节点](master-installation.md)
6. [安装flannel网络插件](flannel-installation.md)
7. [部署node节点](node-installation.md)
8. [安装kubedns插件](kubedns-addon-installation.md)
9. [安装dashboard插件](dashboard-addon-installation.md)
10. [安装heapster插件](heapster-addon-installation.md)
11. [安装EFK插件](efk-addon-installation.md)
