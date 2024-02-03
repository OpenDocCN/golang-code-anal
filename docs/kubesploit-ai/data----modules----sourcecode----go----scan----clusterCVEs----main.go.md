# `kubesploit\data\modules\sourcecode\go\scan\clusterCVEs\main.go`

```go
package main

import (
    "crypto/tls"  // 导入加密/解密包
    "encoding/json"  // 导入 JSON 编解码包
    "fmt"  // 导入格式化包
    "io/ioutil"  // 导入输入输出工具包
    "log"  // 导入日志包
    "net/http"  // 导入 HTTP 包
    "strconv"  // 导入字符串转换包
    "strings"  // 导入字符串处理包
    "time"  // 导入时间包
)

type KubernetesVersion struct {
    Major string `json:"major"`  // 定义 Kubernetes 版本结构体，包含主版本号
    Minor string `json:"minor"`  // 定义 Kubernetes 版本结构体，包含次版本号
    GitVersion string `json:"gitVersion"`  // 定义 Kubernetes 版本结构体，包含 Git 版本号
}

type Version struct {
    Major int  // 定义版本结构体，包含主版本号
    Minor int  // 定义版本结构体，包含次版本号
    Patch int  // 定义版本结构体，包含修订版本号
    Raw   string  // 定义版本结构体，包含原始版本号
}

type CVE struct {
    FixedVersions []Version  // 定义 CVE 结构体，包含已修复的版本列表
    Description string  // 定义 CVE 结构体，包含描述信息
    CVENumber string  // 定义 CVE 结构体，包含 CVE 编号
}

var KNOWN_KUBERNETES_CVES = []CVE{  // 定义已知的 Kubernetes CVE 列表
    struct {  // 定义 CVE 结构体
        FixedVersions []Version  // 定义 CVE 结构体，包含已修复的版本列表
        Description   string  // 定义 CVE 结构体，包含描述信息
        CVENumber     string  // 定义 CVE 结构体，包含 CVE 编号
    }{
        FixedVersions: []Version{  // 定义已修复的版本列表
            {
                Major: 1,  // 设置主版本号
                Minor: 11,  // 设置次版本号
                Patch: 8,  // 设置修订版本号
                Raw: "1.11.8",  // 设置原始版本号
            },
            {
                Major: 1,  // 设置主版本号
                Minor: 12,  // 设置次版本号
                Patch: 6,  // 设置修订版本号
                Raw: "1.12.6",  // 设置原始版本号
            },
            {
                Major: 1,  // 设置主版本号
                Minor: 13,  // 设置次版本号
                Patch: 4,  // 设置修订版本号
                Raw: "1.13.4",  // 设置原始版本号
            },
        },
        Description: "Kubernetes API DoS Vulnerability.",  // 设置描述信息
        CVENumber: "CVE-2019-1002100",  // 设置 CVE 编号
    },
    {
        FixedVersions: []Version{  // 定义已修复的版本列表
            {
                Major: 1,  // 设置主版本号
                Minor: 10,  // 设置次版本号
                Patch: 11,  // 设置修订版本号
                Raw: "1.10.11",  // 设置原始版本号
            },
            {
                Major: 1,  // 设置主版本号
                Minor: 11,  // 设置次版本号
                Patch: 5,  // 设置修订版本号
                Raw: "1.11.5",  // 设置原始版本号
            },
            {
                Major: 1,  // 设置主版本号
                Minor: 12,  // 设置次版本号
                Patch: 3,  // 设置修订版本号
                Raw: "1.12.3",  // 设置原始版本号
            },
        },
        Description: "Allow an unauthenticated user to perform privilege escalation and gain full admin privileges on a cluster.",  // 设置描述信息
        CVENumber: "CVE-2018-1002105",  // 设置 CVE 编号
    },
    {
        FixedVersions: []Version{
            {
                Major: 1,  # 主版本号为1
                Minor: 7,  # 次版本号为7
                Patch: 0,  # 补丁版本号为0
                Raw: "1.7.0",  # 版本号的原始字符串表示
            },
            {
                Major: 1,  # 主版本号为1
                Minor: 8,  # 次版本号为8
                Patch: 0,  # 补丁版本号为0
                Raw: "1.8.0",  # 版本号的原始字符串表示
            },
            {
                Major: 1,  # 主版本号为1
                Minor: 9,  # 次版本号为9
                Patch: 0,  # 补丁版本号为0
                Raw: "1.9.0",  # 版本号的原始字符串表示
            },
            {
                Major: 1,  # 主版本号为1
                Minor: 10,  # 次版本号为10
                Patch: 0,  # 补丁版本号为0
                Raw: "1.10.0",  # 版本号的原始字符串表示
            },
            {
                Major: 1,  # 主版本号为1
                Minor: 11,  # 次版本号为11
                Patch: 0,  # 补丁版本号为0
                Raw: "1.11.0",  # 版本号的原始字符串表示
            },
            {
                Major: 1,  # 主版本号为1
                Minor: 12,  # 次版本号为12
                Patch: 0,  # 补丁版本号为0
                Raw: "1.12.0",  # 版本号的原始字符串表示
            },
            {
                Major: 1,  # 主版本号为1
                Minor: 13,  # 次版本号为13
                Patch: 9,  # 补丁版本号为9
                Raw: "1.13.9",  # 版本号的原始字符串表示
            },
            {
                Major: 1,  # 主版本号为1
                Minor: 14,  # 次版本号为14
                Patch: 5,  # 补丁版本号为5
                Raw: "1.14.5",  # 版本号的原始字符串表示
            },
            {
                Major: 1,  # 主版本号为1
                Minor: 15,  # 次版本号为15
                Patch: 2,  # 补丁版本号为2
                Raw: "1.15.2",  # 版本号的原始字符串表示
            },
        },
        Description: "Allowing users to read, modify, or delete cluster-wide custom resources \neven if they have RBAC permissions that extend only to namespace resources.",  # 描述信息
        CVENumber: "CVE-2019-11247",  # CVE编号
    },
    {
        FixedVersions: []Version{  # 创建一个空的固定版本列表
            {
                Major: 1,  # 设置主版本号为1
                Minor: 13,  # 设置次版本号为13
                Patch: 12,  # 设置修订版本号为12
                Raw: "1.13.12",  # 设置原始版本号字符串为"1.13.12"
            },
            {
                Major: 1,  # 设置主版本号为1
                Minor: 14,  # 设置次版本号为14
                Patch: 8,  # 设置修订版本号为8
                Raw: "1.14.8",  # 设置原始版本号字符串为"1.14.8"
            },
            {
                Major: 1,  # 设置主版本号为1
                Minor: 15,  # 设置次版本号为15
                Patch: 5,  # 设置修订版本号为5
                Raw: "1.15.5",  # 设置原始版本号字符串为"1.15.5"
            },
            {
                Major: 1,  # 设置主版本号为1
                Minor: 16,  # 设置次版本号为16
                Patch: 2,  # 设置修订版本号为2
                Raw: "1.16.2",  # 设置原始版本号字符串为"1.16.2"
            },
        },
        Description: "Kubernetes billion laughs attack vulnerability that allows an attacker to perform a Denial-of-Service (DoS) \nattack on the Kubernetes API server by uploading a maliciously crafted YAML file.",  # 设置描述信息
        CVENumber: "CVE-2019-11253",  # 设置CVE编号
    },
    {
        FixedVersions: []Version{  # 创建一个空的固定版本列表
            {
                Major: 1,  # 设置主版本号为1
                Minor: 15,  # 设置次版本号为15
                Patch: 10,  # 设置修订版本号为10
                Raw: "1.15.10",  # 设置原始版本号字符串为"1.15.10"
            },
            {
                Major: 1,  # 设置主版本号为1
                Minor: 16,  # 设置次版本号为16
                Patch: 7,  # 设置修订版本号为7
                Raw: "1.16.7",  # 设置原始版本号字符串为"1.16.7"
            },
            {
                Major: 1,  # 设置主版本号为1
                Minor: 17,  # 设置次版本号为17
                Patch: 3,  # 设置修订版本号为3
                Raw: "1.17.3",  # 设置原始版本号字符串为"1.17.3"
            },
        },
        Description: "The Kubernetes API Server component in versions 1.1-1.14, and versions prior to 1.15.10, 1.16.7 " +  # 设置描述信息
            "\nand 1.17.3 allows an authorized user who sends malicious YAML payloads to cause the kube-apiserver to consume excessive CPU cycles while parsing YAML.",
        CVENumber: "CVE-2019-11254",  # 设置CVE编号
    },
    # 第一个漏洞的修复版本信息
    FixedVersions: []Version{
        # 修复版本1.16.11
        {
            Major: 1,
            Minor: 16,
            Patch: 11,
            Raw: "1.16.11",
        },
        # 修复版本1.17.7
        {
            Major: 1,
            Minor: 17,
            Patch: 7,
            Raw: "1.17.7",
        },
        # 修复版本1.18.4
        {
            Major: 1,
            Minor: 18,
            Patch: 4,
            Raw: "1.18.4",
        },
        # 修复版本1.16.2
        {
            Major: 1,
            Minor: 16,
            Patch: 2,
            Raw: "1.16.2",
        },
    },
    # 第一个漏洞的描述
    Description: "The kubelet and kube-proxy were found to contain security issue \nwhich allows adjacent hosts to reach TCP and UDP services bound to 127.0.0.1 running on the node or in the node's network namespace." +
        "\nSuch a service is generally thought to be reachable only by other processes on the same host, \nbut due to this defeect, could be reachable by other hosts on the same LAN as the node, or by containers running on the same node as the service.",
    # 第一个漏洞的CVE编号
    CVENumber: "CVE-2020-8558",
},
{
    # 第二个漏洞的修复版本信息
    FixedVersions: []Version{
        # 修复版本1.16.13
        {
            Major: 1,
            Minor: 16,
            Patch: 13,
            Raw: "1.16.13",
        },
        # 修复版本1.17.9
        {
            Major: 1,
            Minor: 17,
            Patch: 9,
            Raw: "1.17.9",
        },
        # 修复版本1.18.6
        {
            Major: 1,
            Minor: 18,
            Patch: 6,
            Raw: "1.18.6",
        },
    },
    # 第二个漏洞的描述
    Description: "The Kubernetes kube-apiserver is vulnerable to an unvalidated redirect on proxied upgrade requests" +
        " \nthat could allow an attacker to escalate privileges from a node compromise to a full cluster compromise.",
    # 第二个漏洞的CVE编号
    CVENumber: "CVE-2020-8559",
},
// 打印给定 CVE 对象的 ID 和描述信息
func printCVE(cve CVE){
    fmt.Printf("[*] ID: %s\n", cve.CVENumber)  // 打印 CVE 编号
    fmt.Printf("[*] Description: %s\n", cve.Description)  // 打印 CVE 描述
    var rawVersions strings.Builder  // 创建一个字符串构建器
    fixedVersions := cve.FixedVersions  // 获取 CVE 的修复版本列表
    for i, version := range(fixedVersions){  // 遍历修复版本列表
        if i == len(cve.FixedVersions) - 1{  // 如果是最后一个版本
            rawVersions.WriteString(version.Raw)  // 将版本信息添加到字符串构建器
        } else {
            rawVersions.WriteString(version.Raw + ", ")  // 将版本信息添加到字符串构建器，并添加逗号
        }
    }
    fmt.Printf("[*] Fixed versions: %s\n", rawVersions.String())  // 打印修复版本信息
}

// 从 Kubernetes 集群导出版本信息
func exportVersionFromKubernetesCluster(address string) Version {
    tr := &http.Transport{  // 创建 HTTP 传输对象
        TLSClientConfig: &tls.Config{  // 配置 TLS 客户端
            InsecureSkipVerify: true,  // 跳过证书验证
        },
        DisableCompression: true,  // 禁用压缩
        MaxIdleConns:       10,  // 最大空闲连接数
        IdleConnTimeout:    20 * time.Second,  // 空闲连接超时时间
    }

    client := &http.Client{Transport: tr}  // 创建 HTTP 客户端

    resp, err := client.Get(address)  // 发送 GET 请求
    if err != nil {
        //log.Fatal("Failed with error: %s", err.Error())
        fmt.Println("Failed with error: %s", err.Error())  // 打印错误信息
        return Version{}  // 返回空版本信息
    }

    defer resp.Body.Close()  // 延迟关闭响应体
    body, err := ioutil.ReadAll(resp.Body)  // 读取响应体内容

    var kubeVersion KubernetesVersion  // 创建 Kubernetes 版本对象
    err = json.Unmarshal(body, &kubeVersion)  // 解析 JSON 数据到 Kubernetes 版本对象

    if err != nil {
        //log.Fatal("Failed to parse JSON error: %s", err.Error())
        fmt.Println("Failed to parse JSON error: %s", err.Error())  // 打印错误信息
        return Version{}  // 返回空版本信息
    }

    /*
        Support EKS version, example:
          "major": "1",
          "minor": "14+",
          "gitVersion": "v1.14.9-eks-f459c0",
    */

    newVersion := strings.Split(kubeVersion.GitVersion, ".")  // 根据点号分割 Git 版本信息
    majorStr := strings.TrimPrefix(newVersion[0], "v")  // 获取主版本号字符串
    majorInt, err := strconv.Atoi(majorStr)  // 将主版本号字符串转换为整数
    if err != nil {
        //log.Fatal("Failed to parse major version with error: %s", err.Error())
        fmt.Println("Failed to parse major version with error: %s", err.Error())  // 打印错误信息
        return Version{}  // 返回空版本信息
    }

    minorStr := strings.TrimSuffix(newVersion[1], "+")  // 获取次版本号字符串
    // 将 minorStr 转换为整数类型，如果出现错误则打印错误信息并返回空的 Version 结构体
    minorInt, err := strconv.Atoi(minorStr)
    if err != nil {
        fmt.Println("Failed to parse minor version with error: %s", err.Error())
        return Version{}
    }

    // 使用 "-" 分割 newVersion[2] 字符串，并将第一个部分转换为整数类型
    patchSplitted := strings.Split(newVersion[2], "-")
    patchInt, err := strconv.Atoi(patchSplitted[0])
    // 如果出现错误则打印错误信息并返回空的 Version 结构体
    if err != nil {
        fmt.Println("Failed to parse patch version with error: %s", err.Error())
        return Version{}
    }

    // 返回一个包含 majorInt、minorInt、patchInt 和 kubeVersion.GitVersion 的 Version 结构体
    return Version{
        Major: majorInt,
        Minor: minorInt,
        Patch: patchInt,
        Raw:   kubeVersion.GitVersion,
    }
# 基于给定版本检查漏洞
func checkForVulnerabilitiesBaseOnVersion(currentVersion Version){
    # 初始化漏洞标志为假
    vulnerable := false
    # 初始化小于所有版本的计数器
    isSmallerThanAll := 0
    # 获取已知的 Kubernetes 漏洞列表
    knownCVEs := KNOWN_KUBERNETES_CVES
    # 遍历已知漏洞列表
    for _, cve := range knownCVEs {
        # 获取已修复版本列表
        fixedVersions := cve.FixedVersions
        # 遍历已修复版本列表
        for _, cveVersion := range fixedVersions {
            # 检查当前版本是否与已修复版本匹配
            if currentVersion.Major == cveVersion.Major {
                if currentVersion.Minor == cveVersion.Minor {
                    # 如果当前版本的补丁版本小于已修复版本的补丁版本，则标记为有漏洞
                    if currentVersion.Patch < cveVersion.Patch {
                        vulnerable = true
                    }
                    # 跳出循环
                    break
                } else if currentVersion.Minor < cveVersion.Minor{
                    # 如果当前版本的次版本小于已修复版本的次版本，则增加计数器
                    isSmallerThanAll += 1
                    # 如果计数器等于已修复版本列表的长度，则标记为有漏洞
                    if isSmallerThanAll == len(cve.FixedVersions){
                        vulnerable = true
                        # 跳出循环
                        break
                    }
                }
            }
        }
        # 如果存在漏洞，则打印漏洞信息
        if vulnerable {
            printCVE(cve)
            fmt.Println()
        }
    }
}

# 主函数
func mainfunc(urlInput string) {
    # 清理 URL 输入中的空格
    urlInput = strings.TrimSpace(urlInput)
    # 拼接 URL 输入
    urlInput = urlInput + "/version"

    # 打印扫描信息
    fmt.Printf("[*] Scanning Kubernetes cluster: %s\n", urlInput)
    # 从 Kubernetes 集群中导出当前版本
    currentVersion := exportVersionFromKubernetesCluster(urlInput)
    # 如果当前版本不为空
    if (Version{}) != currentVersion {
        # 打印当前集群版本信息
        fmt.Printf("[*] Current cluster version: %s\n\n", currentVersion.Raw)
        # 检查基于版本的漏洞
        checkForVulnerabilitiesBaseOnVersion(currentVersion)
    }

    # 打印完成信息
    fmt.Println("[*] Done")
}
```