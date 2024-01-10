# `kubo\test\cli\http_gateway_over_libp2p_test.go`

```
package cli

import (
    "context"
    "encoding/json"
    "fmt"
    "io"
    "net/http"
    "testing"

    "github.com/ipfs/go-cid"
    "github.com/ipfs/kubo/core/commands"
    "github.com/ipfs/kubo/test/cli/harness"
    "github.com/libp2p/go-libp2p"
    "github.com/libp2p/go-libp2p/core/peer"
    libp2phttp "github.com/libp2p/go-libp2p/p2p/http"
    "github.com/multiformats/go-multiaddr"
    manet "github.com/multiformats/go-multiaddr/net"
    "github.com/stretchr/testify/require"
)

func TestGatewayOverLibp2p(t *testing.T) {
    t.Parallel()
    nodes := harness.NewT(t).NewNodes(2).Init()

    // Setup streaming functionality
    nodes.ForEachPar(func(node *harness.Node) {
        node.IPFS("config", "--json", "Experimental.Libp2pStreamMounting", "true")
    })

    gwNode := nodes[0]
    p2pProxyNode := nodes[1]

    nodes.StartDaemons().Connect()

    // Add data to the gateway node
    cidDataOnGatewayNode := cid.MustParse(gwNode.IPFSAddStr("Hello Worlds2!"))
    r := gwNode.GatewayClient().Get(fmt.Sprintf("/ipfs/%s?format=raw", cidDataOnGatewayNode))
    blockDataOnGatewayNode := []byte(r.Body)

    // Add data to the non-gateway node
    cidDataNotOnGatewayNode := cid.MustParse(p2pProxyNode.IPFSAddStr("Hello Worlds!"))
    r = p2pProxyNode.GatewayClient().Get(fmt.Sprintf("/ipfs/%s?format=raw", cidDataNotOnGatewayNode))
    blockDataNotOnGatewayNode := []byte(r.Body)
    _ = blockDataNotOnGatewayNode

    // Setup one of the nodes as http to http-over-libp2p proxy
    p2pProxyNode.IPFS("p2p", "forward", "--allow-custom-protocol", "/http/1.1", "/ip4/127.0.0.1/tcp/0", fmt.Sprintf("/p2p/%s", gwNode.PeerID()))
    lsOutput := commands.P2PLsOutput{}
    // Unmarshal the output of the p2p ls command into lsOutput struct
    if err := json.Unmarshal(p2pProxyNode.IPFS("p2p", "ls", "--enc=json").Stdout.Bytes(), &lsOutput); err != nil {
        t.Fatal(err)
    }
    // Assert that there is only one listener in the lsOutput
    require.Len(t, lsOutput.Listeners, 1)
    // Convert the listener address to multiaddress
    p2pProxyNodeHTTPListenMA, err := multiaddr.NewMultiaddr(lsOutput.Listeners[0].ListenAddress)
    require.NoError(t, err)
}
    // 将 p2pProxyNodeHTTPListenMA 转换为网络地址，并检查是否有错误
    p2pProxyNodeHTTPListenAddr, err := manet.ToNetAddr(p2pProxyNodeHTTPListenMA)
    require.NoError(t, err)

    // 在没有实验性配置的情况下不起作用
    t.Run("DoesNotWorkWithoutExperimentalConfig", func(t *testing.T) {
        // 发起 HTTP GET 请求，检查是否有错误
        _, err := http.Get(fmt.Sprintf("http://%s/ipfs/%s?format=raw", p2pProxyNodeHTTPListenAddr, cidDataOnGatewayNode))
        require.Error(t, err)
    })

    // 启用实验性功能并重新连接节点
    gwNode.IPFS("config", "--json", "Experimental.GatewayOverLibp2p", "true")
    gwNode.StopDaemon().StartDaemon()
    nodes.Connect()

    // 注意：这里的裸 HTTP 请求假设网关挂载在 `/` 上
    t.Run("WillNotServeRemoteContent", func(t *testing.T) {
        // 发起 HTTP GET 请求，检查是否有错误
        resp, err := http.Get(fmt.Sprintf("http://%s/ipfs/%s?format=raw", p2pProxyNodeHTTPListenAddr, cidDataNotOnGatewayNode))
        require.NoError(t, err)
        require.Equal(t, 500, resp.StatusCode)
    })

    t.Run("WillNotServeDeserializedResponses", func(t *testing.T) {
        // 发起 HTTP GET 请求，检查是否有错误
        resp, err := http.Get(fmt.Sprintf("http://%s/ipfs/%s", p2pProxyNodeHTTPListenAddr, cidDataOnGatewayNode))
        require.NoError(t, err)
        require.Equal(t, http.StatusNotAcceptable, resp.StatusCode)
    })
    t.Run("ServeBlock", func(t *testing.T) {
        // 运行测试用例 "ServeBlock"
        t.Run("UsingKuboProxy", func(t *testing.T) {
            // 使用 Kubo 代理进行测试
            resp, err := http.Get(fmt.Sprintf("http://%s/ipfs/%s?format=raw", p2pProxyNodeHTTPListenAddr, cidDataOnGatewayNode))
            // 发起 HTTP GET 请求
            require.NoError(t, err)
            defer resp.Body.Close()
            require.Equal(t, 200, resp.StatusCode)
            // 确保响应状态码为 200
            body, err := io.ReadAll(resp.Body)
            require.NoError(t, err)
            require.Equal(t, blockDataOnGatewayNode, body)
            // 确保响应体与网关节点的块数据一致
        })
        t.Run("UsingLibp2pClientWithPathDiscovery", func(t *testing.T) {
            // 使用带路径发现的 Libp2p 客户端进行测试
            clientHost, err := libp2p.New(libp2p.NoListenAddrs)
            require.NoError(t, err)
            err = clientHost.Connect(context.Background(), peer.AddrInfo{
                ID:    gwNode.PeerID(),
                Addrs: gwNode.SwarmAddrs(),
            })
            require.NoError(t, err)
            // 确保客户端成功连接到网关节点
            client, err := (&libp2phttp.Host{StreamHost: clientHost}).NamespacedClient("/ipfs/gateway", peer.AddrInfo{ID: gwNode.PeerID()})
            require.NoError(t, err)
            // 创建命名空间客户端
            resp, err := client.Get(fmt.Sprintf("/ipfs/%s?format=raw", cidDataOnGatewayNode))
            require.NoError(t, err)
            defer resp.Body.Close()
            require.Equal(t, 200, resp.StatusCode)
            // 确保响应状态码为 200
            body, err := io.ReadAll(resp.Body)
            require.NoError(t, err)
            require.Equal(t, blockDataOnGatewayNode, body)
            // 确保响应体与网关节点的块数据一致
        })
    })
# 闭合前面的函数定义
```