# `grype\grype\pkg\golang_metadata.go`

```go
# 定义了一个名为 GolangBinMetadata 的结构体，用于存储 Golang 二进制文件的元数据
type GolangBinMetadata struct {
    # 用于存储构建设置的键值对，以 JSON 格式输出时的字段名为 goBuildSettings，如果为空则忽略
    BuildSettings     map[string]string `json:"goBuildSettings,omitempty"`
    # 存储 Golang 编译版本的字符串，以 JSON 格式输出时的字段名为 goCompiledVersion
    GoCompiledVersion string            `json:"goCompiledVersion"`
    # 存储架构信息的字符串，以 JSON 格式输出时的字段名为 architecture
    Architecture      string            `json:"architecture"`
    # 存储 H1 摘要的字符串，以 JSON 格式输出时的字段名为 h1Digest，如果为空则忽略
    H1Digest          string            `json:"h1Digest,omitempty"`
    # 存储主模块信息的字符串，以 JSON 格式输出时的字段名为 mainModule，如果为空则忽略
    MainModule        string            `json:"mainModule,omitempty"`
}

# 定义了一个名为 GolangModMetadata 的结构体，用于存储 Golang 模块文件的元数据
type GolangModMetadata struct {
    # 存储 H1 摘要的字符串，以 JSON 格式输出时的字段名为 h1Digest，如果为空则忽略
    H1Digest string `json:"h1Digest,omitempty"`
}
```