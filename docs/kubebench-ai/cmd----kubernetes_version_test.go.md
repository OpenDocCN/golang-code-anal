# `kubebench-aquasecurity\cmd\kubernetes_version_test.go`

```go
package cmd

import (
    "crypto/tls"  // 导入加密传输层协议包
    "fmt"  // 导入格式化包
    "io/ioutil"  // 导入读取文件包
    "net/http"  // 导入HTTP包
    "net/http/httptest"  // 导入HTTP测试包
    "os"  // 导入操作系统包
    "strconv"  // 导入字符串转换包
    "testing"  // 导入测试包
)

func TestLoadCertficate(t *testing.T) {
    tmp, err := ioutil.TempDir("", "TestFakeLoadCertficate")  // 创建临时目录
    if err != nil {
        t.Fatalf("unable to create temp directory: %v", err)  // 如果创建临时目录失败，则输出错误信息
    }
    defer os.RemoveAll(tmp)  // 在函数返回时删除临时目录

    goodCertFile, _ := ioutil.TempFile(tmp, "good-cert-*")  // 创建临时文件
    _, _ = goodCertFile.Write([]byte(`-----BEGIN CERTIFICATE-----
MIICyDCCAbCgAwIBAgIBADANBgkqhkiG9w0BAQsFADAVMRMwEQYDVQQDEwprdWJl
cm5ldGVzMB4XDTE5MTEwODAxNDAwMFoXDTI5MTEwNTAxNDAwMFowFTETMBEGA1UE
AxMKa3ViZXJuZXRlczCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMn6
wjvhMc9e0MDwpQNhp8SPxmv1DsYJ4Btp1GeScIgKKDwppuoOmVizLiMNdV5+70yI
MgNfm/gwFRNDOtN3R7msfZDD5Dd1vI6qRTP21DFOGVdysFdwqJTs0nGcmfvZEOtw
9cjcsXrBi2Mg54v+X/pq2w51xajCGBt2+bpxJJ3WBiWqKYv0RQdNL0WZGm+V9BuP
pHRWPBeLxuCzt5K3Gx+1QDy8o6Y4sSRPssWC4RhD9Hs5/9eeGRyZslLs+AuqdDLQ
aziiSjHVtgCfRXE9nYVxaDIwTFuh+Q1IvtB36NRLyX47oya+BbX3PoCtSjA36RBb
tcJfulr3oNHnb2ZlfcUCAwEAAaMjMCEwDgYDVR0PAQH/BAQDAgKkMA8GA1UdEwEB
/wQFMAMBAf8wDQYJKoZIhvcNAQELBQADggEBAAeQDkbM6DilLkIVQDyxauETgJDV
2AaVzYaAgDApQGAoYV6WIY7Exk4TlmLeKQjWt2s/GtthQWuzUDKTcEvWcG6gNdXk
gzuCRRDMGu25NtG3m67w4e2RzW8Z/lzvbfyJZGoV2c6dN+yP9/Pw2MXlrnMWugd1
jLv3UYZRHMpuNS8BJU74BuVzVPHd55RAl+bV8yemdZJ7pPzMvGbZ7zRXWODTDlge
CQb9lY+jYErisH8Sq7uABFPvi7RaTh8SS7V7OxqHZvmttNTdZs4TIkk45JK7Y+Xq
FAjB57z2NcIgJuVpQnGRYtr/JcH2Qdsq8bLtXaojUIWOOqoTDRLYozdMOOQ=
-----END CERTIFICATE-----`))  // 写入临时文件内容
    badCertFile, _ := ioutil.TempFile(tmp, "bad-cert-*")  // 创建另一个临时文件

    cases := []struct {
        file string  // 文件名
        fail bool  // 是否失败
    }{
        {
            file: "missing cert file",  // 缺少证书文件
            fail: true,  // 失败
        },
        {
            file: badCertFile.Name(),  // 错误的证书文件名
            fail: true,  // 失败
        },
        {
            file: goodCertFile.Name(),  // 正确的证书文件名
            fail: false,  // 不失败
        },
    }
    # 遍历 cases 列表，id 为索引，c 为值
    for id, c := range cases {
        # 使用测试框架运行子测试，子测试名称为当前索引的字符串形式
        t.Run(strconv.Itoa(id), func(t *testing.T) {
            # 加载证书文件，返回 TLS 证书和可能的错误
            tlsCert, err := loadCertficate(c.file)
            # 如果不是预期的失败情况
            if !c.fail {
                # 如果出现了错误
                if err != nil {
                    t.Errorf("unexpected error: %v", err)
                }
                # 如果返回的 TLS 证书为空
                if tlsCert == nil {
                    t.Errorf("missing returned TLS Certificate")
                }
            } else {
                # 如果是预期的失败情况，但没有出现错误
                if err == nil {
                    t.Errorf("Expected error")
                }
            }

        })
    }
}
# 定义测试函数TestGetWebData，用于测试获取网络数据的函数
func TestGetWebData(t *testing.T) {
    # 定义模拟成功响应的函数okfn
    okfn := func(w http.ResponseWriter, r *http.Request) {
        _, _ = fmt.Fprintln(w, `{
            "major": "1",
            "minor": "15"}`)
    }
    # 定义模拟失败响应的函数errfn
    errfn := func(w http.ResponseWriter, r *http.Request) {
        http.Error(w, http.StatusText(http.StatusInternalServerError),
            http.StatusInternalServerError)
    }
    # 设置一个虚拟的token
    token := "dummyToken"
    # 声明一个空的tls证书
    var tlsCert tls.Certificate

    # 定义测试用例
    cases := []struct {
        fn   http.HandlerFunc
        fail bool
    }{
        {
            fn:   okfn,
            fail: false,
        },
        {
            fn:   errfn,
            fail: true,
        },
    }

    # 遍历测试用例
    for id, c := range cases {
        # 使用t.Run创建子测试，id为测试用例的索引
        t.Run(strconv.Itoa(id), func(t *testing.T) {
            # 创建一个模拟的HTTP服务器
            ts := httptest.NewServer(c.fn)
            # 延迟关闭模拟的HTTP服务器
            defer ts.Close()
            # 调用getWebData函数获取网络数据
            data, err := getWebData(ts.URL, token, &tlsCert)
            # 判断是否预期失败
            if !c.fail {
                # 如果不预期失败，判断是否有错误
                if err != nil {
                    t.Errorf("unexpected error: %v", err)
                }
                # 判断返回的数据长度是否为0
                if len(data) == 0 {
                    t.Errorf("missing data")
                }
            } else {
                # 如果预期失败，判断是否有错误
                if err == nil {
                    t.Errorf("Expected error")
                }
            }
        })
    }

}
# 定义测试函数TestGetWebDataWithRetry，用于测试带重试功能的获取网络数据的函数
func TestGetWebDataWithRetry(t *testing.T) {
    # 定义模拟成功响应的函数okfn
    okfn := func(w http.ResponseWriter, r *http.Request) {
        _, _ = fmt.Fprintln(w, `{
            "major": "1",
            "minor": "15"}`)
    }
    # 定义模拟失败响应的函数errfn
    errfn := func(w http.ResponseWriter, r *http.Request) {
        http.Error(w, http.StatusText(http.StatusInternalServerError),
            http.StatusInternalServerError)
    }
    # 设置一个虚拟的token
    token := "dummyToken"
    # 声明一个空的tls证书
    var tlsCert tls.Certificate

    # 定义测试用例
    cases := []struct {
        fn   http.HandlerFunc
        fail bool
    }{
        {
            fn:   okfn,
            fail: false,
        },
        {
            fn:   errfn,
            fail: true,
        },
    }
    # 遍历测试用例集合，id 为索引，c 为测试用例
    for id, c := range cases {
        # 使用测试用例的索引作为子测试的名称，运行子测试
        t.Run(strconv.Itoa(id), func(t *testing.T) {
            # 创建一个 HTTP 测试服务器，并在函数返回时关闭
            ts := httptest.NewServer(c.fn)
            defer ts.Close()
            # 使用重试机制获取 Web 数据，传入 URL、令牌和 TLS 证书
            data, err := getWebDataWithRetry(ts.URL, token, &tlsCert)
            # 如果测试用例不期望失败
            if !c.fail {
                # 如果出现错误，输出意外的错误信息
                if err != nil {
                    t.Errorf("unexpected error: %v", err)
                }
                # 如果数据长度为 0，输出缺失数据的错误信息
                if len(data) == 0 {
                    t.Errorf("missing data")
                }
            } else {
                # 如果测试用例期望失败，但没有出现错误，输出预期的错误信息
                if err == nil {
                    t.Errorf("Expected error")
                }
            }
        })
    }
}
# 测试从 JSON 数据中提取版本信息
func TestExtractVersion(t *testing.T) {
    # 有效的 JSON 数据
    okJSON := []byte(`{
    "major": "1",
    "minor": "15",
    "gitVersion": "v1.15.3",
    "gitCommit": "2d3c76f9091b6bec110a5e63777c332469e0cba2",
    "gitTreeState": "clean",
    "buildDate": "2019-08-20T18:57:36Z",
    "goVersion": "go1.12.9",
    "compiler": "gc",
    "platform": "linux/amd64"
    }`)

    # 无效的 JSON 数据
    invalidJSON := []byte(`{
    "major": "1",
    "minor": "15",
    "gitVersion": "v1.15.3",
    "gitCommit": "2d3c76f9091b6bec110a5e63777c332469e0cba2",
    "gitTreeState": "clean",`)

    # 测试用例
    cases := []struct {
        data        []byte
        fail        bool
        expectedVer string
    }{
        {
            data:        okJSON,
            fail:        false,
            expectedVer: "1.15",
        },
        {
            data: invalidJSON,
            fail: true,
        },
    }

    # 遍历测试用例
    for id, c := range cases {
        t.Run(strconv.Itoa(id), func(t *testing.T) {
            # 提取版本信息
            ver, err := extractVersion(c.data)
            if !c.fail {
                if err != nil {
                    t.Errorf("unexpected error: %v", err)
                }
                if c.expectedVer != ver.BaseVersion() {
                    t.Errorf("Expected %q but Got %q", c.expectedVer, ver)
                }
            } else {
                if err == nil {
                    t.Errorf("Expected error")
                }
            }
        })
    }
}

# 测试获取 Kubernetes URL
func TestGetKubernetesURL(t *testing.T) {

    # 重置环境变量
    resetEnvs := func() {
        os.Unsetenv("KUBE_BENCH_K8S_ENV")
        os.Unsetenv("KUBERNETES_SERVICE_HOST")
        os.Unsetenv("KUBERNETES_SERVICE_PORT_HTTPS")
    }

    # 设置环境变量
    setEnvs := func() {
        os.Setenv("KUBE_BENCH_K8S_ENV", "1")
        os.Setenv("KUBERNETES_SERVICE_HOST", "testHostServer")
        os.Setenv("KUBERNETES_SERVICE_PORT_HTTPS", "443")
    }

    # 测试用例
    cases := []struct {
        useDefault bool
        expected   string
    # 定义测试用例数组，包含两个测试用例对象
    {
        useDefault: true,
        expected:   "https://kubernetes.default.svc/version",
    },
    {
        useDefault: false,
        expected:   "https://testHostServer:443/version",
    }
    # 遍历测试用例数组，使用 id 和 c 作为索引和值
    for id, c := range cases {
        # 运行子测试，使用 id 作为测试名称
        t.Run(strconv.Itoa(id), func(t *testing.T) {
            # 重置环境变量，确保每个测试用例的环境都是一致的
            resetEnvs()
            # 延迟重置环境变量，确保在测试结束后环境能够被正确恢复
            defer resetEnvs()
            # 如果测试用例中指定不使用默认值，则设置环境变量
            if !c.useDefault {
                setEnvs()
            }
            # 获取 Kubernetes 的 URL
            k8sURL := getKubernetesURL()

            # 如果测试用例中指定不使用默认值，则进行相应的断言
            if !c.useDefault {
                if k8sURL != c.expected {
                    t.Errorf("Expected %q but Got %q", k8sURL, c.expected)
                }
            } else {
                # 如果测试用例中指定使用默认值，则进行相应的断言
                if k8sURL != c.expected {
                    t.Errorf("Expected %q but Got %q", k8sURL, c.expected)
                }
            }
        })
    }
# 闭合前面的函数定义
```