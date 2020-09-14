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
| user_id | 用户标识 |
| type | SK_REQUEST |
| key | 临时密钥 |
| secret | 私钥恢复秘密 |

#### PrivateKeyRespond 

| 域 | 值 |
| --- | --- |
| type | SK_RESPOND |
| user_id | 用户标识 |
| server_id | 服务器标识 |
| private_key | 私钥 |

### 节点退出/私钥撤销 

### 网络域生成 

<img src="../image/domain-request.png" style="display: block; margin: 0 auto; padding: 1em 0; width: 100%; max-width: 600px;" />

#### DomainRequest 

| 域 | 值 |
| --- | --- |
| type | DOMAIN_REQUEST |
| user_id | 发起用户标识 |
| mpk | 发起用户系统参数 |

#### DomainSubmit 

| 域 | 值 |
| --- | --- |
| type | DOMAIN_SUBMIT |
| user_id | 发起用户标识 |
| mpk | 发起用户系统参数 |

#### DomainCommit 

| 域 | 值 |
| --- | --- |
| type | DOMAIN_COMMIT |
| user_id | 发起用户标识 |
| mpk | 发起用户系统参数 |

#### DomainFinish
| 域 | 值 |
| --- | --- |
| type | DOMAIN_FINISH |


### 网络域参数更新

### 网络域退出/证书撤销 

### 安全密钥协商/安全信道建立

我们使用TLS握手协议，将其中发送证书的步骤修改为发送节点所在域的系统参数，单方验证通信流程如下图所示：

<img src="../image/handshake.png" style="display: block; margin: 0 auto; padding: 1em 0; width: 100%; max-width: 600px;" />

在收到上述PublicParameter网络消息之后，验证者需要对收到的系统参数进行验证，一个典型的验证参数证书链如下图所示：

<img src="../image/cert-link.png" style="display: block; margin: 0 auto; padding: 1em 0; width: 100%; max-width: 600px;" />

图中所示的三类证书从不同的密钥分发中心节点获得，其中，根证书是该系统根节点的证书，其系统参数为全局参数，任意用户可以使用该参数和根节点ID验证根证书签名，证书分发证书被颁发给拥有证书颁发权限的密钥分发中心，私钥分发证书被颁发给拥有私钥分发权限的密钥分发中心。与PKI相比，该证书链比PKI的证书链节省一个终端公钥证书。如图所示需要验证三个证书的通信终端处于网络拓扑的第四层，PKI中第四层的节点需要进行四个证书的验证。

除了上述证书的验证，用户还需要分别访问各个密钥分发中心/证书分发机构的撤销列表，以确认通信对手的标识不在相应密钥分发中心的撤销列表上，以及证书链上的证书不在颁发者的撤销列表中。