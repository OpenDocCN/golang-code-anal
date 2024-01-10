# `kubo\core\node\libp2p\dns.go`

```
# 导入 libp2p 包
package libp2p

# 导入必要的包
import (
    "github.com/libp2p/go-libp2p"  # 导入 libp2p 包
    madns "github.com/multiformats/go-multiaddr-dns"  # 导入 madns 包
)

# 定义一个函数，接收一个 madns.Resolver 类型的参数，并返回 Libp2pOpts 和 error 类型的结果
func MultiaddrResolver(rslv *madns.Resolver) (opts Libp2pOpts, err error) {
    # 将 libp2p.MultiaddrResolver(rslv) 的结果追加到 opts.Opts 中
    opts.Opts = append(opts.Opts, libp2p.MultiaddrResolver(rslv))
    # 返回 opts 和 nil（表示没有错误）
    return opts, nil
}
```