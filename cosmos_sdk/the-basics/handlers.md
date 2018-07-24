## Handlers

现在我们有了消息类型和存储接口，我们可以通过handler定义我们自己的状态转换函数

```golang
// Handler defines the core of the state transition function of an application.
type Handler func(ctx Context, msg Msg) Result

```
通过message，handler获取环境信息（a Context），并返回一个Result。处理消息所需的所有信息都应在上下文中提供。

所有这些中的KVStore在哪里？在消息处理程序中访问KVStore受Context通过对象key的限制。只有给予明确访问store密钥的处理程序才能在消息处理过程中访问该store