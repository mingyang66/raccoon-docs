Telepresence是一个开源工具，用于在本地开发环境中调试运行在Kubernetes集群中的服务。Kubeconfig文件是Kubernetes用来配置集群访问权限的文件。在使用Telepresence时，确保你有一个有效的kubeconfig文件，并且Telepresence能够读取它。

### 安装配置：

- 安装Telepresence：通过官方文档安装指南[https://www.telepresence.io/docs/latest/quick-start/](https://www.telepresence.io/docs/latest/quick-start/)

- 将k8s集群中的kubeconfig文件下载到本地，复制到指定目录，如：macbook pro到~/.kube/config文件中，并修改server地址为集群地址

- kubconfig文档介绍[https://kubernetes.io/zh-cn/docs/tasks/access-application-cluster/configure-access-multiple-clusters/](https://kubernetes.io/zh-cn/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

- 集群之间进行切换：

```sh
kubectl config use-context
```

- 检查kubectl是否安装命令：

```sh
kubectl version --client
```



### 客户端命令：

  ```sh
  https://www.getambassador.io/docs/telepresence-oss/latest/reference/client
  ```

  