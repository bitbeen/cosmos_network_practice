## Tx

通过Tx将Msg聚集在一起。虽然Msg包含应用程序中特定功能的内容，但用户提供的实际输入是序列化的Tx。应用程序可能有许多Msg接口实现，但它们应该只有一个实现Tx

```golang
// Transactions wrap messages.
type Tx interface {
	// Gets the Msgs.
	GetMsgs() []Msg
}

```
该`Tx`只是一个包装`[]Msg`，并可能包括额外的认证数据，如签名和帐户随机数。应用程序必须指定它们Tx的解码方式，因为这是应用程序的最终输入。我们稍后会详细讨论Tx类型，特别是在我们介绍时`StdTx`。

在第一个应用程序中，我们根本不会有任何身份验证。这可能在私有网络中有意义，其中访问由客户端TLS证书等替代方法控制，但一般来说，我们希望将身份验证嵌入到我们的状态机中。我们将在下一个应用程序中使用`Tx`。现在，Tx只需嵌入MsgSend并使用JSON：

```golang
/ Simple tx to wrap the Msg.
type app1Tx struct {
	MsgSend
}

// This tx only has one Msg.
func (tx app1Tx) GetMsgs() []sdk.Msg {
	return []sdk.Msg{tx.MsgSend}
}

// JSON decode MsgSend.
func txDecoder(txBytes []byte) (sdk.Tx, sdk.Error) {
	var tx app1Tx
	err := json.Unmarshal(txBytes, &tx)
	if err != nil {
		return nil, sdk.ErrTxDecode(err.Error())
	}
	return tx, nil
}

```