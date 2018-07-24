# Transactions

在之前的应用程序中，我们构建了一个简单的bank用于发送硬币的消息类型和用于存储帐户的一个商店。在这里我们建立App2，App1通过介绍扩展

* 用于发行新硬币的新消息类型
* 一个新的硬币元数据商店（比如谁可以发行硬币）
* 要求交易包括有效签名
  在此过程中，我们将介绍Amino进行编码和解码事务，并介绍AnteHandler进行处理。


## message

```golang
// MsgIssue to allow a registered issuer
// to issue new coins.
type MsgIssue struct {
    Issuer   sdk.AccAddress
    Receiver sdk.AccAddress
    Coin     sdk.Coin
}

// Implements Msg.
func (msg MsgIssue) Type() string { return "issue" }
```

> 请注意该`Type()`方法返回`"issue"`，因此此消息的类型不同，并且将由不同的处理程序执行MsgSend。其他方法MsgIssue类似于MsgSend


## Handler

我们需要一个新的处理程序来支持新的消息类型。根据`MsgIssue`发行商店中的信息，它只检查给定硬币类型的发件人是否是正确的发行人：

```golang
// Handle MsgIssue
func handleMsgIssue(keyIssue *sdk.KVStoreKey, keyAcc *sdk.KVStoreKey) sdk.Handler {
    return func(ctx sdk.Context, msg sdk.Msg) sdk.Result {
        issueMsg, ok := msg.(MsgIssue)
        if !ok {
            return sdk.NewError(2, 1, "MsgIssue is malformed").Result()
        }

        // Retrieve stores
        issueStore := ctx.KVStore(keyIssue)
        accStore := ctx.KVStore(keyAcc)

        // Handle updating coin info
        if res := handleIssuer(issueStore, issueMsg.Issuer, issueMsg.Coin); !res.IsOK() {
            return res
        }

        // Issue coins to receiver using previously defined handleTo function
        if res := handleTo(accStore, issueMsg.Receiver, []sdk.Coin{issueMsg.Coin}); !res.IsOK() {
            return res
        }

        return sdk.Result{
            // Return result with Issue msg tags
            Tags: issueMsg.Tags(),
        }
    }
}

func handleIssuer(store sdk.KVStore, issuer sdk.AccAddress, coin sdk.Coin) sdk.Result {
    // the issuer address is stored directly under the coin denomination
    denom := []byte(coin.Denom)
    infoBytes := store.Get(denom)
    if infoBytes == nil {
        return sdk.ErrInvalidCoins(fmt.Sprintf("Unknown coin type %s", coin.Denom)).Result()
    }

    var coinInfo coinInfo
    err := json.Unmarshal(infoBytes, &coinInfo)
    if err != nil {
        return sdk.ErrInternal("Error when deserializing coinInfo").Result()
    }

    // Msg Issuer is not authorized to issue these coins
    if !bytes.Equal(coinInfo.Issuer, issuer) {
        return sdk.ErrUnauthorized(fmt.Sprintf("Msg Issuer cannot issue tokens: %s", coin.Denom)).Result()
    }

    return sdk.Result{}
}

// coinInfo stores meta data about a coin
type coinInfo struct {
    Issuer sdk.AccAddress `json:"issuer"`
}
```

> 注意我们已经介绍了coinInfo存储每个硬币的发行者地址。我们JSON序列化此类型并将其直接存储在颁发者store的面额下。我们当然可以在此周围添加更多字段和逻辑，例如包括当前存在的硬币供应，并强制执行最大供应.



## Amino

现在我们有两个实现Msg，我们不会事先知道哪个类型包含在序列化的`Tx`中。理想情况下，我们会使用`Msg`在`Tx`接口，但JSON解码器无法解码为接口类型。实际上，没有标准的方法可以在Go中解组接口。这是我们建立Amino primary的主要原因之一 。

虽然SDK开发人员可以编码事务和状态对象，，Amino是推荐的格式。Amino的目标是改进最新版本的Protocol Buffers ,` proto3`. 为此，Amino与proto3的子集兼容。除了`oneof`

虽然oneof提供了联合类型，但Amino旨在提供接口。主要区别在于联合类型，您必须预先知道所有类型。但任何人都可以随时随地实现接口类型。

为了实现接口类型，Amino允许接口的任何具体实现注册一个全局唯一的名称，只要序列化类型，该名称就会被携带。这允许Amino无缝地反序列化为接口类型！

SDK中Amino的主要用途是用于实现Msg接口的消息 。通过使用不同的名称注册每个消息，它们每个都被赋予不同的Amino前缀，允许在事务中容易地区分它们。

