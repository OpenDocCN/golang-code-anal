# `v2ray-core\transport\internet\kcp\xor.go`

```go
// +build !amd64

package kcp

// xorfwd函数执行字节切片x的向前XOR操作，即x[i] ^= x[i-4]，其中i的范围是从0到len(x)
func xorfwd(x []byte) {
    for i := 4; i < len(x); i++ {
        x[i] ^= x[i-4]
    }
}

// xorbkd函数执行字节切片x的向后XOR操作，即x[i] ^= x[i-4]，其中i的范围是从len(x)到0
func xorbkd(x []byte) {
    for i := len(x) - 1; i >= 4; i-- {
        x[i] ^= x[i-4]
    }
}
```