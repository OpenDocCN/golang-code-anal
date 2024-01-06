# `kubebench-aquasecurity\cmd\kubernetes_version.go`

```
package cmd
// 声明包名为cmd，表示该文件属于cmd包

import (
	"crypto/tls"
	// 导入crypto/tls包，用于支持TLS加密通信
	"encoding/json"
	// 导入encoding/json包，用于JSON数据的编解码
	"encoding/pem"
	// 导入encoding/pem包，用于PEM格式数据的编解码
	"fmt"
	// 导入fmt包，用于格式化输入输出
	"io/ioutil"
	// 导入io/ioutil包，用于读取文件内容
	"net/http"
	// 导入net/http包，用于HTTP通信
	"os"
	// 导入os包，用于操作系统功能
	"strings"
	// 导入strings包，用于字符串操作
	"time"
	// 导入time包，用于时间相关操作

	"github.com/golang/glog"
	// 导入github.com/golang/glog包，用于日志记录
)

type KubeVersion struct {
	// 定义KubeVersion结构体，用于存储Kubernetes版本信息
	Major       string
	// 定义Major字段，表示Kubernetes的主版本号
	Minor       string
	// 定义Minor字段，表示Kubernetes的次版本号
	baseVersion string
	// 定义baseVersion字段，表示Kubernetes的基础版本号
// 定义结构体 KubeVersion，包含 GitVersion 字段
type KubeVersion struct {
    GitVersion  string
}

// 定义方法 BaseVersion，返回基础版本号
func (k *KubeVersion) BaseVersion() string {
    // 如果已经有基础版本号，则直接返回
    if k.baseVersion != "" {
        return k.baseVersion
    }
    // 一些提供商返回类似 "15+" 的次要版本号
    // 将 "+" 替换为空字符串
    minor := strings.Replace(k.Minor, "+", "", -1)
    // 格式化主要版本号和次要版本号，组成完整版本号
    ver := fmt.Sprintf("%s.%s", k.Major, minor)
    // 将完整版本号赋值给基础版本号字段
    k.baseVersion = ver
    // 返回完整版本号
    return ver
}

// 从 REST API 获取 Kubernetes 版本信息
func getKubeVersionFromRESTAPI() (*KubeVersion, error) {
    // 获取 Kubernetes 版本信息的 URL
    k8sVersionURL := getKubernetesURL()
    // 设置服务账户路径
    serviceaccount := "/var/run/secrets/kubernetes.io/serviceaccount"
    // 设置 CA 证书文件路径
    cacertfile := fmt.Sprintf("%s/ca.crt", serviceaccount)
    // 设置令牌文件路径
    tokenfile := fmt.Sprintf("%s/token", serviceaccount)
# 加载证书文件，返回证书和错误信息
tlsCert, err := loadCertficate(cacertfile)
if err != nil:
    return nil, err

# 读取 token 文件内容，返回内容和错误信息
tb, err := ioutil.ReadFile(tokenfile)
if err != nil:
    return nil, err
# 去除字符串两端的空白字符，得到 token
token := strings.TrimSpace(string(tb))

# 使用重试机制获取带有证书和 token 的 web 数据，返回数据和错误信息
data, err := getWebDataWithRetry(k8sVersionURL, token, tlsCert)
if err != nil:
    return nil, err

# 从数据中提取 Kubernetes 版本信息，返回版本信息和错误信息
k8sVersion, err := extractVersion(data)
if err != nil:
    return nil, err
// 返回 k8sVersion 和 nil 错误
return k8sVersion, nil
}

// 这个函数的想法是，如果 Kubernetes DNS 没有完全设置，kube-bench 运行的容器需要时间来配置 DNS。
// 基本上尝试 10 次，每次等待 1 秒，直到成功或失败为止。
func getWebDataWithRetry(k8sVersionURL, token string, cacert *tls.Certificate) (data []byte, err error) {
	tries := 0
	// 我们重试几次，以防 DNS 服务还没有时间启动
	for tries < 10 {
		data, err = getWebData(k8sVersionURL, token, cacert)
		if err == nil {
			return
		}
		tries++
		time.Sleep(1 * time.Second)
	}

	return
}
// 定义一个结构体 VersionResponse，包含了版本信息的各个字段
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

// 从字节数据中提取版本信息，并返回 KubeVersion 结构体指针
func extractVersion(data []byte) (*KubeVersion, error) {
    // 创建一个 VersionResponse 结构体对象
    vrObj := &VersionResponse{}
    // 打印日志，输出提取的字节数据
    glog.V(2).Info(fmt.Sprintf("vd: %s\n", string(data)))
    // 将字节数据解析为 JSON 格式，并存入 vrObj 对象中
    err := json.Unmarshal(data, vrObj)
    // 如果解析出错，则返回空指针和错误信息
    if err != nil {
        return nil, err
    }
# 使用 glog 打印日志，输出 vrObj 的详细信息
glog.V(2).Info(fmt.Sprintf("vrObj: %#v\n", vrObj))

# 返回一个 KubeVersion 结构体指针，包含 vrObj 的 Major、Minor 和 GitVersion 字段
return &KubeVersion{
    Major:      vrObj.Major,
    Minor:      vrObj.Minor,
    GitVersion: vrObj.GitVersion,
}, nil
}

# 根据给定的 srvURL、token 和 cacert 获取 web 数据
func getWebData(srvURL, token string, cacert *tls.Certificate) ([]byte, error) {
    # 使用 glog 打印日志，输出 srvURL 的信息
    glog.V(2).Info(fmt.Sprintf("getWebData srvURL: %s\n", srvURL))

    # 创建一个包含 cacert 的 tls 配置
    tlsConf := &tls.Config{
        Certificates:       []tls.Certificate{*cacert},
        InsecureSkipVerify: true,
    }
    # 创建一个自定义的 http Transport，使用上面创建的 tls 配置
    tr := &http.Transport{
        TLSClientConfig: tlsConf,
    }
    # 创建一个自定义的 http Client，使用上面创建的 Transport
    client := &http.Client{Transport: tr}
	// 创建一个新的 HTTP 请求，使用 GET 方法，指定目标 URL，不带任何数据
	req, err := http.NewRequest(http.MethodGet, srvURL, nil)
	// 如果创建请求时出现错误，返回 nil 和错误信息
	if err != nil {
		return nil, err
	}

	// 根据 token 创建认证字符串，并打印日志
	authToken := fmt.Sprintf("Bearer %s", token)
	glog.V(2).Info(fmt.Sprintf("getWebData AUTH TOKEN --[%q]--\n", authToken))
	// 设置请求头中的 Authorization 字段为认证字符串
	req.Header.Set("Authorization", authToken)

	// 发送 HTTP 请求，并获取响应
	resp, err := client.Do(req)
	// 如果发送请求时出现错误，打印日志并返回 nil 和错误信息
	if err != nil {
		glog.V(2).Info(fmt.Sprintf("HTTP ERROR: %v\n", err))
		return nil, err
	}
	// 在函数返回前关闭响应体
	defer resp.Body.Close()

	// 如果响应状态码不是 200 OK，打印日志并返回相应的错误信息
	if resp.StatusCode != http.StatusOK {
		glog.V(2).Info(fmt.Sprintf("URL:[%s], StatusCode:[%d] \n Headers: %#v\n", srvURL, resp.StatusCode, resp.Header))
		err = fmt.Errorf("URL:[%s], StatusCode:[%d]", srvURL, resp.StatusCode)
		return nil, err
	}
// 读取响应体的内容并返回
func loadCertficate(certFile string) (*tls.Certificate, error) {
    // 读取证书文件内容
    cacert, err := ioutil.ReadFile(certFile)
    if err != nil {
        return nil, err
    }

    var tlsCert tls.Certificate
    // 解码证书文件内容
    block, _ := pem.Decode(cacert)
    if block == nil {
        return nil, fmt.Errorf("unable to Decode certificate")
    }

    // 输出日志信息
    glog.V(2).Info("Loading CA certificate")
    // 将解码后的证书内容添加到证书对象中
    tlsCert.Certificate = append(tlsCert.Certificate, block.Bytes)
    // 返回证书对象和空错误
    return &tlsCert, nil
}
// 获取 Kubernetes 的版本信息的 URL
func getKubernetesURL() string {
	// 定义 Kubernetes 版本信息的默认 URL
	k8sVersionURL := "https://kubernetes.default.svc/version"

	// 下面的代码提供了灵活性，可以在主机网络为 true 的情况下使用 K8S 提供的变量
	if !isEmpty(os.Getenv("KUBE_BENCH_K8S_ENV")) {
		// 获取 Kubernetes 服务的主机地址和端口
		k8sHost := os.Getenv("KUBERNETES_SERVICE_HOST")
		k8sPort := os.Getenv("KUBERNETES_SERVICE_PORT_HTTPS")
		// 如果主机地址和端口都不为空，则返回对应的 URL
		if !isEmpty(k8sHost) && !isEmpty(k8sPort) {
			return fmt.Sprintf("https://%s:%s/version", k8sHost, k8sPort)
		}
		// 如果 KUBE_BENCH_K8S_ENV 被设置，但是环境变量 KUBERNETES_SERVICE_HOST 或 KUBERNETES_SERVICE_PORT_HTTPS 没有被设置，则记录日志
		glog.V(2).Info("KUBE_BENCH_K8S_ENV is set, but environment variables KUBERNETES_SERVICE_HOST or KUBERNETES_SERVICE_PORT_HTTPS are not set")
	}

	// 返回默认的 Kubernetes 版本信息的 URL
	return k8sVersionURL
}
抱歉，我无法提供代码注释，因为没有给定任何代码。如果您有任何需要帮助的特定代码，请随时告诉我，我会尽力帮助您进行解释和注释。
```