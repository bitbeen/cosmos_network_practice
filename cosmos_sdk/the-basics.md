# 基础

通过构建APP1来介绍基本的组件,app1是一个银行小应用，用户拥有账户地址和账户，用户可以发送token，但是不能进行身份验证，通过json进行序列化.


## KVStore

KVStore是cosmos-sdk应用的基本存储层

```golang
type KVStore interface {
    Store

    // Get returns nil iff key doesn't exist. Panics on nil key.
    Get(key []byte) []byte

    // Has checks if a key exists. Panics on nil key.
    Has(key []byte) bool

    // Set sets the key. Panics on nil key.
    Set(key, value []byte)

    // Delete deletes the key. Panics on nil key.
    Delete(key []byte)

    // Iterator over a domain of keys in ascending order. End is exclusive.
    // Start must be less than end, or the Iterator is invalid.
    // CONTRACT: No writes may happen within a domain while an iterator exists over it.
    Iterator(start, end []byte) Iterator

    // Iterator over a domain of keys in descending order. End is exclusive.
    // Start must be greater than end, or the Iterator is invalid.
    // CONTRACT: No writes may happen within a domain while an iterator exists over it.
    ReverseIterator(start, end []byte) Iterator

 }

```
> 如果Key为nil,程序会进入恐慌

当前KVStore的主要实现是



## Handlers

现在我们有了消息类型和存储接口，我们可以通过handler定义我们自己的状态转换函数

## Conetext
SDK通过`Context`传播函数的公共信息，Context限制了基于对象功能键的KVStore访问。只有已明确访问密钥的处理程序才能访问相应的存储。

## Result

## Handler

## Tx

## BaseApp

## Execution

## Conclusion



