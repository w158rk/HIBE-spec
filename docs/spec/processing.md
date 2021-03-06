
## 协议流程

### 节点加入

节点通过向某一个密钥生成中心请求用户私钥加入网络系统，协议流程如图所示：

<img src="../image/sk-request.png" style="display: block; margin: 0 auto; padding: 1em 0; width: 100%; max-width: 500px;" />

1. 待加入节点将用户标识、临时密钥和私钥恢复秘密发送给密钥生成中心，会话消息使用密钥生成中心的标识和系统参数进行加密。
2. 密钥生成中心检查ID的合法性（规则由密钥生成中心制定，如ID不可重复等），若同意私钥生成请求，则使用该生成中心的系统参数和待加入节点的标识生成私钥，使用临时密钥加密后发送给待加入节点，并将待加入节点标识和私钥恢复秘密添加到标识库。
3. 待加入节点发送Finish消息到密钥生成中心，节点加入过程结束。

#### PrivateKeyRequest 

| 域 | 值 |
| --- | --- |
| client_id | 客户端标识 |
| type | SK_REQUEST |
| key | 临时密钥 |
| secret | 私钥恢复秘密 |

服务器端在收到SK_REQUEST_SEC(私钥请求)类型的数据包后执行以下流程：

1. 获取密文cipher后，执行ibe_decrypt（mode="admin", c=cipher）操作，对密文进行解密，得到client_id,sm4_key,secret。

2. 执行bytes2str(client_id)，bytes2str(secret)操作，将client_id,secret转换为字符串类型。

3. 执行upload操作，将client_id,secret上传到区块链。

4. 执行str2bytes(client_id)操作，将client_id转换为字节类型。

5. 执行ibe_extract（mode="admin", c_id=client_id）操作，根据client_id为用户分配私钥。

6. 执行make_sk_respond_key_plain(client_sk, client_id, server_id)操作，将客户端私钥、客户端标识、服务器端标识构建为一个包。

7. 将包转换为字节类型并且执行sm4_enc(key=sm4_key, m=plain_text)操作，对明文进行加密。

8. 执行make_sk_respond_key_sec(cipher=cipher)操作，将密文构建为一个包。

9. 向客户端发送密文包。
    
#### PrivateKeyRespond 

| 域 | 值 |
| --- | --- |
| type | SK_RESPOND |
| client_id | 客户端标识 |
| server_id | 服务器标识 |
| private_key | 私钥 |

客户端在收到SK_RESPOND_SEC(私钥响应)类型的数据包后执行以下流程：

1. 获取密文cipher = packet.vals[0]。

2. 执行m = user.sm4_dec(user.sm4_key， c=cipher)操作，使用临时密钥对密文进行解密得到明文m。

3. 从明文m中获取private_key。

### 节点退出/私钥撤销 

若某个节点需要退出系统
1. 发送退出的请求给自己的父节点
2. 父节点收到该请求后，对该子节点退出后的域进行参数更新（包括重新生成系统参数、为子节点重新生成私钥、重新生成自己的管理节点证书及撤销旧的证书信息），具体见参数更新过程
3. 父节点更新完成后返回ACK消息给该需要退出系统节点，即完成节点退出

### 网络域生成 

<img src="../image/domain-request.png" style="display: block; margin: 0 auto; padding: 1em 0; width: 100%; max-width: 600px;" />

1. 发起节点将发起节点标识、发起节点系统参数发送给父节点，如果没有父节点，则报错；如果有父节点，发起节点发送的地址和端口赋上父节点的地址和端口，发起节点使用的主公钥模式为admin，使用local私钥对消息进行签名，发送内容为字节流形式的包。
2. 父节点收到网络包后，取出包中的内容，包含携带的有效数据和签名，之后父节点检查发起节点签名的合法性，若检查不合法，则报错；若检查合法，则同意发起节点的网络域生成请求，父节点将发起节点标识和发起节点的系统参数进行格式化打包，使用admin私钥对消息进行签名，将格式化后的信息和签名上传到区块链，发送内容为字节流形式的包。
3. 父节点向发起节点发送网络域生成结束消息（代码中pass掉了），发起节点收到结束消息的网络包后输出网络域生成结束。

