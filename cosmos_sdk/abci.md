# ABCI

`ABCI`允许应用的拜占庭容错可以有任意一种语言编写，ABCI可以认为是用户编写的应用和`Tendermint core`共识引擎之间的API,`ABCI`主要有三种消息类型，
- `DeliverTX`:链中的每笔交易都通过这个消息进行传送，应用需要基于当前的转态、医用协议和交易的加密证书去验证`DeliverTx`中的每笔交易，一个验证的交易需要去更新应用的状态。
- `CheckTx`:用于交易的验证，`Tendermint core`的内存池首先通过`CheckTx`检验一笔交易的有效性，并且只将有效的交易中继到其他的节点
- `Commit`:计算当前应用状态的一个加密保证，这个加密保证会被放到下一个区块头


ABCI socket连接：
- 内存池连接：CheckTx
- 共识连接:只有 commited 的 transactions 才会被 executed
- 用于查询应用状态

![ABCI](/assets/ABIC)

## ABCI设计
- 消息协议
 - protobuf
 - consensus make requests,application responds
- server/client
 - 共识引擎运行在客户端
 - app作为服务端
 - grpc和async raw bytes
- 区块协议

## initChain
## BeginBlock'
## EndBlock
## Commit
## Query
## CheckTx