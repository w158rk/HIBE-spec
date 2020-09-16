## 系统初始化

* 功能：多个节点使用秘密共享协商系统主私钥并分配相关函数。
### 节点文件结构

```shell
    *
    |
    +--- init_server.json # 节点配置文件
    |
    +--- top_user_list.json # 顶级用户列表
    |
    +--- python #指向/src/python的软链接
```

### `init_server.json`实例

****certificate_file现在变成啥****

```json
    {
        "log" : "server.log",
        "top_user_list": "./top_user_list.json",

        "global_mpk_file" : "mpk-global.conf",
        "global_sk_file" : "sk-global.conf",
        "admin_mpk_file" : "mpk-admin.conf",
        "admin_msk_file" : "msk-admin.conf",
        "admin_sk_file" : "sk-admin.conf",
        "local_certificate_file": "local-certificate.conf",
        "admin_certificate_file": "admin-certificate.conf",

        "id" : "Server1", 
        "addr" : "127.0.0.1",
        "port" : 5003,
        "parent" : null
    }
```

参数描述如下
| 参数 | 描述 | 备注 |
| --- | --- | --- |
| log | log输出文件位置 | ****当前log系统还未完善**** |
| top_user_list | 顶级节点列表位置 | |
| global_mpk_file | 顶级域参数文件 | |
| global_sk_file | 顶级域私钥文件 | |
| admin_mpk_file | 节点自身网络域参数文件 | |
| admin_msk_file | 节点自身主私钥文件 | |
| admin_sk_file | 节点自身网络域与节点相应的私钥文件 | |
| local_certificate_file | 节点自身网络域终端证书（证书链结尾） | |
| admin_certificate_file | 节点自身网络域中间证书（证书链非结尾） | |
| id | 用户标识 | |
| addr | 用户IP地址 | |
| port | 用户端口号 | |
| parent | 用户父节点 | 顶级节点没有父节点 |

### `top_user_list`实例

```json
    [
        {
            "id" : "Server1", 
            "addr" : "127.0.0.1",
            "port" : 5003,
            "parent" : null
        },
        {
            "id" : "Server2", 
            "addr" : "127.0.0.1",
            "port" : 5004,
            "parent" : null
        },
        {
            "id" : "Server3", 
            "addr" : "127.0.0.1",
            "port" : 5005,
            "parent" : null
        }
    ]
```

* 所有的节点parent域必须为null 
* 节点之间地址和端口不冲突，ID互不相同

### 运行方式

1. 假设有N个顶级节点，需要其中N-1个节点运行如下指令

```shell
    python3 python/server.py -c init_server.json
```

2. 最后一个节点，在确认其余节点都已运行`server.py`服务之后运行如下指令

```shell
    python3 python/client.py --action init -c init_server.json 
```

### 运行结果

1. 节点输出`The initialization has finished`消息，初始化结束。
2. 节点输出一系列配置文件，对应配置文件中指定的相关文件名，未在配置文件中指定的文件名可忽略，会于后续版本中逐渐消除。


## 会话密钥协商

