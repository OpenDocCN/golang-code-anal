# `v2ray-core\common\common.go`

```go
// Package common contains common utilities that are shared among other packages.
// See each sub-package for detail.
package common

import (
    "fmt" // 导入 fmt 包，用于格式化输出
    "go/build" // 导入 go/build 包，用于构建 Go 程序
    "io/ioutil" // 导入 ioutil 包，用于读取文件内容
    "os" // 导入 os 包，提供操作系统函数
    "path/filepath" // 导入 filepath 包，用于处理文件路径
    "strings" // 导入 strings 包，提供字符串操作函数

    "v2ray.com/core/common/errors" // 导入自定义的 errors 包
)

//go:generate go run v2ray.com/core/common/errors/errorgen
// 使用 go:generate 命令生成错误处理代码

var (
    // ErrNoClue is for the situation that existing information is not enough to make a decision. For example, Router may return this error when there is no suitable route.
    ErrNoClue = errors.New("not enough information for making a decision") // 定义 ErrNoClue 错误变量
)

// Must panics if err is not nil.
func Must(err error) {
    if err != nil {
        panic(err) // 如果 err 不为空，则触发 panic
    }
}

// Must2 panics if the second parameter is not nil, otherwise returns the first parameter.
func Must2(v interface{}, err error) interface{} {
    Must(err) // 调用 Must 函数
    return v // 返回参数 v
}

// Error2 returns the err from the 2nd parameter.
func Error2(v interface{}, err error) error {
    return err // 返回第二个参数 err
}

// envFile returns the name of the Go environment configuration file.
// Copy from https://github.com/golang/go/blob/c4f2a9788a7be04daf931ac54382fbe2cb754938/src/cmd/go/internal/cfg/cfg.go#L150-L166
func envFile() (string, error) {
    if file := os.Getenv("GOENV"); file != "" {
        if file == "off" {
            return "", fmt.Errorf("GOENV=off") // 如果 GOENV 为 off，则返回错误
        }
        return file, nil // 返回 GOENV 的值
    }
    dir, err := os.UserConfigDir() // 获取用户配置目录
    if err != nil {
        return "", err // 如果出错，则返回错误
    }
    if dir == "" {
        return "", fmt.Errorf("missing user-config dir") // 如果用户配置目录为空，则返回错误
    }
    return filepath.Join(dir, "go", "env"), nil // 返回 Go 环境配置文件的路径
}

// GetRuntimeEnv returns the value of runtime environment variable,
// that is set by running following command: `go env -w key=value`.
func GetRuntimeEnv(key string) (string, error) {
    file, err := envFile() // 获取环境配置文件
    if err != nil {
        return "", err // 如果出错，则返回错误
    }
    if file == "" {
        return "", fmt.Errorf("missing runtime env file") // 如果环境配置文件为空，则返回错误
    }
    var data []byte
    var runtimeEnv string
    // 读取文件内容并返回数据和可能的错误
    data, readErr := ioutil.ReadFile(file)
    // 如果读取文件时发生错误，则返回空字符串和错误信息
    if readErr != nil {
        return "", readErr
    }
    // 将文件内容按换行符分割成字符串数组
    envStrings := strings.Split(string(data), "\n")
    // 遍历每个环境变量字符串
    for _, envItem := range envStrings {
        // 去除每个环境变量字符串末尾的回车符
        envItem = strings.TrimSuffix(envItem, "\r")
        // 将环境变量字符串按等号分割成键值对
        envKeyValue := strings.Split(envItem, "=")
        // 如果键的前后空格忽略大小写后与目标键相等，则将对应的值赋给 runtimeEnv
        if strings.EqualFold(strings.TrimSpace(envKeyValue[0]), key) {
            runtimeEnv = strings.TrimSpace(envKeyValue[1])
        }
    }
    // 返回最终的运行时环境变量和空错误
    return runtimeEnv, nil
// GetGOBIN 返回 GOBIN 环境变量的字符串。它不会为空。
func GetGOBIN() string {
    // 由用户显式设置的，如 `export GOBIN=/path` 或 `env GOBIN=/path command`
    GOBIN := os.Getenv("GOBIN")
    if GOBIN == "" {
        var err error
    // 由用户运行 `go env -w GOBIN=/path` 设置的
        GOBIN, err = GetRuntimeEnv("GOBIN")
        if err != nil {
            // Golang 使用的默认值
            return filepath.Join(build.Default.GOPATH, "bin")
        }
        if GOBIN == "" {
            return filepath.Join(build.Default.GOPATH, "bin")
        }
        return GOBIN
    }
    return GOBIN
}

// GetGOPATH 返回 GOPATH 环境变量的字符串。它不会为空。
func GetGOPATH() string {
    // 由用户显式设置的，如 `export GOPATH=/path` 或 `env GOPATH=/path command`
    GOPATH := os.Getenv("GOPATH")
    if GOPATH == "" {
        var err error
        // 由用户运行 `go env -w GOPATH=/path` 设置的
        GOPATH, err = GetRuntimeEnv("GOPATH")
        if err != nil {
            // Golang 使用的默认值
            return build.Default.GOPATH
        }
        if GOPATH == "" {
            return build.Default.GOPATH
        }
        return GOPATH
    }
    return GOPATH
}

// GetModuleName 返回 `go.mod` 文件中模块的值。
func GetModuleName(pathToProjectRoot string) (string, error) {
    var moduleName string
    loopPath := pathToProjectRoot
    # 无限循环，直到条件满足或者遇到 break 语句
    for {
        # 查找 loopPath 中最后一个文件路径分隔符的索引
        if idx := strings.LastIndex(loopPath, string(filepath.Separator)); idx >= 0 {
            # 拼接路径，获取 go.mod 文件的路径
            gomodPath := filepath.Join(loopPath, "go.mod")
            # 读取 go.mod 文件的内容
            gomodBytes, err := ioutil.ReadFile(gomodPath)
            # 如果读取文件出错，则更新 loopPath 并继续循环
            if err != nil {
                loopPath = loopPath[:idx]
                continue
            }

            # 将文件内容转换为字符串
            gomodContent := string(gomodBytes)
            # 查找 "module " 的索引
            moduleIdx := strings.Index(gomodContent, "module ")
            # 查找换行符的索引
            newLineIdx := strings.Index(gomodContent, "\n")

            # 如果找到 "module "，则提取模块名并返回
            if moduleIdx >= 0 {
                if newLineIdx >= 0:
                    # 提取模块名并去除空格和换行符
                    moduleName = strings.TrimSpace(gomodContent[moduleIdx+6 : newLineIdx])
                    moduleName = strings.TrimSuffix(moduleName, "\r")
                else:
                    # 提取模块名并去除空格
                    moduleName = strings.TrimSpace(gomodContent[moduleIdx+6:])
                return moduleName, nil
            }
            # 如果未找到 "module "，则返回错误信息
            return "", fmt.Errorf("can not get module path in `%s`", gomodPath)
        }
        # 结束循环
        break
    }
    # 返回错误信息，表示在每个父目录中都没有找到 go.mod 文件
    return moduleName, fmt.Errorf("no `go.mod` file in every parent directory of `%s`", pathToProjectRoot)
# 闭合前面的函数定义
```