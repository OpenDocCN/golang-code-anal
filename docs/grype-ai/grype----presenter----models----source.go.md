# `grype\grype\presenter\models\source.go`

```
package models

import (
	"fmt"

	syftSource "github.com/anchore/syft/syft/source"
)

type source struct {
	Type   string      `json:"type"`  // 定义 source 结构体的类型字段，并指定其在 JSON 中的键名为 "type"
	Target interface{} `json:"target"`  // 定义 source 结构体的目标字段，并指定其在 JSON 中的键名为 "target"
}

// newSource creates a new source object to be represented into JSON.
// newSource 函数用于创建一个新的 source 对象，并将其表示为 JSON。
func newSource(src syftSource.Description) (source, error) {
	switch m := src.Metadata.(type) {
	case syftSource.StereoscopeImageSourceMetadata:
		// ensure that empty collections are not shown as null
		// 确保空集合不会显示为 null
		if m.RepoDigests == nil {
			m.RepoDigests = []string{}  // 如果 RepoDigests 为空，则将其设置为一个空字符串数组
		}
		// 如果标签为空，则初始化为空数组
		if m.Tags == nil {
			m.Tags = []string{}
		}

		// 返回图片类型的源数据
		return source{
			Type:   "image",
			Target: m,
		}, nil
	case syftSource.DirectorySourceMetadata:
		// 返回目录类型的源数据
		return source{
			Type:   "directory",
			Target: m.Path,
		}, nil
	case syftSource.FileSourceMetadata:
		// 返回文件类型的源数据
		return source{
			Type:   "file",
			Target: m.Path,
		}, nil
	case nil:
// 如果显示的结果来自不支持源信息的输入源，则返回未知类型和目标
return source{
    Type:   "unknown",
    Target: "unknown",
}, nil
// 如果输入源不支持，则返回空的源和错误信息
default:
    return source{}, fmt.Errorf("unsupported source: %T", src.Metadata)
}
```