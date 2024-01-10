# `v2ray-core\app\dispatcher\dispatcher.go`

```
// +build !confonly
// 设置构建标记，表示该文件不仅仅是用于配置
package dispatcher

//go:generate go run v2ray.com/core/common/errors/errorgen
// 使用go:generate命令来生成错误处理代码
```