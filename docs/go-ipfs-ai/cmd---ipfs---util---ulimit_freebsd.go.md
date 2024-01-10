# `kubo\cmd\ipfs\util\ulimit_freebsd.go`

```
// 如果编译目标是 FreeBSD，则构建此包
// +build freebsd

package util

import (
    "errors"  // 导入 errors 包
    "math"    // 导入 math 包

    unix "golang.org/x/sys/unix"  // 导入 unix 包
)

func init() {
    supportsFDManagement = true  // 初始化文件描述符管理为 true
    getLimit = freebsdGetLimit    // 获取限制调用 freebsdGetLimit 函数
    setLimit = freebsdSetLimit    // 设置限制调用 freebsdSetLimit 函数
}

func freebsdGetLimit() (uint64, uint64, error) {
    rlimit := unix.Rlimit{}  // 创建 unix.Rlimit 结构体
    err := unix.Getrlimit(unix.RLIMIT_NOFILE, &rlimit)  // 获取文件描述符限制
    if (rlimit.Cur < 0) || (rlimit.Max < 0) {  // 如果当前或最大限制小于 0
        return 0, 0, errors.New("invalid rlimits")  // 返回错误信息
    }
    return uint64(rlimit.Cur), uint64(rlimit.Max), err  // 返回当前和最大限制
}

func freebsdSetLimit(soft uint64, max uint64) error {
    if (soft > math.MaxInt64) || (max > math.MaxInt64) {  // 如果软限制或最大限制大于 math 包中的最大值
        return errors.New("invalid rlimits")  // 返回错误信息
    }
    rlimit := unix.Rlimit{  // 创建 unix.Rlimit 结构体
        Cur: int64(soft),  // 设置当前限制
        Max: int64(max),   // 设置最大限制
    }
    return unix.Setrlimit(unix.RLIMIT_NOFILE, &rlimit)  // 设置文件描述符限制
}
```