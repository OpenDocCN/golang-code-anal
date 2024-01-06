# `grype\test\cli\utils_test.go`

```
package cli
// 导入 cli 包

import (
	"bytes"
	"fmt"
	"io"
	"os"
	"os/exec"
	"path"
	"path/filepath"
	"runtime"
	"strings"
	"testing"
	"time"

	"github.com/stretchr/testify/require"
	// 导入必要的包
	"github.com/anchore/stereoscope/pkg/imagetest"
	// 导入 stereoscope 包中的 imagetest 模块
)
# 获取测试图像的路径
func getFixtureImage(tb testing.TB, fixtureImageName string) string {
    tb.Helper()
    // 从 imagetest 包中获取指定名称的 fixture 图像
    imagetest.GetFixtureImage(tb, "docker-archive", fixtureImageName)
    // 返回 fixture 图像的 tar 文件路径
    return imagetest.GetFixtureImageTarPath(tb, fixtureImageName)
}

# 获取 grype 命令
func getGrypeCommand(tb testing.TB, args ...string) *exec.Cmd {
    tb.Helper()
    argsWithConfig := args
    // 如果 grype 命令没有配置参数，则添加默认配置参数
    if !grypeCommandHasConfigArg(argsWithConfig...) {
        argsWithConfig = append(
            []string{"-c", "../grype-test-config.yaml"},
            args...,
        )
    }
    // 返回带有配置参数的 grype 命令
    return exec.Command(
        getGrypeSnapshotLocation(tb, runtime.GOOS),
        argsWithConfig...,
    )
}
// 检查参数中是否包含配置参数 "-c" 或 "--config"，如果有则返回 true，否则返回 false
func grypeCommandHasConfigArg(args ...string) bool {
    for _, arg := range args {
        if arg == "-c" || arg == "--config" {
            return true
        }
    }
    return false
}

// 获取 Grype 快照的位置
func getGrypeSnapshotLocation(tb testing.TB, goOS string) string {
    if os.Getenv("GRYPE_BINARY_LOCATION") != "" {
        // 如果环境变量 GRYPE_BINARY_LOCATION 存在，则返回其值作为快照的位置
        return os.Getenv("GRYPE_BINARY_LOCATION")
    }

    // 注意：对于 amd64 架构，需要使用 v1 后缀更新快照位置
    // 参考：https://goreleaser.com/customization/build/#why-is-there-a-_v1-suffix-on-amd64-builds
}
// 获取当前系统架构
archPath := runtime.GOARCH
// 如果系统架构为amd64，则在架构路径后面添加"_v1"
if runtime.GOARCH == "amd64" {
	archPath = fmt.Sprintf("%s_v1", archPath)
}

// 根据不同的操作系统，返回不同的路径
switch goOS {
case "darwin", "linux":
	return path.Join(repoRoot(tb), fmt.Sprintf("snapshot/%s-build_%s_%s/grype", goOS, goOS, archPath))
default:
	tb.Fatalf("unsupported OS: %s", runtime.GOOS)
}
// 如果不符合任何操作系统，则返回空字符串
return ""
}

// 获取Docker运行命令
func getDockerRunCommand(tb testing.TB, args ...string) *exec.Cmd {
	tb.Helper()

// 返回一个执行docker命令的*exec.Cmd对象
return exec.Command(
	"docker",
	// 将参数添加到docker命令中
	append(
// 定义一个字符串切片，包含单个字符串"run"
[]string{"run"},
// 将args切片中的元素传递给函数，并将结果作为参数传递给runGrype函数
args...,
// 调用runGrype函数
)...,
)

// 定义一个名为runGrype的函数，接受一个testing.TB类型的参数，一个map[string]string类型的参数和一个可变长度的字符串参数
func runGrype(tb testing.TB, env map[string]string, args ...string) (*exec.Cmd, string, string) {
    // 标记该函数是一个辅助函数
    tb.Helper()

    // 获取一个Grype命令的执行对象
    cmd := getGrypeCommand(tb, args...)
    // 如果环境变量为空，则创建一个空的环境变量map
    if env == nil {
        env = make(map[string]string)
    }

    // 禁止测试检查应用程序更新
    env["GRYPE_CHECK_FOR_APP_UPDATE"] = "false"

    // 运行命令并返回标准输出和标准错误输出
    stdout, stderr := runCommand(cmd, env)
    return cmd, stdout, stderr
}
# 运行给定的命令并返回标准输出和标准错误输出
func runCommand(cmd *exec.Cmd, env map[string]string) (string, string) {
    # 如果环境变量不为空，则将环境变量添加到命令的环境变量中
    if env != nil {
        cmd.Env = append(os.Environ(), envMapToSlice(env)...)
    }
    # 创建存储标准输出和标准错误输出的缓冲区
    var stdout, stderr bytes.Buffer
    # 将命令的标准输出和标准错误输出重定向到缓冲区
    cmd.Stdout = &stdout
    cmd.Stderr = &stderr

    # 忽略错误，因为这可能是测试所期望的结果
    cmd.Run()

    # 返回标准输出和标准错误输出的字符串形式
    return stdout.String(), stderr.String()
}

# 将环境变量映射转换为字符串切片
func envMapToSlice(env map[string]string) (envList []string) {
    # 遍历环境变量映射，将键值对转换为字符串并添加到切片中
    for key, val := range env {
        if key == "" {
            continue
        }
		# 将环境变量列表中的每个键值对格式化为"key=value"的形式，并添加到envList中
		envList = append(envList, fmt.Sprintf("%s=%s", key, val))
	}
	return
}

# 获取当前代码库的根目录
func repoRoot(tb testing.TB) string:
	tb.Helper()
	# 执行git命令获取当前代码库的根目录
	root, err := exec.Command("git", "rev-parse", "--show-toplevel").Output()
	# 如果执行出错，则输出错误信息
	if err != nil:
		tb.Fatalf("unable to find repo root dir: %+v", err)
	# 获取代码库根目录的绝对路径
	absRepoRoot, err := filepath.Abs(strings.TrimSpace(string(root)))
	# 如果获取绝对路径出错，则输出错误信息
	if err != nil:
		tb.Fatal("unable to get abs path to repo root:", err)
	# 返回代码库根目录的绝对路径
	return absRepoRoot
}

# 将文件附加到命令的标准输入中
func attachFileToCommandStdin(tb testing.TB, file io.Reader, command *exec.Cmd):
	tb.Helper()
// 从文件中读取所有内容并检查是否有错误
b, err := io.ReadAll(file)
require.NoError(tb, err)

// 将读取的内容作为标准输入流
command.Stdin = bytes.NewReader(b)
}

// 确保命令执行成功
func assertCommandExecutionSuccess(t testing.TB, cmd *exec.Cmd) {
	// 获取命令的组合输出
	_, err := cmd.CombinedOutput()
	if err != nil {
		// 如果有错误，检查是否是退出错误，如果是则输出退出错误
		if exitErr, ok := err.(*exec.ExitError); ok {
			t.Fatal(exitErr)
		}
		// 如果不是退出错误，则输出无法运行命令的错误
		t.Fatalf("unable to run command %q: %v", cmd, err)
	}
}

// 使用超时进行测试
func testWithTimeout(t *testing.T, name string, timeout time.Duration, test func(*testing.T)) {
	// 创建一个通道用于通知测试完成
	done := make(chan bool)
	// 启动一个 goroutine 运行测试
	go func() {
		// 在新的 goroutine 中运行测试函数
		t.Run(name, test)
		// 发送完成信号到通道
		done <- true
	}()

	// 选择不同的通道操作
	select {
	// 如果超时，输出错误信息
	case <-time.After(timeout):
		t.Fatal("test timed out")
	// 如果完成，继续执行
	case <-done:
	}
}
```