
## 为什么使用Kubernetes

> Kubernetes——让容器应用进入大规模工业生产。

**Kubernetes是容器编排系统的事实标准**

在单机上运行容器，无法发挥它的最大效能，只有形成集群，才能最大程度发挥容器的良好隔离、资源分配与编排管理的优势，而对于容器的编排管理，Swarm、Mesos和Kubernetes的大战已经基本宣告结束，Kubernetes成为了无可争议的赢家。

下面这张图是Kubernetes的架构图，其中显示了组件之间交互的接口CNI、CRI、OCI等，这些将Kubernetes与某款具体产品解耦，给用户最大的定制程度，使得Kubernetes有机会成为跨云的真正的云原生应用的操作系统。

![Kubernetes架构](../images/kubernetes-high-level-component-archtecture.jpg)


**云原生的核心目标**

![Cloud Native Core target](../images/cloud-native-core-target.jpg)

## Cloud Native

> DevOps——通向云原生的云梯

CNCF（云原生计算基金会）给出了云原生应用的三大特征：

- **容器化包装**：软件应用的进程应该包装在容器中独立运行。
- **动态管理**：通过集中式的编排调度系统来动态的管理和调度。
- **微服务化**：明确服务间的依赖，互相解耦。

云原生所需要的能力和特征。

![Cloud Native Features](https://jimmysong.io/kubernetes-handbook/images/cloud-native-architecutre-mindnode.jpg)


**使用Kubernetes构建云原生应用**

我们都是知道Heroku推出了适用于PaaS的[12 factor app](https://12factor.net/)的规范，包括如下要素：

1. 基准代码
2. 依赖管理
3. 配置
4. 后端服务
5. 构建，发布，运行
6. 无状态进程
7. 端口绑定
8. 并发
9. 易处理
10. 开发环境与线上环境等价
11. 日志作为事件流
12. 管理进程


- API声明管理
- 认证和授权
- 监控与告警

使用Kubernetes构建云原生架构：

![Building a Cloud Native Architecture with Kubernetes followed 12 factor app](../images/building-cloud-native-architecture-with-kubernetes.png)



## Open Source

Kubernetes调研方案选择。

![Kubernetes solutions](../images/20210207101715.jpg)
