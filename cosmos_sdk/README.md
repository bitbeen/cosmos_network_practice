# cosmos sdk

cosmos-sdk是一个构建Tendermint ABCI应用的框架，它允许开发人员通过cosmos-network开发跨链的应用程序，实现链链之间的互操作，cosmos-sdk通过golang语言实现.

## 目录结构
sdk包含在如下目录：
* `baseapp`:定义`ABCI`应用程序模板，以便Cosmos-sdk应用程序可以与tendrmint节点通信.
* `client`:`CLI`和`REST`服务组件用于与cosmos-sdk应用程序进行交互
* `examples`:一些应用程序的例子
* `server`:在Tendermint上运行用户程序的全节点服务器.
* `store`:he database of the SDK - a Merkle multistore supporting multiple types of underling Merkle key-value stores.
* `types`:sdk应用程序中的常见类型.
* `x`:核心扩展，定义了所有消息和处理程序.

## 应用结构

cosmos sdk包含很多不痛级别的应用,ABCI app、BaseApp、BasecoinApp和第三方开发者开发的app.

### ABCI app

基本的ABCI接口允许tendermint使用事务驱动应用程序状态机.


### Base app

通过multiStore实现ABCI应用程序的持久化,使用路由来处理事务,目标是在存储和可扩展状态机之间提供安全接口，同时尽可能少地定义该状态机（保持对ABCI的真实性）

BaseApp要求通过功能键安装商店 - 处理程序只能访问为其提供密钥的商店。BaseApp可确保正确加载，缓存和提交所有商店。一个已安装的存储被认为是“主要” - 它包含最新的块头，我们可以从中找到并加载最新的状态。

BaseApp有两种处理程序类型 - AnteHandler和MsgHandler。前者是全局有效性检查（检查nonces，sigs和足够的余额以支付费用，例如适用于所有模块的所有事务的事物），后者是完整的状态转换函数。在CheckTx期间，状态转换函数仅应用于`checkTxState`，并且应该在运行任何昂贵的状态转换之前返回（这取决于每个开发人员）。它还需要返回估计的`gas`费用。

在DeliverTx期间，状态转换功能应用于确定区块链状态中`transaction`是否执行成功。

BaseApp负责管理传递给处理程序的上下文 - 它使块头可用，并为CheckTx和DeliverTx提供正确的存储,BaseApp对于序列化格式是完全不可知的。


### Basecoin

Basecoine是一个完整的应用程序，完整的应用程序需要扩展SDK的核心模块才能实现处理程序功能

sdk的原生扩展适用于构建`cosmos zones` 可以使用`x`,BaseApp使用x/auth和x/bank扩展实现状态机，他定义了如何验证交易签名者以及转移token,此外，还使用了x/ibc.

BaseCoin和原生的`x`扩展使用go-amino来满足所有序列化需求包括交易和账户。


### 第三方app

用户开发APP时候可以基于Basecoin,拷贝源码下的目录examples/basecoin，然后根据自己的需求去修改：
* 向Accounts中添加字段
* 复制并修改handlers
* 为新的交易类型添加handler
* 添加新的store隔离handler 

cosmos采用基于basecoin,添加了store并且扩展handler来处理其他事物类型和逻辑，如高级存储逻辑和治理过程。


## Ethermint
