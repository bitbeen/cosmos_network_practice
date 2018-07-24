# cosmos sdk

## 应用结构
### ABCI app
基本的ABCI接口允许tendermint使用事务驱动应用程序状态机.


### Base app

通过multiStore实现ABCI应用程序的持久化,使用路由来处理事务,目标是在存储和可扩展状态机之间提供安全接口，同时尽可能少地定义该状态机（保持对ABCI的真实性）

BaseApp要求通过功能键安装商店 - 处理程序只能访问为其提供密钥的商店。BaseApp可确保正确加载，缓存和提交所有商店。一个已安装的存储被认为是“主要” - 它包含最新的块头，我们可以从中找到并加载最新的状态。

BaseApp有两种处理程序类型 - AnteHandler和MsgHandler。前者是全局有效性检查（检查nonces，sigs和足够的余额以支付费用，例如适用于所有模块的所有事务的事物），后者是完整的状态转换函数。在CheckTx期间，状态转换函数仅应用于`checkTxState`，并且应该在运行任何昂贵的状态转换之前返回（这取决于每个开发人员）。它还需要返回估计的`gas`费用。

在DeliverTx期间，状态转换功能应用于确定区块链状态中`transaction`是否执行成功。

BaseApp负责管理传递给处理程序的上下文 - 它使块头可用，并为CheckTx和DeliverTx提供正确的存储。BaseApp与序列化格式完全无关。

＃

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
Ethermint是Base App的实现，其实现不依赖Base App，是基于go-ethereum做的