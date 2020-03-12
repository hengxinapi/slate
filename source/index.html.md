---
title: 恒信API文档

language_tabs: # must be one of https://git.io/vQNgJ
  - go

toc_footers:
  # - <a href='#'>Sign Up for a Developer Key</a>
  # - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:
  # - errors

search: false
---

# 恒信简介

Hengxin Network是使用有向无环图实现的分布式账簿系统。在Hengxin Network中，每一次记账行为就是往主网上发送了一次Transaction；主网将为每一个被确认了的Transaction生成一个快照Snapshot。基于Snapshot，各个节点构建了有向无环图，从而实现了高效，低功耗的分布式账簿系统。

与其他区块链项目不同，Hengxin Network从一开始就兼顾了隐私保护。Hengxin Network上的地址都包含三对公私钥：Private View Key / Public View Key, Private Spend Key / Public Spend Key, Private Encrypt Key / Public Encrypt Key。其中, Public View Key, Public Spend Key 以及 Public Encrypt Key构成了Address。只有持有了相应的私钥，才可以分辨出指向该地址的记录:

若持有某Address，则可以向该Address发送Transaction
若持有某Address的Private View Key，则可以识别出链上所有指向该Address的UTXO
若持有某Address的Private View Key 以及 Private Spend Key，则可以消费与该Address相关的UTXO
另外，Hengxin Network中亦引入了Asset概念。实际使用中，可以使用不同的Asset来区分不同的业务方。

Hengxin Network是 Hengxin 的无币区块链解决方案，采取了数据链下加密存储，数据文件hash上链的方式，提供了一套完整的无币区块链解决方案。

恒信网络有以下两个部分组成   
 1. Hengxin Kernel   
 2. Hengxin Wallet   

本文档主要介绍如何部署恒信网络中的 Hengxin Kernel以及Hengxin Wallet热钱包。


# 恒信 Kernel

Hengxin Kernel即恒信主网，负责处理所有的链上事物，对外提供稳定的区块链服务；

## 部署 Hengxin Kernel

Hengxin Kernel 作为Hengxin Network的底层链，其测试网络已经平稳运营了超过一年，产生了1500多万条snapshot。Hengxin Kernel包含两部分，其一为核心链，其二为Hengxin Network Kernel，向外提供REST 接口。

### 部署Hengxin Kernel流程:

> 配置文件模版 config.template.json

```json
{
  "cache-ttl": 3600,
  "listener": "127.0.0.1:7001",
  "max-cache-size": 128,
  "signer": "bf80380abb0011dbe573a462b81e591defc04f8ea5021eccb5b7f1de9794c701"
}
```

1. 创建一台Linux主机，建议配置在8核 32G内存 200G ssd硬盘以上
2. Hengxin将下发Kernel可执行文件: hxkernel
3. 使用可执行文件，创建两组私钥(payee，signer)，并将signer的public spend key发送给Hengxin: ___./hxkernel createaddress --public___
4. Hengxin收集齐了所有public spend key之后，将向各节点下发genesis.json, 以及 nodes.json文件
5. 各节点方将按照模版生成config.json，并将其与genesis.json, nodes.json放置在同一目录
6. 配置节点运行，/usr/local/hxkernel kernel -dir /home/hengxin/hxkernel/hxkernel -port 7001


键  | 说明
---------  | -----------
cache-ttl | 缓存的生存周期
listener| 监听端口
max-cache-size | 最大缓存大小
signer | 签名密钥


## 部署 Hengxin Kernel Gateway

1. Hengxin 将下发 Hengxin Kernel Gateway可执行文件
2. 各节点方部署运行: ___./hxkernel-gw server --port 8081 --kernel-node http://localhost:8001___

**注意，kernel-node的端口为kernel的端口 + 1000。** 如上述kernel使用了7001端口，此处则为8001。


## 接入 Hengxin Wallet

## 银行接入:

银行将部署Hengxin Wallet时创建的用户Address及需要开展的业务提交给Hengxin，Hengxin将为该银行的相应业务生成专属Asset，并将Asset信息发送给银行以及应急中心。

### 上链流程:

1. 当用户A在银行B完成了 开户操作，将开户结果发送给应急中心
2. 应急中心查出银行B的开户业务的专属Asset
3. 应急中心按照规则生成链上数据
4. 应急中心生成一个指向用户A的Address，银行B的Address（以及各监管方的Address，如金融局等）的Transaction，附带上相应数据，发送到主网
5. 主网收到Transaction后确认，并生成相应的Snapshot
6. 银行A 收到对应Transaction时，向应急中心请求对应数据，与memo中的hash比对，完成数据审计

