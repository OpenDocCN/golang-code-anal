# `kubo\test\cli\testutils\requires.go`

```
package testutils

import (
    "os"  // 导入操作系统相关的包
    "runtime"  // 导入运行时相关的包
    "testing"  // 导入测试相关的包
)

func RequiresDocker(t *testing.T) {
    // 如果环境变量中未设置 TEST_DOCKER 为 1，则跳过测试
    if os.Getenv("TEST_DOCKER") != "1" {
        t.SkipNow()
    }
}

func RequiresFUSE(t *testing.T) {
    // 如果环境变量中未设置 TEST_FUSE 为 1，则跳过测试
    if os.Getenv("TEST_FUSE") != "1" {
        t.SkipNow()
    }
}

func RequiresExpensive(t *testing.T) {
    // 如果环境变量中设置了 TEST_EXPENSIVE 为 1 或者测试为短测试模式，则跳过测试
    if os.Getenv("TEST_EXPENSIVE") == "1" || testing.Short() {
        t.SkipNow()
    }
}

func RequiresPlugins(t *testing.T) {
    // 如果环境变量中未设置 TEST_PLUGIN 为 1，则跳过测试
    if os.Getenv("TEST_PLUGIN") != "1" {
        t.SkipNow()
    }
}

func RequiresLinux(t *testing.T) {
    // 如果运行环境不是 Linux，则跳过测试
    if runtime.GOOS != "linux" {
        t.SkipNow()
    }
}
```