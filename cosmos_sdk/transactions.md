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

> 请注意该Type\(\)方法返回"issue"，因此此消息的类型不同，并且将由不同的处理程序执行MsgSend。其他方法MsgIssue类似于MsgSend

## Handler

我们需要一个新的处理程序来支持新的消息类型。根据MsgIssue发行商店中的信息，它只检查给定硬币类型的发件人是否是正确的发行人：

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

> 注意我们已经介绍了coinInfo存储每个硬币的发行者地址的类型。我们JSON序列化此类型并将其直接存储在颁发者商店中的面额下。我们当然可以在此周围添加更多字段和逻辑，例如包括当前存在的硬币供应，并强制执行最大供应，但这仍然是读者的练习😃。



## Amino

现在我们有两个实现Msg，我们不会事先知道哪个类型包含在序列化中Tx。理想情况下，我们会Msg在Tx实现中使用 接口，但JSON解码器无法解码为接口类型。实际上，没有标准的方法可以在Go中解组接口。这是我们建立Amino primary的主要原因之一 。

虽然SDK开发人员可以编码事务和状态对象，但他们喜欢，Amino是推荐的格式。Amino的目标是改进最新版本的Protocol Buffers , proto3. 为此，Amino与proto3排除oneof关键字的子集兼容。

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

## AntHandler

## App2

## Conclusion



