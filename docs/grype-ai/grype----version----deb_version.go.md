# `grype\grype\version\deb_version.go`

```go
package version

import (
    "fmt"

    deb "github.com/knqyf263/go-deb-version"
)

type debVersion struct {
    obj deb.Version
}

func newDebVersion(raw string) (*debVersion, error) {
    // 使用 go-deb-version 库创建一个 deb.Version 对象
    ver, err := deb.NewVersion(raw)
    if err != nil {
        return nil, err
    }
    // 返回一个包含 deb.Version 对象的 debVersion 结构体指针
    return &debVersion{
        obj: ver,
    }, nil
}

func (d *debVersion) Compare(other *Version) (int, error) {
    // 检查给定的 Version 对象是否为 DebFormat，如果不是则返回错误
    if other.Format != DebFormat {
        return -1, fmt.Errorf("unable to compare deb to given format: %s", other.Format)
    }
    // 检查给定的 Version 对象是否为空，如果是则返回错误
    if other.rich.debVer == nil {
        return -1, fmt.Errorf("given empty debVersion object")
    }

    // 调用 go-deb-version 库中的 Compare 方法比较两个 deb.Version 对象
    return other.rich.debVer.obj.Compare(d.obj), nil
}
```