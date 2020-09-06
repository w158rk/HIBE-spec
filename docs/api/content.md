## REST API

本节用于描述各类节点功能的http接口和使用方式

### 服务端

### 客户端

#### 获取私钥`/sk`

****<未实现>****

如果`user.local_sk`存在，返回base64编码后的私钥内容，否则，返回空。

#### 请求私钥`/sk/request`

如果`user.local_sk`存在，返回base64编码后的私钥内容；否则，向父节点请求私钥，保存到`user.local_sk`并返回base64编码后的私钥内容。

#### 加密数据 `/encrypt`

```json
{
  "message": "SGVsbG8=",
  "mpk": "MIHqBgoqgRzPVQGCLgYBBgkqgRzPVQGCLgMGCiqBHM9VAYIuBAEEQQQ_J5Nihxu3dttODZjkVdWKYbzfP2_v4XMCG-EFfFvf9bCNCtWF35qg46M6ewD3bZhCZ0g3Mz9zeCJ7djEM7vrSBIGBBGEcSeqm5v9Mwmk8epi2-6ONYB3BgvTX-1kvVGSxuswdsvKnAHTijwYEFvQuJRVLrD99qhy-HYnSxvBiVrkcn4Cm-YJqldVtTKXM66MvB7YthtAnnWVhFbxLaNl_DweNILJS266VGwvf4ycxgKOVdkCmubUO-zhtsyGn5URX307i",
  "user_id": "Client"
}
```

上图是一个请求body的示例，`message`和`user_id`分别为待加密消息（`urlsafe_b64encode`编码）和通信对手的标识，`mpk`为`urlsafe_b64encode`编码的公共参数文件。上述请求得到的结果如下：

```json
{
    "cipher": "MIGtBIGBBB0eQ1OhIg8kxGM9RsaDFOur4eepGe9X-abkXnkzaW97nLiuXyGibouf0pfOSmiuok27WnvyMp6ZitzDqwMzrM2k4ZYJUZxWFjCh-DT9zgDWkEK0sPvxEmbrYc6-g7tLg6IqAhdtRbWT2Ud7qiQDwDUiPIzT1u463TTAhQ_7hqSQBAUP64qHDwQg_GW-vYpER7-gmMdmknjyFoNOxrQSII0SDrh4ils8Gik="
}
```

#### 解密数据 `/decrypt`

```json
{
    "sk": "MIIBeAYKKoEcz1UBgi4GAQYJKoEcz1UBgi4DBgoqgRzPVQGCLgQBBEEEPyeTYocbt3bbTg2Y5FXVimG83z9v7-FzAhvhBXxb3_WwjQrVhd-aoOOjOnsA922YQmdINzM_c3gie3YxDO760gSBgQRhHEnqpub_TMJpPHqYtvujjWAdwYL01_tZL1RksbrMHbLypwB04o8GBBb0LiUVS6w_faocvh2J0sbwYla5HJ-ApvmCapXVbUylzOujLwe2LYbQJ51lYRW8S2jZfw8HjSCyUtuulRsL3-MnMYCjlXZAprm1Dvs4bbMhp-VEV99O4gQGQ2xpZW50BEEELy4AEBX4UdCoG47PF0J0b6t7p-lP6EzUywxF0aXeLTOxQDQm4H321ALRZdE9GqVKYDBymYVsCk1PatEBncZ73QRBBAQJNViWyWCAqKBV4SlsuPAVHR6F7eZ_iqrHj3RBuJ6wNpsbMIb31w2AtTCAvoFSGtauDxqYHq14ABimHs-vys8=",
    "cipher":"MIGtBIGBBB0eQ1OhIg8kxGM9RsaDFOur4eepGe9X-abkXnkzaW97nLiuXyGibouf0pfOSmiuok27WnvyMp6ZitzDqwMzrM2k4ZYJUZxWFjCh-DT9zgDWkEK0sPvxEmbrYc6-g7tLg6IqAhdtRbWT2Ud7qiQDwDUiPIzT1u463TTAhQ_7hqSQBAUP64qHDwQg_GW-vYpER7-gmMdmknjyFoNOxrQSII0SDrh4ils8Gik="
}

上图为解密参数，两个参数都是`urlsafe_b64encode`编码后的相关文件，解密后的结果如下：

```json
{
    "dmessage": "SGVsbG8="
}
```

