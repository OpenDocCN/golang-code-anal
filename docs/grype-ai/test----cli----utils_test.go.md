# `grype\test\cli\utils_test.go`

```go
package cli

import (
    "bytes"  // 导入 bytes 包，用于操作字节
    "fmt"  // 导入 fmt 包，用于格式化输出
    "io"  // 导入 io 包，用于进行 I/O 操作
    "os"  // 导入 os 包，提供了对操作系统功能的访问
    "os/exec"  // 导入 os/exec 包，用于执行外部命令
    "path"  // 导入 path 包，用于处理文件路径
    "path/filepath"  // 导入 path/filepath 包，用于处理文件路径
    "runtime"  // 导入 runtime 包，提供了与 Go 运行时环境交互的函数
    "strings"  // 导入 strings 包，用于操作字符串
    "testing"  // 导入 testing 包，用于编写测试函数
    "time"  // 导入 time 包，提供了时间的显示和测量用的函数

    "github.com/stretchr/testify/require"  // 导入 testify/require 包，提供了测试断言功能

    "github.com/anchore/stereoscope/pkg/imagetest"  // 导入 imagetest 包，用于图像测试
)

func getFixtureImage(tb testing.TB, fixtureImageName string) string {
    tb.Helper()  // 标记当前测试函数为辅助函数

    imagetest.GetFixtureImage(tb, "docker-archive", fixtureImageName)  // 获取指定名称的测试图像
    return imagetest.GetFixtureImageTarPath(tb, fixtureImageName)  // 返回测试图像的路径
}

func getGrypeCommand(tb testing.TB, args ...string) *exec.Cmd {
    tb.Helper()  // 标记当前测试函数为辅助函数
    argsWithConfig := args  // 将参数赋值给 argsWithConfig
    if !grypeCommandHasConfigArg(argsWithConfig...) {  // 如果参数中不包含配置参数
        argsWithConfig = append(  // 添加配置参数到参数列表
            []string{"-c", "../grype-test-config.yaml"},  // 配置参数
            args...,  // 原参数
        )
    }

    return exec.Command(  // 返回执行命令对象
        getGrypeSnapshotLocation(tb, runtime.GOOS),  // 获取 grype 快照位置
        argsWithConfig...,  // 带有配置参数的参数列表
    )
}

func grypeCommandHasConfigArg(args ...string) bool {
    for _, arg := range args {  // 遍历参数列表
        if arg == "-c" || arg == "--config" {  // 如果参数是配置参数
            return true  // 返回 true
        }
    }
    return false  // 返回 false
}

func getGrypeSnapshotLocation(tb testing.TB, goOS string) string {
    if os.Getenv("GRYPE_BINARY_LOCATION") != "" {  // 如果环境变量中有快照二进制位置
        // GRYPE_BINARY_LOCATION is the absolute path to the snapshot binary
        return os.Getenv("GRYPE_BINARY_LOCATION")  // 返回快照二进制位置
    }

    // note: for amd64 we need to update the snapshot location with the v1 suffix
    // see : https://goreleaser.com/customization/build/#why-is-there-a-_v1-suffix-on-amd64-builds
    archPath := runtime.GOARCH  // 获取当前架构
    if runtime.GOARCH == "amd64" {  // 如果是 amd64 架构
        archPath = fmt.Sprintf("%s_v1", archPath)  // 更新架构路径
    }

    switch goOS {  // 根据操作系统类型进行判断
    case "darwin", "linux":  // 如果是 darwin 或 linux
        return path.Join(repoRoot(tb), fmt.Sprintf("snapshot/%s-build_%s_%s/grype", goOS, goOS, archPath))  // 返回快照位置
    default:  // 其他情况
        tb.Fatalf("unsupported OS: %s", runtime.GOOS)  // 报错，不支持的操作系统
    }
    return ""  // 返回空字符串
}

func getDockerRunCommand(tb testing.TB, args ...string) *exec.Cmd {
    tb.Helper()  // 标记当前测试函数为辅助函数
    # 返回一个表示在命令行中执行 Docker 命令的对象
    return exec.Command(
        # 使用 "docker" 作为命令
        "docker",
        # 将参数 args 添加到命令中
        append(
            # 添加 "run" 到命令中
            []string{"run"},
            # 添加额外的参数到命令中
            args...,
        )...,
    )
}

// 运行 Grype 命令，并返回命令对象、标准输出和标准错误输出
func runGrype(tb testing.TB, env map[string]string, args ...string) (*exec.Cmd, string, string) {
    tb.Helper()

    // 获取 Grype 命令对象
    cmd := getGrypeCommand(tb, args...)
    if env == nil {
        env = make(map[string]string)
    }

    // 设置环境变量，禁止应用程序更新检查
    env["GRYPE_CHECK_FOR_APP_UPDATE"] = "false"

    // 运行命令并获取标准输出和标准错误输出
    stdout, stderr := runCommand(cmd, env)
    return cmd, stdout, stderr
}

// 运行命令并返回标准输出和标准错误输出
func runCommand(cmd *exec.Cmd, env map[string]string) (string, string) {
    if env != nil {
        cmd.Env = append(os.Environ(), envMapToSlice(env)...)
    }
    var stdout, stderr bytes.Buffer
    cmd.Stdout = &stdout
    cmd.Stderr = &stderr

    // 忽略错误，因为这可能是测试所期望的结果
    cmd.Run()

    return stdout.String(), stderr.String()
}

// 将环境变量映射转换为切片
func envMapToSlice(env map[string]string) (envList []string) {
    for key, val := range env {
        if key == "" {
            continue
        }
        envList = append(envList, fmt.Sprintf("%s=%s", key, val))
    }
    return
}

// 获取仓库根目录
func repoRoot(tb testing.TB) string {
    tb.Helper()
    root, err := exec.Command("git", "rev-parse", "--show-toplevel").Output()
    if err != nil {
        tb.Fatalf("unable to find repo root dir: %+v", err)
    }
    absRepoRoot, err := filepath.Abs(strings.TrimSpace(string(root)))
    if err != nil {
        tb.Fatal("unable to get abs path to repo root:", err)
    }
    return absRepoRoot
}

// 将文件附加到命令的标准输入
func attachFileToCommandStdin(tb testing.TB, file io.Reader, command *exec.Cmd) {
    tb.Helper()

    b, err := io.ReadAll(file)
    require.NoError(tb, err)
    command.Stdin = bytes.NewReader(b)
}

// 断言命令执行成功
func assertCommandExecutionSuccess(t testing.TB, cmd *exec.Cmd) {
    _, err := cmd.CombinedOutput()
    if err != nil {
        if exitErr, ok := err.(*exec.ExitError); ok {
            t.Fatal(exitErr)
        }

        t.Fatalf("unable to run command %q: %v", cmd, err)
    }
}
# 使用超时机制进行测试，如果测试超时则报错
func testWithTimeout(t *testing.T, name string, timeout time.Duration, test func(*testing.T)) {
    # 创建一个通道用于标记测试是否完成
    done := make(chan bool)
    # 启动一个 goroutine 运行测试，并在完成后向通道发送标记
    go func() {
        t.Run(name, test)
        done <- true
    }()

    # 使用 select 语句监听通道和超时事件
    select {
    # 如果超时则报错
    case <-time.After(timeout):
        t.Fatal("test timed out")
    # 如果测试完成则继续执行
    case <-done:
    }
}
```