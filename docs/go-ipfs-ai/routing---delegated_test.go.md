# `kubo\routing\delegated_test.go`

```
package routing

import (
    "crypto/rand"  // 导入加密随机数生成包
    "encoding/base64"  // 导入base64编解码包
    "testing"  // 导入测试包

    "github.com/ipfs/kubo/config"  // 导入配置包
    "github.com/libp2p/go-libp2p/core/crypto"  // 导入libp2p核心加密包
    "github.com/libp2p/go-libp2p/core/peer"  // 导入libp2p核心peer包
    "github.com/stretchr/testify/require"  // 导入测试断言包
)

func TestParser(t *testing.T) {
    require := require.New(t)  // 创建测试断言对象

    pid, sk, err := generatePeerID()  // 生成对等节点ID和私钥
    require.NoError(err)  // 断言没有错误发生

    router, err := Parse(config.Routers{  // 解析路由器配置
        "r1": config.RouterParser{  // 路由器1配置
            Router: config.Router{  // 路由器配置
                Type: config.RouterTypeHTTP,  // 路由器类型为HTTP
                Parameters: &config.HTTPRouterParams{  // HTTP路由器参数
                    Endpoint: "testEndpoint",  // 设置端点为testEndpoint
                },
            },
        },
        "r2": config.RouterParser{  // 路由器2配置
            Router: config.Router{  // 路由器配置
                Type: config.RouterTypeSequential,  // 路由器类型为Sequential
                Parameters: &config.ComposableRouterParams{  // 可组合路由器参数
                    Routers: []config.ConfigRouter{  // 配置路由器数组
                        {
                            RouterName: "r1",  // 路由器名称为r1
                        },
                    },
                },
            },
        },
    }, config.Methods{  // 方法配置
        config.MethodNameFindPeers: config.Method{  // 查找对等节点方法配置
            RouterName: "r1",  // 使用r1路由器
        },
        config.MethodNameFindProviders: config.Method{  // 查找提供者方法配置
            RouterName: "r1",  // 使用r1路由器
        },
        config.MethodNameGetIPNS: config.Method{  // 获取IPNS方法配置
            RouterName: "r1",  // 使用r1路由器
        },
        config.MethodNamePutIPNS: config.Method{  // 存储IPNS方法配置
            RouterName: "r2",  // 使用r2路由器
        },
        config.MethodNameProvide: config.Method{  // 提供方法配置
            RouterName: "r2",  // 使用r2路由器
        },
    }, &ExtraDHTParams{}, &ExtraHTTPParams{  // 额外的DHT参数和HTTP参数
        PeerID:     string(pid),  // 设置对等节点ID
        PrivKeyB64: sk,  // 设置私钥
    })

    require.NoError(err)  // 断言没有错误发生

    comp, ok := router.(*Composer)  // 将路由器转换为Composer类型
    require.True(ok)  // 断言转换成功

    require.Equal(comp.FindPeersRouter, comp.FindProvidersRouter)  // 断言查找对等节点路由器和查找提供者路由器相等
    require.Equal(comp.ProvideRouter, comp.PutValueRouter)  // 断言提供路由器和存储值路由器相等
}

func TestParserRecursive(t *testing.T) {
    require := require.New(t)  // 创建测试断言对象

    pid, sk, err := generatePeerID()  // 生成对等节点ID和私钥
    // 检查错误是否为空，如果不为空则抛出错误
    require.NoError(err)

    // 解析路由配置，创建路由对象
    router, err := Parse(config.Routers{
        // 创建名为 "http1" 的 HTTP 路由对象
        "http1": config.RouterParser{
            Router: config.Router{
                Type: config.RouterTypeHTTP,
                Parameters: &config.HTTPRouterParams{
                    Endpoint: "testEndpoint1",
                },
            },
        },
        // 创建名为 "http2" 的 HTTP 路由对象
        "http2": config.RouterParser{
            Router: config.Router{
                Type: config.RouterTypeHTTP,
                Parameters: &config.HTTPRouterParams{
                    Endpoint: "testEndpoint2",
                },
            },
        },
        // 创建名为 "http3" 的 HTTP 路由对象
        "http3": config.RouterParser{
            Router: config.Router{
                Type: config.RouterTypeHTTP,
                Parameters: &config.HTTPRouterParams{
                    Endpoint: "testEndpoint3",
                },
            },
        },
        // 创建名为 "composable1" 的顺序路由对象
        "composable1": config.RouterParser{
            Router: config.Router{
                Type: config.RouterTypeSequential,
                Parameters: &config.ComposableRouterParams{
                    Routers: []config.ConfigRouter{
                        {
                            RouterName: "http1",
                        },
                        {
                            RouterName: "http2",
                        },
                    },
                },
            },
        },
        // 创建名为 "composable2" 的并行路由对象
        "composable2": config.RouterParser{
            Router: config.Router{
                Type: config.RouterTypeParallel,
                Parameters: &config.ComposableRouterParams{
                    Routers: []config.ConfigRouter{
                        {
                            RouterName: "composable1",
                        },
                        {
                            RouterName: "http3",
                        },
                    },
                },
            },
        },
    }, config.Methods{  # 创建一个包含多个方法的 Methods 对象
        config.MethodNameFindPeers: config.Method{  # 添加查找对等节点的方法到 Methods 对象
            RouterName: "composable2",  # 设置方法的路由名称
        },
        config.MethodNameFindProviders: config.Method{  # 添加查找提供者的方法到 Methods 对象
            RouterName: "composable2",  # 设置方法的路由名称
        },
        config.MethodNameGetIPNS: config.Method{  # 添加获取 IPNS 的方法到 Methods 对象
            RouterName: "composable2",  # 设置方法的路由名称
        },
        config.MethodNamePutIPNS: config.Method{  # 添加存储 IPNS 的方法到 Methods 对象
            RouterName: "composable2",  # 设置方法的路由名称
        },
        config.MethodNameProvide: config.Method{  # 添加提供数据的方法到 Methods 对象
            RouterName: "composable2",  # 设置方法的路由名称
        },
    }, &ExtraDHTParams{}, &ExtraHTTPParams{  # 创建一个包含额外 DHT 参数和额外 HTTP 参数的对象
        PeerID:     string(pid),  # 设置对等节点的 ID
        PrivKeyB64: sk,  # 设置 Base64 编码的私钥
    })

    require.NoError(err)  # 断言错误为空

    _, ok := router.(*Composer)  # 将 router 转换为 Composer 类型，并判断是否成功
    require.True(ok)  # 断言转换成功
}
// 测试递归循环的解析器
func TestParserRecursiveLoop(t *testing.T) {
    require := require.New(t)

    // 解析配置和方法，检查是否存在依赖循环
    _, err := Parse(config.Routers{
        "composable1": config.RouterParser{
            Router: config.Router{
                Type: config.RouterTypeSequential,
                Parameters: &config.ComposableRouterParams{
                    Routers: []config.ConfigRouter{
                        {
                            RouterName: "composable2",
                        },
                    },
                },
            },
        },
        "composable2": config.RouterParser{
            Router: config.Router{
                Type: config.RouterTypeParallel,
                Parameters: &config.ComposableRouterParams{
                    Routers: []config.ConfigRouter{
                        {
                            RouterName: "composable1",
                        },
                    },
                },
            },
        },
    }, config.Methods{
        config.MethodNameFindPeers: config.Method{
            RouterName: "composable2",
        },
        config.MethodNameFindProviders: config.Method{
            RouterName: "composable2",
        },
        config.MethodNameGetIPNS: config.Method{
            RouterName: "composable2",
        },
        config.MethodNamePutIPNS: config.Method{
            RouterName: "composable2",
        },
        config.MethodNameProvide: config.Method{
            RouterName: "composable2",
        },
    }, &ExtraDHTParams{}, nil)

    // 断言错误信息中包含指定的依赖循环信息
    require.ErrorContains(err, "dependency loop creating router with name \"composable2\"")
}

// 生成对等节点ID
func generatePeerID() (string, string, error) {
    // 生成 Ed25519 密钥对
    sk, pk, err := crypto.GenerateEd25519Key(rand.Reader)
    if err != nil {
        return "", "", err
    }

    // 将私钥编码为字节流
    bytes, err := crypto.MarshalPrivateKey(sk)
    if err != nil {
        return "", "", err
    }

    // 使用标准 base64 编码将字节流转换为字符串
    enc := base64.StdEncoding.EncodeToString(bytes)
    if err != nil {
        return "", "", err
    }
}
    # 通过公钥创建对等节点ID
    pid, err := peer.IDFromPublicKey(pk)
    # 返回对等节点ID的字符串表示形式、加密后的数据和可能的错误
    return pid.String(), enc, err
# 闭合前面的函数定义
```