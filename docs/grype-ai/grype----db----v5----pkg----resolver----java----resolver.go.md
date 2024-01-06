# `grype\grype\db\v5\pkg\resolver\java\resolver.go`

```
// 导入所需的包
package java

import (
	"fmt"
	"strings"

	grypePkg "github.com/anchore/grype/grype/pkg"
	"github.com/anchore/grype/internal/log"
	"github.com/anchore/grype/internal/stringutil"
	"github.com/anchore/packageurl-go"
)

// 定义 Resolver 结构体
type Resolver struct {
}

// 定义 Normalize 方法，用于规范化包名
func (r *Resolver) Normalize(name string) string {
	return strings.ToLower(name)
}

// 定义 Resolve 方法，用于解析包
func (r *Resolver) Resolve(p grypePkg.Package) []string {
	// 创建一个新的字符串集合
	names := stringutil.NewStringSet()

	// 当前 Java 生态系统的默认标识符形式是 "<group-name>:<artifact-name>"
	// 如果元数据是 JavaMetadata 类型，则将其转换为 Maven 风格的标识符，并添加到字符串集合中
	if metadata, ok := p.Metadata.(grypePkg.JavaMetadata); ok {
		if metadata.PomGroupID != "" {
			if metadata.PomArtifactID != "" {
				names.Add(r.Normalize(fmt.Sprintf("%s:%s", metadata.PomGroupID, metadata.PomArtifactID)))
			}
			if metadata.ManifestName != "" {
				names.Add(r.Normalize(fmt.Sprintf("%s:%s", metadata.PomGroupID, metadata.ManifestName)))
			}
		}
	}

	// 如果存在 Package URL (PURL)，则尝试解析并添加到字符串集合中
	if p.PURL != "" {
		purl, err := packageurl.FromString(p.PURL)
		if err != nil {
			// 如果解析失败，则记录警告日志
			log.Warnf("unable to resolve java package identifier from purl=%q: %+v", p.PURL, err)
		} else {
# 将格式化后的命名空间和名称组合成一个字符串，并使用规范化函数将其添加到名称列表中
names.Add(r.Normalize(fmt.Sprintf("%s:%s", purl.Namespace, purl.Name)))
# 返回名称列表的切片
return names.ToSlice()
```