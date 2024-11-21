
AI之旅实现的第一个功能是基于大模型的 vector embedding 进行语义搜索（semantic search）。


![](https://img2024.cnblogs.com/blog/35695/202411/35695-20241120115645119-795064269.png)


（图片来源：[kdnuggets.com](https://github.com)）


基于大模型实现的聊天机器人虽然能打字和你聊天，但大模型大字不识一个，它只识数（向量）与只会计算，它不会玩文字游戏，只会玩数字游戏。


任何一段文字，在大模型的眼里只是一个向量，这个向量是包含n个元素的浮点型数组（n对应大模型的n维空间），数组中的一组浮点数字表示的是大模型n维空间中的坐标，大模型n维空间可以理解为大模型在自己的「大脑」中对文字的语义进行建模的空间，大模型将对一段文字的理解映射到n维空间中的一个点。不顾多么长多么复杂的一段文字，都可以用一个点表示，这就是多维空间的神奇力量。这个向量，这个数组，这个坐标，这个点，就是大名鼎鼎的 vector embedding，大模型通过 vector embedding 将文字的语义嵌入到自己的n维空间，然后通过对向量的计算操作玩转数字，从而玩转文字。如果你想更多了解 vector embedding，推荐阅读 [Embeddings: What they are and why they matter](https://github.com) 。


![](https://img2024.cnblogs.com/blog/35695/202411/35695-20241120163650749-64390177.jpg)


![](https://img2024.cnblogs.com/blog/35695/202411/35695-20241120163718019-1716765116.jpg)


（图片来源：[simonwillison.net](https://github.com)）


由于有了 vector embedding，只要计算向量之间的距离，大模型就可以知道两段文字在语义上是否相似，语义搜索就是基于这个原理。


![](https://img2024.cnblogs.com/blog/35695/202411/35695-20241119123132522-2142166462.png)


（图片来源：[causewriter.ai](https://github.com)）


![](https://img2024.cnblogs.com/blog/35695/202411/35695-20241119125928642-844872515.jpg)


（图片来源：[datastax.com](https://github.com)）


我们要实现园子博文的语义搜索，只要使用一种大模型生成博文内容的 vector embedding 存入向量数据库（类似于传统搜索的建索引），然后搜索时将搜索文字通过大模型生成 vector embedding，通过 vector embedding 在向量数据库中查找语义相近的博文内容即可。


第一步准备工作是选择一种向量数据库并完成部署，我们选择的是开源向量数据库 [qdrant](https://github.com)，由于我们的应用都部署在 k8s 集群上，所以我们选择在 k8s 集群上部署 qdrant，qdrant 提供了 [helm chart](https://github.com):[veee加速器](https://youhaochi.com)，k8s 上的部署更便捷了。


先添加 qdrant 的 helm 仓库



```
helm repo add qdrant https://qdrant.github.io/qdrant-helm
helm repo update

```

然后编写 qdrant\-values.yaml 清单文件



```
config:
  service:
    enable_tls: false

persistence:
  storageClassName: nas-db
  size: 50G

snapshotPersistence:
  storageClassName: nas-db
  size: 50G

resources: 
  limits:
    cpu: 2
    memory: 4Gi
  requests:
     cpu: 1
     memory: 1Gi

```

需要注意的是，这是数据库，需要将数据文件保存在 nas 存储上，上面的 storageClassName 用于指定所使用的 nas 存储，nas\-db 是我们的 k8s 集群之前已经部署好的使用阿里云 nas 的 StorageClass。


通过下面的命令进行部署



```
helm upgrade -i qdrant qdrant/qdrant -f qdrant-values.yaml

```

然后查看部署是否成功



```
pod/qdrant-0               1/1          Running
service/qdrant             ClusterIP    10.99.106.66    6333/TCP,6334/TCP,6335/TCP 
service/qdrant-headless    ClusterIP    None            6333/TCP,6334/TCP,6335/TCP
statefulset.apps/qdrant    1/1          45h

```

pod 处于 Running 状态，说明部署成功了。


第一步准备工作完成了，接下来就是调用大模型（我们选用的是通义千问）把园子的博文内容生成 vector embedding 存入 qdrant 向量数据库，然后就可以实现语义搜索，这将是AI之旅下一篇博文的主要内容。


参考资料：


* [Embeddings: What they are and why they matter](https://github.com)
* [What is vector embedding?](https://github.com)
* [What are Vector Embeddings? Understanding the Foundation of Data Representation](https://github.com)
* [LLM Basics: Embedding Spaces \- Transformer Token Vectors Are Not Points in Space](https://github.com)
* [From Words to Vectors: Inside the LLM Transformer Architecture](https://github.com)


