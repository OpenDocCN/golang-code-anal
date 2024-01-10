# `v2ray-core\infra\conf\general_test.go`

```
package conf_test

import (
    "encoding/json"  // 导入 JSON 编码解码包
    "testing"  // 导入测试包

    "github.com/golang/protobuf/proto"  // 导入 protobuf 包
    "v2ray.com/core/common"  // 导入 v2ray 核心通用包
    . "v2ray.com/core/infra/conf"  // 导入 v2ray 核心配置包
)

func loadJSON(creator func() Buildable) func(string) (proto.Message, error) {
    return func(s string) (proto.Message, error) {
        instance := creator()  // 创建一个实例
        if err := json.Unmarshal([]byte(s), instance); err != nil {  // 使用 JSON 解码字符串到实例
            return nil, err
        }
        return instance.Build()  // 返回实例的构建结果
    }
}

type TestCase struct {
    Input  string  // 测试用例输入
    Parser func(string) (proto.Message, error)  // 解析器函数
    Output proto.Message  // 期望输出
}

func runMultiTestCase(t *testing.T, testCases []TestCase) {
    for _, testCase := range testCases {  // 遍历测试用例
        actual, err := testCase.Parser(testCase.Input)  // 使用解析器解析输入
        common.Must(err)  // 检查错误
        if !proto.Equal(actual, testCase.Output) {  // 检查实际输出和期望输出是否相等
            t.Fatalf("Failed in test case:\n%s\nActual:\n%v\nExpected:\n%v", testCase.Input, actual, testCase.Output)  // 输出测试失败信息
        }
    }
}
```