# Transactions

在之前的应用程序中，我们构建了一个简单的bank用于发送硬币的消息类型和用于存储帐户的一个商店。在这里我们建立App2，App1通过介绍扩展
- 用于发行新硬币的新消息类型
- 一个新的硬币元数据商店（比如谁可以发行硬币）
- 要求交易包括有效签名
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
>请注意该Type()方法返回"issue"，因此此消息的类型不同，并且将由不同的处理程序执行MsgSend。其他方法MsgIssue类似于MsgSend

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
## Amino
## Tx
## AntHandler
## App2
## Conclusion
