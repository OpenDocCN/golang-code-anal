# `kubebench-aquasecurity\integration\integration_test.go`

```
// +build integration
// 标记此文件为集成测试文件

package integration
// 声明包名为integration

import (
	"bufio"
	"bytes"
	"flag"
	"fmt"
	"io/ioutil"
	"strings"
	"testing"
	"time"
)
// 导入所需的包

var kubebenchImg = flag.String("kubebenchImg", "aquasec/kube-bench:latest", "kube-bench image used as part of this test")
// 定义一个命令行标志，用于指定kube-bench镜像

var timeout = flag.Duration("timeout", 10*time.Minute, "Test Timeout")
// 定义一个命令行标志，用于指定测试超时时间

func testCheckCISWithKind(t *testing.T, testdataDir string) {
	// 定义一个测试函数，接受一个*testing.T类型的参数和一个字符串参数
	flag.Parse()
	// 解析命令行标志
	// 打印输出 kube-bench 容器镜像的信息
	fmt.Printf("kube-bench Container Image: %s\n", *kubebenchImg)

	// 定义一个包含测试用例的结构体切片
	cases := []struct {
		TestName      string
		KubebenchYAML string
		ExpectedFile  string
		ExpectError   bool
	}{
		// 第一个测试用例
		{
			TestName:      "kube-bench",
			KubebenchYAML: "../job.yaml",
			ExpectedFile:  fmt.Sprintf("./testdata/%s/job.data", testdataDir),
		},
		// 第二个测试用例
		{
			TestName:      "kube-bench-node",
			KubebenchYAML: "../job-node.yaml",
			ExpectedFile:  fmt.Sprintf("./testdata/%s/job-node.data", testdataDir),
		},
		// 第三个测试用例
		{
			TestName:      "kube-bench-master",
// 设置 KubebenchYAML 和 ExpectedFile 的值
KubebenchYAML: "../job-master.yaml",
ExpectedFile:  fmt.Sprintf("./testdata/%s/job-master.data", testdataDir),

// 使用 setupCluster 函数设置 KIND 集群，并返回上下文和错误信息
ctx, err := setupCluster("kube-bench", fmt.Sprintf("./testdata/%s/add-tls-kind.yaml", testdataDir), *timeout)
if err != nil {
    t.Fatalf("failed to setup KIND cluster error: %v", err)
}

// 延迟执行删除 KIND 集群的操作
defer func() {
    ctx.Delete()
}()

// 从 Docker 加载 kubebenchImg 到 KIND 集群中
if err := loadImageFromDocker(*kubebenchImg, ctx); err != nil {
    t.Fatalf("failed to load kube-bench image from Docker to KIND error: %v", err)
}

// 获取连接到 Kubernetes 集群的客户端集合和错误信息
clientset, err := getClientSet(ctx.KubeConfigPath())
if err != nil {
    t.Fatalf("failed to connect to Kubernetes cluster error: %v", err)
}
# 遍历测试用例列表
for _, c := range cases {
    # 使用测试名称创建子测试
    t.Run(c.TestName, func(t *testing.T) {
        # 运行测试并获取结果数据和可能的错误
        resultData, err := runWithKind(ctx, clientset, c.TestName, c.KubebenchYAML, *kubebenchImg, *timeout)
        # 如果有错误，输出错误信息
        if err != nil {
            t.Errorf("unexpected error: %v", err)
        }

        # 读取预期结果文件内容
        c, err := ioutil.ReadFile(c.ExpectedFile)
        if err != nil {
            t.Error(err)
        }

        # 去除预期结果和实际结果的空白字符
        expectedData := strings.TrimSpace(string(c))
        resultData = strings.TrimSpace(resultData)
        
        # 比较预期结果和实际结果，如果不相等，输出差异信息
        if expectedData != resultData {
            t.Errorf("expected results\n\nExpected\t(<)\nResult\t(>)\n\n%s\n\n", generateDiff(expectedData, resultData))
        }
    })
}
// 测试函数，用于检查带有特定类型的 CIS 1.5 的情况
func TestCheckCIS15WithKind(t *testing.T) {
    testCheckCISWithKind(t, "cis-1.5")
}

// 测试函数，用于检查带有特定类型的 CIS 1.6 的情况
func TestCheckCIS16WithKind(t *testing.T) {
    testCheckCISWithKind(t, "cis-1.6")
}

// 该函数用于生成两个包含多行的字符串之间的简单“diff”。
// 它不是两个字符串之间的全面差异。
// 它不指示何时删除了行。
func generateDiff(source, target string) string {
    // 创建一个新的字节缓冲区
    buf := new(bytes.Buffer)
    // 创建一个从源字符串读取的扫描器
    ss := bufio.NewScanner(strings.NewReader(source))
    // 创建一个从目标字符串读取的扫描器
    ts := bufio.NewScanner(strings.NewReader(target))

    // 源字符串是否为空
    emptySource := false
    // 目标字符串是否为空
    emptyTarget := false
# 循环开始
loop:
	# 递增行号
	for ln := 1; ; ln++ {
		# 初始化左右两个字符串
		var ll, rl string

		# 从源扫描器中扫描下一行，如果有则赋值给ll
		sourceScan := ss.Scan()
		if sourceScan {
			ll = ss.Text()
		}

		# 从目标扫描器中扫描下一行，如果有则赋值给rl
		targetScan := ts.Scan()
		if targetScan {
			rl = ts.Text()
		}

		# 根据扫描结果进行不同的处理
		switch {
		# 如果源和目标扫描器都没有扫描到新行，则跳出循环
		case !sourceScan && !targetScan:
			# no more lines
			break loop
		# 如果源和目标扫描器都扫描到了新行
		case sourceScan && targetScan:
# 如果左右两行内容不相等
if ll != rl:
    # 输出行号
    fmt.Fprintf(buf, "line: %d\n", ln)
    # 输出左边内容
    fmt.Fprintf(buf, "< %s\n", ll)
    # 输出右边内容
    fmt.Fprintf(buf, "> %s\n", rl)
# 如果不是目标扫描
case !targetScan:
    # 如果目标不为空
    if !emptyTarget:
        # 输出行号
        fmt.Fprintf(buf, "line: %d\n", ln)
    # 输出左边内容
    fmt.Fprintf(buf, "< %s\n", ll)
    # 将目标置为空
    emptyTarget = true
# 如果不是源扫描
case !sourceScan:
    # 如果源不为空
    if !emptySource:
        # 输出行号
        fmt.Fprintf(buf, "line: %d\n", ln)
    # 输出右边内容
    fmt.Fprintf(buf, "> %s\n", rl)
    # 将源置为空
    emptySource = true
# 如果源数据为空，则向缓冲区中写入指示没有更多数据的消息
if emptySource:
    fmt.Fprintf(buf, "< [[NO MORE DATA]]")

# 如果目标数据为空，则向缓冲区中写入指示没有更多数据的消息
if emptyTarget:
    fmt.Fprintf(buf, "> [[NO MORE DATA]]")

# 返回缓冲区中的字符串
return buf.String()
```