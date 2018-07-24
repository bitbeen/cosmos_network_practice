

## Result

Hander拥有两个参数Contex和Msg以及返回一个Result,Result is motivated by the corresponding ABCI result，它包含有关Transaction的返回值、错误信息、日志和元数据。

```golang
// Result is the union of ResponseDeliverTx and ResponseCheckTx.
type Result struct {

	// Code is the response code, is stored back on the chain.
	Code ABCICodeType

	// Data is any data returned from the app.
	Data []byte

	// Log is just debug information. NOTE: nondeterministic.
	Log string

	// GasWanted is the maximum units of work we allow this tx to perform.
	GasWanted int64

	// GasUsed is the amount of gas actually consumed. NOTE: unimplemented
	GasUsed int64

	// Tx fee amount and denom.
	FeeAmount int64
	FeeDenom  string

	// Tags are used for transaction indexing and pubsub.
	Tags Tags
}

```

现在，请注意，该 0值的值Code被视为成功，而其他所有值都是失败的。该Tags可包含有关将使我们能够很容易地查找，涉及到特定的账户或交易行为交易的元数据


