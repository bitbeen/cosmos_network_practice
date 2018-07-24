## Handler

定义App1的handler
```golang
// Handle MsgSend.
// NOTE: msg.From, msg.To, and msg.Amount were already validated
// in ValidateBasic().
func handleMsgSend(ctx sdk.Context, key *sdk.KVStoreKey, msg MsgSend) sdk.Result {
	// Load the store.
	store := ctx.KVStore(key)

	// Debit from the sender.
	if res := handleFrom(store, msg.From, msg.Amount); !res.IsOK() {
		return res
	}

	// Credit the receiver.
	if res := handleTo(store, msg.To, msg.Amount); !res.IsOK() {
		return res
	}

	// Return a success (Code 0).
	// Add list of key-value pair descriptors ("tags").
	return sdk.Result{
		Tags: msg.Tags(),
	}
}

```

我们只有一个消息类型，因此只需定义一个特定于消息的函数，`handleMsgSend`。

请注意，此处理程序可以无限制地访问功能键指定的存储keyAcc，因此必须定义要存储的内容以及如何对其进行编码。稍后，我们将引入更高级别的抽象，以便处理程序在他们能做的事情上受到限制。对于第一个示例，我们使用一个JSON编码的简单帐户：

```golang
type appAccount struct {
	Coins sdk.Coins `json:"coins"`
}

```
Coins是SDK为多资产帐户提供的有用类型。我们可以在这里使用一个整数作为单一硬币类型。

现在我们已经准备好处理MsgSend的两个部分：

```golang
func handleFrom(store sdk.KVStore, from sdk.AccAddress, amt sdk.Coins) sdk.Result {
	// Get sender account from the store.
	accBytes := store.Get(from)
	if accBytes == nil {
		// Account was not added to store. Return the result of the error.
		return sdk.NewError(2, 101, "Account not added to store").Result()
	}

	// Unmarshal the JSON account bytes.
	var acc appAccount
	err := json.Unmarshal(accBytes, &acc)
	if err != nil {
		// InternalError
		return sdk.ErrInternal("Error when deserializing account").Result()
	}

	// Deduct msg amount from sender account.
	senderCoins := acc.Coins.Minus(amt)

	// If any coin has negative amount, return insufficient coins error.
	if !senderCoins.IsNotNegative() {
		return sdk.ErrInsufficientCoins("Insufficient coins in account").Result()
	}

	// Set acc coins to new amount.
	acc.Coins = senderCoins

	// Encode sender account.
	accBytes, err = json.Marshal(acc)
	if err != nil {
		return sdk.ErrInternal("Account encoding error").Result()
	}

	// Update store with updated sender account
	store.Set(from, accBytes)
	return sdk.Result{}
}

func handleTo(store sdk.KVStore, to sdk.AccAddress, amt sdk.Coins) sdk.Result {
	// Add msg amount to receiver account
	accBytes := store.Get(to)
	var acc appAccount
	if accBytes == nil {
		// Receiver account does not already exist, create a new one.
		acc = appAccount{}
	} else {
		// Receiver account already exists. Retrieve and decode it.
		err := json.Unmarshal(accBytes, &acc)
		if err != nil {
			return sdk.ErrInternal("Account decoding error").Result()
		}
	}

	// Add amount to receiver's old coins
	receiverCoins := acc.Coins.Plus(amt)

	// Update receiver account
	acc.Coins = receiverCoins

	// Encode receiver account
	accBytes, err := json.Marshal(acc)
	if err != nil {
		return sdk.ErrInternal("Account encoding error").Result()
	}

	// Update store with updated receiver account
	store.Set(to, accBytes)
	return sdk.Result{}
}

```

处理程序很简单。我们首先使用授权的功能密钥从上下文加载KVStore。然后我们进行两个状态转换：一个用于发送者，一个用于接收者。每一个都涉及JSON解组来自store的帐户字节，将CoinsJSON和JSON编组回store。


