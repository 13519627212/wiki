# Nebulas 101 - 03 编写并运行智能合约

今天我们会学习怎样在Nebulas中编写、部署并执行智能合约。

## 准备工作

在进入智能合约之前，先温习下先前学过的内容：
1. 安装、编译并启动neb应用
2. 创建钱包地址，设置coinbase，并开始挖矿
3. 查询neb节点信息、钱包地址余额等
4. 发送转账交易，并验证交易是否成功

如果对上述的内容有疑惑的同学可以重新去学习前面的章节，接下来我们会通过下面的步骤来学习和使用智能合约：
1. 编写智能合约
2. 部署智能合约
3. 调用智能合约，验证合约执行结果


## 编写智能合约

跟以太坊类似，Nebulas实现了NVM虚拟机来运行智能合约，NVM的实现使用了JavaScript V8引擎，所以当前的开发版，我们可以使用JavaScript、TypeScript来编写智能合约。

编写智能合约的简要规范：
1. 智能合约代码必须是一个Prototype的对象；
2. 智能合约代码必须有一个init()的方法，这个方法只会在部署的时候被执行一次；
3. 智能合约里面的私有方法是以_开头的方法，私有方法不能被外部直接调用；

下面我们使用JavaScript来编写第一个智能合约：银行保险柜。
这个智能合约需要实现以下功能：
1. 用户可以向这个银行保险柜存钱。
2. 用户可以从这个银行保险柜取钱。

智能合约示例：

```js
"use strict";

var BankVaultContract = function() {
   // 星云链的智能合约运行环境内置了存储对象LocalContractStorage, 可以存储JavaScript对象
   LocalContractStorage.defineMapProperty(this, "bankVault");
};

BankVaultContract.prototype = {
   init:function() {},
   save:function() {
       var deposit = this.bankVault.get(Blockchain.transaction.from);
       var value = new BigNumber(Blockchain.transaction.value);
       if (deposit != null && deposit.balance.length > 0) {
           var balance = new BigNumber(deposit.balance);
           value = value.plus(balance);
       }
       var content = {
           balance:value.toString()
       };
       this.bankVault.put(Blockchain.transaction.from, content);
   },
   takeout:function(amount) {
       var deposit = this.bankVault.get(Blockchain.transaction.from);
       if (deposit == null) {
           return 0;
       }
       var balance = new BigNumber(deposit.balance);
       var value = new BigNumber(amount);
       if (balance.lessThan(value)) {
           return 0;
       }
       var result = Blockchain.transfer(Blockchain.transaction.from, value);
       if (result > 0) {
           deposit.balance = balance.dividedBy(value).toString();
           this.bankVault.put(Blockchain.transaction.from, deposit);
       }
       return result;
   }
};

module.exports = BankVaultContract;

```
上面智能合约的示例可以看到，`BankVaultContract`是一个prototype对象，这个对象有一个init()方法，满足了我们说的编写智能合约最基础的规范。
`BankVaultContract`实现了另外两个方法：
- save(): 用户可以通过调用save()方法向银行保险柜存钱；
- takeout(): 用户可以通过调用takeout()方法向银行保险柜取钱；

上面的合约代码用到了内置的`Blockchain`对象和内置的`BigNumber()`方法，下面我们来逐行拆解分析合约代码：
save():
```js
// 从保险柜查看合约的余额信息
var deposit = this.bankVault.get(Blockchain.transaction.from);
// 用户本次存钱的金额
var value = new BigNumber(Blockchain.transaction.value);
if (deposit != null && deposit.balance.length > 0) {
   var balance = new BigNumber(deposit.balance);
   // 更新保险柜的余额信息（新的余额=之前的余额+本次存钱的金额）
   value = value.plus(balance);
}
var content = {
   balance:value.toString()
};
// 将新的余额信息存储到链上
this.bankVault.put(Blockchain.transaction.from, content);
```

takeout():
```js
// 从保险柜的余额
var deposit = this.bankVault.get(Blockchain.transaction.from);
// 如果余额信息不存在，直接返回失败
if (deposit == null) {
   return 0;
}
var balance = new BigNumber(deposit.balance);
var value = new BigNumber(amount);
// 如果保险柜余额少于待取的金额，直接返回失败
if (balance.lessThan(value)) {
   return 0;
}
// 调用blockchain的transfer接口，将待取金额赚到用户的钱包地址
var result = Blockchain.transfer(Blockchain.transaction.from, value);
if (result > 0) {
   // 更新保险柜的余额（新的余额=之前的余额-待取金额）
   deposit.balance = balance.dividedBy(value).toString();
   // 将新的余额信息存储到链上
   this.bankVault.put(Blockchain.transaction.from, deposit);
}
```

## 部署智能合约
上面介绍了在Nebulas中怎么去编写一个智能合约，现在我们需要把智能合约部署到链上。
前面有介绍过用户如何在Nebulas中进行转账交易，我们使用sendTransation()接口来发起一笔转账交易。在Nebulas中部署一个智能合约其实也是发送一个transation来实现，即也是通过调用sendTransation()接口，只是参数不一样。
```js
sendTransation(from, to, nonce, source, args)
```
我们约定：如果from和to是同一个地址，就认为是部署一个智能合约。
source：待部署的智能合约的源代码
args：部署智能合约用到的参数

