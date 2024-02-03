# `v2ray-core\transport\internet\tls\config_other.go`

```go
// +build !windows
// +build !confonly

package tls

import (
    "crypto/x509"  // 导入crypto/x509包，用于操作X.509证书
    "sync"  // 导入sync包，用于实现同步功能
)

type rootCertsCache struct {  // 定义rootCertsCache结构体
    sync.Mutex  // 互斥锁
    pool *x509.CertPool  // X.509证书池
}

func (c *rootCertsCache) load() (*x509.CertPool, error) {  // rootCertsCache结构体的load方法，用于加载证书池
    c.Lock()  // 加锁
    defer c.Unlock()  // 延迟解锁

    if c.pool != nil {  // 如果证书池不为空
        return c.pool, nil  // 返回证书池
    }

    pool, err := x509.SystemCertPool()  // 获取系统证书池
    if err != nil {  // 如果出现错误
        return nil, err  // 返回空和错误
    }
    c.pool = pool  // 将系统证书池赋值给c.pool
    return pool, nil  // 返回证书池和空
}

var rootCerts rootCertsCache  // 定义rootCerts变量，类型为rootCertsCache

func (c *Config) getCertPool() (*x509.CertPool, error) {  // Config结构体的getCertPool方法，用于获取证书池
    if c.DisableSystemRoot {  // 如果禁用系统根证书
        return c.loadSelfCertPool()  // 返回加载自定义证书池的结果
    }

    if len(c.Certificate) == 0 {  // 如果证书列表为空
        return rootCerts.load()  // 返回加载系统证书池的结果
    }

    pool, err := x509.SystemCertPool()  // 获取系统证书池
    if err != nil {  // 如果出现错误
        return nil, newError("system root").AtWarning().Base(err)  // 返回空和错误信息
    }
    for _, cert := range c.Certificate {  // 遍历证书列表
        if !pool.AppendCertsFromPEM(cert.Certificate) {  // 如果无法将证书添加到证书池
            return nil, newError("append cert to root").AtWarning().Base(err)  // 返回空和错误信息
        }
    }
    return pool, err  // 返回证书池和错误信息
}
```