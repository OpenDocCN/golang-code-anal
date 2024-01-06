# `grype\grype\version\apk_version.go`

```
// 声明一个名为 version 的包
package version

// 引入 fmt 包，用于格式化输出
import (
	"fmt"

	// 引入 go-apk-version 包，用于处理 APK 版本
	apk "github.com/knqyf263/go-apk-version"
)

// 定义一个结构体 apkVersion，包含一个 apk.Version 对象
type apkVersion struct {
	obj apk.Version
}

// 定义一个函数 newApkVersion，用于创建一个 apkVersion 对象
func newApkVersion(raw string) (*apkVersion, error) {
	// 调用 go-apk-version 包中的 NewVersion 函数，将原始字符串转换为版本对象
	ver, err := apk.NewVersion(raw)
	// 如果转换过程中出现错误，则返回错误信息
	if err != nil {
		return nil, err
	}

	// 返回一个包含版本对象的 apkVersion 对象
	return &apkVersion{
		obj: ver,
// 定义apkVersion结构体的Compare方法，用于比较两个版本号
func (a *apkVersion) Compare(other *Version) (int, error) {
    // 检查传入的版本号格式是否为APK格式，如果不是则返回错误
    if other.Format != ApkFormat {
        return -1, fmt.Errorf("unable to compare apk to given format: %s", other.Format)
    }
    // 检查传入的版本号是否为空，如果为空则返回错误
    if other.rich.apkVer == nil {
        return -1, fmt.Errorf("given empty apkVersion object")
    }
    // 调用其他版本号对象的Compare方法，比较当前版本号和传入版本号的大小关系
    return other.rich.apkVer.obj.Compare(a.obj), nil
}
```