使用curl方式部署智能合约示例：
```
// Request
curl -i -H 'Accept: application/json' -X POST http://localhost:8090/v1/transaction -H 'Content-Type: application/json' -d '{"from":"8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf","to":"8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf","nonce":1,"source":"\"use strict\";var BankVaultContract=function(){LocalContractStorage.defineMapProperty(this,\"bankVault\")};BankVaultContract.prototype={init:function(){},save:function(){var deposit=this.bankVault.get(Blockchain.transaction.from);var value=new BigNumber(Blockchain.transaction.value);if(deposit!=null&&deposit.balance.length>0){var balance=new BigNumber(deposit.balance);value=value.plus(balance)}var content={balance:value.toString()};this.bankVault.put(Blockchain.transaction.from,content)},takeout:function(amount){var deposit=this.bankVault.get(Blockchain.transaction.from);if(deposit==null){return 0}var balance=new BigNumber(deposit.balance);var value=new BigNumber(amount);if(balance.lessThan(value)){return 0}var result=Blockchain.transfer(Blockchain.transaction.from,value);if(result>0){deposit.balance=balance.dividedBy(value).toString();this.bankVault.put(Blockchain.transaction.from,deposit)}return result}};module.exports=BankVaultContract;", "args":""}'

// Result
{
    "txhash":"9ea7c434f717123ef335fc4e74671996a95b3dcef7873cbe77dc679d6b500a8c",
    "contract_address":"a111489535426e93883981f6a3de84ada9bdf2fbab714079"
}
```
部署智能合约的返回值是transaction的hash地址`txhash`（蓝色标识）和合约的部署地址`contract_address`（红色标识）。
得到返回值并不能保证合约已经部署成功，因为sendTransaction()是一个异步的过程，需要经过矿工打包，正如之前的转账交易一样，转账并不能实时到账，依赖矿工打包的速度，所以需要等待一段时间（约1分钟），然后可以通过查询合约地址的信息或者调用智能合约来验证合约是否部署成功。

## 验证合约是否部署成功
在部署智能合约的时候得到了合约的地址`contract_address`，我们可以很方便的使用console控制台查询合约的地址信息来验证合约是否部署成功。

如上图所示，如果我们通过合约的地址可以查询到合约的信息，就表示合约部署成功了。

## 调用智能合约
在Nebulas中调用智能合约的方式也很简单，可以通过rpc接口call()方法来调用智能合约。
```js
call(from, to, nonce, value, function_name, args)
```
from: 用户钱包地址
to: 智能合约地址
nonce: 用户transaction标识，顺序增长
value: 调用智能合约用于转账的金额
function_name: 待调用的方法
args: 调用智能合约方法用到的参数

调用智能合约的save()方法：
```js
// Request
curl -i -H 'Accept: application/json' -X POST http://localhost:8090/v1/call -H 'Content-Type: application/json' -d '{"from":"9341709022928b38dae1f9e1cfbad25611e81f736fd192c5","to":"a111489535426e93883981f6a3de84ada9bdf2fbab714079","value": "100","nonce":2,"function":"save","args":""}'

// Result
{
  "hash": "b425abe633c4a8896aac0d02d7c75ae5eaa38318dd4c19ba3eec2f8be4bd5050"
}
```
智能合约的调用本质也是提交一个transation，所以也依赖矿工打包，矿工将交易打包成功以后调用才算成功，所以智能合约的调用也不是立即生效。我们需要等待一段时间（约一分钟），然后可以验证我们的调用是否成功。
上面我们调用save()方法向银行保险柜存储了金额100的资金，需要先从用户的余额扣除100，所以有个转账的过程，转账的金额需要通过value字段来传递。合约调用之后，只需要验证智能合约的地址余额是否是100。
通过console控制台可以很方便的查询到当前智能合约地址的金额：


调用智能合约的takeout()方法：

```js
// Request
curl -i -H 'Accept: application/json' -X POST http://localhost:8090/v1/call -H 'Content-Type: application/json' -d '{"from":"9341709022928b38dae1f9e1cfbad25611e81f736fd192c5","to":"a111489535426e93883981f6a3de84ada9bdf2fbab714079","nonce":3,"function":"takeout","args":"[50]"}'

// Result
{
  "hash": "03bd2bbeafa03e2432d774a2b52e570b0f2e615b8a6c457b0e1ae4668faf1a15"
}
```
上面takeout()方法与save()方法有所不同，只是从保险柜取出50的金额，将取出的金额转给用户是智能合约内部的操作，所以value参数不需要有值，取出的金额是操作的智能合约相关的参数，所以通过args参数来传递。
然后我们需要验证当前智能合约地址的余额是不是50：


上图可以看到智能合约调用结果无误，智能合约的部署到调用都是成功的。
