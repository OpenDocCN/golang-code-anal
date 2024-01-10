# `kubo\cmd\ipfs\kubo\pinmfs_test.go`

```
package kubo

import (
    "context"  // 导入上下文包
    "fmt"  // 导入格式化包
    "strings"  // 导入字符串处理包
    "testing"  // 导入测试包
    "time"  // 导入时间包

    merkledag "github.com/ipfs/boxo/ipld/merkledag"  // 导入 IPLD MerkleDAG 包
    ipld "github.com/ipfs/go-ipld-format"  // 导入 IPLD 格式包
    config "github.com/ipfs/kubo/config"  // 导入配置包
    "github.com/libp2p/go-libp2p/core/host"  // 导入 libp2p 核心主机包
    peer "github.com/libp2p/go-libp2p/core/peer"  // 导入 libp2p 核心对等体包
)

type testPinMFSContext struct {
    ctx context.Context  // 定义上下文
    cfg *config.Config  // 定义配置对象指针
    err error  // 定义错误对象
}

func (x *testPinMFSContext) Context() context.Context {
    return x.ctx  // 返回上下文
}

func (x *testPinMFSContext) GetConfig() (*config.Config, error) {
    return x.cfg, x.err  // 返回配置对象和错误
}

type testPinMFSNode struct {
    err error  // 定义错误对象
}

func (x *testPinMFSNode) RootNode() (ipld.Node, error) {
    return merkledag.NewRawNode([]byte{0x01}), x.err  // 返回 MerkleDAG 新原始节点和错误
}

func (x *testPinMFSNode) Identity() peer.ID {
    return peer.ID("test_id")  // 返回测试对等体 ID
}

func (x *testPinMFSNode) PeerHost() host.Host {
    return nil  // 返回空的主机
}

var testConfigPollInterval = time.Second  // 定义测试配置轮询间隔为1秒

func isErrorSimilar(e1, e2 error) bool {
    switch {
    case e1 == nil && e2 == nil:
        return true
    case e1 != nil && e2 == nil:
        return false
    case e1 == nil && e2 != nil:
        return false
    default:
        return strings.Contains(e1.Error(), e2.Error()) || strings.Contains(e2.Error(), e1.Error())
    }
}

func TestPinMFSConfigError(t *testing.T) {
    ctx := &testPinMFSContext{
        ctx: context.Background(),  // 创建测试上下文
        cfg: nil,  // 配置对象为空
        err: fmt.Errorf("couldn't read config"),  // 设置错误为无法读取配置
    }
    node := &testPinMFSNode{}  // 创建测试节点
    errCh := make(chan error)  // 创建错误通道
    go pinMFSOnChange(testConfigPollInterval, ctx, node, errCh)  // 在新的 goroutine 中执行 pinMFSOnChange 函数
    if !isErrorSimilar(<-errCh, ctx.err) {  // 如果错误通道中的错误与上下文的错误不相似
        t.Errorf("error did not propagate")  // 输出错误信息
    }
    if !isErrorSimilar(<-errCh, ctx.err) {  // 如果错误通道中的错误与上下文的错误不相似
        t.Errorf("error did not propagate")  // 输出错误信息
    }
}

func TestPinMFSRootNodeError(t *testing.T) {
    ctx := &testPinMFSContext{
        ctx: context.Background(),  // 创建测试上下文
        cfg: &config.Config{
            Pinning: config.Pinning{},  // 设置配置对象的固定值
        },
        err: nil,  // 错误为空
    }
    // 创建一个 testPinMFSNode 结构体指针，并设置 err 字段为无法创建根节点的错误
    node := &testPinMFSNode{
        err: fmt.Errorf("cannot create root node"),
    }
    // 创建一个用于传递错误的通道
    errCh := make(chan error)
    // 在新的 goroutine 中执行 pinMFSOnChange 函数，传入测试配置轮询间隔、上下文、节点和错误通道
    go pinMFSOnChange(testConfigPollInterval, ctx, node, errCh)
    // 如果从错误通道中接收到的错误与节点的错误不相似，则输出错误信息
    if !isErrorSimilar(<-errCh, node.err) {
        t.Errorf("error did not propagate")
    }
    // 如果从错误通道中接收到的错误与节点的错误不相似，则输出错误信息
    if !isErrorSimilar(<-errCh, node.err) {
        t.Errorf("error did not propagate")
    }
func TestPinMFSService(t *testing.T) {
    // 创建一个无效的间隔配置
    cfgInvalidInterval := &config.Config{
        Pinning: config.Pinning{
            RemoteServices: map[string]config.RemotePinningService{
                "disabled": {
                    Policies: config.RemotePinningServicePolicies{
                        MFS: config.RemotePinningServiceMFSPolicy{
                            Enable: false,
                        },
                    },
                },
                "invalid_interval": {
                    Policies: config.RemotePinningServicePolicies{
                        MFS: config.RemotePinningServiceMFSPolicy{
                            Enable:        true,
                            RepinInterval: "INVALID_INTERVAL",
                        },
                    },
                },
            },
        },
    }
    // 创建一个有效但未命名的配置
    cfgValidUnnamed := &config.Config{
        Pinning: config.Pinning{
            RemoteServices: map[string]config.RemotePinningService{
                "valid_unnamed": {
                    Policies: config.RemotePinningServicePolicies{
                        MFS: config.RemotePinningServiceMFSPolicy{
                            Enable:        true,
                            PinName:       "",
                            RepinInterval: "2s",
                        },
                    },
                },
            },
        },
    }
    // 创建一个有效且命名的配置
    cfgValidNamed := &config.Config{
        Pinning: config.Pinning{
            RemoteServices: map[string]config.RemotePinningService{
                "valid_named": {
                    Policies: config.RemotePinningServicePolicies{
                        MFS: config.RemotePinningServiceMFSPolicy{
                            Enable:        true,
                            PinName:       "pin_name",
                            RepinInterval: "2s",
                        },
                    },
                },
            },
        },
    }
}
    # 使用给定的测试参数调用 testPinMFSServiceWithError 函数，验证远程固定服务是否会返回错误信息
    testPinMFSServiceWithError(t, cfgInvalidInterval, "remote pinning service \"invalid_interval\" has invalid MFS.RepinInterval")
    # 使用给定的测试参数调用 testPinMFSServiceWithError 函数，验证远程固定服务是否会返回错误信息
    testPinMFSServiceWithError(t, cfgValidUnnamed, "error while listing remote pins: empty response from remote pinning service")
    # 使用给定的测试参数调用 testPinMFSServiceWithError 函数，验证远程固定服务是否会返回错误信息
    testPinMFSServiceWithError(t, cfgValidNamed, "error while listing remote pins: empty response from remote pinning service")
func testPinMFSServiceWithError(t *testing.T, cfg *config.Config, expectedErrorPrefix string) {
    // 创建一个带有取消函数的上下文
    goctx, cancel := context.WithCancel(context.Background())
    // 创建测试用的 PinMFS 上下文
    ctx := &testPinMFSContext{
        ctx: goctx,
        cfg: cfg,
        err: nil,
    }
    // 创建测试用的 PinMFS 节点
    node := &testPinMFSNode{
        err: nil,
    }
    // 创建一个用于接收错误的通道
    errCh := make(chan error)
    // 在新的 goroutine 中执行 pinMFSOnChange 函数
    go pinMFSOnChange(testConfigPollInterval, ctx, node, errCh)
    // 延迟执行取消函数
    defer cancel()
    // 第一次通过固定循环
    err := <-errCh
    // 检查错误信息是否包含预期的错误前缀
    if !strings.Contains((err).Error(), expectedErrorPrefix) {
        t.Errorf("expecting error containing %q", expectedErrorPrefix)
    }
    // 第二次通过固定循环
    if !strings.Contains((err).Error(), expectedErrorPrefix) {
        t.Errorf("expecting error containing %q", expectedErrorPrefix)
    }
}
```