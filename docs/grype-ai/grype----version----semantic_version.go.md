# `grype\grype\version\semantic_version.go`

```
package version

import (
    "fmt"

    hashiVer "github.com/anchore/go-version"
)

type semanticVersion struct {
    verObj *hashiVer.Version
}

func newSemanticVersion(raw string) (*semanticVersion, error) {
    // 使用正则表达式替换原始字符串，创建语义版本对象
    verObj, err := hashiVer.NewVersion(normalizer.Replace(raw))
    if err != nil {
        // 如果创建失败，返回错误信息
        return nil, fmt.Errorf("unable to create semver obj: %w", err)
    }
    // 返回新创建的语义版本对象
    return &semanticVersion{
        verObj: verObj,
    }, nil
}

func (v *semanticVersion) Compare(other *Version) (int, error) {
    // 检查给定的版本对象格式是否为语义版本格式
    if other.Format != SemanticFormat {
        // 如果不是，返回错误信息
        return -1, fmt.Errorf("unable to compare semantic version to given format: %s", other.Format)
    }
    // 检查给定的版本对象是否为空
    if other.rich.semVer == nil {
        // 如果是，返回错误信息
        return -1, fmt.Errorf("given empty semanticVersion object")
    }

    // 比较当前语义版本对象和给定版本对象的大小关系
    return other.rich.semVer.verObj.Compare(v.verObj), nil
}
```