#### DomainRequest 

| 域 | 值 |
| --- | --- |
| type | DOMAIN_REQUEST |
| user_id | 发起用户标识 |
| mpk | 发起用户系统参数 |


父节点在收到DOMAIN_REQUEST_SEC(网络域生成请求）类型的数据包后执行以下流程：
1. 获取信息payload
2. 获取签名sig
3. 将获取的信息解密然后转化成int类型
4. 获取发起用户标识user_id
5. 检查发起节点签名的合法性
6. 如果签名不合法，则报错
7. 如果签名合法，则对消息用admin私钥对消息进行签名


#### DomainSubmit 

| 域 | 值 |
| --- | --- |
| type | DOMAIN_SUBMIT |
| user_id | 发起用户标识 |
| mpk | 发起用户系统参数 |

父节点在检查签名合法后执行以下流程：
1. 将消息和签名进行打包
2. 将打包的消息上传到区块链
3. 向发起节点发送网络域生成结束消息



#### DomainFinish
| 域 | 值 |
| --- | --- |
| type | DOMAIN_FINISH |


### 网络域参数更新

1. 管理节点为更新后的管理域生成新的参数信息${s_{new}}P$。

   ​	此时管理节点会产生一个数据包，该数据包的结构如下：

   ​                   <img src="../image/domain_update_request_plain.png" style="zoom:50%;" />

   ​	然后将该数据包作为载荷数据，并对该载荷数据进行签名，然后生成一个新的数据包，该数据包的结构如下：  

   ​                  <img src="../image/domain_update_request_sec.png" style="zoom:50%;" />

   ​	上述数据包作为新的管理域参数信息。

2. 管理节点发送参数更新的请求，并将新的管理域参数信息${s_{new}}P$发送给自己的父节点。

3. 父节点收到由下层发来的更新请求，若同意该请求，则为该更新的管理节点生成新的参数信息。

   ​	父节点对新收到的的参数信息进行签名验证，若验证通过，此时，父节点生成一个数据包，该数据包的结构如下：

   ​	               <img src="../image/domain_request_plain.png" style="zoom:50%;" />

   ​	然后将该数据包作为载荷数据，并对该载荷数据进行签名，然后生成一个新的数据包，该数据包的结构如下：

   ​                   <img src="../image/data.png" style="zoom:50%;" />

   ​	最后将该数据包上传到区块链上。

4. 网络域参数更新结束。

   ​	管理节点生成**Domain generation finished**消息，父节点生成**insert an entry to the database**消息，至此网络域参数更新结束。

### 网络域退出/证书撤销 

### 安全密钥协商/安全信道建立

我们使用TLS握手协议，将其中发送证书的步骤修改为发送节点所在域的系统参数，单方验证通信流程如下图所示：

<img src="../image/handshake.png" style="display: block; margin: 0 auto; padding: 1em 0; width: 100%; max-width: 600px;" />

在收到上述PublicParameter网络消息之后，验证者需要对收到的系统参数进行验证，一个典型的验证参数证书链如下图所示：

<img src="../image/cert-link.png" style="display: block; margin: 0 auto; padding: 1em 0; width: 100%; max-width: 600px;" />

图中所示的三类证书从不同的密钥分发中心节点获得，其中，根证书是该系统根节点的证书，其系统参数为全局参数，任意用户可以使用该参数和根节点ID验证根证书签名，证书分发证书被颁发给拥有证书颁发权限的密钥分发中心，私钥分发证书被颁发给拥有私钥分发权限的密钥分发中心。与PKI相比，该证书链比PKI的证书链节省一个终端公钥证书。如图所示需要验证三个证书的通信终端处于网络拓扑的第四层，PKI中第四层的节点需要进行四个证书的验证。

除了上述证书的验证，用户还需要分别访问各个密钥分发中心/证书分发机构的撤销列表，以确认通信对手的标识不在相应密钥分发中心的撤销列表上，以及证书链上的证书不在颁发者的撤销列表中。





