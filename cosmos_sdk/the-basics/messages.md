## Messages

消息是应用程序状态机的主要输入，通过Messages定义`Transaction`的内容，Messages可以包含任意的信息，程序员可以通过实现Msg接口来创建`Messages`

```golang

type Msg interface {
    // Return the message type.
    // Must be alphanumeric or empty.
    // Must correspond to name of message handler (XXX).
    //返回消息的类型
    //返回值要么26个字母构成的字符中，要么为空
    //返回值必须对应于消息处理程序的名称
    Type() string

    // ValidateBasic does a simple validation check that
    // doesn't require access to any other information.
    //简单验证，检查是否有权限访问其他信息
    ValidateBasic() error

    // Get the canonical byte representation of the Msg.
    // This is what is signed.
    //获取签名后的字节数
    GetSignBytes() []byte

    // Signers returns the addrs of signers that must sign.
    // CONTRACT: All signatures must be present to be valid.
    // CONTRACT: Returns addrs in some deterministic order.
    //获取签名者
    //所有签名者必须存在才有效。
    //以某种确定性顺序返回addrs。
    GetSigners() []AccAddress
}
```
Msg允许messages定义基本的验证，和需要签名的数据，以及谁会去签名。

```golang
// MsgSend to send coins from Input to Output
type MsgSend struct {
	From   sdk.AccAddress `json:"from"`
	To     sdk.AccAddress `json:"to"`
	Amount sdk.Coins   `json:"amount"`
}

// Implements Msg.
func (msg MsgSend) Type() string { return "bank" }

// Implements Msg. JSON encode the message.
func (msg MsgSend) GetSignBytes() []byte {
	bz, err := json.Marshal(msg)
	if err != nil {
		panic(err)
	}
	return bz
}

// Implements Msg. Return the signer.
func (msg MsgSend) GetSigners() []sdk.AccAddress {
	return []sdk.AccAddress{msg.From}
}


// Implements Msg. Ensure the addresses are good and the
// amount is positive.
func (msg MsgSend) ValidateBasic() sdk.Error {
	if len(msg.From) == 0 {
		return sdk.ErrInvalidAddress("From address is empty")
	}
	if len(msg.To) == 0 {
		return sdk.ErrInvalidAddress("To address is empty")
	}
	if !msg.Amount.IsPositive() {
		return sdk.ErrInvalidCoins("Amount is not positive")
	}
	return nil
}
```
ValidateBasic会自动被SDK调用


