# `grype\grype\version\maven_version.go`

```
package version

import (
    "fmt"

    mvnv "github.com/masahiro331/go-mvn-version"
)

type mavenVersion struct {
    raw     string
    version mvnv.Version
}

func newMavenVersion(raw string) (*mavenVersion, error) {
    // 使用原始字符串创建新的 Maven 版本对象
    ver, err := mvnv.NewVersion(raw)
    if err != nil {
        // 如果出错，返回错误信息
        return nil, fmt.Errorf("could not generate new java version from: %s; %w", raw, err)
    }

    // 返回新的 Maven 版本对象
    return &mavenVersion{
        raw:     raw,
        version: ver,
    }, nil
}

// Compare returns 0 if j2 == j, 1 if j2 > j, and -1 if j2 < j.
// If an error returns the int value is -1
func (j *mavenVersion) Compare(j2 *Version) (int, error) {
    // 检查给定的版本对象格式是否为 Maven 格式
    if j2.Format != MavenFormat {
        return -1, fmt.Errorf("unable to compare java to given format: %s", j2.Format)
    }
    // 检查给定的版本对象是否为空
    if j2.rich.mavenVer == nil {
        return -1, fmt.Errorf("given empty mavenVersion object")
    }

    // 获取给定版本对象的版本号
    submittedVersion := j2.rich.mavenVer.version
    // 比较版本号，返回相应的结果和错误信息
    if submittedVersion.Equal(j.version) {
        return 0, nil
    }
    if submittedVersion.LessThan(j.version) {
        return -1, nil
    }
    if submittedVersion.GreaterThan(j.version) {
        return 1, nil
    }

    // 如果无法比较，返回错误信息
    return -1, fmt.Errorf(
        "could not compare java versions: %v with %v",
        submittedVersion.String(),
        j.version.String())
}
```