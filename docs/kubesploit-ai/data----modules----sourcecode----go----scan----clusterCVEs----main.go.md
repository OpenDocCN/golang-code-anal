# `kubesploit\data\modules\sourcecode\go\scan\clusterCVEs\main.go`

```
package main
// 声明当前文件所属的包

import (
	"crypto/tls"
	// 导入加密/解密包
	"encoding/json"
	// 导入 JSON 编解码包
	"fmt"
	// 导入格式化包
	"io/ioutil"
	// 导入读取文件包
	"log"
	// 导入日志包
	"net/http"
	// 导入 HTTP 包
	"strconv"
	// 导入字符串转换包
	"strings"
	// 导入字符串处理包
	"time"
	// 导入时间包
)

type KubernetesVersion struct {
	// 定义结构体 KubernetesVersion
	Major string `json:"major"`
	// 定义字段 Major，使用 JSON 标签
	Minor string `json:"minor"`
	// 定义字段 Minor，使用 JSON 标签
	GitVersion string `json:"gitVersion"`
	// 定义字段 GitVersion，使用 JSON 标签
}
# 定义一个名为 Version 的结构体，包含 Major、Minor、Patch 和 Raw 四个字段，用于表示版本号
type Version struct {
    Major int    # 主版本号
    Minor int    # 次版本号
    Patch int    # 补丁版本号
    Raw   string # 原始版本号字符串
}

# 定义一个名为 CVE 的结构体，包含 FixedVersions、Description 和 CVENumber 三个字段，用于表示 CVE（公共漏洞和暴露）信息
type CVE struct {
    FixedVersions []Version  # 修复的版本号列表
    Description   string     # 漏洞描述
    CVENumber     string     # CVE 编号
}

# 定义一个名为 KNOWN_KUBERNETES_CVES 的变量，类型为 CVE 切片，用于存储已知的 Kubernetes 漏洞信息
var KNOWN_KUBERNETES_CVES = []CVE{
    # 创建一个匿名结构体，包含 FixedVersions、Description 和 CVENumber 三个字段的值
    struct {
        FixedVersions []Version  # 修复的版本号列表
        Description   string     # 漏洞描述
        CVENumber     string     # CVE 编号
    }{
# 定义一个包含 Version 结构的 FixedVersions 数组
FixedVersions: []Version{
    # 定义第一个 Version 结构，包含主版本号、次版本号、修订号和原始版本号
    {
        Major: 1,
        Minor: 11,
        Patch: 8,
        Raw: "1.11.8",
    },
    # 定义第二个 Version 结构，包含主版本号、次版本号、修订号和原始版本号
    {
        Major: 1,
        Minor: 12,
        Patch: 6,
        Raw: "1.12.6",
    },
    # 定义第三个 Version 结构，包含主版本号、次版本号、修订号和原始版本号
    {
        Major: 1,
        Minor: 13,
        Patch: 4,
        Raw: "1.13.4",
    },
},
		Description: "Kubernetes API DoS Vulnerability.",  # 描述漏洞的名称
		CVENumber: "CVE-2019-1002100",  # 漏洞的CVE编号
	},
	{  # 修复的版本信息
		FixedVersions: []Version{  # 修复的版本列表
			{
				Major: 1,  # 主版本号
				Minor: 10,  # 次版本号
				Patch: 11,  # 补丁版本号
				Raw: "1.10.11",  # 完整版本号字符串
			},
			{
				Major: 1,
				Minor: 11,
				Patch: 5,
				Raw: "1.11.5",
			},
			{
				Major: 1,
				Minor: 12,  # 修复的版本信息
		{
			// 定义修复版本的数组
			FixedVersions: []Version{
				// 定义修复版本1
				{
					Major: 1,
					Minor: 7,
					Patch: 0,
					Raw: "1.7.0",
				},
				// 定义修复版本2
				{
					Major: 1,
					Minor: 8,
					Patch: 0,
					Raw: "1.8.0",
				}
			},
			// 描述漏洞的描述
			Description: "Allow an unauthenticated user to perform privilege escalation and gain full admin privileges on a cluster.",
			// 漏洞的 CVE 编号
			CVENumber: "CVE-2018-1002105",
		},
		// 下一个漏洞的信息...
		{
			// ...
		}
# 创建一个包含多个对象的列表，每个对象都包含了主版本号、次版本号、补丁号和原始版本号
{
    Major: 1,   # 主版本号为1
    Minor: 9,   # 次版本号为9
    Patch: 0,   # 补丁号为0
    Raw: "1.9.0",   # 原始版本号为"1.9.0"
},
{
    Major: 1,   # 主版本号为1
    Minor: 10,  # 次版本号为10
    Patch: 0,   # 补丁号为0
    Raw: "1.10.0",  # 原始版本号为"1.10.0"
},
{
    Major: 1,   # 主版本号为1
    Minor: 11,  # 次版本号为11
    Patch: 0,   # 补丁号为0
    Raw: "1.11.0",  # 原始版本号为"1.11.0"
},
{
			{
				// 定义版本号对象，主版本号为1，次版本号为12，补丁版本号为0，原始版本号为"1.12.0"
				Major: 1,
				Minor: 12,
				Patch: 0,
				Raw: "1.12.0",
			},
			{
				// 定义版本号对象，主版本号为1，次版本号为13，补丁版本号为9，原始版本号为"1.13.9"
				Major: 1,
				Minor: 13,
				Patch: 9,
				Raw: "1.13.9",
			},
			{
				// 定义版本号对象，主版本号为1，次版本号为14，补丁版本号为5，原始版本号为"1.14.5"
				Major: 1,
				Minor: 14,
				Patch: 5,
				Raw: "1.14.5",
			},
			{
				// 定义版本号对象，主版本号为1，次版本号为15，补丁版本号未指定
				Major: 1,
				Minor: 15,
# 定义一个包含修复版本信息的结构体
{
    # 修复版本信息中的版本号
    FixedVersions: []Version{
        # 第一个修复版本的具体信息
        {
            # 修复版本的主版本号
            Major: 1,
            # 修复版本的次版本号
            Minor: 13,
            # 修复版本的补丁号
            Patch: 12,
            # 修复版本的原始字符串表示
            Raw: "1.13.12",
        },
        # 第二个修复版本的具体信息
        {
            # 修复版本的主版本号
            Major: 1,
            # 修复版本的次版本号
            Minor: 14,
            # 修复版本的补丁号
            Patch: 8,
            # 修复版本的原始字符串表示
            Raw: "1.14.8",
        }
    },
    # 漏洞描述
    Description: "Allowing users to read, modify, or delete cluster-wide custom resources \neven if they have RBAC permissions that extend only to namespace resources.",
    # 漏洞的CVE编号
    CVENumber: "CVE-2019-11247",
},
# 定义一个包含多个版本信息的列表
		},
		{
			# 定义一个版本对象，包含主版本号、次版本号、修订号和原始版本号
			Major: 1,
			Minor: 15,
			Patch: 5,
			Raw: "1.15.5",
		},
		{
			# 定义另一个版本对象，包含主版本号、次版本号、修订号和原始版本号
			Major: 1,
			Minor: 16,
			Patch: 2,
			Raw: "1.16.2",
		},
	},
	# 定义一个CVE编号
	CVENumber: "CVE-2019-11253",
},
{
	# 定义一个包含修复版本信息的列表
	FixedVersions: []Version{
		{
			# 定义一个版本对象，包含主版本号
			Major: 1,
# 定义一个包含多个版本信息的列表
Versions: [
    {
        Major: 1,
        Minor: 11,
        Patch: 0,
        Raw: "1.11.0",
    },
    {
        Major: 1,
        Minor: 12,
        Patch: 0,
        Raw: "1.12.0",
    },
    {
        Major: 1,
        Minor: 13,
        Patch: 0,
        Raw: "1.13.0",
    },
    {
        Major: 1,
        Minor: 14,
        Patch: 0,
        Raw: "1.14.0",
    },
    {
        Major: 1,
        Minor: 15,
        Patch: 10,
        Raw: "1.15.10",
    },
    {
        Major: 1,
        Minor: 16,
        Patch: 7,
        Raw: "1.16.7",
    },
    {
        Major: 1,
        Minor: 17,
        Patch: 3,
        Raw: "1.17.3",
    },
],
# 描述漏洞的信息
Description: "The Kubernetes API Server component in versions 1.1-1.14, and versions prior to 1.15.10, 1.16.7 " +
    "\nand 1.17.3 allows an authorized user who sends malicious YAML payloads to cause the kube-apiserver to consume excessive CPU cycles while parsing YAML.",
# 漏洞的 CVE 编号
CVENumber: "CVE-2019-11254",
	},
	{   # 开始定义一个新的元素
		FixedVersions: []Version{   # 定义一个包含 Version 元素的数组
			{   # 开始定义一个新的 Version 元素
				Major: 1,   # 设置主版本号为 1
				Minor: 16,  # 设置次版本号为 16
				Patch: 11,  # 设置修订版本号为 11
				Raw: "1.16.11",  # 设置原始版本号字符串为 "1.16.11"
			},  # 结束定义第一个 Version 元素
			{   # 开始定义一个新的 Version 元素
				Major: 1,   # 设置主版本号为 1
				Minor: 17,  # 设置次版本号为 17
				Patch: 7,   # 设置修订版本号为 7
				Raw: "1.17.7",   # 设置原始版本号字符串为 "1.17.7"
			},  # 结束定义第二个 Version 元素
			{   # 开始定义一个新的 Version 元素
				Major: 1,   # 设置主版本号为 1
				Minor: 18,  # 设置次版本号为 18
				Patch: 4,   # 设置修订版本号为 4
				Raw: "1.18.4",   # 设置原始版本号字符串为 "1.18.4"
		},
		{
			// 定义版本信息
			Major: 1,
			Minor: 16,
			Patch: 2,
			Raw: "1.16.2",
		},
	},
	// 定义CVE编号
	CVENumber: "CVE-2020-8558",
},
{
	// 定义修复版本信息
	FixedVersions: []Version{
		{
			Major: 1,
			Minor: 16,
			Patch: 13,
			Raw: "1.16.13",
		},
		{
			Major: 1,
# 定义一个结构体，包含了多个漏洞信息
var cveList = []CVE{
	{
		// 第一个漏洞信息
		Versions: []Version{
			{
				// 版本号为 1.17.9
				Major: 1,
				Minor: 17,
				Patch: 9,
				Raw: "1.17.9",
			},
			{
				// 版本号为 1.18.6
				Major: 1,
				Minor: 18,
				Patch: 6,
				Raw: "1.18.6",
			},
		},
		// 漏洞描述
		Description: "The Kubernetes kube-apiserver is vulnerable to an unvalidated redirect on proxied upgrade requests" +
			" \nthat could allow an attacker to escalate privileges from a node compromise to a full cluster compromise.",
		// 漏洞的 CVE 编号
		CVENumber: "CVE-2020-8559",
	},
}

# 定义一个函数，用于打印漏洞信息
func printCVE(cve CVE){
	# 打印漏洞的 CVE 编号
	fmt.Printf("[*] ID: %s\n", cve.CVENumber)
	# 打印漏洞的描述
	fmt.Printf("[*] Description: %s\n", cve.Description)
}
// 创建一个字符串构建器，用于存储版本信息
var rawVersions strings.Builder
// 获取CVE的修复版本列表
fixedVersions := cve.FixedVersions
// 遍历修复版本列表，将版本信息添加到字符串构建器中
for i, version := range(fixedVersions){
    // 判断是否为最后一个版本，如果是则不添加逗号
    if i == len(cve.FixedVersions) - 1{
        rawVersions.WriteString(version.Raw)
    } else {
        rawVersions.WriteString(version.Raw + ", ")
    }
}
// 打印修复版本信息
fmt.Printf("[*] Fixed versions: %s\n", rawVersions.String())

// 从Kubernetes集群中导出版本信息
func exportVersionFromKubernetesCluster(address string) Version {
    // 创建一个HTTP传输对象
    tr := &http.Transport{
        // 配置TLS客户端，跳过证书验证
        TLSClientConfig: &tls.Config{
            InsecureSkipVerify: true,
        },
        // 禁用压缩
        DisableCompression: true,
        // 最大空闲连接数
        MaxIdleConns:       10,
        // 空闲连接超时时间
        IdleConnTimeout:    20 * time.Second,
	}

	// 创建一个带有自定义传输方式的 HTTP 客户端
	client := &http.Client{Transport: tr}

	// 发送 GET 请求获取响应
	resp, err := client.Get(address)
	if err != nil {
		// 如果请求失败，打印错误信息并返回空的 Version 结构体
		fmt.Println("Failed with error: %s", err.Error())
		return Version{}
	}

	// 确保在函数返回前关闭响应体
	defer resp.Body.Close()

	// 读取响应体的内容
	body, err := ioutil.ReadAll(resp.Body)

	// 将 JSON 数据解析为 KubernetesVersion 结构体
	var kubeVersion KubernetesVersion
	err = json.Unmarshal(body, &kubeVersion)

	// 如果解析失败，打印错误信息
	if err != nil {
		//log.Fatal("Failed to parse JSON error: %s", err.Error())
	// 打印错误信息并返回空的 Version 结构体
	fmt.Println("Failed to parse JSON error: %s", err.Error())
	return Version{}
}

/*
	支持 EKS 版本，例如：
	  "major": "1",
	  "minor": "14+",
	  "gitVersion": "v1.14.9-eks-f459c0",
*/

// 根据 git 版本号分割出主版本号
newVersion := strings.Split(kubeVersion.GitVersion, ".")
// 去除版本号中的前缀 "v"，并转换为整数
majorStr := strings.TrimPrefix(newVersion[0], "v")
majorInt, err := strconv.Atoi(majorStr)
if err != nil {
	// 打印错误信息并返回空的 Version 结构体
	fmt.Println("Failed to parse major version with error: %s", err.Error())
	return Version{}
}
// 去除newVersion[1]中的后缀，得到次版本号的字符串
minorStr := strings.TrimSuffix(newVersion[1], "+")
// 将次版本号的字符串转换为整数
minorInt, err := strconv.Atoi(minorStr)
if err != nil {
    // 如果转换出错，打印错误信息并返回空的Version结构体
    fmt.Println("Failed to parse minor version with error: %s", err.Error())
    return Version{}
}

// 使用"-"分割newVersion[2]，得到补丁版本号的字符串数组
patchSplitted := strings.Split(newVersion[2], "-")
// 将补丁版本号的字符串转换为整数
patchInt, err := strconv.Atoi(patchSplitted[0])
if err != nil {
    // 如果转换出错，打印错误信息并返回空的Version结构体
    fmt.Println("Failed to parse patch version with error: %s", err.Error())
    return Version{}
}

// 返回包含主版本号、次版本号和补丁版本号的Version结构体
return Version{
    Major: majorInt,
    Minor: minorInt,
    Patch: patchInt,
}
// 检查当前版本是否存在漏洞
func checkForVulnerabilitiesBaseOnVersion(currentVersion Version){
	// 初始化漏洞标志和小于所有版本号的计数器
	vulnerable := false
	isSmallerThanAll := 0
	// 获取已知的 Kubernetes 漏洞列表
	knownCVEs := KNOWN_KUBERNETES_CVES
	// 遍历已知漏洞列表
	for _, cve := range knownCVEs {
		// 获取已修复的版本号列表
		fixedVersions := cve.FixedVersions
		// 遍历已修复的版本号列表
		for _, cveVersion := range fixedVersions {
			// 检查当前版本是否与已修复版本号匹配
			if currentVersion.Major == cveVersion.Major {
				if currentVersion.Minor == cveVersion.Minor {
					// 如果当前版本的补丁版本号小于已修复版本号，则标记为存在漏洞
					if currentVersion.Patch < cveVersion.Patch {
						vulnerable = true
					}
					break
				} else if currentVersion.Minor < cveVersion.Minor{
					// 如果当前版本的次版本号小于已修复版本号的次版本号，则增加计数器
					isSmallerThanAll += 1
					// 如果计数器等于已修复版本号列表的长度，则表示当前版本小于所有已修复版本号
					if isSmallerThanAll == len(cve.FixedVersions){
						vulnerable = true  // 设置变量vulnerable为true，表示存在漏洞
						break  // 跳出当前循环
					}
				}
			}
		}

		if vulnerable {  // 如果存在漏洞
			printCVE(cve)  // 调用printCVE函数打印漏洞信息
			fmt.Println()  // 打印空行
		}
	}
}

func mainfunc(urlInput string) {
	// The Run() function in ../pkg/modules/modules.go might return the URL with spaces, need to clean it
	urlInput = strings.TrimSpace(urlInput)  // 去除URL输入的空格
	urlInput = urlInput + "/version"  // 在URL后面添加"/version"

	fmt.Printf("[*] Scanning Kubernetes cluster: %s\n", urlInput)  // 打印扫描Kubernetes集群的信息
# 从 Kubernetes 集群的 URL 输入中导出版本号
currentVersion := exportVersionFromKubernetesCluster(urlInput)

# 如果当前版本不为空，则打印当前集群版本号
if (Version{}) != currentVersion {
    fmt.Printf("[*] Current cluster version: %s\n\n", currentVersion.Raw)
    # 临时更改当前版本号为指定的版本
    /*
        currentVersion = Version{
            Major: 1,
            Minor: 11,
            Patch: 3,
        }
    */

    # 根据当前版本号检查漏洞
    checkForVulnerabilitiesBaseOnVersion(currentVersion)
}

# 打印完成信息
fmt.Println("[*] Done")
```