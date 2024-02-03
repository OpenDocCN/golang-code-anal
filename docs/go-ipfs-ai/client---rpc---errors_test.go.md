# `kubo\client\rpc\errors_test.go`

```go
package rpc

import (
    "errors"  // 导入 errors 包，用于处理错误
    "fmt"  // 导入 fmt 包，用于格式化输出
    "testing"  // 导入 testing 包，用于编写测试用例

    "github.com/ipfs/go-cid"  // 导入 go-cid 包，用于处理 CID
    ipld "github.com/ipfs/go-ipld-format"  // 导入 go-ipld-format 包，用于处理 IPLD
    mbase "github.com/multiformats/go-multibase"  // 导入 go-multibase 包，用于处理 multibase
    mh "github.com/multiformats/go-multihash"  // 导入 go-multihash 包，用于处理 multihash
)

var randomSha256MH = mh.Multihash{0x12, 0x20, 0x88, 0x82, 0x73, 0x37, 0x7c, 0xc1, 0xc9, 0x96, 0xad, 0xee, 0xd, 0x26, 0x84, 0x2, 0xc9, 0xc9, 0x5c, 0xf9, 0x5c, 0x4d, 0x9b, 0xc3, 0x3f, 0xfb, 0x4a, 0xd8, 0xaf, 0x28, 0x6b, 0xca, 0x1a, 0xf2}  // 定义 randomSha256MH 变量，存储 Multihash 值

func doParseIpldNotFoundTest(t *testing.T, original error) {
    originalMsg := original.Error()  // 获取原始错误的消息

    rebuilt := parseErrNotFoundWithFallbackToMSG(originalMsg)  // 调用 parseErrNotFoundWithFallbackToMSG 函数，传入原始消息进行重建

    rebuiltMsg := rebuilt.Error()  // 获取重建后的错误消息

    if originalMsg != rebuiltMsg {  // 比较原始消息和重建消息是否相同
        t.Errorf("expected message to be %q; got %q", originalMsg, rebuiltMsg)  // 输出错误信息
    }

    originalNotFound := ipld.IsNotFound(original)  // 检查原始错误是否为 NotFound
    rebuiltNotFound := ipld.IsNotFound(rebuilt)  // 检查重建后的错误是否为 NotFound
    if originalNotFound != rebuiltNotFound {  // 比较原始错误和重建后的错误是否相同
        t.Errorf("for %q expected Ipld.IsNotFound to be %t; got %t", originalMsg, originalNotFound, rebuiltNotFound)  // 输出错误信息
    }
}

func TestParseIPLDNotFound(t *testing.T) {
    t.Parallel()  // 并行执行测试用例

    if err := parseErrNotFoundWithFallbackToMSG(""); err != nil {  // 调用 parseErrNotFoundWithFallbackToMSG 函数，传入空字符串进行测试
        t.Errorf("expected empty string to give no error; got %T %q", err, err.Error())  // 输出错误信息
    }

    cidBreaks := make([]string, len(cidBreakSet))  // 创建一个与 cidBreakSet 长度相同的字符串切片
    for i, v := range cidBreakSet {  // 遍历 cidBreakSet
        cidBreaks[i] = "%w" + string(v)  // 将 %w 与 cidBreakSet 中的字符拼接，存入 cidBreaks 中
    }

    base58BTCEncoder, err := mbase.NewEncoder(mbase.Base58BTC)  // 创建 Base58BTC 编码器
    if err != nil {  // 检查是否有错误
        t.Fatalf("expected to find Base58BTC encoder; got error %q", err.Error())  // 输出错误信息
    }

    for _, wrap := range append(cidBreaks,  // 遍历 cidBreaks 切片
        "",
        "merkledag: %w",
        "testing: %w the test",
        "%w is wrong",
    ) {
        // 遍历错误数组，对每个错误进行处理
        for _, err := range [...]error{
            // 创建新的错误对象，提示找不到指定内容
            errors.New("ipld: could not find "),
            // 创建新的错误对象，提示找不到指定内容
            errors.New("ipld: could not find Bad_CID"),
            // 创建新的错误对象，测试只接受 CIDv0 和 base32 CIDs
            errors.New("ipld: could not find " + cid.NewCidV1(cid.Raw, randomSha256MH).Encode(base58BTCEncoder)),
            // 创建新的错误对象，提示网络连接超时
            errors.New("network connection timeout"),
            // 创建新的错误对象，指定未找到的 CID 为 cid.Undef
            ipld.ErrNotFound{Cid: cid.Undef},
            // 创建新的错误对象，指定未找到的 CID 为随机生成的 CIDv0
            ipld.ErrNotFound{Cid: cid.NewCidV0(randomSha256MH)},
            // 创建新的错误对象，指定未找到的 CID 为指定的 CIDv1
            ipld.ErrNotFound{Cid: cid.NewCidV1(cid.Raw, randomSha256MH)},
        } {
            // 如果有包装信息，则使用包装信息格式化错误对象
            if wrap != "" {
                err = fmt.Errorf(wrap, err)
            }
            // 执行解析 IPLD 未找到的测试
            doParseIpldNotFoundTest(t, err)
        }
    }
func TestBlockstoreNotFoundMatchingIPLDErrNotFound(t *testing.T) {
    // 定义一个测试函数，用于测试未找到匹配的 IPLD 错误是否能正确匹配未找到错误
    t.Parallel()

    // 如果 blockstoreNotFoundMatchingIPLDErrNotFound 类型不匹配 IsNotFound 接口
    if !ipld.IsNotFound(blockstoreNotFoundMatchingIPLDErrNotFound{}) {
        // 报错，期望 blockstoreNotFoundMatchingIPLDErrNotFound 能匹配 IsNotFound，实际得到 false
        t.Fatalf("expected blockstoreNotFoundMatchingIPLDErrNotFound to match ipld.IsNotFound; got false")
    }

    // 遍历不同的错误包装格式
    for _, wrap := range [...]string{
        "",
        "merkledag: %w",
        "testing: %w the test",
        "%w is wrong",
    } {
        // 遍历不同的错误类型
        for _, err := range [...]error{
            errors.New("network connection timeout"),
            blockstoreNotFoundMatchingIPLDErrNotFound{"blockstore: block not found"},
        } {
            // 如果包装格式不为空
            if wrap != "" {
                // 使用包装格式对错误进行格式化
                err = fmt.Errorf(wrap, err)
            }

            // 执行解析 IPLD 未找到错误的测试
            doParseIpldNotFoundTest(t, err)
        }
    }
}
```