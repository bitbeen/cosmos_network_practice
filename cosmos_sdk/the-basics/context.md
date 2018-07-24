## Conetext

SDK通过`Context`传播函数的公共信息，Context限制了基于对象功能键的KVStore访问。只有已明确拥有访问密钥的handler才能访问相应的存储。

```golang
// newFooHandler returns a Handler that can access a single store.
func newFooHandler(key sdk.StoreKey) sdk.Handler {
    return func(ctx sdk.Context, msg sdk.Msg) sdk.Result {
        store := ctx.KVStore(key)
        // ...
    }
}

```

Context在Golang context.Context之后建模 ，它已经在网络中间件和路由应用程序中无处不在，作为通过处理程序函数轻松传播请求上下文的一种手段。SDK对象上的许多方法都接收上下文作为第一个参数。

Context还包含区块块头，其中包含来自区块链的最新时间戳以及有关最新块的其他信息。
