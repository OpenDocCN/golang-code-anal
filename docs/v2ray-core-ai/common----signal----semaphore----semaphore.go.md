# `v2ray-core\common\signal\semaphore\semaphore.go`

```
// Instance 是信号量的实现
type Instance struct {
    token chan struct{}  // 信号量的通道
}

// New 用于创建具有 n 个许可的新信号量
func New(n int) *Instance {
    s := &Instance{  // 创建一个新的 Instance 结构体
        token: make(chan struct{}, n),  // 初始化信号量通道，容量为 n
    }
    for i := 0; i < n; i++ {  // 循环 n 次
        s.token <- struct{}{}  // 将 n 个空结构体发送到信号量通道中
    }
    return s  // 返回新创建的信号量实例
}

// Wait 返回一个用于获取许可的通道
func (s *Instance) Wait() <-chan struct{} {
    return s.token  // 返回信号量的通道
}

// Signal 释放一个许可到信号量中
func (s *Instance) Signal() {
    s.token <- struct{}{}  // 向信号量通道中发送一个空结构体
}
```