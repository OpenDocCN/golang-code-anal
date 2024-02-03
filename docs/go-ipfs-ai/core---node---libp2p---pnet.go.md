# `kubo\core\node\libp2p\pnet.go`

```go
package libp2p

import (
    "bytes"  // 导入 bytes 包，用于操作字节流
    "context"  // 导入 context 包，用于控制程序的上下文
    "fmt"  // 导入 fmt 包，用于格式化输出
    "time"  // 导入 time 包，用于时间相关操作

    "github.com/ipfs/kubo/repo"  // 导入外部依赖包

    "github.com/libp2p/go-libp2p"  // 导入 libp2p 包
    "github.com/libp2p/go-libp2p/core/host"  // 导入 libp2p 的 host 包
    "github.com/libp2p/go-libp2p/core/pnet"  // 导入 libp2p 的 pnet 包
    "go.uber.org/fx"  // 导入外部依赖包
    "golang.org/x/crypto/salsa20"  // 导入 salsa20 加密算法包
    "golang.org/x/crypto/sha3"  // 导入 sha3 加密算法包
)

type PNetFingerprint []byte  // 定义 PNetFingerprint 类型为字节流切片

func PNet(repo repo.Repo) (opts Libp2pOpts, fp PNetFingerprint, err error) {
    swarmkey, err := repo.SwarmKey()  // 从 repo 中获取 SwarmKey
    if err != nil || swarmkey == nil {  // 如果出错或者 swarmkey 为空
        return opts, nil, err  // 返回空的 opts 和 fp，以及错误信息
    }

    psk, err := pnet.DecodeV1PSK(bytes.NewReader(swarmkey))  // 解码 swarmkey 为 psk
    if err != nil {  // 如果解码出错
        return opts, nil, fmt.Errorf("failed to configure private network: %s", err)  // 返回错误信息
    }

    opts.Opts = append(opts.Opts, libp2p.PrivateNetwork(psk))  // 将 PrivateNetwork(psk) 添加到 opts.Opts 中

    return opts, pnetFingerprint(psk), nil  // 返回 opts 和 psk 的指纹，以及空的错误信息
}

func PNetChecker(repo repo.Repo, ph host.Host, lc fx.Lifecycle) error {
    // TODO: better check?  // 待办事项：更好的检查？

    swarmkey, err := repo.SwarmKey()  // 从 repo 中获取 SwarmKey
    if err != nil || swarmkey == nil {  // 如果出错或者 swarmkey 为空
        return err  // 返回错误信息
    }

    done := make(chan struct{})  // 创建一个结构体类型的通道

    lc.Append(fx.Hook{  // 在生命周期中添加钩子
        OnStart: func(_ context.Context) error {  // 在启动时执行
            go func() {  // 启动一个 goroutine
                t := time.NewTicker(30 * time.Second)  // 创建一个每 30 秒触发一次的定时器
                defer t.Stop()  // 在函数返回时停止定时器

                <-t.C  // 从定时器的通道中接收一个值（忽略）
                for {  // 无限循环
                    select {  // 选择不同的 case 执行
                    case <-t.C:  // 当定时器触发时
                        if len(ph.Network().Peers()) == 0 {  // 如果主机的网络中没有对等节点
                            log.Warn("We are in private network and have no peers.")  // 输出警告信息
                            log.Warn("This might be configuration mistake.")  // 输出警告信息
                        }
                    case <-done:  // 当 done 通道关闭时
                        return  // 返回
                    }
                }
            }()
            return nil  // 返回空的错误信息
        },
        OnStop: func(_ context.Context) error {  // 在停止时执行
            close(done)  // 关闭 done 通道
            return nil  // 返回空的错误信息
        },
    })
    return nil  // 返回空的错误信息
}

func pnetFingerprint(psk pnet.PSK) []byte {
    var pskArr [32]byte  // 声明一个长度为 32 的字节数组
    copy(pskArr[:], psk)  // 将 psk 复制到 pskArr 中
    // 创建一个长度为64的字节切片，用于存储加密后的数据
    enc := make([]byte, 64)
    // 创建一个长度为64的字节切片，用于存储全零数据
    zeros := make([]byte, 64)
    // 创建一个长度为16的字节切片，用于存储哈希后的数据

    // 首先对数据进行加密，这样我们就不会将PSK传递给哈希函数。
    // Salsa20函数是不可逆的，因此增加了我们的安全边界。
    salsa20.XORKeyStream(enc, zeros, []byte("finprint"), &pskArr)

    // 然后进行Shake-128哈希运算以减少其长度。
    // 这样，如果由于某种原因Shake被破解并且Salsa20的前像是可能的，
    // 攻击者只有一半的字节来重新创建PSK。
    sha3.ShakeSum128(out, enc)

    // 返回哈希后的数据
    return out
# 闭合前面的函数定义
```