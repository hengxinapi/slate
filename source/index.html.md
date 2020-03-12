---
title: 恒信网络API文档

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

1. 若持有某Address，则可以向该Address发送Transaction
2. 若持有某Address的Private View Key，则可以识别出链上所有指向该Address的UTXO
3. 若持有某Address的Private View Key 以及 Private Spend Key，则可以消费与该Address相关的UTXO

另外，Hengxin Network中亦引入了Asset概念。实际使用中，可以使用不同的Asset来区分不同的业务方。


# 部署 Hengxin Network

Hengxin Network 包含 Hengxin Kernel以及Hengxin Wallet热钱包两部分。

其中Hengxin Kernel即恒信主网，负责处理所有的链上事物，对外提供稳定的区块链服务；Hengxin Wallet是基于Hengxin Kernel的热钱包，它托管了用户私钥，为用户同步Snapshot，以及生成Transaction。对于普通用户来说，使用Hengxin Wallet可以完全不了解区块链知识而实现数据上链的需求。

# 部署 Hengxin Kernel

Hengxin Kernel 作为Hengxin Network的底层链，其测试网络已经平稳运营了超过一年，产生了1500多万条snapshot。Hengxin Kernel包含两部分，其一为核心链，其二为Hengxin Network Kernel，向外提供REST 接口。

## 部署Hengxin Kernel流程:

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



## 部署 Hengxin Kernel Gateway:

1. Hengxin 将下发 Hengxin Kernel Gateway可执行文件
2. 各节点方部署运行: ___./hxkernel-gw server --port 8081 --kernel-node http://localhost:8001___

**注意，kernel-node的端口为kernel的端口 + 1000。** 如上述kernel使用了7001端口，此处则为8001。

###部署 Hengxin Wallet

Hengxin Wallet是Hengxin实现的一个热钱包。热钱包实现了私钥托管，同步UTXO，消费UTXO的功能。通过REST接口，可以轻松完成链上资源读，写。

### 部署流程：

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

1. 创建数据库服务器，创建数据库，以及配置数据库账号
2. Hengxin将下发Hengxin Wallet可执行文件
3. 使用 hxkernel createaddress 随机生成新的view key，记录下来
4. 按照配置文件模版生成配置文件 config.yaml
5. 部署运行 Hengxin Wallet Worker, 运行指令 __./hxwallet worker --config config.yaml__
6. 部署运行 Hengxin Wallet Server, 运行指令 __./hxwallet server --config config.yaml --port 8082__
7. 运行createuser command以创建根用户并保管好私钥信息，将地址信息发送给Hengxin，Hengxin将为该地址发送足够的余额以进行链的写入操作



> 创建用户

```shell
./hxwallet createuser --config config.yaml

user id: f6c86fe8-ff97-3c64-8e3a-bacdef6b2bed
address: HX2XmbfUqaANYWSPCh2ZcGrBE1Z7pjkrsWufowGPPBVnvaNLbTQvmY6W7KWgUKbxBV1oZ5XWsKUsc2Uq4cKkjeKJT7Nkv8RF2z2fohLn9QG56y5PYd3vFqyMaDEbGUfAXonbHpnnbXBMbixKc38F2dHMBXuQffTwXkpQ5Scvsi42f6i3VGy4R4bqvnnysZACmAC3Y7ycS129Dyh1njV2xzYJ9erz
```

# 接入 Hengxin Wallet

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

## Auth

请求时，添加Http Header "Authorization: Basic base64(user_id)", __根用户的user_id在上述部署阶段生成__

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



## 创建新地址

### HTTP 请求
`POST /users`

> 接口返回的JSON结构:

```json
{
  "code": 0,
  "data": {
    "created_at": "2020-03-05T22:21:13.321112+08:00",
    "user_id": "6c54d04f-585a-3c0b-869f-6e63f6912410",
    "address": "HXxxx" 
  }
}
```

键  | 说明
---------  | -----------
created_at | 创建时间
user_id| 在访问该用户资源(如snapshot列表)时使用，通过user_id完成用户鉴权
address | 在生成Transaction时使用


## 提交 Transaction

### HTTP 请求
`POST /transactions`

### Query 参数

```json
{
  "asset": "b9f49cf777dc4d03bc54cd1367eebca319f8603ea1ce18910d09e2c540c630d8",
  "opponent_addresses": ["HX266NGnUZ5HmXzzgBW4C1asDcPGqxteTJZu9EaoFUGLxT6KZEcH96bNE1L4zqy3neBbjdZLScbr1vMM8rB186qrHFERetcwY36ecxW1yToJG3MZ6gw3Pf4oGx7NyELanCzNWbnRL8NmjoRQsuUtPKEgXegoq5oh6yyJ5y9prH43gTj7zjYxgkkcvnGj9osFkKb9vk6q4nDGL5vtpa63kUjV6oSe"], 
  "memo": "{\"t\":\"reg\",\"d\":\"{}\",\"h\":\"xxxx\"}" // 
}
```

键  | 描述
---------  | -----------
asset | 资产类型
opponent_addresses| 指向用户Address，银行Address，同时还可以添加其他监管机构Address，如金融局地址
memo | 链上数据，此数据为公开可见数据，不建议添加隐私数据




> 接口返回的JSON:

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

## 读取 记账记录列表

### HTTP 请求
`GET /snapshots?asset = b9f49cf777dc4d03bc54cd1367eebca319f8603ea1ce18910d09e2c540c630d8&from=1&limit=1&order=ASC`


### Query 参数

键  | 描述
---------  | -----------
asset | 可选，若为空则查询所有
from| 上一条snapshot深度。ASC时，查询height > from的数据; DESC时查询height < from的数据
limit | 返回数据条目。最大 500
order | 排序方式

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