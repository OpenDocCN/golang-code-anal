# `kubebench-aquasecurity\integration\integration_test.go`

```go
// +build integration

package integration

import (
    "bufio"  // 导入 bufio 包，用于提供带缓冲的 I/O
    "bytes"  // 导入 bytes 包，用于操作字节切片
    "flag"   // 导入 flag 包，用于命令行参数解析
    "fmt"    // 导入 fmt 包，用于格式化 I/O
    "io/ioutil"  // 导入 ioutil 包，用于 I/O 实用函数
    "strings"    // 导入 strings 包，用于操作字符串
    "testing"    // 导入 testing 包，用于编写测试函数
    "time"       // 导入 time 包，用于时间相关操作
)

var kubebenchImg = flag.String("kubebenchImg", "aquasec/kube-bench:latest", "kube-bench image used as part of this test")  // 定义命令行参数 kubebenchImg，指定 kube-bench 镜像
var timeout = flag.Duration("timeout", 10*time.Minute, "Test Timeout")  // 定义命令行参数 timeout，指定测试超时时间

func testCheckCISWithKind(t *testing.T, testdataDir string) {
    flag.Parse()  // 解析命令行参数
    fmt.Printf("kube-bench Container Image: %s\n", *kubebenchImg)  // 打印 kube-bench 镜像信息

    cases := []struct {  // 定义测试用例结构体切片
        TestName      string  // 测试名称
        KubebenchYAML string  // kube-bench YAML 文件路径
        ExpectedFile  string  // 期望的文件路径
        ExpectError   bool    // 是否期望出错
    }{
        {
            TestName:      "kube-bench",
            KubebenchYAML: "../job.yaml",
            ExpectedFile:  fmt.Sprintf("./testdata/%s/job.data", testdataDir),
        },
        {
            TestName:      "kube-bench-node",
            KubebenchYAML: "../job-node.yaml",
            ExpectedFile:  fmt.Sprintf("./testdata/%s/job-node.data", testdataDir),
        },
        {
            TestName:      "kube-bench-master",
            KubebenchYAML: "../job-master.yaml",
            ExpectedFile:  fmt.Sprintf("./testdata/%s/job-master.data", testdataDir),
        },
    }
    ctx, err := setupCluster("kube-bench", fmt.Sprintf("./testdata/%s/add-tls-kind.yaml", testdataDir), *timeout)  // 设置 KIND 集群
    if err != nil {
        t.Fatalf("failed to setup KIND cluster error: %v", err)  // 如果设置集群失败，则输出错误信息
    }
    defer func() {
        ctx.Delete()  // 延迟执行删除集群操作
    }()

    if err := loadImageFromDocker(*kubebenchImg, ctx); err != nil {
        t.Fatalf("failed to load kube-bench image from Docker to KIND error: %v", err)  // 如果加载 kube-bench 镜像失败，则输出错误信息
    }

    clientset, err := getClientSet(ctx.KubeConfigPath())  // 获取 Kubernetes 客户端
    if err != nil {
        t.Fatalf("failed to connect to Kubernetes cluster error: %v", err)  // 如果连接 Kubernetes 集群失败，则输出错误信息
    }
}
    # 遍历测试用例集合
    for _, c := range cases {
        # 使用测试名称创建子测试，运行测试函数
        t.Run(c.TestName, func(t *testing.T) {
            # 运行指定测试名称的测试，并返回结果数据和可能的错误
            resultData, err := runWithKind(ctx, clientset, c.TestName, c.KubebenchYAML, *kubebenchImg, *timeout)
            # 如果有错误，输出错误信息
            if err != nil {
                t.Errorf("unexpected error: %v", err)
            }

            # 读取预期文件的内容
            c, err := ioutil.ReadFile(c.ExpectedFile)
            # 如果有错误，输出错误信息
            if err != nil {
                t.Error(err)
            }

            # 去除预期数据和结果数据的首尾空白字符
            expectedData := strings.TrimSpace(string(c))
            resultData = strings.TrimSpace(resultData)
            # 如果预期数据和结果数据不相等，输出差异信息
            if expectedData != resultData {
                t.Errorf("expected results\n\nExpected\t(<)\nResult\t(>)\n\n%s\n\n", generateDiff(expectedData, resultData))
            }
        })
    }
// 测试函数，用于检查带有特定类型的 CIS 1.5
func TestCheckCIS15WithKind(t *testing.T) {
    testCheckCISWithKind(t, "cis-1.5")
}

// 测试函数，用于检查带有特定类型的 CIS 1.6
func TestCheckCIS16WithKind(t *testing.T) {
    testCheckCISWithKind(t, "cis-1.6")
}

// 生成两个包含多行的字符串之间的简单“diff”。
// 这不是两个字符串之间的全面差异。
// 它不指示何时删除了行。
func generateDiff(source, target string) string {
    // 创建一个新的字节缓冲区
    buf := new(bytes.Buffer)
    // 创建一个从源字符串读取的扫描器
    ss := bufio.NewScanner(strings.NewReader(source))
    // 创建一个从目标字符串读取的扫描器
    ts := bufio.NewScanner(strings.NewReader(target))

    // 标记源字符串是否为空
    emptySource := false
    // 标记目标字符串是否为空
    emptyTarget := false

loop:
    for ln := 1; ; ln++ {
        var ll, rl string

        // 从源字符串扫描器中读取下一行
        sourceScan := ss.Scan()
        if sourceScan {
            ll = ss.Text()
        }

        // 从目标字符串扫描器中读取下一行
        targetScan := ts.Scan()
        if targetScan {
            rl = ts.Text()
        }

        switch {
        case !sourceScan && !targetScan:
            // 没有更多的行
            break loop
        case sourceScan && targetScan:
            // 如果源字符串和目标字符串的当前行不相等，则将差异信息写入缓冲区
            if ll != rl {
                fmt.Fprintf(buf, "line: %d\n", ln)
                fmt.Fprintf(buf, "< %s\n", ll)
                fmt.Fprintf(buf, "> %s\n", rl)
            }
        case !targetScan:
            // 如果目标字符串已经为空，则将源字符串的当前行写入缓冲区
            if !emptyTarget {
                fmt.Fprintf(buf, "line: %d\n", ln)
            }
            fmt.Fprintf(buf, "< %s\n", ll)
            emptyTarget = true
        case !sourceScan:
            // 如果源字符串已经为空，则将目标字符串的当前行写入缓冲区
            if !emptySource {
                fmt.Fprintf(buf, "line: %d\n", ln)
            }
            fmt.Fprintf(buf, "> %s\n", rl)
            emptySource = true
        }
    }

    // 如果源字符串为空，则在缓冲区中写入相应信息
    if emptySource {
        fmt.Fprintf(buf, "< [[NO MORE DATA]]")
    }

    // 如果目标字符串为空，则在缓冲区中写入相应信息
    if emptyTarget {
        fmt.Fprintf(buf, "> [[NO MORE DATA]]")
    }

    // 返回缓冲区中的字符串表示
    return buf.String()
}
```