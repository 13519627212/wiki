# How to Join Nebulas Testnet

## Introduction

We are glad to release Nebulas Testnet here. It simulate the Nebulas network and NVM, and allow developers to interact with Nebulas without paying the cost of gas.

#### Configuration

The testnet configuration files are in folder [`testnet/conf`](https://github.com/nebulasio/go-nebulas/tree/master/testnet/conf).

Information about it is below.

> Genesis.conf

Genesis.conf should be as same as [here](resources/conf/testnet/genesis.conf) exactly.

> neb.conf

Refer to neb.conf [here](resources/conf/neb.conf).

Notice:

* testnet seed node:

```
seed:["/ip4/35.154.108.11/tcp/8680/ipfs/QmTvcQ3E1yGdKoW6rSQYVNpbLUivXtb44Py72EvySSLaQx"]
```
* `chain_id` should be `1001`, as same as [genesis.conf](resources/conf/testnet/genesis.conf).
* `datadir` should be different with private chain.

The example of testnet conf [here](resources/conf/testnet/config.conf).

#### API List

Test Endpoint:

| API | URL | Protocol |
|-------|:------------:|:------------:|
| RESTful | https://testnet.nebulas.io/ | HTTP |

* [GetNebState](https://github.com/nebulasio/wiki/blob/master/rpc.md#getnebstate) : returns nebulas client info.
* [GetAccountState](https://github.com/nebulasio/wiki/blob/master/rpc.md#getaccountstate): returns the account balance and nonce.
* [SendRawTransaction](https://github.com/nebulasio/wiki/blob/master/rpc.md#sendrawtransaction): submit signed transaction. The transaction must be signed before send.
* [Call](https://github.com/nebulasio/wiki/blob/master/rpc.md#call): execute smart contract local, don't submit on chain.
* [SendRawTransaction](https://github.com/nebulasio/wiki/blob/master/rpc.md#sendrawtransaction): submit the signed transaction.
* [GetTransactionReceipt](https://github.com/nebulasio/wiki/blob/master/rpc.md#gettransactionreceipt): get transaction receipt info by tansaction hash.

More Nebulas APIs at [RPC](https://github.com/nebulasio/wiki/blob/master/rpc.md).

#### Token Claim

Every email can claim some tokens every day at [here](https://testnet.nebulas.io/claim).

## Tutorials

#### English

1. [Installation](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2001%20Installation.md) (thanks [Victor](https://github.com/victorychain))
2. [Sending a Transaction](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2002%20Transaction.md) (thanks [Victor](https://github.com/victorychain))
3. [Writing Smart Contract in JavaScript](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2003%20Smart%20Contracts%20JavaScript.md) (thanks [otto](https://github.com/ottokafka))
4. [Introducing Smart Contract Storage](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2004%20Smart%20Contract%20Storage.md) (thanks [Victor](https://github.com/victorychain))
5. [Interacting with Nebulas by RPC API](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2005%20Interacting%20with%20Nebulas%20by%20RPC%20API.md) (thanks [Victor](https://github.com/victorychain))
6. [Connecting to the test network i.e. Testnet](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2006%20Testnet.md)

#### 中文

1. [编译安装及运行neb](https://github.com/nebulasio/wiki/blob/master/tutorials/%5B中文%5D%20Nebulas%20101%20-%2001%20编译安装.md)
2. [在星云链上发送交易](https://github.com/nebulasio/wiki/blob/master/tutorials/%5B中文%5D%20Nebulas%20101%20-%2002%20发送交易.md)
3. [使用JavaScript编写智能合约](https://github.com/nebulasio/wiki/blob/master/tutorials/%5B中文%5D%20Nebulas%20101%20-%2003%20编写智能合约.md)
4. [智能合约存储区介绍](https://github.com/nebulasio/wiki/blob/master/tutorials/%5B中文%5D%20Nebulas%20101%20-%2004%20智能合约存储区.md)
5. [通过RPC接口与星云链交互](https://github.com/nebulasio/wiki/blob/master/tutorials/%5B中文%5D%20Nebulas%20101%20-%2005%20通过RPC接口与星云链交互.md)
6. [连接测试网络](https://github.com/nebulasio/wiki/blob/master/tutorials/%5B中文%5D%20Nebulas%20101%20-%2006%20测试网络.md)

## Contribution

Feel free to join Nebulas Testnet. If you did find something wrong, please [submit a issue](https://github.com/nebulasio/go-nebulas/issues/new) or [submit a pull request](https://github.com/nebulasio/go-nebulas/pulls) to let us know, we will add your name and url to this page soon.
