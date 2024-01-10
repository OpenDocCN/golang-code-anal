# `v2ray-core\transport\internet\xtls\config_other.go`

```
// +build !windows
// +build !confonly

package xtls

import (
    "crypto/x509"  // 导入加密/解密相关的包
    "sync"  // 导入同步操作相关的包
)

type rootCertsCache struct {  // 定义 rootCertsCache 结构体
    sync.Mutex  // 互斥锁
    pool *x509.CertPool  // x509 证书池
}

func (c *rootCertsCache) load() (*x509.CertPool, error) {  // 定义 load 方法，返回证书池和错误
    c.Lock()  // 加锁
    defer c.Unlock()  // 延迟解锁

    if c.pool != nil {  // 如果证书池不为空
        return c.pool, nil  // 返回证书池和空错误
    }

    pool, err := x509.SystemCertPool()  // 获取系统证书池
    if err != nil {  // 如果出现错误
        return nil, err  // 返回空证书池和错误
    }
    c.pool = pool  // 将获取的系统证书池赋值给结构体中的证书池
    return pool, nil  // 返回证书池和空错误
}

var rootCerts rootCertsCache  // 定义 rootCerts 变量为 rootCertsCache 类型

func (c *Config) getCertPool() (*x509.CertPool, error) {  // 定义 getCertPool 方法，返回证书池和错误
    if c.DisableSystemRoot {  // 如果禁用系统根证书
        return c.loadSelfCertPool()  // 返回加载自定义证书池的结果
    }

    if len(c.Certificate) == 0 {  // 如果证书列表为空
        return rootCerts.load()  // 返回加载系统根证书池的结果
    }

    pool, err := x509.SystemCertPool()  // 获取系统证书池
    if err != nil {  // 如果出现错误
        return nil, newError("system root").AtWarning().Base(err)  // 返回空证书池和包含错误信息的新错误
    }
    for _, cert := range c.Certificate {  // 遍历证书列表
        if !pool.AppendCertsFromPEM(cert.Certificate) {  // 如果无法将证书添加到证书池
            return nil, newError("append cert to root").AtWarning().Base(err)  // 返回空证书池和包含错误信息的新错误
        }
    }
    return pool, err  // 返回证书池和错误
}
```