# `v2ray-core\infra\conf\policy_test.go`

```go
package conf_test

import (
    "testing"

    "v2ray.com/core/common"
    . "v2ray.com/core/infra/conf"
)

func TestBufferSize(t *testing.T) {
    cases := []struct {  // 定义测试用例结构体
        Input  int32       // 输入缓冲区大小
        Output int32       // 期望输出缓冲区大小
    }{
        {
            Input:  0,     // 输入为0
            Output: 0,     // 期望输出为0
        },
        {
            Input:  -1,    // 输入为-1
            Output: -1,    // 期望输出为-1
        },
        {
            Input:  1,     // 输入为1
            Output: 1024,  // 期望输出为1024
        },
    }

    for _, c := range cases {  // 遍历测试用例
        bs := int32(c.Input)  // 将输入缓冲区大小转换为int32类型
        pConf := Policy{      // 创建Policy对象
            BufferSize: &bs,  // 设置缓冲区大小
        }
        p, err := pConf.Build()  // 构建Policy对象
        common.Must(err)         // 检查错误
        if p.Buffer.Connection != c.Output {  // 检查实际输出是否与期望输出相符
            t.Error("expected buffer size ", c.Output, " but got ", p.Buffer.Connection)  // 输出错误信息
        }
    }
}
```