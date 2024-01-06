# `grype\grype\version\semantic_version.go`

```
// 声明 version 包
package version

// 引入 fmt 和 hashiVer 包
import (
	"fmt"
	hashiVer "github.com/anchore/go-version"
)

// 定义语义版本结构体
type semanticVersion struct {
	verObj *hashiVer.Version
}

// 创建新的语义版本对象
func newSemanticVersion(raw string) (*semanticVersion, error) {
	// 使用 hashiVer 包中的 NewVersion 方法创建版本对象
	verObj, err := hashiVer.NewVersion(normalizer.Replace(raw))
	// 如果创建失败，返回错误
	if err != nil {
		return nil, fmt.Errorf("unable to create semver obj: %w", err)
	}
	// 创建成功，返回语义版本对象
	return &semanticVersion{
		verObj: verObj,
	}, nil
}
// 定义一个方法，用于比较语义版本号对象和另一个版本号对象的大小
func (v *semanticVersion) Compare(other *Version) (int, error) {
    // 如果另一个版本号对象的格式不是语义版本号格式，则返回错误
    if other.Format != SemanticFormat {
        return -1, fmt.Errorf("unable to compare semantic version to given format: %s", other.Format)
    }
    // 如果另一个版本号对象的语义版本号对象为空，则返回错误
    if other.rich.semVer == nil {
        return -1, fmt.Errorf("given empty semanticVersion object")
    }
    // 调用语义版本号对象的比较方法，返回比较结果和错误信息
    return other.rich.semVer.verObj.Compare(v.verObj), nil
}
```