对于应急中心来说，当完成了第5步，即完成了链的写入操作。

## 链上数据结构

Hengxin Network的Transaction中，可以自定义的设置Memo，长度最大为5120。在eKYC系统中，Memo为json marshal的字符串，结构如:

```json
{
  "t":"reg", 
  "d":"{}",
  "h": "raw_data_hash" 
}
```

键  | 说明
---------  | -----------
t | type
d| data
h | sha256(原始数据)


# 恒信 Wallet
## 简介

Hengxin Wallet作为Hengxin Network的热钱包，提供创建地址，发送transaction，以及获取地址snapshot列表的功能。   
Hengxin Wallet分为两部分，worker & server，其中worker将同步所有链上snapshot，并映射到对应的地址；server则对外提供rest接口。



## 编译和部署工程
钱包工程需要首先编译成二进制运行文件，需要安装Go依赖库

安装 Go 语言[Go Install](https://golang.org/doc/install)

`
go install github.com/fox-one/hxwallet
`

> 配置文件模版 config.yaml

```yaml
db:
  dialect: mysql # mysql, sqlite3
  host: localhost
  host_read: localhost
  port: 3306
  user: root
  password:
  database: hxwallet
  debug: true


kernel_node: http://localhost:8081 # kernel gateway

server:
  verify_auth_token: false
  verify_request_sig: false
```


### 钱包的运行
复制config.template.yaml到config.yaml，并适当配置。

### 设置数据库

`./hxwallet setdb --config config.local.yaml`

### 创建系统跟账号

> Shell 返回结果

```bash
user id: 6e5afa15-eb51-3d58-87cb-70dda3231df0
address: HXvrMpQKfFNTnLFSuGULugcsLxxxMWV3qBUnWc2HH81RhAdXKtUNxFnfoN7qXaNqs6LGgx1JL6ry1nd9sKjMDoRFJUjw6QRLJW9D7meogsdYHjWYEZKWVJzbMN6v5wYHxYnraimXzebPt772279NFQe6mWBFWwytTwU9mWxBXBmXV1JywPewNmZzGVYG8vHa8ySrbss9oU6Eso5CpG13rdJeV32
keystore: HXKSxUSRLudmJ4yxr7aC6Nq4Q1RPVim9woufmxyUGmvWcMizDshdvBiqWq54qj6TX7K592xzoy1vyrhRodnqSwGQzH4ZW7gKaF3K4EqNy8UNUo6DfsZoLzRH8jwP9e5K7xT316zX1zNSmmcfCobUJ4CfFR44poW3VwZweUFzwvGiMmkqGhpm5gKyXN53vaqAciVReBFzfEdEyAQiBm5oFbTfB5kBz1bUFJ3BJbfz1Ac6MauFpzotToYJD85fDCtxfEKekj
```

`./hxwallet createuser --config config.local.yaml`

键  | 说明
---------  | -----------
user id| 用户ID
address | 地址
keystore | 私钥

此账号作为系统账号，将address提供给kernel运行方，运行方将会提供足够的balance以使得系统可以正常运转。


### 启动Work

`./hxwallet worker --debug --config config.local.yaml`

### 启动Server
`./hxwallet server --debug --config config.local.yaml --port 8082`

## 鉴权

Hengxin Wallet实现了两种鉴权，

1. Basic: "Authorization: Basic base64(user_id)"  
2. JWT: "Authorization: Bearer jwt_token"  
 
若配置文件中开启了verify_auth_token, 则必须使用Bearer方式；否则两种皆可。

### JWT Token

JWT 的签名算法使用了国家标准密码算法.

[访问了解JWT](https://jwt.io/)   
[访问了解国家标准密码](http://www.oscca.gov.cn/sca/index.shtml)   

> Header:


```json
{
  "alg": "SM2",
  "typ": "JWT"
}
```

键  | 说明
---------  | -----------
alg| 使用的签名算法类型
typ | token 的种类



> Payload:

```json
{
  "exp": 1584064554, 
  "iat": 1583978154, 
  "iss": "HXy9YaTXHcNcQUXAHS2sBFX2A8yd2zEiePL4Bo5ZmwRGtgXbtnv142aja1Qw4k8m1pWfPaSS4cddx8GjDDdUFV5hgHZT1M5VrRBVsiiGb4iU953i9M9tKhUXWtZFKKZHaNWNnAiMnsVHUifyubY5Mm5PtqwJDNNzU2hYt1Mtqvhcd8ZnJwBeNygvgjicnBajr52ZjKPtd85bezMnVUFCrK4iUDn", 
  "nc": "__uuid__",
  "sig": "xxxx", 
}
```

键  | 说明
---------  | -----------
exp| token过期时间。Unix Timestamp, 以s为单位
iat | token签发时间，为当前时间即可。Unix Timestamp, 以s为单位
iss | 根用户的 address
nc | 每次生成一个唯一值
sig | optional. request signature，sm3.hash(METHOD+URI+BODY)，注意，URI不包含host，例如sms.hash("GET/snapshots")。若Wallet未开启verify_request_signature，则"sig"可以唯恐


> Token 生成的方法:

```go
func GenerateToken(ekInPEM, address string, expires int64) (string, error) {
  block, _ := pem.Decode([]byte(ekInPEM))
  if block == nil {
    return "", errors.New("failed to decode private key")
  }
  ek, err := sm2.ParsePKCS8PrivateKey(block.Bytes, nil)
  if err != nil {
    return "", err
  }

  header := `{"alg":"SM2","typ":"JWT"}`
  iat := time.Now().Unix()
  payload := fmt.Sprintf(`{"exp":%d,"iat":%d,"iss":"%s"}`, iat+expires, iat, address)

  signedData := strings.Join([]string{
    strings.TrimRight(base64.URLEncoding.EncodeToString([]byte(header)), "="),
    strings.TrimRight(base64.URLEncoding.EncodeToString([]byte(payload)), "="),
  }, ".")

  hasher := sm3.New()
  hasher.Write([]byte(signedData))

  sig, err := ek.Sign(rand.Reader, hasher.Sum(nil), nil)
  if err != nil {
    return "", err
  }

  return strings.Join([]string{
    signedData,
    strings.TrimRight(base64.URLEncoding.EncodeToString([]byte(sig)), "="),
  }, "."), nil
}
```


## 获取用户信息 API

### HTTP 请求
`GET /info`

### 返回值

> 接口返回的JSON结构:

```json
{
  "code": 0,
  "data": {
    "user_id": "6c54d04f-585a-3c0b-869f-6e63f6912410",
    "created_at": "2020-03-12T02:15:11Z",
    "address": "HX266NGnUZ5HmXzzgBW4C1asDcPGqxteTJZu9EaoFUGLxT6KZEcH96bNE1L4zqy3neBbjdZLScbr1vMM8rB186qrHFERetcwY36ecxW1yToJG3MZ6gw3Pf4oGx7NyELanCzNWbnRL8NmjoRQsuUtPKEgXegoq5oh6yyJ5y9prH43gTj7zjYxgkkcvnGj9osFkKb9vk6q4nDGL5vtpa63kUjV6oSe"
  }
}
```

键  | 说明
---------  | -----------
user_id| 在访问该用户资源(如snapshot列表)时使用，通过user_id完成用户鉴权
created_at | 创建时间
address | 获取用户的地址



## 创建用户 API

### HTTP 请求
`POST /users`

### 返回值

> 接口返回的JSON结构:

```json
{
  "code": 0,
  "data": {
    "user_id": "6c54d04f-585a-3c0b-869f-6e63f6912410",
    "created_at": "2020-03-05T22:21:13.321112+08:00",
    "address": "HX266NGnUZ5HmXzzgBW4C1asDcPGqxteTJZu9EaoFUGLxT6KZEcH96bNE1L4zqy3neBbjdZLScbr1vMM8rB186qrHFERetcwY36ecxW1yToJG3MZ6gw3Pf4oGx7NyELanCzNWbnRL8NmjoRQsuUtPKEgXegoq5oh6yyJ5y9prH43gTj7zjYxgkkcvnGj9osFkKb9vk6q4nDGL5vtpa63kUjV6oSe",
    "keystore": "HXKS7ET32Wgs4JntsxqewKMPQTR19EAxjFtxMhVvUMAP6uu2nxYZUtLgJMmNA6nv1Jr7eVWdEJxSrn1ggjVjEYarPEKpiogyNvHrLKX5sZcnsdeYYgYT2uK9KZJayNHJ3u7bJfSuKB7ZoXcPMst8fuSCMPjmFLq5h3dN8i6pLK2RFVuXbAYiAsDfxrPK7aiXZgq71u8FxEgszVjZn2HBezVFtUmy6VhdRcNSYgnNevmhiKWYG4aVpJtFDQ1xJi6yWnaENh"
  }
}
```

键  | 说明
---------  | -----------
user_id| 在访问该用户资源(如snapshot列表)时使用，通过user_id完成用户鉴权
created_at | 创建时间
address | 在生成Transaction时使用
keystore | 用户的私钥

## 提交 Transaction API

### HTTP 请求
`POST /transactions`

### Query 参数

```json
{
  "asset": "b9f49cf777dc4d03bc54cd1367eebca319f8603ea1ce18910d09e2c540c630d8",
  "opponent_addresses": ["HX266NGnUZ5HmXzzgBW4C1asDcPGqxteTJZu9EaoFUGLxT6KZEcH96bNE1L4zqy3neBbjdZLScbr1vMM8rB186qrHFERetcwY36ecxW1yToJG3MZ6gw3Pf4oGx7NyELanCzNWbnRL8NmjoRQsuUtPKEgXegoq5oh6yyJ5y9prH43gTj7zjYxgkkcvnGj9osFkKb9vk6q4nDGL5vtpa63kUjV6oSe"],
  "trace_id": "6c54d04f-585a-3c0b-869f-6e63f6912410",  
  "memo": "{\"t\":\"reg\",\"d\":\"{}\",\"h\":\"xxxx\"}" 
}
```

键  | 描述
---------  | -----------
asset | 资产类型
opponent_addresses| 指向用户Address，银行Address，同时还可以添加其他监管机构Address，如金融局地址
trace_id | 任意字符串，最大长度为36。在Hengxin Wallet内唯一，从而保证一个Transaction只能被记录一次。实际使用中，可以根据业务的信息来生成，这样在请求失败之后可以重复提交保证数据上链成功，而又不会发生重复上链的事件
memo | 链上数据，此数据为公开可见数据，不建议添加隐私数据

### 返回值

> 接口返回的JSON结构:

```json
{
  "code": 0,
  "data": {
    "tx_hash": "2d259a9cbe49eccd7878112e291e378fee7c08af0b443c598b1fbc091d7345fc"
  }
}
```


键  | 说明
---------  | -----------
tx_hash |  Transaction Hash 值


## 内转转账 API
### HTTP 请求
`POST /transfer`

### 参数
```json
{
  "asset": "b9f49cf777dc4d03bc54cd1367eebca319f8603ea1ce18910d09e2c540c630d8",
  "opponent_id": "c95a14b8-14fe-3228-aa2b-a28c88bd1591", 
  "trace_id": "6c54d04f-585a-3c0b-869f-6e63f6912410",   
  "memo": "{}"
}
```
键  | 描述
---------  | -----------
asset | 资产类型
opponent_id| transaction接收方user_id
trace_id | 任意字符串，最大长度为36。在Hengxin Wallet内唯一，从而保证一个Transaction只能被记录一次。实际使用中，可以根据业务的信息来生成，这样在请求失败之后可以重复提交保证数据上链成功，而又不会发生重复上链的事件
memo | 备注

### 返回值

> 接口返回的JSON结构:

```json
{
  "code": 0,
  "data": {
    "trace_id": "6c54d04f-585a-3c0b-869f-6e63f6912410",
    "tx_hash": "2d259a9cbe49eccd7878112e291e378fee7c08af0b443c598b1fbc091d7345fc"
  }
}
```

键  | 说明
---------  | -----------
trace_id | 用于追溯
tx_hash | 链上 Hash 地址


## 读取记账记录列表 API

### HTTP 请求
`GET /snapshots?asset = b9f49cf777dc4d03bc54cd1367eebca319f8603ea1ce18910d09e2c540c630d8&from=1&limit=1&order=ASC`


### Query 参数

键  | 描述
---------  | -----------
asset | 可选，若为空则查询所有
from| 上一条snapshot深度。ASC时，查询height > from的数据; DESC时查询height < from的数据
limit | 返回数据条目。最大 500
order | 排序方式


### 返回值

> 接口返回的JSON结构:

```json
{
  "code": 0,
  "data": [
    {
      "created_at": "2020-03-05T13:17:37Z",
      "asset": "b9f49cf777dc4d03bc54cd1367eebca319f8603ea1ce18910d09e2c540c630d8",
      "user_id": "c852a70d-86ca-3832-b7ab-bbfdf68bc39b",
      "amount": "0.00000001", 
      "opponent_id": "c95a14b8-14fe-3228-aa2b-a28c88bd1591", 
      "memo": "{}",
      "height": 100 
    }
  ]
}
```

键  | 说明
---------  | -----------
created_at | 创建时间
asset | 资产类型
user_id| 在访问该用户资源(如snapshot列表)时使用，通过user_id完成用户鉴权
amount | 金额数量 正数时为用户接收到transaction；负数时为用户发起的transaction
opponent_id | 接收方 optional，指向transaction的对方。若用户为接收方则指向发起方，若用户为发起方则指向接收方。只有opponent私钥托管在Hengxin Wallet时，此字段才会有数值。
memo | 备注
height | Hengxin Wallet中的snapshot深度, 下次查询时