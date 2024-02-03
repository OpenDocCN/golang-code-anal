# `v2ray-core\common\interfaces.go`

```go
// common 包包含了一些常用的接口和函数
package common

import "v2ray.com/core/common/errors"

// Closable 是一个释放资源的接口
//
// v2ray:api:beta
type Closable interface {
    // Close 释放对象的所有资源，包括 goroutines
    Close() error
}

// Interruptible 是一个可以在完成之前停止的接口
//
// v2ray:api:beta
type Interruptible interface {
    Interrupt()
}

// Close 如果 obj 是 Closable，则关闭它
//
// v2ray:api:beta
func Close(obj interface{}) error {
    if c, ok := obj.(Closable); ok {
        return c.Close()
    }
    return nil
}

// Interrupt 如果对象实现了 Interruptible 接口，则调用 Interrupt()，否则调用 Close()
//
// v2ray:api:beta
func Interrupt(obj interface{}) error {
    if c, ok := obj.(Interruptible); ok {
        c.Interrupt()
        return nil
    }
    return Close(obj)
}

// Runnable 是一个可以根据需要启动和停止的接口
type Runnable interface {
    // Start 启动可运行对象。方法返回 nil 后，对象开始正常运行。
    Start() error

    Closable
}

// HasType 是一个知道自己类型的接口
type HasType interface {
    // Type 返回对象的类型。
    // 通常返回对象的 (*Type)(nil)。
    Type() interface{}
}

// ChainedClosable 是由多个 Closable 对象组成的 Closable
type ChainedClosable []Closable

// Close 实现了 Closable 接口
func (cc ChainedClosable) Close() error {
    var errs []error
    for _, c := range cc {
        if err := c.Close(); err != nil {
            errs = append(errs, err)
        }
    }
    return errors.Combine(errs...)
}
```