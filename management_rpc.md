# Management RPC

Beside the [NEB API RPC](https://github.com/nebulasio/wiki/blob/master/rpc.md) interface nebulas provides additional management APIs. Neb console supports both API and management interfaces. Management RPC has separate gRPC and HTTP port, which also binds [NEB API RPC](https://github.com/nebulasio/wiki/blob/master/rpc.md) interfaces.

Nebulas provide both [gRPC](https://grpc.io) and RESTful management APIs, let users interact with Nebulas. Our management [proto](https://github.com/nebulasio/go-nebulas/blob/develop/rpc/pb/management_rpc.proto) file defines all management APIs. **We recommend using the console access management interfaces, or restricting the port of the management RPC to local access.**

Default management RPC Endpoint:

| API | URL | Protocol |
|-------|:------------:|:------------:|
| gRPC |  http://localhost:52520 | Protobuf
| RESTful |http://localhost:8191 | HTTP |


## Management RPC methods

* [NewAccount](#newaccount)
* [UnlockAccount](#unlockaccount)
* [LockAccount](#lockaccount)
* [SignTransaction](#signtransaction)
* [SendTransactionWithPassphrase](#sendtransactionwithpassphrase)

## Management RPC API Reference

#### NewAccount
NewAccount create a new account with passphrase.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  NewAccount |
| HTTP | POST |  /v1/account/new |


###### Parameters
`passphrase` New account passphrase.

###### Returns
`address` New Account address.

###### HTTP Example
```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8191/v1/account/new -d '{"passphrase":"passphrase"}'

// Result

{
    "address":"71e616cb8bcd1270eee26f54247bd45dd0e5e0032bc04279"
}

```
***

#### UnlockAccount
UnlockAccount unlock account with passphrase.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  UnlockAccount |
| HTTP | POST |  /v1/account/unlock |


###### Parameters
`address` Unlock account address.

`passphrase` Unlock account passphrase.

###### Returns
`result` Unlock account result.

###### HTTP Example
```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8191/v1/account/unlock -d '{"address":"8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf", "passphrase":"passphrase"}'

// Result
{
    "result":true
}

```

***

#### LockAccount
LockAccount lock account.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  LockAccount |
| HTTP | POST |  /v1/account/lock |


###### Parameters
`address` Lock account address.

###### Returns
`result` Lock account result.

###### HTTP Example
```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8191/v1/account/lock -d '{"address":"8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf"}'

// Result
{
    "result":true
}

```
***

#### SignTransaction
SignTransaction sign transaction.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  SignTransaction |
| HTTP | POST |  /v1/sign |


###### Parameters
`from` Hex string of the sender account addresss.

`to` Hex string of the receiver account addresss.

`value` Amount of value sending with this transaction.

`nonce` Transaction nonce.

`source` Contract source code.

`args` The params of contract.

`gas_price` GasPrice sending with this transaction.

`gas_limit` GasLimit sending with this transaction.

###### Returns
`data` Signed transaction data.

###### Example
```
// Request
curl -i -H 'Accept: application/json' -X POST http://localhost:8191/v1/sign -H 'Content-Type: application/json' -d '{"from":"8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf","to":"8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf","nonce":1,"source":"\"use strict\";var BankVaultContract=function(){LocalContractStorage.defineMapProperty(this,\"bankVault\")};BankVaultContract.prototype={init:function(){},save:function(height){var deposit=this.bankVault.get(Blockchain.transaction.from);var value=new BigNumber(Blockchain.transaction.value);if(deposit!=null&&deposit.balance.length>0){var balance=new BigNumber(deposit.balance);value=value.plus(balance)}var content={balance:value.toString(),height:Blockchain.block.height+height};this.bankVault.put(Blockchain.transaction.from,content)},takeout:function(amount){var deposit=this.bankVault.get(Blockchain.transaction.from);if(deposit==null){return 0}if(Blockchain.block.height<deposit.height){return 0}var balance=new BigNumber(deposit.balance);var value=new BigNumber(amount);if(balance.lessThan(value)){return 0}var result=Blockchain.transfer(Blockchain.transaction.from,value);if(result>0){deposit.balance=balance.dividedBy(value).toString();this.bankVault.put(Blockchain.transaction.from,deposit)}return result}};module.exports=BankVaultContract;", "args":""}'

// Result
{
    "data":"CiDQYaX62Yj2F0vuovLgD0X+BMD1ADXJZxLnqXqnPNv1hBIYiiCc7ALL6rfi90rZadLf6N0kQWqmVYm/GhiKIJzsAsvqt+L3Stlp0t/o3SRBaqZVib8iEAAAAAAAAAAAAAAAAAAAAAAoATCFtOTQBToICgZiaW5hcnlAAUoQAAAAAAAAAAAAAAAAAAAAAFIQAAAAAAAAAAAAAAAAAAAAAFgBYkF8DUuC4/6u3YDNPMx8Xq7wQu7bywLObYMjBmA4BBFcECYKUNz15jFdGq/L/VBodoN0+eoUmW8VstB1l+j94BOiAA=="
}
```
***

#### SendTransactionWithPassphrase
SendTransactionWithPassphrase send transaction with passphrase.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  SendTransactionWithPassphrase |
| HTTP | POST |  /v1/transactionWithPassphrase |


###### Parameters
`from` Hex string of the sender account addresss.

`to` Hex string of the receiver account addresss.

`value` Amount of value sending with this transaction.

`nonce` Transaction nonce.

`source` Contract source code.

`args` The params of contract.

`gas_price` GasPrice sending with this transaction.

`gas_limit` GasLimit sending with this transaction.

`passphrase` From address passphrase.

###### Returns
if general transaction:
`hash` Hex string of transaction hash.
if deploy & init smart contract:
`hash` transaction hash + '$' + address of contract

###### Example
```
// Request
curl -i -H 'Accept: application/json' -X POST http://localhost:8191/v1/transactionWithPassphrase -H 'Content-Type: application/json' -d '{"from":"8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf","to":"8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf","value": "1","nonce":1,"source":"\"use strict\";var BankVaultContract=function(){LocalContractStorage.defineMapProperty(this,\"bankVault\")};BankVaultContract.prototype={init:function(){},save:function(height){var deposit=this.bankVault.get(Blockchain.transaction.from);var value=new BigNumber(Blockchain.transaction.value);if(deposit!=null&&deposit.balance.length>0){var balance=new BigNumber(deposit.balance);value=value.plus(balance)}var content={balance:value.toString(),height:Blockchain.block.height+height};this.bankVault.put(Blockchain.transaction.from,content)},takeout:function(amount){var deposit=this.bankVault.get(Blockchain.transaction.from);if(deposit==null){return 0}if(Blockchain.block.height<deposit.height){return 0}var balance=new BigNumber(deposit.balance);var value=new BigNumber(amount);if(balance.lessThan(value)){return 0}var result=Blockchain.transfer(Blockchain.transaction.from,value);if(result>0){deposit.balance=balance.dividedBy(value).toString();this.bankVault.put(Blockchain.transaction.from,deposit)}return result}};module.exports=BankVaultContract;", "args":"","passphrase":"passphrase"}'

// Result
{
    "hash":"00a631ebf5e1b02e9d8ad3f714d8b4ae64aca211e65323d41469d233270c9dc5"
}
```
***
