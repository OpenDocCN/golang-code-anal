# `grype\grype\version\apk_version.go`

```
package version

import (
    "fmt"

    apk "github.com/knqyf263/go-apk-version"
)

type apkVersion struct {
    obj apk.Version
}

// 创建一个新的 apkVersion 对象
func newApkVersion(raw string) (*apkVersion, error) {
    // 使用原始字符串创建一个 apk.Version 对象
    ver, err := apk.NewVersion(raw)
    if err != nil {
        return nil, err
    }

    return &apkVersion{
        obj: ver,
    }, nil
}

// 比较当前 apkVersion 对象和另一个 Version 对象
func (a *apkVersion) Compare(other *Version) (int, error) {
    // 如果另一个对象的格式不是 APK 格式，则返回错误
    if other.Format != ApkFormat {
        return -1, fmt.Errorf("unable to compare apk to given format: %s", other.Format)
    }
    // 如果另一个对象的 apkVersion 为空，则返回错误
    if other.rich.apkVer == nil {
        return -1, fmt.Errorf("given empty apkVersion object")
    }

    // 比较当前对象和另一个对象的版本号
    return other.rich.apkVer.obj.Compare(a.obj), nil
}
```