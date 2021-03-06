# Execution

我们现在完成了应用程序的核心逻辑！从这里开始，我们可以在Go中编写测试，用帐户初始化`store`并通过调用`app.DeliverTx`方法执行事务。

在实际设置中，应用程序将在Tendermint共识引擎之上作为`ABCI`应用程序运行。它将由`Genesis`文件初始化，并且它将由基础Tendermint共识所提交的事务块驱动。我们将更多地讨论ABCI以及稍后如何工作，但请随时查看 规范。我们还将了解如何将我们的应用程序连接到一整套组件，以便运行和使用实时区块链应用程序。

现在，我们注意到在收到交易时（通过`app.DeliverTx`）会发生以下事件序列：

- 序列化交易是由 `app.DeliverTx`
- 事务使用反序列化 `TxDecoder`
- 对于事务中的每条消息，运行 `msg.ValidateBasic()`
- 对于事务中的每条消息，加载适当的处理程序并使用消息执行它