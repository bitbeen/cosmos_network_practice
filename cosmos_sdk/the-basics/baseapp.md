## BaseApp

BaseApp只是在`Tendermint ABCI`上的换一个抽象，通过处理常见的低级别问题简化了应用程序开发。它充当SDK应用程序的两个关键组件之间的中介：`store`和`message handler`,Baseapp实现接口`abci.Appication`

以下是App1的完整设置：
```
func NewApp1(logger log.Logger, db dbm.DB) *bapp.BaseApp {
    cdc := wire.NewCodec()

    // Create the base application object.
    app := bapp.NewBaseApp(app1Name, cdc, logger, db)

    // Create a capability key for accessing the account store.
    keyAccount := sdk.NewKVStoreKey("acc")

    // Determine how transactions are decoded.
    app.SetTxDecoder(txDecoder)

    // Register message routes.
    // Note the handler receives the keyAccount and thus
    // gets access to the account store.
    app.Router().
    	AddRoute("bank", NewApp1Handler(keyAccount))

    // Mount stores and load the latest state.
    app.MountStoresIAVL(keyAccount)
    err := app.LoadLatestVersion(keyAccount)
    if err != nil {
    	cmn.Exit(err.Error())
    }
    return app
}

```
每个应用程序都将具有定义应用程序设置的功能。它通常包含在app.go文件中。我们将在本教程后面讨论如何用CLI，REST API与app进行通信。

在这里，我们只有一个Msg类型`ank`，一个store服务于account和handler。通过为处理程序提供访问store的功能密钥，在未来的应用程序中，我们将拥有多个商店和处理程序，并不是每个处理程序都可以访问每个商店。

设置`transaction`解码器和消息处理路由后，最后一步是挂载store并加载最新版本。由于我们只有一个商店，我们只挂一个
