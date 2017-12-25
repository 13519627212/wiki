
# Nebulas 101 - 05 通过RPC接口与星云链交互

星云链节点启动后可以通过RPC远程控制访问。星云链提供了一系列API来获取节点的信息，账号余额，发送交易和部署调用智能合约。

星云链的远程访问是[gRPC](https://grpc.io)实现的，通过代理([grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway))也可以通过HTTP访问。HTTP访问是RESTful实现的接口，参数与gRPC的调用接口参数相同。

## gRPC访问
gRPC是一个高性能、通用的开源RPC框架，由Google主要面向移动应用开发并基于HTTP/2协议标准而设计，基于[ProtoBuf](https://github.com/google/protobuf)序列化协议开发，且支持众多开发语言。

星云链的gRPC接口是用go语言开发的，并提供了一个go语言写的客户端[demo](https://github.com/nebulasio/go-nebulas/blob/develop/rpc/testing/client/main.go)。这里就主要介绍使用go实现的gRPC接口使用。（gRPC支持多种语言，也可以使用其他语言访问gRPC。）

gRPC使用ProtoBuf来定义服务,protobuf的定义在官方代码的[/rpc/pb](https://github.com/nebulasio/go-nebulas/tree/master/rpc/pb)中：

```
// API接口，定义了节点、账号地址信息获取，发送交易等接口
api_rpc.proto

```
使用的时候可以在`rpc/pb`文件夹中执行`make`：

```
cd rpc/pb
make
```
生成对应的go版本grpc接口代码。官方代码go版本已经生成了，使用的时候可以不用重新生成。gRPC的端口可以在配置文件(eg:`conf/default/seed.conf`)中修改。配置文件的中的端口配置项如下：

```
# 用户与节点交互的服务配置，同一台机器启动多个时注意修改端口防止占用
rpc {
    # gRPC API服务端口
    rpc_listen: ["127.0.0.1:51510"]
    # HTTP API服务端口
    http_listen: ["127.0.0.1:8090"]
    # 开放可对外提供http服务的模块
    http_module: ["api","admin"]
}
```
默认的配置端口为上述的`API:51510`。

go的gRPC访问代码如下：

```go
// gRPC服务器连接地址配置
addr := fmt.Sprintf("127.0.0.1:%d", uint32(51510))
conn, err := grpc.Dial(addr, grpc.WithInsecure())
if err != nil {
	log.Warn("rpc.Dial() failed: ", err)
}
defer conn.Close()

// API接口访问，获取节点状态信息
api := rpcpb.NewAPIServiceClient(conn)
resp, err := ac.GetNebState(context.Background(), &rpcpb.GetNebStateRequest{})
if err != nil {
	log.Println("GetNebState", "failed", err)
} else {
	//tail := r.GetTail()
	log.Println("GetNebState tail", resp)
}

// API接口访问,锁定账号地址
management := rpcpb.NewManagementServiceClient(conn)
from := "8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf"
resp, err = management.LockAccount(context.Background(), &rpcpb.LockAccountRequest{Address: from})
if err != nil {
	log.Println("LockAccount", from, "failed", err)
} else {
	log.Println("LockAccount", from, "result", resp)
}
```
API的接口定义在通过proto文件生成的go接口文件中:
`api_rpc.pb.go`

## HTTP访问
星云链的HTTP访问使用了RESTful风格的API。使用HTTP接口可以很方便的获取节点的信息，账号地址的余额，发送交易和部署调用智能合约。

官方默认端口：

* 8090：默认API端口，可以访问[RPC](https://github.com/nebulasio/wiki/blob/master/rpc.md)的接口，有获取节点信息，发送交易等功能；可以对外部用户开放。

一些使用HTTP访问接口的例子：

##### 获取节点信息
返回节点信息。

| Protocol | Method | API |
|----------|--------|-----|
| HTTP | GET |  /v1/user/nodeinfo |

###### Parameters
none

###### Returns
`id` 节点ID.

`chain_id` 区块链ID.

`version` 节点版本.

`peer_count` 当前连接的节点数.

`synchronized` 节点同步状态.

`bucket_size` 节点路由表保存节点个数.

`relay_cache_size` 转播缓存大小.

`stream_store_size` 节点流数据仓库size.

`stream_store_extend_size` 节点流数据仓库扩展size.

`protocol_version` 网络协议版本.

`RouteTable route_table` 路由表

```
message RouteTable {
	string id = 1;
	repeated string address = 2;
}
```

###### HTTP Example
```
// Request
curl -i -H Accept:application/json -X GET http://localhost:8090/v1/user/nodeinfo

// Result
{
    "id":"QmPyr4ZbDmwF1nWxymTktdzspcBFPL6X1v3Q5nT7PGNtUN",
    "chain_id":100,
    "version":1,
    "bucket_size":16,
    "relay_cache_size":65536,
    "stream_store_size":128,
    "stream_store_extend_size":32,
    "protocol_version":"/neb/1.0.0"
}
```
##### 账号列表
返回节点存在的账号列表。

| Protocol | Method | API |
|----------|--------|-----|
| HTTP | GET |  /v1/user/accounts |

##### Parameters
无

##### Returns
`addresses` 账号列表

##### HTTP Example
```
// Request
curl -i -H Accept:application/json -X GET http://localhost:8090/v1/user/accounts

// Result
{
    "addresses":[
        "16464b93292d7c99099d4d982a05140f12779f5e299d6eb4",
        "22ac3a9a2b1c31b7a9084e46eae16e761f83f02324092b09",
        "5cdadc1cfe3da0a3d067e9f1b195b90c5aebfb5afc8d43b4",
        "8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf"
    ]
}
```
#### 获取账号信息
返回账号信息，包括账号地址的余额和当前交易nonce。

| Protocol | Method | API |
|----------|--------|-----|
| HTTP | POST |  /v1/user/accountstate |

###### Parameters
`address` 地址哈希.

###### Returns
`balance` 当前余额 单位： 1/(10^18) nas.

`nonce` 当前交易nonce.

###### HTTP Example
```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8090/v1/user/accountstate -d '{"address":"22ac3a9a2b1c31b7a9084e46eae16e761f83f02324092b09"}'

// Result
{
    "balance":"5",
    "nonce": "0"
}
```
#### 解锁账号
使用密码解锁账号.

| Protocol | Method | API |
|----------|--------|-----|
| HTTP | POST |  /v1/admin/account/unlock |


###### Parameters
`address` 账号地址哈希.

`passphrase` 账号密码.

###### Returns
`result` 解锁结果.

###### HTTP Example
```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8191/v1/admin/account/unlock -d '{"address":"8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf", "passphrase":"passphrase"}'

// Result
{
    "result":true
}

```
#### 发送交易
发送交易，提交合约接口

| Protocol | Method | API |
|----------|--------|-----|
| HTTP | POST |  /v1/user/transaction |

###### Parameters
`from` 发送账号地址哈希.

`to` 接收账号地址哈希.

`value` 金额，单位 1/(10^18)nas.

`nonce` 交易nonce.

`source` 合约代码.

`args` 合约参数.

`gas_price` 交易gas价格.

`gas_limit` 交易gas使用上限.

###### Returns
`txhash` 交易hash;

如果是部署合约，还有合约地址信息：

`contract_address` 合约地址信息；

###### Example
```
// Request
curl -i -H 'Accept: application/json' -X POST http://localhost:8090/v1/user/transaction -H 'Content-Type: application/json' -d '{"from":"83a78219edbdeee19eefc48b8d9a4a7cfa02704518b54511","to":"8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf","nonce":1,"source":"\"use strict\";var BankVaultContract=function(){LocalContractStorage.defineMapProperty(this,\"bankVault\")};BankVaultContract.prototype={init:function(){},save:function(height){var deposit=this.bankVault.get(Blockchain.transaction.from);var value=new BigNumber(Blockchain.transaction.value);if(deposit!=null&&deposit.balance.length>0){var balance=new BigNumber(deposit.balance);value=value.plus(balance)}var content={balance:value.toString(),height:Blockchain.block.height+height};this.bankVault.put(Blockchain.transaction.from,content)},takeout:function(amount){var deposit=this.bankVault.get(Blockchain.transaction.from);if(deposit==null){return 0}if(Blockchain.block.height<deposit.height){return 0}var balance=new BigNumber(deposit.balance);var value=new BigNumber(amount);if(balance.lessThan(value)){return 0}var result=Blockchain.transfer(Blockchain.transaction.from,value);if(result>0){deposit.balance=balance.dividedBy(value).toString();this.bankVault.put(Blockchain.transaction.from,deposit)}return result}};module.exports=BankVaultContract;", "args":""}'

// Result
{
    "txhash":"896f813a4560732793cdceb9133682277de20df502a35f7af2a845e6606c450b",
    "contract_address":"695392515cfb8fbfc1f50b7eec02e79a1fbb31e710178b0f"
}
```
#### 获取交易信息
通过交易哈希返回交易信息.

| Protocol | Method | API |
|----------|--------|-----|
| HTTP | POST |  /v1/user/getTransactionReceipt |

###### Parameters
`hash` 交易哈希.

###### Returns
`hash` 交易哈希.

`from` 交易发送地址哈希.

`to` 交易接收地址哈希.

`nonce` 交易nonce.

`timestamp` 交易时间戳.

`data` 交易数据.

`chainId` 交易链ID.

`contract_address`合约地址.

###### HTTP Example
```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8090/v1/user/getTransactionReceipt -d '{"hash":"f37acdf93004f7a3d72f1b7f6e56e70a066182d85c186777a2ad3746b01c3b52"}'

// Result
{
    "hash":"f37acdf93004f7a3d72f1b7f6e56e70a066182d85c186777a2ad3746b01c3b52",
    "from":"8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf",
    "to":"22ac3a9a2b1c31b7a9084e46eae16e761f83f02324092b09",
    "nonce":"12",
    "timestamp":"1511519091",
    "chainId":1
}
```


详细的接口使用和参数说明，可以参考官方文档[RPC](https://github.com/nebulasio/wiki/blob/master/rpc.md)和[Management RPC](https://github.com/nebulasio/wiki/blob/master/management_rpc.md)。

