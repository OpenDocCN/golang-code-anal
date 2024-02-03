# `kubebench-aquasecurity\cmd\kubernetes_version.go`

```go
package cmd

import (
    "crypto/tls"  // 导入加密通信包
    "encoding/json"  // 导入 JSON 编解码包
    "encoding/pem"  // 导入 PEM 编解码包
    "fmt"  // 导入格式化包
    "io/ioutil"  // 导入 I/O 工具包
    "net/http"  // 导入 HTTP 包
    "os"  // 导入操作系统包
    "strings"  // 导入字符串处理包
    "time"  // 导入时间包

    "github.com/golang/glog"  // 导入日志包
)

type KubeVersion struct {
    Major       string  // 主版本号
    Minor       string  // 次版本号
    baseVersion string  // 基础版本号
    GitVersion  string  // Git 版本号
}

func (k *KubeVersion) BaseVersion() string {
    if k.baseVersion != "" {
        return k.baseVersion
    }
    // 一些提供商返回类似 "15+" 的次版本号
    minor := strings.Replace(k.Minor, "+", "", -1)
    ver := fmt.Sprintf("%s.%s", k.Major, minor)
    k.baseVersion = ver
    return ver
}

func getKubeVersionFromRESTAPI() (*KubeVersion, error) {
    k8sVersionURL := getKubernetesURL()  // 获取 Kubernetes URL
    serviceaccount := "/var/run/secrets/kubernetes.io/serviceaccount"  // 服务账户路径
    cacertfile := fmt.Sprintf("%s/ca.crt", serviceaccount)  // CA 证书文件路径
    tokenfile := fmt.Sprintf("%s/token", serviceaccount)  // Token 文件路径

    tlsCert, err := loadCertficate(cacertfile)  // 加载证书
    if err != nil {
        return nil, err
    }

    tb, err := ioutil.ReadFile(tokenfile)  // 读取 Token 文件
    if err != nil {
        return nil, err
    }
    token := strings.TrimSpace(string(tb))  // 去除 Token 字符串两端的空白字符

    data, err := getWebDataWithRetry(k8sVersionURL, token, tlsCert)  // 获取 Web 数据并进行重试
    if err != nil {
        return nil, err
    }

    k8sVersion, err := extractVersion(data)  // 提取版本信息
    if err != nil {
        return nil, err
    }
    return k8sVersion, nil
}

// 该函数的思想是，如果 Kubernetes DNS 没有完全设置，kube-bench 运行的容器需要时间来配置 DNS。
// 基本上尝试 10 次，每次等待 1 秒，直到成功或失败为止。
func getWebDataWithRetry(k8sVersionURL, token string, cacert *tls.Certificate) (data []byte, err error) {
    tries := 0  // 尝试次数
    // 我们重试几次，以防 DNS 服务没有足够的时间启动
    # 设置尝试次数的上限为10次
    for tries < 10 {
        # 调用 getWebData 函数获取 web 数据，并将结果赋值给 data 和 err
        data, err = getWebData(k8sVersionURL, token, cacert)
        # 如果没有错误，则直接返回结果
        if err == nil {
            return
        }
        # 如果有错误，则尝试次数加1
        tries++
        # 休眠1秒
        time.Sleep(1 * time.Second)
    }

    # 循环结束后直接返回
    return
}

// 定义 VersionResponse 结构体，包含版本信息的各个字段
type VersionResponse struct {
    Major        string
    Minor        string
    GitVersion   string
    GitCommit    string
    GitTreeState string
    BuildDate    string
    GoVersion    string
    Compiler     string
    Platform     string
}

// 从数据中提取版本信息并返回 KubeVersion 对象
func extractVersion(data []byte) (*KubeVersion, error) {
    // 创建 VersionResponse 对象
    vrObj := &VersionResponse{}
    // 打印数据日志
    glog.V(2).Info(fmt.Sprintf("vd: %s\n", string(data)))
    // 解析 JSON 数据到 VersionResponse 对象
    err := json.Unmarshal(data, vrObj)
    if err != nil {
        return nil, err
    }
    // 打印解析后的版本信息日志
    glog.V(2).Info(fmt.Sprintf("vrObj: %#v\n", vrObj))

    // 返回 KubeVersion 对象
    return &KubeVersion{
        Major:      vrObj.Major,
        Minor:      vrObj.Minor,
        GitVersion: vrObj.GitVersion,
    }, nil
}

// 从指定 URL 获取数据并返回字节流
func getWebData(srvURL, token string, cacert *tls.Certificate) ([]byte, error) {
    // 打印获取数据的 URL 日志
    glog.V(2).Info(fmt.Sprintf("getWebData srvURL: %s\n", srvURL))

    // 创建 TLS 配置
    tlsConf := &tls.Config{
        Certificates:       []tls.Certificate{*cacert},
        InsecureSkipVerify: true,
    }
    // 创建 HTTP 传输对象
    tr := &http.Transport{
        TLSClientConfig: tlsConf,
    }
    // 创建 HTTP 客户端
    client := &http.Client{Transport: tr}
    // 创建 HTTP 请求对象
    req, err := http.NewRequest(http.MethodGet, srvURL, nil)
    if err != nil {
        return nil, err
    }

    // 设置授权 Token
    authToken := fmt.Sprintf("Bearer %s", token)
    glog.V(2).Info(fmt.Sprintf("getWebData AUTH TOKEN --[%q]--\n", authToken))
    req.Header.Set("Authorization", authToken)

    // 发起 HTTP 请求并获取响应
    resp, err := client.Do(req)
    if err != nil {
        glog.V(2).Info(fmt.Sprintf("HTTP ERROR: %v\n", err))
        return nil, err
    }
    defer resp.Body.Close()

    // 检查响应状态码
    if resp.StatusCode != http.StatusOK {
        glog.V(2).Info(fmt.Sprintf("URL:[%s], StatusCode:[%d] \n Headers: %#v\n", srvURL, resp.StatusCode, resp.Header))
        err = fmt.Errorf("URL:[%s], StatusCode:[%d]", srvURL, resp.StatusCode)
        return nil, err
    }

    // 读取并返回响应体数据
    return ioutil.ReadAll(resp.Body)
}

// 从指定文件加载证书并返回 TLS 证书对象
func loadCertficate(certFile string) (*tls.Certificate, error) {
    // 读取证书文件内容
    cacert, err := ioutil.ReadFile(certFile)
    if err != nil {
        return nil, err
    }
    // 声明一个变量tlsCert，用于存储TLS证书
    var tlsCert tls.Certificate
    // 解码证书的PEM编码块
    block, _ := pem.Decode(cacert)
    // 如果解码块为空，则返回错误
    if block == nil {
        return nil, fmt.Errorf("unable to Decode certificate")
    }
    // 输出日志信息，表示正在加载CA证书
    glog.V(2).Info("Loading CA certificate")
    // 将解码后的证书字节添加到tlsCert的证书列表中
    tlsCert.Certificate = append(tlsCert.Certificate, block.Bytes)
    // 返回加载好的TLS证书
    return &tlsCert, nil
// 获取 Kubernetes 的版本信息的 URL
func getKubernetesURL() string {
    // 定义 Kubernetes 版本信息的默认 URL
    k8sVersionURL := "https://kubernetes.default.svc/version"

    // 下面的代码提供了灵活性，可以在 hostNetwork: true 的情况下使用 K8S 提供的变量
    if !isEmpty(os.Getenv("KUBE_BENCH_K8S_ENV")) {
        // 获取 Kubernetes 服务的主机地址和端口
        k8sHost := os.Getenv("KUBERNETES_SERVICE_HOST")
        k8sPort := os.Getenv("KUBERNETES_SERVICE_PORT_HTTPS")
        // 如果主机地址和端口都不为空，则返回对应的 URL
        if !isEmpty(k8sHost) && !isEmpty(k8sPort) {
            return fmt.Sprintf("https://%s:%s/version", k8sHost, k8sPort)
        }
        // 如果 KUBE_BENCH_K8S_ENV 被设置，但是环境变量 KUBERNETES_SERVICE_HOST 或 KUBERNETES_SERVICE_PORT_HTTPS 没有被设置，则记录日志信息
        glog.V(2).Info("KUBE_BENCH_K8S_ENV is set, but environment variables KUBERNETES_SERVICE_HOST or KUBERNETES_SERVICE_PORT_HTTPS are not set")
    }

    // 返回默认的 Kubernetes 版本信息的 URL
    return k8sVersionURL
}
```