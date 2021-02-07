## 硬件环境

| 主机名称 | 操作系统 | 系统配置 | 备注
| :--- | --- |  --- | --- | --- |
| k8s-master1 | CentOS-7 | 8核16g |  |
| k8s-master2 | CentOS-7 | 8核16g |  |
| k8s-master3 | CentOS-7 | 8核16g |  |
| k8s-node1 | CentOS-7 | 8核16g |  |
| k8s-node2 | CentOS-7 | 8核16g |  |
| commonNode | CentOS-7 | 8核16g | 提供镜像仓库，nuget私有包，gitlab  |

## 运维工作
  
![Cloud Native Core target](../images/handbook.png)
  
  **内网操作**

 1. 运维人员从线下渠道获取项目发布脚本和镜像
 2. 运维人员将镜像推送到镜像仓库中
 3. 运维人员通过运维工具(jenkins)执行脚本
 4. 部署完成

  **外网操作**
 1. 运维人员通过运维工具(jenkins)运执行脚本
 2. 部署完成