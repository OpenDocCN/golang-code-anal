# `grype\grype\pkg\golang_metadata.go`

```
// 定义一个名为 GolangBinMetadata 的结构体，包含构建设置、Go 编译版本、架构、H1 摘要和主模块
type GolangBinMetadata struct {
    BuildSettings     map[string]string `json:"goBuildSettings,omitempty"` // 构建设置，以键值对形式存储
    GoCompiledVersion string            `json:"goCompiledVersion"`         // Go 编译版本
    Architecture      string            `json:"architecture"`              // 架构
    H1Digest          string            `json:"h1Digest,omitempty"`        // H1 摘要，可选
    MainModule        string            `json:"mainModule,omitempty"`      // 主模块，可选
}

// 定义一个名为 GolangModMetadata 的结构体，包含 H1 摘要
type GolangModMetadata struct {
    H1Digest string `json:"h1Digest,omitempty"` // H1 摘要，可选
}
```