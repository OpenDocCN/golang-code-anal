# `v2ray-core\features\feature.go`

```go
package features

import "v2ray.com/core/common"

//go:generate go run v2ray.com/core/common/errors/errorgen
// 生成错误代码的工具

// Feature is the interface for V2Ray features. All features must implement this interface.
// All existing features have an implementation in app directory. These features can be replaced by third-party ones.
// Feature 是 V2Ray 功能的接口。所有功能都必须实现这个接口。所有现有功能都在 app 目录中有一个实现。这些功能可以被第三方功能替换。
type Feature interface {
    common.HasType
    common.Runnable
}

// PrintDeprecatedFeatureWarning prints a warning for deprecated feature.
// PrintDeprecatedFeatureWarning 打印已弃用功能的警告。
func PrintDeprecatedFeatureWarning(feature string) {
    newError("You are using a deprecated feature: " + feature + ". Please update your config file with latest configuration format, or update your client software.").WriteToLog()
}
```