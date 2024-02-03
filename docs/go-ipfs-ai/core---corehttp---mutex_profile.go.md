# `kubo\core\corehttp\mutex_profile.go`

```go
package corehttp

import (
    "net"  // 导入 net 包，提供了用于网络通信的基本接口
    "net/http"  // 导入 net/http 包，提供了 HTTP 客户端和服务器的实现
    "runtime"  // 导入 runtime 包，提供了与 Go 运行时系统交互的函数
    "strconv"  // 导入 strconv 包，提供了基本数据类型与其字符串表示之间的转换函数

    core "github.com/ipfs/kubo/core"  // 导入自定义包 core，用于引用其中的 IpfsNode 结构体
)

// MutexFractionOption allows to set runtime.SetMutexProfileFraction via HTTP
// using POST request with parameter 'fraction'.
// MutexFractionOption 允许通过 HTTP 设置 runtime.SetMutexProfileFraction，使用带有参数 'fraction' 的 POST 请求。
func MutexFractionOption(path string) ServeOption {
    return func(_ *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
        mux.HandleFunc(path, func(w http.ResponseWriter, r *http.Request) {
            if r.Method != http.MethodPost {  // 如果请求方法不是 POST
                http.Error(w, "only POST allowed", http.StatusMethodNotAllowed)  // 返回错误信息
                return
            }
            if err := r.ParseForm(); err != nil {  // 解析请求的表单数据
                http.Error(w, err.Error(), http.StatusBadRequest)  // 返回错误信息
                return
            }

            asfr := r.Form.Get("fraction")  // 获取表单中的 'fraction' 参数值
            if len(asfr) == 0 {  // 如果 'fraction' 参数值为空
                http.Error(w, "parameter 'fraction' must be set", http.StatusBadRequest)  // 返回错误信息
                return
            }

            fr, err := strconv.Atoi(asfr)  // 将 'fraction' 参数值转换为整数
            if err != nil {  // 如果转换出错
                http.Error(w, err.Error(), http.StatusBadRequest)  // 返回错误信息
                return
            }
            log.Infof("Setting MutexProfileFraction to %d", fr)  // 记录设置 MutexProfileFraction 的操作
            runtime.SetMutexProfileFraction(fr)  // 设置 MutexProfileFraction
        })

        return mux, nil  // 返回处理后的 ServeMux 对象和 nil 错误
    }
}

// BlockProfileRateOption allows to set runtime.SetBlockProfileRate via HTTP
// using POST request with parameter 'rate'.
// The profiler tries to sample 1 event every <rate> nanoseconds.
// If rate == 1, then the profiler samples every blocking event.
// To disable, set rate = 0.
// BlockProfileRateOption 允许通过 HTTP 设置 runtime.SetBlockProfileRate，使用带有参数 'rate' 的 POST 请求。
// 分析器尝试每 <rate> 纳秒采样 1 个事件。
// 如果 rate == 1，则分析器会对每个阻塞事件进行采样。
// 要禁用采样，设置 rate = 0。
func BlockProfileRateOption(path string) ServeOption {
    # 定义一个函数，接受一个IpfsNode对象，一个net.Listener对象和一个http.ServeMux对象作为参数，并返回一个http.ServeMux对象和一个错误
    return func(_ *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
        # 为指定的路径注册一个处理函数
        mux.HandleFunc(path, func(w http.ResponseWriter, r *http.Request) {
            # 如果请求方法不是POST，则返回“只允许POST请求”的错误
            if r.Method != http.MethodPost {
                http.Error(w, "only POST allowed", http.StatusMethodNotAllowed)
                return
            }
            # 解析请求的表单数据，如果出错则返回错误信息
            if err := r.ParseForm(); err != nil {
                http.Error(w, err.Error(), http.StatusBadRequest)
                return
            }

            # 获取表单中名为'rate'的参数值
            rateStr := r.Form.Get("rate")
            # 如果参数值为空，则返回“必须设置参数'rate'”的错误
            if len(rateStr) == 0 {
                http.Error(w, "parameter 'rate' must be set", http.StatusBadRequest)
                return
            }

            # 将参数值转换为整数，如果出错则返回错误信息
            rate, err := strconv.Atoi(rateStr)
            if err != nil {
                http.Error(w, err.Error(), http.StatusBadRequest)
                return
            }
            # 记录日志，设置BlockProfileRate为指定的值
            log.Infof("Setting BlockProfileRate to %d", rate)
            runtime.SetBlockProfileRate(rate)
        })
        # 返回处理函数注册后的http.ServeMux对象和nil错误
        return mux, nil
    }
# 闭合前面的函数定义
```