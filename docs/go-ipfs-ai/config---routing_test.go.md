# `kubo\config\routing_test.go`

```
package config

import (
    "encoding/json"  // 导入 JSON 编解码包
    "testing"  // 导入测试包
    "time"  // 导入时间包

    "github.com/stretchr/testify/require"  // 导入断言包
)

func TestRouterParameters(t *testing.T) {
    require := require.New(t)  // 创建测试断言对象
    sec := time.Second  // 定义时间间隔为一秒
    min := time.Minute  // 定义时间间隔为一分钟
    }

    out, err := json.Marshal(r)  // 将对象 r 转换为 JSON 格式
    require.NoError(err)  // 断言转换过程中没有错误发生

    r2 := &Routing{}  // 创建 Routing 结构体对象的指针

    err = json.Unmarshal(out, r2)  // 将 JSON 数据解码为对象 r2
    require.NoError(err)  // 断言解码过程中没有错误发生

    require.Equal(5, len(r2.Methods))  // 断言 r2.Methods 的长度为 5

    dhtp := r2.Routers["router-dht"].Parameters  // 获取指定路由器的参数
    require.IsType(&DHTRouterParams{}, dhtp)  // 断言参数类型为 DHTRouterParams

    sp := r2.Routers["router-sequential"].Parameters  // 获取指定路由器的参数
    require.IsType(&ComposableRouterParams{}, sp)  // 断言参数类型为 ComposableRouterParams

    pp := r2.Routers["router-parallel"].Parameters  // 获取指定路由器的参数
    require.IsType(&ComposableRouterParams{}, pp)  // 断言参数类型为 ComposableRouterParams
}

func TestMethods(t *testing.T) {
    require := require.New(t)  // 创建测试断言对象

    methodsOK := Methods{  // 创建 Methods 结构体对象
        MethodNameFindPeers: {  // 定义 MethodNameFindPeers 的值
            RouterName: "router-wrong",  // 指定路由器名称
        },
        MethodNameFindProviders: {  // 定义 MethodNameFindProviders 的值
            RouterName: "router-wrong",  // 指定路由器名称
        },
        MethodNameGetIPNS: {  // 定义 MethodNameGetIPNS 的值
            RouterName: "router-wrong",  // 指定路由器名称
        },
        MethodNameProvide: {  // 定义 MethodNameProvide 的值
            RouterName: "router-wrong",  // 指定路由器名称
        },
        MethodNamePutIPNS: {  // 定义 MethodNamePutIPNS 的值
            RouterName: "router-wrong",  // 指定路由器名称
        },
    }

    require.NoError(methodsOK.Check())  // 断言 methodsOK 检查没有错误发生

    methodsMissing := Methods{  // 创建 Methods 结构体对象
        MethodNameFindPeers: {  // 定义 MethodNameFindPeers 的值
            RouterName: "router-wrong",  // 指定路由器名称
        },
        MethodNameGetIPNS: {  // 定义 MethodNameGetIPNS 的值
            RouterName: "router-wrong",  // 指定路由器名称
        },
        MethodNameProvide: {  // 定义 MethodNameProvide 的值
            RouterName: "router-wrong",  // 指定路由器名称
        },
        MethodNamePutIPNS: {  // 定义 MethodNamePutIPNS 的值
            RouterName: "router-wrong",  // 指定路由器名称
        },
    }

    require.Error(methodsMissing.Check())  // 断言 methodsMissing 检查发生错误
}
```