Amino也可用于持久存储接口。

要使用Amino，只需创建一个编解码器，然后注册类型：

```golang

func NewCodec() *wire.Codec {
	cdc := wire.NewCodec()
	cdc.RegisterInterface((*sdk.Msg)(nil), nil)
	cdc.RegisterConcrete(MsgSend{}, "example/MsgSend", nil)
	cdc.RegisterConcrete(MsgIssue{}, "example/MsgIssue", nil)
	return cdc
}

```
>Amino支持二进制和JSON格式的编码和解码


## Tx

现在我们正在使用Amino，我们可以在`Tx`中嵌入`Msg`。我们还可以添加公钥和签名进行身份验证

```golang
// Simple tx to wrap the Msg.
type app2Tx struct {
    sdk.Msg
    
    PubKey    crypto.PubKey
    Signature crypto.Signature
}

// This tx only has one Msg.
func (tx app2Tx) GetMsgs() []sdk.Msg {
	return []sdk.Msg{tx.Msg}
}

```
>我们不再需要自定义TxDecoder功能，因为我们只使用Amino编解码器！

## AntHandler

既然我们的实现Tx包含的不仅仅是Msg，我们还需要指定如何验证和处理额外的信息。这是作用的AnteHandler。ante这里的单词表示“之前”，因为 AnteHandler它在a之前运行Handler。虽然应用程序可以有多个处理程序，每个消息集一个，但它只能有一个AnteHandler与其单个实现相对应的处理程序Tx。

AnteHandler类似于处理程序：

```golang
type AnteHandler func(ctx Context, tx Tx) (newCtx Context, result Result, abort bool)

```

与Handler一样，AnteHandler根据授予的任何功能键采用Context来限制其对商店的访问。Msg然而，它需要一个而不是一个Tx。

像Handler一样，AnteHandler返回一个Result类型，但它也返回一个新的 Context和一个abort bool。

因为App2，我们只是检查PubKey是否与地址匹配，并且Signature使用PubKey进行验证：
## App2
```golang
// Simple anteHandler that ensures msg signers have signed.
// Provides no replay protection.
func antehandler(ctx sdk.Context, tx sdk.Tx) (_ sdk.Context, _ sdk.Result, abort bool) {
	appTx, ok := tx.(app2Tx)
	if !ok {
		// set abort boolean to true so that we don't continue to process failed tx
		return ctx, sdk.ErrTxDecode("Tx must be of format app2Tx").Result(), true
	}

	// expect only one msg in app2Tx
	msg := tx.GetMsgs()[0]

	signerAddrs := msg.GetSigners()

	if len(signerAddrs) != len(appTx.GetSignatures()) {
		return ctx, sdk.ErrUnauthorized("Number of signatures do not match required amount").Result(), true
	}

	signBytes := msg.GetSignBytes()
	for i, addr := range signerAddrs {
		sig := appTx.GetSignatures()[i]

		// check that submitted pubkey belongs to required address
		if !bytes.Equal(sig.PubKey.Address(), addr) {
			return ctx, sdk.ErrUnauthorized("Provided Pubkey does not match required address").Result(), true
		}

		// check that signature is over expected signBytes
		if !sig.PubKey.VerifyBytes(signBytes, sig.Signature) {
			return ctx, sdk.ErrUnauthorized("Signature verification failed").Result(), true
		}
	}

	// authentication passed, app to continue processing by sending msg to handler
	return ctx, sdk.Result{}, false
}

```

与App1相比，这里的主要区别在于，我们对第二个商店使用第二个功能键，该商店仅传递给第二个处理程序，即 handleMsgIssue。第一个handleMsgSend商店无法访问这第二家商店，无法读取或写入，确保关注点的强烈分离。

>另请注意，我们不需要在SetTxDecoder这里使用- 现在我们正在使用Amino，我们只需创建一个编解码器，在编解码器上注册我们的类型，然后将编解码器传递给NewBaseApp。SDK为我们完成剩下的工作

## Conclusion

我们通过添加用于发行硬币的新消息类型以及通过检查签名来扩展我们的第一个应用程序。我们学习了如何使用Amino解码到接口类型，允许我们支持多种Msg类型，并且我们学习了如何使用AnteHandler来验证事务。

不幸的是，我们的应用程序仍然不安全，因为任何有效的事务都可以多次重放以消耗某人帐号！此外，验证签名和防止重放不是开发人员应该考虑的事情。

在下一节中，我们介绍了内置SDK模块`auth`和`bank`，分别为我们所有的交易认证和coin transfering提供必要的安全.



