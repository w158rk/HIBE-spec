## 网络结构


### 网络拓扑

<img src="../image/topo.png" style="display: block; margin: 0 auto; padding: 1em 0; width: 100%; max-width: 500px;" />

上图是一个单一根节点HIBE系统的拓扑结构，其中，根KGC和KGC1同时具有密钥生成中心和证书管理结构两种结构的全部功能，而KGC2和KGC3只具有私钥分发功能，而不具有证书分发功能，因而这两个KGC的子节点中只能有终端节点。

### 密钥生成中心 

<img src="../image/kgc.png" style="display: block; margin: 0 auto; padding: 1em 0; width: 100%; max-width: 200px;" />

密钥生成中心是一个网络域的管理者，负责向网络域中的其他节点分发私钥，密钥生成中心需要向上级证书管理机构请求私钥分发证书，同时需要维护标识库和撤销列表，撤销列表用于对该密钥生成中心分发的私钥进行撤销。

### 证书管理机构

<img src="../image/ca.png" style="display: block; margin: 0 auto; padding: 1em 0; width: 100%; max-width: 200px;" />

证书管理机构可以向密钥生成中心分发证书。在当前版本中，一个证书管理机构必须首先是一个密钥生成中心。