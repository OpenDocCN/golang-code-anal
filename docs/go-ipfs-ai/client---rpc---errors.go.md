# `kubo\client\rpc\errors.go`

```go
package rpc

import (
    "errors" // 引入 errors 包，用于创建错误
    "strings" // 引入 strings 包，用于字符串操作
    "unicode/utf8" // 引入 unicode/utf8 包，用于处理 UTF-8 编码

    "github.com/ipfs/go-cid" // 引入 go-cid 包，用于处理 CID
    ipld "github.com/ipfs/go-ipld-format" // 引入 go-ipld-format 包，用于处理 IPLD 数据
    mbase "github.com/multiformats/go-multibase" // 引入 go-multibase 包，用于处理多种基数编码
)

// This file handle parsing and returning the correct ABI based errors from error messages
// 该文件处理解析并根据错误消息返回正确的 ABI 错误

type prePostWrappedNotFoundError struct {
    pre  string
    post string
    wrapped ipld.ErrNotFound
}

func (e prePostWrappedNotFoundError) String() string {
    return e.Error()
}

func (e prePostWrappedNotFoundError) Error() string {
    return e.pre + e.wrapped.Error() + e.post
}

func (e prePostWrappedNotFoundError) Unwrap() error {
    return e.wrapped
}

func parseErrNotFoundWithFallbackToMSG(msg string) error {
    err, handled := parseErrNotFound(msg) // 调用 parseErrNotFound 函数解析错误消息
    if handled {
        return err // 如果解析成功，返回错误
    }

    return errors.New(msg) // 如果解析失败，返回新的错误
}

func parseErrNotFoundWithFallbackToError(msg error) error {
    err, handled := parseErrNotFound(msg.Error()) // 调用 parseErrNotFound 函数解析错误消息
    if handled {
        return err // 如果解析成功，返回错误
    }

    return msg // 如果解析失败，返回原始错误
}

func parseErrNotFound(msg string) (error, bool) {
    if msg == "" {
        return nil, true // 如果消息为空，直接返回空值和 true
    }

    if err, handled := parseIPLDErrNotFound(msg); handled {
        return err, true // 调用 parseIPLDErrNotFound 函数解析错误消息，如果解析成功，返回错误和 true
    }

    if err, handled := parseBlockstoreNotFound(msg); handled {
        return err, true // 调用 parseBlockstoreNotFound 函数解析错误消息，如果解析成功，返回错误和 true
    }

    return nil, false // 如果无法解析，返回空值和 false
}

// Assume CIDs break on:
// - Whitespaces: " \t\n\r\v\f"
// - Semicolon: ";" this is to parse ipld.ErrNotFound wrapped in multierr
// - Double Quotes: "\"" this is for parsing %q and %#v formating.
const cidBreakSet = " \t\n\r\v\f;\"" // 定义 CID 分隔符集合

func parseIPLDErrNotFound(msg string) (error, bool) {
    // The patern we search for is:
    const ipldErrNotFoundKey = "ipld: could not find " /*CID*/
    // We try to parse the CID, if it's invalid we give up and return a simple text error.
    // We also accept "node" in place of the CID because that means it's an Undefined CID.

    keyIndex := strings.Index(msg, ipldErrNotFoundKey) // 在错误消息中查找指定的模式
    if keyIndex < 0 { // 如果索引小于0，表示未知错误
        return nil, false  // 返回空值和false
    }

    cidStart := keyIndex + len(ipldErrNotFoundKey)  // 计算CID的起始位置

    msgPostKey := msg[cidStart:]  // 获取消息中CID后面的部分
    var c cid.Cid  // 声明CID变量
    var postIndex int  // 声明后缀索引变量
    if strings.HasPrefix(msgPostKey, "node") {  // 如果消息后缀以"node"开头
        // 回退情况
        c = cid.Undef  // 将CID设置为未定义
        postIndex = len("node")  // 设置后缀索引为"node"的长度
    } else {
        postIndex = strings.IndexFunc(msgPostKey, func(r rune) bool {  // 否则，查找消息后缀中满足条件的索引
            return strings.ContainsAny(string(r), cidBreakSet)  // 返回消息后缀中包含在cidBreakSet中的字符的索引
        })
        if postIndex < 0 {  // 如果索引小于0
            // 没有断点，意味着字符串看起来像这样 something + "ipld: could not find bafy"
            postIndex = len(msgPostKey)  // 设置后缀索引为消息后缀的长度
        }

        cidStr := msgPostKey[:postIndex]  // 获取消息后缀中CID的部分

        var err error
        c, err = cid.Decode(cidStr)  // 解码CID字符串
        if err != nil {  // 如果有错误
            // 解码CID失败，放弃
            return nil, false  // 返回空值和false
        }

        // 检查CID是CIDv0还是base32 multibase，因为这是ipld.ErrNotFound.Error() -> cid.Cid.String()目前的情况
        if c.Version() != 0 {  // 如果CID的版本不是0
            baseRune, _ := utf8.DecodeRuneInString(cidStr)  // 解码CID字符串中的符文
            if baseRune == utf8.RuneError || baseRune != mbase.Base32 {  // 如果符文是错误的或者不是base32
                // 不是我们期望的multibase，放弃
                return nil, false  // 返回空值和false
            }
        }
    }

    err := ipld.ErrNotFound{Cid: c}  // 创建一个包含CID的ErrNotFound错误
    pre := msg[:keyIndex]  // 获取消息中关键索引之前的部分
    post := msgPostKey[postIndex:]  // 获取消息中关键索引之后的部分

    if len(pre) > 0 || len(post) > 0 {  // 如果前缀部分或后缀部分的长度大于0
        return prePostWrappedNotFoundError{  // 返回包含前缀、后缀和错误的结构体
            pre:     pre,
            post:    post,
            wrapped: err,
        }, true
    }

    return err, true  // 返回错误和true
// 定义一个简单的错误类型，将 msg 作为 Error() 返回。
// 但是当调用 Is(err) 时，它也匹配 ipld.ErrNotFound。
// 这是为了保持与使用 string.Contains(err.Error(), "blockstore: block not found") 的代码和使用 ipld.ErrNotFound 的代码的兼容性。
type blockstoreNotFoundMatchingIPLDErrNotFound struct {
    msg string
}

func (e blockstoreNotFoundMatchingIPLDErrNotFound) String() string {
    return e.Error()
}

func (e blockstoreNotFoundMatchingIPLDErrNotFound) Error() string {
    return e.msg
}

func (e blockstoreNotFoundMatchingIPLDErrNotFound) Is(err error) bool {
    _, ok := err.(ipld.ErrNotFound)
    return ok
}

// 解析 blockstoreNotFound 错误信息
func parseBlockstoreNotFound(msg string) (error, bool) {
    // 如果 msg 中不包含 "blockstore: block not found"，则返回 nil 和 false
    if !strings.Contains(msg, "blockstore: block not found") {
        return nil, false
    }
    // 否则返回 blockstoreNotFoundMatchingIPLDErrNotFound 结构体实例和 true
    return blockstoreNotFoundMatchingIPLDErrNotFound{msg: msg}, true
}
```