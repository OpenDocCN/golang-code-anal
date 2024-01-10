# `grype\grype\presenter\models\source.go`

```
package models

import (
    "fmt"

    syftSource "github.com/anchore/syft/syft/source"
)

type source struct {
    Type   string      `json:"type"`  // 定义 source 结构体，包含类型和目标
    Target interface{} `json:"target"`  // 目标可以是任意类型
}

// newSource creates a new source object to be represented into JSON.
// 创建一个新的 source 对象，用于转换成 JSON 格式
func newSource(src syftSource.Description) (source, error) {
    switch m := src.Metadata.(type) {
    case syftSource.StereoscopeImageSourceMetadata:
        // ensure that empty collections are not shown as null
        // 确保空集合不会显示为 null
        if m.RepoDigests == nil {
            m.RepoDigests = []string{}
        }
        if m.Tags == nil {
            m.Tags = []string{}
        }

        return source{
            Type:   "image",  // 设置类型为 image
            Target: m,  // 设置目标为 m
        }, nil
    case syftSource.DirectorySourceMetadata:
        return source{
            Type:   "directory",  // 设置类型为 directory
            Target: m.Path,  // 设置目标为 m 的路径
        }, nil
    case syftSource.FileSourceMetadata:
        return source{
            Type:   "file",  // 设置类型为 file
            Target: m.Path,  // 设置目标为 m 的路径
        }, nil
    case nil:
        // we may be showing results from a input source that does not support source information
        // 可能显示来自不支持源信息的输入源的结果
        return source{
            Type:   "unknown",  // 设置类型为 unknown
            Target: "unknown",  // 设置目标为 unknown
        }, nil
    default:
        return source{}, fmt.Errorf("unsupported source: %T", src.Metadata)  // 返回错误信息，表示不支持的源类型
    }
}
```