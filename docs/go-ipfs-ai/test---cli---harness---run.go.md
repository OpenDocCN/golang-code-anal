# `kubo\test\cli\harness\run.go`

```go
package harness

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "io"   // 导入 io 包，用于实现 I/O 操作
    "os/exec"  // 导入 os/exec 包，用于执行外部命令
    "strings"  // 导入 strings 包，用于处理字符串
)

// Runner is a process runner which can run subprocesses and aggregate output.
type Runner struct {
    Env     map[string]string  // 环境变量
    Dir     string  // 运行命令的目录
    Verbose bool  // 是否输出详细信息
}

type (
    CmdOpt  func(*exec.Cmd)  // 定义一个函数类型，用于修改 exec.Cmd
    RunFunc func(*exec.Cmd) error  // 定义一个函数类型，用于运行 exec.Cmd
)

var RunFuncStart = (*exec.Cmd).Start  // 定义一个变量，用于启动 exec.Cmd

type RunRequest struct {
    Path string  // 命令路径
    Args []string  // 命令参数
    // Options that are applied to the exec.Cmd just before running it
    CmdOpts []CmdOpt  // 用于在运行命令之前应用到 exec.Cmd 的选项
    // Function to use to run the command.
    // If not specified, defaults to cmd.Run
    RunFunc func(*exec.Cmd) error  // 用于运行命令的函数
    Verbose bool  // 是否输出详细信息
}

type RunResult struct {
    Stdout  *Buffer  // 标准输出
    Stderr  *Buffer  // 标准错误输出
    Err     error  // 错误信息
    ExitErr *exec.ExitError  // 退出错误
    Cmd     *exec.Cmd  // 执行的命令
}

func (r *RunResult) ExitCode() int {
    return r.Cmd.ProcessState.ExitCode()  // 获取命令的退出码
}

func environToMap(environ []string) map[string]string {
    m := map[string]string{}  // 创建一个空的字符串映射
    for _, e := range environ {
        kv := strings.Split(e, "=")  // 以 "=" 分割环境变量
        // Skip environment variables that start with =
        // These can occur in Windows https://github.com/golang/go/issues/61956
        if kv[0] == "" {
            continue  // 跳过以 "=" 开头的环境变量
        }
        m[kv[0]] = kv[1]  // 将环境变量添加到映射中
    }
    return m  // 返回映射
}

func (r *Runner) Run(req RunRequest) *RunResult {
    cmd := exec.Command(req.Path, req.Args...)  // 创建一个命令
    stdout := &Buffer{}  // 创建一个标准输出缓冲区
    stderr := &Buffer{}  // 创建一个标准错误输出缓冲区
    cmd.Stdout = stdout  // 将标准输出设置为缓冲区
    cmd.Stderr = stderr  // 将标准错误输出设置为缓冲区
    cmd.Dir = r.Dir  // 设置命令的运行目录

    for k, v := range r.Env {
        cmd.Env = append(cmd.Env, fmt.Sprintf("%s=%s", k, v))  // 将环境变量添加到命令中
    }

    for _, o := range req.CmdOpts {
        o(cmd)  // 应用命令选项
    }

    if req.RunFunc == nil {
        req.RunFunc = (*exec.Cmd).Run  // 如果未指定运行函数，则默认为 cmd.Run
    }

    log.Debugf("running %v", cmd.Args)  // 输出运行的命令

    err := req.RunFunc(cmd)  // 运行命令

    result := RunResult{
        Stdout: stdout,
        Stderr: stderr,
        Cmd:    cmd,
        Err:    err,
    }

    if exitErr, ok := err.(*exec.ExitError); ok {
        result.ExitErr = exitErr  // 如果有退出错误，则设置退出错误字段
    }

    return &result  // 返回运行结果
}
// MustRun 方法运行命令，并在命令失败时使测试失败。
func (r *Runner) MustRun(req RunRequest) *RunResult {
    // 运行命令
    result := r.Run(req)
    // 断言结果中没有错误
    r.AssertNoError(result)
    // 返回结果
    return result
}

// AssertNoError 方法用于断言运行结果中是否有错误，并在有错误时输出错误信息。
func (r *Runner) AssertNoError(result *RunResult) {
    // 如果有退出错误，则输出错误信息
    if result.ExitErr != nil {
        log.Panicf("'%s' returned error, code: %d, err: %s\nstdout:%s\nstderr:%s\n",
            result.Cmd.Args, result.ExitErr.ExitCode(), result.ExitErr.Error(), result.Stdout.String(), result.Stderr.String())
    }
    // 如果有其他错误，则输出错误信息
    if result.Err != nil {
        log.Panicf("unable to run %s: %s", result.Cmd.Path, result.Err)
    }
}

// RunWithEnv 方法用于设置命令的环境变量。
func RunWithEnv(env map[string]string) CmdOpt {
    return func(cmd *exec.Cmd) {
        // 遍历环境变量，设置到命令中
        for k, v := range env {
            cmd.Env = append(cmd.Env, fmt.Sprintf("%s=%s", k, v))
        }
    }
}

// RunWithPath 方法用于设置命令的执行路径。
func RunWithPath(path string) CmdOpt {
    return func(cmd *exec.Cmd) {
        var newEnv []string
        // 遍历命令的环境变量
        for _, env := range cmd.Env {
            e := strings.Split(env, "=")
            // 如果是 PATH 环境变量，则添加新的路径
            if e[0] == "PATH" {
                paths := strings.Split(e[1], ":")
                paths = append(paths, path)
                e[1] = strings.Join(paths, ":")
                fmt.Printf("path: %s\n", strings.Join(e, "="))
            }
            newEnv = append(newEnv, strings.Join(e, "="))
        }
        cmd.Env = newEnv
    }
}

// RunWithStdin 方法用于设置命令的标准输入。
func RunWithStdin(reader io.Reader) CmdOpt {
    return func(cmd *exec.Cmd) {
        cmd.Stdin = reader
    }
}

// RunWithStdinStr 方法用于设置命令的标准输入，输入为字符串。
func RunWithStdinStr(s string) CmdOpt {
    return RunWithStdin(strings.NewReader(s))
}

// RunWithStdout 方法用于设置命令的标准输出。
func RunWithStdout(writer io.Writer) CmdOpt {
    return func(cmd *exec.Cmd) {
        cmd.Stdout = io.MultiWriter(writer, cmd.Stdout)
    }
}

// RunWithStderr 方法用于设置命令的标准错误输出。
func RunWithStderr(writer io.Writer) CmdOpt {
    return func(cmd *exec.Cmd) {
        cmd.Stderr = io.MultiWriter(writer, cmd.Stdout)
    }
}
```