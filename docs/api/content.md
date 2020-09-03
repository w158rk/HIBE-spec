## REST API

本节用于描述各类节点功能的http接口和使用方式

### 服务端

### 客户端

#### 获取私钥`/sk`

****<未实现>****

如果`user.local_sk`存在，返回base64编码后的私钥内容，否则，返回空。

#### 请求私钥`/sk/request`

如果`user.local_sk`存在，返回base64编码后的私钥内容；否则，向父节点请求私钥，保存到`user.local_sk`并返回base64编码后的私钥内容。
