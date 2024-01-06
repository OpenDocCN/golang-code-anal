# `grype\grype\version\maven_version.go`

```
// 声明一个名为 version 的包
package version

// 引入 fmt 和 mvnv 包
import (
	"fmt"
	mvnv "github.com/masahiro331/go-mvn-version"
)

// 定义 mavenVersion 结构体，包含原始版本字符串和 mvnv.Version 类型的版本
type mavenVersion struct {
	raw     string
	version mvnv.Version
}

// 定义一个新的 mavenVersion 对象的构造函数，接受原始版本字符串作为参数
func newMavenVersion(raw string) (*mavenVersion, error) {
	// 使用 mvnv 包中的 NewVersion 函数创建一个新的版本对象
	ver, err := mvnv.NewVersion(raw)
	// 如果出现错误，返回错误信息
	if err != nil {
		return nil, fmt.Errorf("could not generate new java version from: %s; %w", raw, err)
	}

	// 返回一个包含原始版本字符串和版本对象的 mavenVersion 对象指针
	return &mavenVersion{
		raw:     raw,  // 将 raw 赋值给结构体字段 raw
		version: ver,  // 将 ver 赋值给结构体字段 version
	}, nil  // 返回结构体和空错误

// Compare 方法用于比较两个 mavenVersion 对象的版本号大小
// 如果 j2 == j，则返回 0；如果 j2 > j，则返回 1；如果 j2 < j，则返回 -1
// 如果出现错误，则返回 -1
func (j *mavenVersion) Compare(j2 *Version) (int, error) {
	if j2.Format != MavenFormat {  // 如果 j2 的格式不是 MavenFormat，则返回错误
		return -1, fmt.Errorf("unable to compare java to given format: %s", j2.Format)
	}
	if j2.rich.mavenVer == nil {  // 如果 j2 的 mavenVersion 对象为空，则返回错误
		return -1, fmt.Errorf("given empty mavenVersion object")
	}

	submittedVersion := j2.rich.mavenVer.version  // 获取 j2 的版本号
	if submittedVersion.Equal(j.version) {  // 如果 j2 的版本号等于当前对象的版本号，则返回 0
		return 0, nil
	}
	if submittedVersion.LessThan(j.version) {  // 如果 j2 的版本号小于当前对象的版本号，则返回 -1
		// 如果提交的版本大于当前版本，则返回1和空错误
		return 1, nil
	}
	// 如果当前版本大于提交的版本，则返回-1和空错误
	if submittedVersion.GreaterThan(j.version) {
		return -1, nil
	}

	// 如果无法比较Java版本，则返回-1和带有错误信息的错误
	return -1, fmt.Errorf(
		"could not compare java versions: %v with %v",
		submittedVersion.String(),
		j.version.String())
}
```