# `v2ray-core\infra\conf\transport_test.go`

```
package conf_test

import (
    "encoding/json"  // 导入 JSON 包，用于处理 JSON 数据
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/golang/protobuf/proto"  // 导入 protobuf 包，用于处理 protobuf 数据
    "v2ray.com/core/common/protocol"  // 导入 protocol 包
    "v2ray.com/core/common/serial"  // 导入 serial 包
    . "v2ray.com/core/infra/conf"  // 导入 conf 包
    "v2ray.com/core/transport"  // 导入 transport 包
    "v2ray.com/core/transport/internet"  // 导入 internet 包
    "v2ray.com/core/transport/internet/headers/http"  // 导入 http 包
    "v2ray.com/core/transport/internet/headers/noop"  // 导入 noop 包
    "v2ray.com/core/transport/internet/headers/tls"  // 导入 tls 包
    "v2ray.com/core/transport/internet/kcp"  // 导入 kcp 包
    "v2ray.com/core/transport/internet/quic"  // 导入 quic 包
    "v2ray.com/core/transport/internet/tcp"  // 导入 tcp 包
    "v2ray.com/core/transport/internet/websocket"  // 导入 websocket 包
)

func TestSocketConfig(t *testing.T) {
    createParser := func() func(string) (proto.Message, error) {  // 创建一个函数，用于解析字符串并返回 proto.Message 和 error
        return func(s string) (proto.Message, error) {  // 返回一个函数，该函数解析字符串并返回 proto.Message 和 error
            config := new(SocketConfig)  // 创建一个 SocketConfig 对象
            if err := json.Unmarshal([]byte(s), config); err != nil {  // 使用 JSON 包解析字符串并将结果存储到 SocketConfig 对象中
                return nil, err  // 如果解析出错，返回 nil 和错误信息
            }
            return config.Build()  // 调用 SocketConfig 对象的 Build 方法并返回结果
        }
    }

    runMultiTestCase(t, []TestCase{  // 运行多个测试用例
        {
            Input: `{
                "mark": 1,
                "tcpFastOpen": true
            }`,  // 设置输入的 JSON 字符串
            Parser: createParser(),  // 使用 createParser 函数作为解析器
            Output: &internet.SocketConfig{  // 设置期望的输出结果
                Mark: 1,  // 设置 Mark 字段为 1
                Tfo:  internet.SocketConfig_Enable,  // 设置 Tfo 字段为 SocketConfig_Enable
            },
        },
    })
}

func TestTransportConfig(t *testing.T) {
    createParser := func() func(string) (proto.Message, error) {  // 创建一个函数，用于解析字符串并返回 proto.Message 和 error
        return func(s string) (proto.Message, error) {  // 返回一个函数，该函数解析字符串并返回 proto.Message 和 error
            config := new(TransportConfig)  // 创建一个 TransportConfig 对象
            if err := json.Unmarshal([]byte(s), config); err != nil {  // 使用 JSON 包解析字符串并将结果存储到 TransportConfig 对象中
                return nil, err  // 如果解析出错，返回 nil 和错误信息
            }
            return config.Build()  // 调用 TransportConfig 对象的 Build 方法并返回结果
        }
    }

    })
}
```