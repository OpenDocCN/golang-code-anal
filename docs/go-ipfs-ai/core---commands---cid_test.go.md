# `kubo\core\commands\cid_test.go`

```
package commands

import (
    "testing"  // 导入测试包

    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入命令包
    "github.com/multiformats/go-multibase"  // 导入多重基础包
)

func TestCidFmtCmd(t *testing.T) {
    t.Parallel()  // 并行执行测试用例

    // Test 'error when -v 0 is present and a custom -b is passed'  // 测试当 -v 0 存在并且传递了自定义 -b 时出现错误
    # 运行测试用例 "ipfs cid format <cid> -b z -v 0"
    t.Run("ipfs cid format <cid> -b z -v 0", func(t *testing.T) {
        # 并行执行测试
        t.Parallel()

        # 定义结构体 testV0PresentAndCustomBaseCase
        type testV0PresentAndCustomBaseCase struct {
            MultibaseName  string  # 多重基名称
            ExpectedErrMsg string  # 预期错误信息
        }

        # 定义测试用例数组 testV0PresentAndCustomBaseCases
        var testV0PresentAndCustomBaseCases []testV0PresentAndCustomBaseCase

        # 遍历 multibase.EncodingToStr
        for _, e := range multibase.EncodingToStr {
            # 定义 testCase 结构体
            var testCase testV0PresentAndCustomBaseCase

            # 如果 e 等于 "base58btc"
            if e == "base58btc" {
                # 设置 testCase 的 MultibaseName 和 ExpectedErrMsg
                testCase.MultibaseName = e
                testCase.ExpectedErrMsg = ""
                # 将 testCase 添加到 testV0PresentAndCustomBaseCases 数组中
                testV0PresentAndCustomBaseCases = append(testV0PresentAndCustomBaseCases, testCase)
                # 继续下一次循环
                continue
            }
            # 设置 testCase 的 MultibaseName 和 ExpectedErrMsg
            testCase.MultibaseName = e
            testCase.ExpectedErrMsg = "cannot convert to CIDv0 with any multibase other than the implicit base58btc"
            # 将 testCase 添加到 testV0PresentAndCustomBaseCases 数组中
            testV0PresentAndCustomBaseCases = append(testV0PresentAndCustomBaseCases, testCase)
        }

        # 遍历 testV0PresentAndCustomBaseCases 数组
        for _, e := range testV0PresentAndCustomBaseCases {

            # 模拟请求
            req := &cmds.Request{
                Options: map[string]interface{}{
                    cidVerisonOptionName:   "0",
                    cidMultibaseOptionName: e.MultibaseName,
                    cidFormatOptionName:    "%s",
                },
            }

            # 响应发射器
            resp := cmds.ResponseEmitter(nil)

            # 调用 CidFmtCmd 函数，传入模拟请求和响应
            err := cidFmtCmd.Run(req, resp, nil)
            # 如果没有错误并且 MultibaseName 为 "base58btc"，则继续下一次循环
            if err == nil && e.MultibaseName == "base58btc" {
                continue
            }

            # 获取错误信息
            errMsg := err.Error()
            # 如果错误信息不等于预期错误信息，则输出错误信息
            if errMsg != e.ExpectedErrMsg {
                t.Errorf("Expected %s, got %s instead", e.ExpectedErrMsg, errMsg)
            }
        }
    })

    # 测试 'upgrade CID to v1 when passing a custom -b and no -v is specified'
    t.Run("ipfs cid format <cid-version-0> -b z", func(t *testing.T) {
        t.Parallel()

        type testImplicitVersionAndCustomMultibaseCase struct {
            Ver           string  # 定义测试用例结构体字段：版本号
            CidV1         string  # 定义测试用例结构体字段：CID 版本 1
            CidV0         string  # 定义测试用例结构体字段：CID 版本 0
            MultibaseName string  # 定义测试用例结构体字段：多重基数编码名称
        }

        var testCases = []testImplicitVersionAndCustomMultibaseCase{  # 定义测试用例切片
            {
                Ver:           "",  # 设置测试用例字段值：版本号为空字符串
                CidV1:         "zdj7WWwMSWGoyxYkkT7mHgYvr6tV8CYd77aYxxqSbg9HsiMcE",  # 设置测试用例字段值：CID 版本 1
                CidV0:         "QmPr755CxWUwt39C2Yiw4UGKrv16uZhSgeZJmoHUUS9TSJ",  # 设置测试用例字段值：CID 版本 0
                MultibaseName: "z",  # 设置测试用例字段值：多重基数编码名称为 "z"
            },
            {
                Ver:           "",  # 设置测试用例字段值：版本号为空字符串
                CidV1:         "CAFYBEIDI7ZABPGG3S63QW3AJG2XAZNE4NJQPN777WLWYRAIDG3TE5QFN3A======",  # 设置测试用例字段值：CID 版本 1
                CidV0:         "QmVQVyEijmLb2cBQrowNQsaPbnUnJhfDK1sYe3wepm6ySf",  # 设置测试用例字段值：CID 版本 0
                MultibaseName: "base32padupper",  # 设置测试用例字段值：多重基数编码名称为 "base32padupper"
            },
        }
        for _, e := range testCases {  # 遍历测试用例切片
            // Mock request  # 模拟请求
            req := &cmds.Request{  # 创建请求对象
                Options: map[string]interface{  # 设置请求对象的选项
                    cidVerisonOptionName:   e.Ver,  # 使用测试用例中的版本号设置选项值
                    cidMultibaseOptionName: e.MultibaseName,  # 使用测试用例中的多重基数编码名称设置选项值
                    cidFormatOptionName:    "%s",  # 设置格式选项值为 "%s"
                },
            }

            // Response emitter  # 响应发射器
            resp := cmds.ResponseEmitter(nil)  # 创建响应发射器对象

            // Call the CidFmtCmd function with the mock request and response  # 调用 CidFmtCmd 函数，传入模拟请求和响应
            err := cidFmtCmd.Run(req, resp, nil)  # 运行 CidFmtCmd 函数，并将结果赋值给 err

            if err != nil {  # 如果出现错误
                t.Error(err)  # 输出错误信息
            }
        }
    })
# 闭合前面的函数定义
```