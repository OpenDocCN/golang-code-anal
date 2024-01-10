# `kubo\core\corehttp\metrics.go`

```
// 导入所需的包
package corehttp

import (
    "net"
    "net/http"
    "time"

    core "github.com/ipfs/kubo/core"
    "go.opencensus.io/stats/view"
    "go.opencensus.io/zpages"

    ocprom "contrib.go.opencensus.io/exporter/prometheus"
    prometheus "github.com/prometheus/client_golang/prometheus"
    promhttp "github.com/prometheus/client_golang/prometheus/promhttp"
)

// MetricsScrapingOption 添加用于 Prometheus 获取指标的抓取端点
func MetricsScrapingOption(path string) ServeOption {
    return func(n *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
        // 将 Prometheus 默认收集器的处理程序添加到指定路径
        mux.Handle(path, promhttp.HandlerFor(prometheus.DefaultGatherer, promhttp.HandlerOpts{}))
        return mux, nil
    }
}

// MetricsOpenCensusCollectionOption 添加 OpenCensus 指标的收集
func MetricsOpenCensusCollectionOption() ServeOption {
    return func(_ *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
        // 初始化 OpenCensus
        log.Info("Init OpenCensus")

        // 创建 Prometheus 注册表
        promRegistry := prometheus.NewRegistry()
        // 创建 OpenCensus Prometheus 导出器
        pe, err := ocprom.NewExporter(ocprom.Options{
            Namespace: "ipfs_oc",
            Registry:  promRegistry,
            OnError: func(err error) {
                log.Errorw("OC ERROR", "error", err)
            },
        })
        if err != nil {
            return nil, err
        }

        // 将 Prometheus 注册到 OpenCensus
        view.RegisterExporter(pe)
        view.SetReportingPeriod(2 * time.Second)

        // 构建 mux
        zpages.Handle(mux, "/debug/metrics/oc/debugz")
        mux.Handle("/debug/metrics/oc", pe)

        return mux, nil
    }
}

// MetricsOpenCensusDefaultPrometheusRegistry 将默认的 Prometheus 注册表注册为 OpenCensus 指标的导出器
func MetricsOpenCensusDefaultPrometheusRegistry() ServeOption {
    // 添加代码实现
}
    // 返回一个函数，该函数接受一个IpfsNode指针、net.Listener和http.ServeMux指针作为参数，并返回一个http.ServeMux指针和一个错误
    return func(_ *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
        // 输出日志信息，初始化OpenCensus并使用默认的prometheus注册表
        log.Info("Init OpenCensus with default prometheus registry")

        // 创建一个OpenCensus的prometheus导出器
        pe, err := ocprom.NewExporter(ocprom.Options{
            Registry: prometheus.DefaultRegisterer.(*prometheus.Registry),
            OnError: func(err error) {
                // 在发生错误时输出日志信息
                log.Errorw("OC default registry ERROR", "error", err)
            },
        })
        if err != nil {
            // 如果创建导出器时发生错误，则返回nil和错误
            return nil, err
        }

        // 将prometheus注册到opencensus
        view.RegisterExporter(pe)

        // 返回传入的http.ServeMux指针和nil错误
        return mux, nil
    }
// MetricsCollectionOption函数用于添加net/http相关指标的收集选项
func MetricsCollectionOption(handlerName string) ServeOption {
    // 这里应该包含具体的实现逻辑，暂时缺失
}

// 创建一个描述IPFS节点连接的指标
var peersTotalMetric = prometheus.NewDesc(
    prometheus.BuildFQName("ipfs", "p2p", "peers_total"), // 创建指标的命名空间、子系统和名称
    "Number of connected peers", // 指标的描述信息
    []string{"transport"}, // 指标的标签
    nil, // 指标的额外帮助信息
)

// IpfsNodeCollector结构体用于收集IPFS节点的指标
type IpfsNodeCollector struct {
    Node *core.IpfsNode
}

// 实现描述方法，向通道中发送peersTotalMetric指标的描述信息
func (IpfsNodeCollector) Describe(ch chan<- *prometheus.Desc) {
    ch <- peersTotalMetric
}

// 实现收集方法，遍历节点的连接并发送指标值到通道中
func (c IpfsNodeCollector) Collect(ch chan<- prometheus.Metric) {
    for tr, val := range c.PeersTotalValues() {
        ch <- prometheus.MustNewConstMetric(
            peersTotalMetric,
            prometheus.GaugeValue,
            val,
            tr,
        )
    }
}

// 获取节点的连接数值
func (c IpfsNodeCollector) PeersTotalValues() map[string]float64 {
    vals := make(map[string]float64)
    if c.Node.PeerHost == nil {
        return vals
    }
    for _, peerID := range c.Node.PeerHost.Network().Peers() {
        conns := c.Node.PeerHost.Network().ConnsToPeer(peerID)
        if len(conns) == 0 {
            continue
        }
        tr := ""
        for _, proto := range conns[0].RemoteMultiaddr().Protocols() {
            tr = tr + "/" + proto.Name
        }
        vals[tr] = vals[tr] + 1
    }
    return vals
}
```